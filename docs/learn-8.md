### All Network Utils
```
# dig
本地dns->根dns服务器->权威dns服务器的dns查询。

dig www.baidu.com

dig @114.114.114.114 www.baidu.com   // 指定dns服务器114.114.114.114进行解析。

# traceroute 发出icmp包进行路由探查。

traceroute www.baidu.com

# telnet  侦测远端服务端口是否可用

telnet www.baidu.com 80


# curl 发送http请求

curl -v "http://www.baidu.com"

# wget 下载资源

wget http://logtail-release-cn-hangzhou.oss-cn-hangzhou.aliyuncs.com/linux64/logtail.sh -O logtail.sh;chmod 755 logtail.sh // -O 指定文件名称


```
