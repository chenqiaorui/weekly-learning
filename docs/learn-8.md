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

# 验证ref
curl -so /dev/null -v 'https://example.com/web/home/feature_3_bg.png' -H 'referer: http://servicewechat.com/'

# get请求
curl -i -X GET http://httpbin.org:80/anything/foo?arg=1  # -i表示打印响应头

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

##### DNS安装和配置
环境: centos7    

IP: 192.168.1.181

```
# 安装
yum install vim bind*-y

# 备份
mv /etc/named.conf /etc/named.confbak

# vim /etc/named.conf，修改：
listen-on port 53 { any; };  // "127.0.0.1"改为"any"
allow-query     { any; };    // "localhost"改为"any"

# named.conf 继续添加：
zone "emogricky.com" { // 增加正向解析区域
    type master;    // 类型为master
    file "emogricky.com.zone";    // 区域为解析文件为"/var/named/emogricky.com"
};

# 新建文件 /var/named/emogricky.com.zone
$TTL 1D
@       IN SOA  emogricky.com. root.emogricky.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                NS      emogricky.com.
                A       192.168.1.181
dns     IN      A       192.168.1.181
www     IN      A       192.168.1.181

# 关闭防火墙或开放53端口
systemctl stop firewalld

# 重启dns服务
systemctl restart named

以上DNS服务器就搭建好了。
# 验证：找一台linux服务器，/etc/resolv.conf文件添加：
nameserver 192.168.1.181

nslookup www.emogricky.com

返回如下结果就算成功：
Server:		192.168.1.181
Address:	192.168.1.181#53

Name:	www.emogricky.com
Address: 192.168.1.181
```
其他：

稍显复杂的zone配置
```
$TTL 1D
@       IN SOA  emogricky.com. root.emogricky.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                NS      emogricky.com.
                A       192.168.1.181
                AAAA    ::1
                MX 10   mail.emogricky.com
web     IN      CNAME   www.emogricky.com
dns     IN      A       192.168.1.181
www     IN      A       192.168.1.181
cookie     IN   A       192.168.1.167
```
#### telnet
本质是上一个远程登录工具，只不过是明文传输数据。

常用功能：
- 测试远程端口是否可用

centos7安装telnet
yum install -y telnet

如果返回`Connection refused`说明服务端口没有被占用，反之。

测试mysql是否可用
```
$ telnet localhost 3306
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
R
5.5.65-MariaDB/+R<uX;Z?;fp"Ug%}k"bKmysql_native_passwordConnection closed by foreign host.
```
以上表示mysql服务可连。在`foreign host.`后面输入quit可退出 或在后面输入`info`可查返回细节。

#### ssh简介
远程连接工具，传输的信息被加密

#### 原理
1.举例

我(windows)想通过ssh连到服务器(192.168.1.189)，怎么办？

首先，我打开一个powershell窗口，输入：ssh root@192.168.1.189，回车。

请求被发到了服务器，服务器给我返回了它自己的公钥，这个时候，我的powershell提示：

`The authenticity of host '192.168.1.189 (192.168.1.189)' can't be established...` 问我要不要继续连接。输入`yes`则会在我的~/.ssh/known_hosts文件生成相关内容，表示把192.168.1.189添加到我的可信赖列表。至于问为什么要有这个步骤，是因为我的请求可能被中间攻击，公钥不是我请求的服务器返回给我的，那我的登录信息就给中间人获取了。

接着，powershell显示我要输入登录服务器的密码了，输完密码登录进去后，输入exit退出，再ssh登录，奇怪，又要我输入密码，太烦了。上面是用ssh进行登录的方法之一。

好了，有没有输入`ssh root@192.168.1.189`之后不用输密码就能登录的办法？有的。


首先，将我(windows)的`~/.ssh/id_rsa.pub`公钥粘贴追加到服务器(192.168.1.189)的`/root/.ssh/authorized_keys`文件里面(没有就创建)。

注意：windows没有`id_rsa.pub`的话用powershell执行`ssh-keygen`生成。服务器(192.168.1.189)的authorized_keys文件权限可能要在`-rw-r--r--`或比它低的权限，可以用`chmod 644 authorized_keys`修改权限。如果authorized_keys权限过高，我(windows)ssh登录还会要求输入密码。

放完之后，我(windows)再输入：ssh root@192.168.1.189 就可以实现免密登录了。这是登录ssh的方式二。

##### 总结
ssh实现远程免密登录的过程(这里不说要密码登录的方式)：客户端把自己的公钥id_rsa.pub 内容复制到服务端的authorized_keys，客户端登录，服务器发现有客户端的公钥就让客户端登录了。

##### 关于ssh请求连接数过多引起的问题
```
ssh_exchange_identification: Connection closed by remote host
```
原因：存在一些远程恶意登录机器的尝试连接
解决：
```
查看 /var/log/auth.log，加ip黑名单。
```
#### SSL简介
SSL协议，只是一套规范，实现这套规范的有openssl等。为什么需要SSL证书？http传输的数据是明文的，容易被中间人攻击，如果是电商网站用户的登录信息很容易被窃取，所以需要ssl证书加密。

#### SSL加密原理
例子：

我(浏览器)发出一个加密请求，"say hello"，这个请求叫ClientHello请求。还夹带我(浏览器)支持的TLS(SSL后更名TLS)版本(如TLS v1.0)、随机数(用于后来加密)、加密算法(rsa)等。

服务器收到后，发了一个"ServerHello"，夹带使用的TLS版本、随机密钥、`服务器证书`。

客户端收到后，要确认服务器的`服务器证书`，如果证书不是可信机构、证书记录的域名和实际访问域名不符、或过期，浏览器这时候就会弹出来一个框问：是否还要继续访问？

客户端确认没问题了，就从`服务器证书`拿出公钥，接着发给服务器：随机数、根据前面所发内容产生的hash值(供服务器检验)

服务器回应，发送以下内容：根据客户端的随机数生成会话密钥，根据前面所发内容产生的hash值(供客户端检验)

至此，双方握手结束，开始加密交流。

#### 总结
ssl加密就是：要发的信息+会话密钥被算法加密成hash串进行传输。

#### SSL证书类型
- DV型：单域名的DV证书有免费的。企业一般是买泛域名DV，价格估计在600-1500不等。至于说为什么有免费的不用？免费的只支持一个域名、Let's Encrypt签出来的证书不稳定，可能用3个月就失效了。

除了单域名、泛域名(比如说你买了`*.haha.com`这个泛域名证书，那么a.haha.com和b.haha.com就能用了，但c.a.haha.com这种三级域名泛域名证书就不支持了)，还有多域名(如a.youpai.com、a.youpai1.com)

DV型证书只需要对域名的所属权进行验证，CA机构就能签发给你。CA机构一般是那些权威的机构担任，如：Digicert、Sectigo

- OV型：CA机构将对你的企业资质和域名所有全进行验证，相对更靠谱，但价格更贵，有7000左右的。

- EV型：老贵了，一般是大型金融机构才买。



