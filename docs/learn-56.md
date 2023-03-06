#### Etcd读写数据模式
集群内任何节点都可以处理读写请求，不同节点的数据都是强一致性的，读请求任一节点可读。对于写，当写请求先到了leader节点，由leader节点进行写操作再分发到follower；但当写请求先到follower，写请求只会被转发到leader写完再进行分发。

#### leader选举过程
集群创建支持，不存在leader，每个node具备一个计时器Timer，现有A,B,C三个节点。假设节点A的Timer先计算完后，率先向另外两节点发成为leader的请求，后被投票后A成为leader。A成为leader后，定时发心跳确保leader还在，如果超时则重新选举leader。

#### 高可用节点为什么是奇数的理解
因为遵循过半原则，以节点7为例，可用节点数应大于 7/2，也就是割裂后的集群节点数至少要大于3个，最小可用数为4，容忍度为3。对于节点数为8的集群，8/2意味着最小节点数为5，容忍度为3。既然7和8的容忍度一致，用8只会多占据资源机器。那为什么要遵循过半原则？对于节点7的集群，不遵循的话，假设坏了3个，剩下4个正常的A,B,C,D，然而AB和CD因为网络问题(可能是防火墙)被分成两个集群，此时两边的可用节点数皆为2，会分别选出1个leader出来。过了一会，AB和CD的网络又行了，这个时候有2个leader，这就是所谓的脑裂，会导致数据不知道交给谁处理的问题。

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