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

# tcpdump 抓post请求并存放到文件
nohup tcpdump -s 0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354' | grep -C 100 "/api/screenshot" >> file.log &

more tcpdump examples see: https://hackertarget.com/tcpdump-examples/ 
```
#### CDN介绍
CDN，content delivery network，内容分发网络，是说所有服务节点组成一个网络，源站内容缓存到就近服务节点，供用户访问，降低源站压力，加快访问速度。

cdn加速好处：

- 举例：一般而言，前端如果有较大体积的图片，可以存放到oss，通过cdn加速可以实现文件快速访问。返回文件可以经过gzip，加水印，格式转化。
- 刷新预热，比如说现在办一场秒杀活动，到点面临流量突增的情况，可以提前将资源缓存到加速节点，缓解源站压力。手机中端加载网站，因为文件数量太多导致加载时间太长了可以用CDN。
- 不同地区用户访问质量不同，用cdn。
- 大文件下载，像手机安装包，文件大小20M以上。
- 视频点播加速。
- HSTS Http Strict Transport Security，Http严格传输安全，强制浏览器只能以HTTPS协议访问服务器
- 防盗链 

提升资源命中率？设置资源缓存时间，减少回源，提高性能。

