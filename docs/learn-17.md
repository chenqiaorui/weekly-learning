#### docker-compose 安装
```
#下载可执行包

curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# docker-compose版本见：`https://github.com/docker/compose/releases`

# 赋可执行权限
chmod +x /usr/local/bin/docker-compose

# 测试是否可用
docker-compose --version
```