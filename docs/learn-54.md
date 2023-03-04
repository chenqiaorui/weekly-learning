#### 通过clash设置代理拉取国外docker镜像
```
# 下载clash
wget https://github.com/Dreamacro/clash/releases/download/v1.11.12/clash-linux-amd64-v1.11.12.gz

# 解压
gzip -cd clash-linux-amd64-v1.11.12.gz > /usr/bin/clash

# 设置权限
chmod u+x /usr/bin/clash

# 创建目录
mkdir /etc/clash

# 指定配置文件位置，默认生成config.yaml和Country.mmdb
clash -d /etc/clash

# 下载config.yanl
curl -o /etc/clash/config.yaml clash订阅链接

# 下载Country.mmdb
wget -O /etc/clash/Country.mmdb https://ghproxy.com/https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb

# 前台启动clash
clash -d /etc/clash

# 怎么做到后台启动？
 vi /etc/systemd/system/clash.service

[Unit]
Description=clash daemon
[Service]
Type=simple
User=root
ExecStart=/usr/bin/clash -d /etc/clash/
Restart=on-failure
[Install]
WantedBy=multi-user.target

# 加载守护进程配置
systemctl daemon-reload

systemctl enable clash

systemctl start clash

# 上面的做法还没设置代理服务器，临时配置全局代理如下：

export ALL_PROXY="socks5h://localhost:7890"
export HTTP_PROXY="http://localhost:7890"
export HTTPS_PROXY="http://localhost:7890"

# 测试，返回文档则成功
curl https://www.google.com

# 但对于docker pull，还需设置代理，创建目录
/etc/systemd/system/docker.service.d

# 配置docker走代理
vim /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=ttp://localhost:7890"
Environment="HTTPS_PROXY=http://localhost:7890"

# 重启
systemctl daemon-reload

systemctl restart docker

# 检查代理
docker info |grep Proxy

# 拉取国外镜像
docker pull k8s.gcr.io/etcd:3.4.3-0

说明：
- config.yaml取自订阅 URL，里面设置了代理服务器的相关信息；

- Country.mmdb 为全球 IP 库

```
参考：

https://juejin.cn/post/7163543975419183118

https://www.cnblogs.com/linjiangCN/p/16135203.html