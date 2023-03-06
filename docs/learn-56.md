#### 实验部署
编辑docker-compose.yml
```
version: '3'

######## 项目依赖的环境，启动项目之前要先启动此环境 #######
######## The environment that the project depends on, starting this environment before starting the project #######

services:
  etcd:
    image: bitnami/etcd:3.4.15
    container_name: etcd
    restart: always
    volumes:
      - etcd_data:/bitnami/etcd
    environment:
      ETCD_ENABLE_V2: "true"
      ALLOW_NONE_AUTHENTICATION: "yes"
      ETCD_ADVERTISE_CLIENT_URLS: "http://0.0.0.0:2379"
      ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
    ports:
      - "2379:2379/tcp"
    networks:
      - etcd_net

  #prometheus监控 — Prometheus for monitoring
  prometheus:
    image: prom/prometheus:v2.28.1
    container_name: prometheus
    environment:
      # 时区上海 - Time zone Shanghai (Change if needed)
      TZ: Asia/Shanghai
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data/prometheus:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    restart: always
    user: root
    ports:
      - 9090:9090
    networks:
      - etcd_net

  #查看prometheus监控数据 - Grafana to view Prometheus monitoring data
  grafana:
    image: grafana/grafana:8.0.6
    container_name: grafana
    hostname: grafana
    user: root
    environment:
      # 时区上海 - Time zone Shanghai (Change if needed)
      TZ: Asia/Shanghai
    restart: always
    volumes:
        - ./grafana/data:/var/lib/grafana
    ports:
        - "3001:3000"
    networks:
        - etcd_net

networks:
  etcd_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.30.0.0/16

volumes:
  etcd_data:
    driver: local

```
编辑prometheus/prometheus.yml
```
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "etcd"
    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    static_configs:
      - targets: ["etcd:2379"]
```
启动: ```docker-compose up -d```

grafana添加prometheus数据源后，导入etcd dashboard。面板json文件见：[https://grafana.com/grafana/dashboards/3070-etcd/]

#### 一个key占据多个G内存案例
```
# 进入etcd容器
docker exec -ti etcd bash

# put同一个key，执行1000次
for i in {1..1000}; do dd if=/dev/urandom bs=1024 
count=1024  | ETCDCTL_API=3 etcdctl put key  || break; done

# 获取最新revision，并压缩
etcdctl compact `(etcdctl endpoint status --write-out="json" | egrep -o '"revision":[0-9]*' | egrep -o '[0-9].*')`

# 对集群所有节点进行碎片整理
etcdctl defrag --cluster
```