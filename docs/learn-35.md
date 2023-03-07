##### 书籍
https://pages.cs.wisc.edu/~remzi/OSTEP/Chinese/01.pdf

#### 操作系统学习笔记
学习方法：首先听课，做笔记，阅读笔记，巩固知识，实践

##### 1.程序运行时发生了什么？
```
一个程序运行时在做什么？执行指令。cpu从内存get到一条指令，对其解码然后执行它，比如说两个数相加。完成指令后执行下一条，依此类推，直到程序完成。
```

问题：如何将硬件资源虚拟化？
```
通过操作系统实现，操作系统（也可称之为虚拟机）提供API，利用虚拟机的功能（如访问文件、内存、计算），让用户可以通过软件的形式告诉操作系统做什么。操作系统api称之为系统调用（system call），让应用程序调用。由于操作系统提供了这些调用来运行程序，访问内存、文件等资源，有时也会称操作系统提供了标准库(standard library)。
```
##### 2.虚拟化CPU
虚拟化CPU有什么好处？
```
运行多个程序在一个单核cpu机器上，看起来有多个程序`同时`运行。实际上，cpu是在几个程序间不断地切换运行。

可以通过接口的形式去暂停程序或查看程序。
```
##### 3.虚拟化内存
物理内存的模式本质上是一个存满字节的数组。读内存就是读取某个位置上的地址，再读取地址上的值。

再来说说程序运行发生了什么？
```
程序运行时，代码数据以某种数据结构被加载到内存，通过指令操作内存上的数据。
```

场景：当1套代码同时运行2个程序，程序运行的代码逻辑是：分配1个内存，打印其内存地址。结果两个程序打印出来的内存地址是一样的，那么它们是互相覆盖吗？

不会，因为看到的内存地址是虚拟内存地址，映射到实际的物理内存地址是两个不同的地方。

实际上，不同的程序是用到的资源是隔离、互不影响的。

##### 4. 并发
同一个主程序有两个线程在操纵同一个内存，会发生什么事？

##### 5.持久性
内存在断电或系统崩溃的时候会丢失数据。持久化数据，通过软件系统和硬件磁盘IO来实现。

软件系统称之为文件系统(file system)。

如何实现持久化数据到文件？
- 调用open()创建和打开文件
- write()写入文件
- close()关闭文件

注意：为防止写时系统崩溃或高效写入，文件系统有写时复制、写入特定数据结构(简单列表或复杂B树)。

##### 6.操作系统的发展历史
- 早期：只有一些库，操作人员排列任务进行批处理。
- 为了安全性，如防止一些文件被访问，增加系统调用的概念，通过内核接口形式只暴露出特定硬件的能力，让用户态的应用程序只能有特定权限。
- 多程序时代，操作系统不应该同时只跑一个任务

##### 关于虚拟化的对话
虚拟化cpu是抽象出来的概念，比如说有一个桃子，让每个人都看起来像是拥有一个“桃子”而不自知，这便是虚拟化的奥义所在。

##### 7. 抽象：进程
进程是在运行的程序。进程本身不具备生命周期，它只是磁盘上的一堆数据(字节)，是操作系统让程序真正运行起来。

cpu虚拟化的假象是怎么做到的？
```
操作系统通过时分共享技术，允许进程使用一段时间cpu后，切换到另一个进程。操作系统还有不同的算法调度策略，例如哪个进程优先级高，哪个运行时间更长等。
```

进程的组成成分：
```
进程含有内存，内存用于存储代码指令，程序生成的数据也被写入内存，当然，进程也操纵文件系统当它用到文件的时候
```

与进程相关的系统调用：
- create() 创建进程
- destory() 销毁进程
- status() 进程状态

问：操作系统运行进程(程序)的细节？
```
操作系统将代码和静态文件加载到内存中，这个过程需要从磁盘读取字节。程序在运行前会分配运行时栈，用于存放局部变量等。程序运行时，在C语言需要显示分配动态内存堆（heap）用于存放产生的动态数据。

数据结构（如链表、散列表、树和其他有趣的数据结构）需要堆。起初堆会很小。随着程序运行，通过 malloc()库 API 请求更多内存，操作系统可能会参与分配更多内存给进程，以满足这些调用。

操作系统还将执行一些初始化任务，如为每个进程打开标准输入、标准输出、错误三个文件描述符（file descriptor）。

做完初始化任务、分配stack内存这一系列任务后，开始启动进程，即运行main()。
```

进程状态
- 运行（running） 进程在使用cpu，意味着cpu在执行指令
- 就绪（ready） 进程准备好了，但由于某种原因，操作系统决定不在此时运行
- 阻塞（blocked）进程执行了某种操作，直到发生某个时间才运行。如写文件到磁盘，它就会被阻塞，其他进程可以使用cpu

##### 一、什么是性能问题？
```
突然有一天，公司开发在夜里打电话给你，“现在有很多用户说系统弹出一个错误，系统繁忙”之类的。
开发同事一看接口返回500，看了后端日志好像没有特别明显的报错信息。

这时，你赶忙上服务器看下nginx日志，好家伙，一看nginx日志发现后发现接口请求过了60s还没有响应。
`top`看一下服务器资源，用户空间的cpu使用率100%，是哪些进程呢？php-fpm。里面发生了什么？我不知道啊？
唉，重启服务能解决90%的问题，那就重启看看吧。果然，重启后用户反馈正常了。害，暂时是没有问题了，那下次再来一次怎么办？...无后续
```

看，cpu100%这类问题就归属性能问题。

##### 研究性能问题需要学习操作系统主要组件原理
CPU性能、内存、磁盘IO性能、网络

衡量性能的指标：
并发（吞吐）、响应快（时延）

##### 二、CPU性能篇

##### 什么是平均负载
看一个例子：`uptime`
```
$ uptime
02:34:03 up 2 days, 20:14,  1 user,  load average: 0.63, 0.83, 0.88
```
说明：
- `02:34:03` 当前时间
- `up 2 days, 20:14` 系统从开机后运行的时长
- `1 user` 正在登录的用户数
- `load average: 0.63, 0.83, 0.88` 最近1分钟、5分钟、15分钟的平均负载(Load average)

`平均负载`是指单位时间内，系统处于可运行(Running或ready)和不可中断(blocked)的进程数。结合最近1分钟、5分钟、15分钟的平均负载，我们可以全面了解cpu的使用情况。像了解一天早中晚的气候变化。

Running是指正在使用cpu；Ready是指代码(指令)已经加载到了内存，等cpu来执行指令；

不可中断是指进程在使用cpu，突然需要进行磁盘IO(读写)的长时间操作，先不用cpu，等IO操作完毕再回来使用cpu。

一般而言，平均负载超过cpu核数70%就要检查cpu的使用情况，考虑是否优化。

查看cpu核数
```
grep 'model name' /proc/cpuinfo|wc -l
```

##### 什么是CPU使用率
cpu使用率衡量了单位时间cpu的繁忙程度。

对于I/O密集型进程，平均负载就高，但cpu却不繁忙，也就是cpu使用率不高。

看一个例子：`top`
```
$ top
top - 11:15:27 up 40 days, 51 min,  1 user,  load average: 0.32, 0.27, 0.26
Tasks: 356 total,   1 running, 355 sleeping,   0 stopped,   0 zombie
%Cpu(s):  4.2 us,  5.6 sy,  0.0 ni, 90.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7777.8 total,    281.6 free,   4394.8 used,   3101.3 buff/cache
MiB Swap:   2048.0 total,    580.2 free,   1467.8 used.   3015.1 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                                                 
3811294 root      20   0   15440   4540   3768 R  20.0   0.1   0:00.03 top                                                                                                                     
    682 avahi     20   0   10524   5696   3120 S   6.7   0.1 110:29.16 avahi-daemon                                                                                                            
   7726 root      20   0  750872  21192   5772 S   6.7   0.3 259:56.50 travel-api  
```
说明：
- `%Cpu(s)` 比如说有4个cpu，%Cpu(s)代表这4个的平均使用率。

  cpu = 用户空间使用率(us) + 内核空间使用率(sy) + 空闲(id)

  `ni` 用户空间通过改变进程优先级占用的cpu百分比

  `wa` 等待io操作占用的cpu百分比

  `hi/si` 硬/软中断进行cpu上下文切换占用的百分比

- `RES` 使用的真实物理内存（KB）
- `%CPU` 一个cpu的使用率，毕竟一个进程只占用一个cpu
- `TIME+` 累计使用cpu时间

附top使用快捷键说明：
- `shift + m` # 按照内存使用率排序，shift m 等价于大写M
- `shift + p` # 按照cpu使用率排序
- `c` # 显示命令全路径
- `F` # 挑选你要选择展示的列，按下空格选中，* 代表会展示的列(列会出现在最后)，按q退出。
    可以展示进程使用哪一个cpu
- 按`1` # 展示每个cpu的使用情况

#### 工具合集
```
# 压测
ab -c 100 -n 1000 http://192.168.0.10:10000/   # 一次并发100，共1000
...
Requests per second: 87.86 [#/sec] (mean)   # 平均每秒处理87个请求
Time per request: 1138.229 [ms] (mean)     # 平均1个1.13秒
...


# 根据名称查看进程树
pstree | grep stress

# 监视短时进程
execsnoop

# 监视IO
dstat 1 10 # 每隔1s输出，共10组，关注输出read和writ
pidstat -d -p 4344 1 3 # 指定进程的读写速率
strace -p 6082 # 追踪进程

# 模拟网络SYN FLOOD 攻击
hping3 -S -p 80 -i u100 192.168.0.30 
说明：
-S参数表示设置TCP协议的SYN（同步序列号）
-i u100表示每隔100微秒发送一个网络帧

# 网络
yum install sysstat
sar -n DEV 1 # -n DEV 表示显示网络收发的报告，间隔1秒输出一组数据

tcpdump -i eth0 -n tcp port 80 # 抓包，-n不解析协议名和主机名

# 列出cpu个数
lscpu

# 查看进程父子关系
yum install -y psmisc
pstree -aps $pid

# 资源优化
应用角度：吞吐 & 延迟
系统角度：cpu使用率

# 内存
free
说明：
- available：未使用内存 + 可回收缓存
- used：已使用内存，包含共享内存
- cache/buff：cache是文件内存，buff是磁盘内存
top
说明：
- VIRT：虚拟内存，进程申请的但还未分配实际物理内存
- RES: 实际占用的物理内存
- SHR: 共享内存，一般是公共库、动态链接库
- %MEM：物理内存占总内存百分比

# 内存泄漏分析
vmstat 3 # 实时查看内存变动
yum install bcc
memleak -a -p $pid

# Swap
swapoff -a # 关闭

# IO
df -i

iostat -d -x 1 # -d -x 显示所有磁盘指标
说明：
- r/s 每s向磁盘发送的读请求数
- rkB/s 每s从磁盘读的数据量
- %util IO使用量

# 进程io使用情况
pidstat -d 1

# 慢sql场景
1.接口返回时间长；
show full processlist; # 查看所有会话
explain select * from products where productName='time';
说明：
- type为ALL代表全表查询
- key为null代表无索引建立
- rows代表扫描行数

CREATE INDEX products_index ON products (productName); # 建立索引

# 慢redis查询
# redis持久化还是会用到磁盘
strace -f -T -tt -p 9085 # -f表示跟踪子进程和子线程，-T表示显示系统调用的时长，-tt表示显示跟踪时间；-e fdatasync 可以指定函数

# 网络
应用 - 提供统一接口
表示 - 数据转换
会话 - 维护通信连接
传输 - 加表头成包进行传输
网络 - 路由
数据链路 - MAC寻址
物理 - 物理网络传输帧

MTU - 1500 规定包大小

ss -ltnp | head -n 3
说明：
Recv，Send-Q 接收(未被程序处理)和发送包队列(未被远端确认)，不为0代表包堆积
state有Listening和Established

# 网络吞吐
sar -n DEV 1
ethtool ens18 | grep Speed # 千兆网卡

# 理解一台机器65536个端口
对于客户端，确实受限；但对服务端，可以组合客户端ip+端口。

# 测试某台机器tcp吞吐(带宽能力)
yum install iperf3
服务器1：iperf3 -s -i 1 -p 10000 # -s表示启动服务端，-i表示汇报间隔，-p表示监听端口

# -c表示启动客户端，192.168.0.30为目标服务器的IP# -b表示目标带宽(单位是bits/s)# -t表示测试时间# -P表示并发数，-p表示目标服务器监听端口
服务器2：iperf3 -c 192.168.0.30 -b 1G -t 15 -P 2 -p 10000

回服务端看接口：SUM 行就是测试的汇总结果。receiver 表接收，Bandwidth是带宽。

# 压测
yum install -y httpd-tools
ab -c 1000 -n 10000 http://192.168.0.30/  # -c表示并发请求数为1000，-n表示总的请求数为10000
说明：
Requests per second # 平均每个请求花费时间
第二个Time per request # 实际请求的响应时间 

# 域名
nslookup time.geekbang.org

dig +trace @114.114.114.114 +nodnssec time.geekbang.org # +trace表示开启跟踪查询# +nodnssec表示禁止DNS安全扩展；@114.114.114.114指定使用的dns服务器
dns流程查询说明：client(time.geekbang.org) -> 114.114.114.114(可能存在time.geekbang.org缓存) -> NS .org -> m.root-servers.net -> dns9.hichina.com -> ip返回114 DNS服务器。

# 内网域名解析可以通过自建DNS服务器或配置/etc/hosts文件
# 强制使用https好处：防止dns劫持
# 抓包
tcpdump -nn udp port 53 or host 35.190.27.188 
说明：
-nn ，表示不解析抓包中的域名（即不反向解析）、协议以及端口号。
udp port 53 ，表示只显示 UDP 协议的端口号（包括源端口和目的端口）为 53 的包。
host 35.190.27.188 ，表示只显示 IP 地址（包括源地址和目的地址）为 35.190.27.188 的包。
or 表 或

第一条：
36909+ 表示查询标识值，它也会出现在响应中，加号表示启用递归查询
A? 表示查询 A 记录。
geektime.org. 表示待查询的域名。
30 表示报文长度。

第二条：
则是从 114.114.114.114 发送回来的 DNS 响应—-域名 geektime.org. 的 A 记录值为 35.190.27.188。

第三条和第四条，是 ICMP echo request 和 ICMP echo reply，响应包的时间戳 14:02:31.539667，减去请求包的时间戳 14:02:31.508164 ，就可以得到，这次 ICMP 所用时间为 30ms

第5第6条：
反向地址解析 PTR 请求，只有请求包，却没有应答包。ping -n 可禁止ptr解析。

# tcpdump选项解析
-A 以ASCII格式显示网络包(不指定则只显示头信息)
-i 指定网口
-nn 不反向解析
-w 保存到文件，以.pcap后缀结尾

host 主机过滤
port 端口过滤
tcp 协议过滤
and/or/not 逻辑表达

# tcpdump输出格式
时间戳 协议 源地址.源端口 > 目的地址.目的端口 网络包详细信息
tcpdump -nn udp port 53 or host 35.190.27.188 -w ping.pcap

# tcpdump & wireshark 抓包 tcp & http案例分析三次握手和四次挥手工作原理
dig +short example.com93.184.216.34
tcpdump -nn host 93.184.216.34 -w web.pcap

curl http://example.com

wireshark中分析：
由于 HTTP 基于 TCP，所以最先看到的三个包，分别是 TCP 三次握手的包。接下来，中间的才是 HTTP 请求和响应包，而最后的三个包，则是 TCP 连接断开时的三次挥手包。

从菜单栏中，点击 Statistics -> Flow Graph，然后，在弹出的界面中的 Flow type 选择 TCP Flows，可以更清晰的看到，整个过程中 TCP 流的执行过程。

之所以有三个包，是因为服务器端收到客户端的 FIN 后，服务器端同时也要关闭连接，这样就可以把 ACK 和 FIN 合并到一起发送，节省了一个包，变成了“三次挥手”。

而通常情况下，服务器端收到客户端的 FIN 后，很可能还没发送完数据，所以就会先回复客户端一个 ACK 包。稍等一会儿，完成所有数据包的发送后，才会发送 FIN 包。这也就是四次挥手了。

# HTTP分析工具：fiddler
$ curl -s -w 'Http code: %{http_code}\nTotal time:%{time_total}s\n' -o /dev/null http://192.168.0.30/ # 获取状态码和时间

# 模拟ddos攻击
hping3 -S -p 80 -i u10 192.168.0.30 #  -S参数表示设置TCP协议的SYN（同步序列号），-p表示目的端口为80。-i u10表示每隔10微秒发送一个网络帧

curl -w 'Http code: %{http_code}\nTotal time:%{time_total}s\n' -o /dev/null --connect-timeout 10 http://192.168.0.30 # --connect-timeout表示连接超时时间

sar -n DEV 1 # 观察收发情况

tcpdump -i eth0 -n tcp port 80
结果：Flags [S] 表示这是一个 SYN 包。大量的 SYN 包表明，这是一个 SYN Flood 攻击。即客户端构造大量的 SYN 包，请求建立 TCP 连接；而服务器收到包后，会向源 IP 发送 SYN+ACK 报文，并等待三次握手的最后一次 ACK 报文，直到超时。

TCP 半开连接的方法，关键在于 SYN_RECEIVED 状态的连接。
netstat -n -p | grep SYN_REC # -n表示不解析名字，-p表示显示连接所属进程

iptables -I INPUT -s 192.168.0.2 -p tcp -j REJECT # 封ip
iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT # 限制syn并发数为每秒1次
iptables -I INPUT -p tcp --dport 80 --syn -m recent --name SYN_FLOOD --update --seconds 60 --hitcount 10 -j REJECT # 限制单个IP在60秒新建立的连接数为10

# 半开状态的连接数是有限制的
sysctl net.ipv4.tcp_max_syn_backlog

sysctl -w net.ipv4.tcp_max_syn_backlog=1024 # 增加半开连接容量

sysctl -w net.ipv4.tcp_synack_retries=1 # 减少半开连接重试次数

# 开启 SYN Cookies不维护半开连接状态
sysctl -w net.ipv4.tcp_syncookies=1

# 持久化保存
$ cat /etc/sysctl.conf
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_max_syn_backlog = 1024

# 基于一些服务端会禁用icmp，使用hping3 or traceroute 测试延迟
hping3 -c 3 -S -p 80 baidu.com # -c表示发送3次请求，-S表示设置TCP SYN，-p表示端口号为80

traceroute --tcp -p 80 -n baidu.com  # --tcp表示使用TCP协议，-p表示端口号，-n表示不对结果中的IP地址执行反向域名解析

# NAT
SNAT(转换源ip，实现上网)
192.168.0.2 -> 路由器（NAT网关）转换成公网ip：100.100.100.100 -> 百度

DNAT(暴露内网服务)
baidu.com 发回响应包 -> 路由器 -> 公网ip替换成192.168.0.2 -> 192.168.0.2

# NAT实现原理
Linux 内核提供的 Netfilter ，具体可通过工具iptables或firewalld实现链配置。

对SNAT: iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -j MASQUERADE
对DNAT: iptables -t nat -A PREROUTING -d 100.100.100.100 -j DNAT --to-destination 192.168.0.2

# 在使用 iptables 配置 NAT 规则时，Linux 需要转发来自其他 IP 的网络包，需开启 Linux 的 IP 转发功能
查看：sysctl net.ipv4.ip_forward
开启：sysctl -w net.ipv4.ip_forward=1
持久化保存：
cat /etc/sysctl.conf | grep ip_forward
net.ipv4.ip_forward=1

# 网络工具合集
sar # 可查看网络接口、进程、IP地址吞吐量(BPS) 
netstat或ss # 网络连接
ping或hping3 # 网络延迟
traceroute # 查看路由链路
nslookup或dig # DNS解析
iptables # 防火墙或NAT
tcpdump & wireshark # 抓包

# 网络优化内核参数设置
- 增大每个套接字的缓冲区大小 net.core.optmem_max；推荐81920
- 增大套接字接收缓冲区大小 net.core.rmem_max 和发送缓冲区大小 net.core.wmem_max； 513920/513920

# cat /proc/sys/net/ipv4/tcp_mem 查看配置
#cat /proc/net/sockstat 查看当前tcp的统计
#sysctl -w net.ipv4.tcp_mem=新配置 来增大
```