# 每周学习第 1 期

这里记录过去一周，我看到的值得学习的东西，每周六发布。

# 搭建Nginx和PHP-FPM网站
## 环境和软件安装
机器：centos7

安装
```
# 安装epel和remi
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm

# 假如想下载最新nginx，新增文件/etc/yum.repos.d/nginx.repo，并写下以下内容

[nginx] 
name=nginx repo 
baseurl=http://nginx.org/packages/centos/7/$basearch/ 
gpgcheck=0 
enabled=1

# 安装nginx
yum install nginx

# 应用remi-php73.repo作为源并下载php7.3版本
yum-config-manager --enable remi-php73
yum install -y php php-common php-fpm
yum install -y php-mysql php-pecl-memcache php-pecl-memcached php-gd php-mbstring php-mcrypt php-xml php-pecl-apc php-cli php-pear php-pdo

# 修改FastCGI进程所属用户和用户组，编辑/etc/php-fpm.d/www.conf
# 默认是apache，现在修改成nginx

user = nginx
group = nginx

# 查看FastCGI监听的端口
listen = 127.0.0.1:9000

# 设置开机自启动和启动php-fpm
systemctl enable php-fpm
systemctl start php-fpm

# 设置开机自启动和启动nginx
systemctl enable nginx
systemctl start nginx

# 设置网站
mkdir -p /var/www/html/example1.com/ 
mkdir -p /var/log/nginx/example1.com/ 

# 设置nginx server block配置
# vi /etc/nginx/conf.d/example1.com.conf

server {
  listen 80;
  server_name example1.com www.example1.com;

  root   /var/www/html/example1.com/;
  index index.php index.html index.htm;

  #charset koi8-r;
  access_log /var/log/nginx/example1.com/example1_access_log;
  error_log   /var/log/nginx/example1.com/example1_error_log   error;

  location / {
    try_files $uri $uri/ /index.php?$query_string;
  }

  # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
  location ~ \.php$ {

    root    /var/www/html/example1.com/;
    fastcgi_pass   127.0.0.1:9000;	#set port for php-fpm to listen on
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include         fastcgi_params;
    include /etc/nginx/fastcgi_params;
  }
}

# 写index.php文件
echo "<?php phpinfo(); ?>" > /var/www/html/example1.com/index.php

# 检查nginx配置和重载配置
nginx -t
nginx -s reload 

# 配置/etc/hosts文件
192.168.31.87   example1.com   example1

# 访问服务
curl http://example1.com/index.php

# 如果服务不可访问，可能要关闭防火墙
systemctl stop iptables
```
说明：

- CentOS系统内置的yum源中，没有想要的安装包，可额外添加yum源安装。安装epel和remi会在/etc/yum.repos.d/下生成相关的remi*.repo和epel*.repo文件。
- 假设有一个请求`curl localhost`进到nginx，会先匹配`location /`的内容，通过try_files找到的index.php再匹配到`location ~ \.php$`。
- `try_files $uri $uri/ /index.php?$query_string;` 按顺序查找文件。如`$uri`查找http://localhost/下的index文件;`$uri/`查找http://localhost/path/下的index文件;`/index.php?$query_string;`查找http://localhost/index.php?xx。
- `fastcgi_pass`指定将与nginx通讯的fastcgi位置。
- `fastcgi_index  index.php;`，index.php将被存到`$fastcgi_script_name`这个变量上。
- `fastcgi_param`用于nginx给fastcgi传递参数。`SCRIPT_FILENAME`可以被cgi识别， `$document_root`取自nginx设置的root路径，`$document_root$fastcgi_script_name`等于nginx告诉cgi去`/var/www/html/example1.com/`找`index.php`，处理完结果返回给nginx。
- `include /etc/nginx/fastcgi_params`，举个例子：fastcgi_params文件里面有这么一段`fastcgi_param  REMOTE_ADDR $remote_addr;`，它表示将nginx的$remote_addr赋值给REMOTE_ADDR，REMOTE_ADDR又能被cgi识别，这样就实现了变量的传递。

## PHP-FPM参数调优
```
# 切换到/etc/php-fpm.d，查看www.conf
user = apache # 设置fpm子进程的启动用户
[www] # 设置fpm进程池的名称 

pm = dynamic # 动态生成fpm子进程的个数
pm.max_children = 50 # 最多50个子进程
pm.start_servers = 5 # 初始启动的子进程个数
pm.min_spare_servers = 5 # 最少的子进程个数

pm = static # 设置成静态规则，这个时候只有pm.max_children起作用的，pm.min_spare_servers无效。

pm.max_requests = 500 # 每个子进程处理过500个请求后会重启，可以解决内存泄漏问题。
```
# 服务器操作审计日志设置
```
# 编辑/etc/profile文件，添加：

export HISTTIMEFORMAT="[%Y.%m.%d %H:%M:%S]"
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
HISTDIR=/var/log/.hist
if [ -z $USER_IP  ]
then
  USER_IP=`hostname`
fi
if [ ! -d $HISTDIR ]
then
   mkdir -p $HISTDIR
   chmod 777 $HISTDIR
fi
if [ ! -d $HISTDIR/${LOGNAME} ]
then
    mkdir -p $HISTDIR/${LOGNAME}
    chmod 300 $HISTDIR/${LOGNAME}
fi
export HISTSIZE=4096
DT=`date +%Y%m%d_%H%M%S`
export HISTFILE="$HISTDIR/${LOGNAME}/${USER_IP}.hist.$DT"
chmod 600 $HISTDIR/${LOGNAME}/*.hist* 2>/dev/null
```
说明：

-  `who -u am i`获取当前登录用户，登录时间，登录的远程ip
-  `[ -z $USER_IP ]`如果$USER_IP字符长度为0，返回真

# shell编程之if
```
# false
if [ 1 -eq 2 ];then
  echo true;
else
  echo false;
fi

# 简写: false
[ 1 -eq 2 ] && echo true || echo false

# 简写: false and false
[ 1 -eq 2 ] && echo true || (echo false; echo false)

# -o逻辑或: true, 逻辑与-a
if [ 1 -eq 0 -o 1 -eq 1 ];then
  echo true;
else
  echo false;
fi

# !逻辑非
if [ ! 1 -eq 0 ];then
  echo true; 
fi

# -d目录存在返回true；-f文件存在返回true
if [ -d "/opt" ];then
  echo true;
fi
```
# 成为一个可靠的SRE入门教程
## 基础系列
### linux基本命令
```
ls # 列出文件和目录
pwd
cd

# 操控文件和目录
touch
mkdir
cp
mv
rm

# 查看文件
cat
head
tail
more # 空格向下翻页，enter一行，b向上翻页
less
seq 1 100 > a.txt # 制造一个测试文件

# 文本处理
grep
sed
sort

# 多用户
id
whoami

# 文件
/etc/passwd
/etc/group

useradd
passwd
userdel

useradd aim -s /bin/sh # 创建用户并指定sh
usermod aim -s /bin/bash # 修改用户属性成/bin/bash

groupadd sre # 添加组
usermod -a -G sre aim # 给用户aim多加一个sre组
groups aim # 查看aim所在组，id aim也可以。

# 变成超级用户：拥有sudo能力
grep "wheel" /etc/sudoers
usermod -a -G wheel aim
id aim
su aim
sudo cat /etc/shadow # 检验是否可以打开

# 文件权限
drwxrwxr-- # 文件类型/用户r(4)w(2)x(1)/组rwx/其他人r--
chmod 644 file_name
chown root:root file_name
chgrp root # 改变group

# 远程连接ssh
ssh-keygen # 产生一对key到~/.ssh目录
ssh-copy-id root@192.168.1.1 # 把自己的id_rsa.pub拷贝到192.168.1.1的~/.ssh/authorized_keys
ssh root@192.168.1.1 # 连接
ssh root@192.168.1.1 'ps aux' # 连接执行命令

# 复制文件到远程服务器
scp a.log root@192.168.1.1:/tmp

# 软件包管理
yum search httpd # 查找是否有可安装的包
yum install httpd # 安装
yum remove httpd # 卸载

# 进程管理
ps
top

# 内存管理
free

vmstat # 内存 + io + cpu信息展示

# 磁盘
df
du

# 后台服务：systemd
/usr/lib/systemd/system
systemctl reload name.service # 重载服务配置
systemctl start name.service

# 日志
/var/log
dmesg # 展示内核日志
```
### GIT基础
```
git init # 初始化git仓库，生成.git目录
git add # 追踪文件
git commit # 保存成本地快照
git log # 查看commit历史：作者、commitid、提交日期
git branch b1 # 创建分支，b1分支代码取决于当时创建的分支
git checkout b1 # 切换分支
git merge b1 # 把b1分支代码合并到master，同时添加一个merge commit。
git reset --hard commitid # 重置merge到上一个状态
git rebase master # 假设当前为b2分支，git rebase master将以master最新代码为基底加上b2的commit
```

### 网络基础
```
# DNS 域名
# /etc/hosts文件格式
127.0.0.1 test.linkedin.com # ip FQDN(全限定域名)

dig +trace 域名 # 追踪域名查询整个流程，A记录代表返回域名对应的ip地址，AAAA为ipv6，NS返回记录域名的权威名称服务器，CNAME返回域名的别名

dig A static.example.com +short
dig NS static.example.com +short
dig CNAME static.example.com +short
```

### Python练习
```

```

### 日志轮询 + nginx请求分析
背景：流量大的服务常常会产生很多大文件日志文件，占用磁盘空间，怎么去管理呢？使用日志切割服务logrotate。

logrotate可以对日志进行截断、压缩、删除旧文件。比如说配置logrotate，让/var/log/nginx每30天轮询，并删除超过60天的日志，此过程完全自动化。

安装logrotate，centos7默认自带
```
yum install logrotate
```

##### 运行原理
logrotate的运行依赖`crontab`, 安装logrotate后, 自动在 /etc/cron.daily 目录下添加
logrotate 文件

说明：
- 配置文件：/etc/logrotate.conf，一般不变动
- 设置独立的轮询配置文件，/etc/logrotate.d

#### 二、实践配置
编辑文件/var/log/nginx/access.log
```
192.168.1.167 - - [19/Oct/2022:16:35:15 +0800] "GET /favicon.ico HTTP/1.1" 404 134 "http://192.168.1.167:891/test.log" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:105.0) Gecko/20100101 Firefox/105.0"
192.168.1.167 - - [19/Oct/2022:16:35:45 +0800] "GET /api/status HTTP/1.1" 200 50 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:105.0) Gecko/20100101 Firefox/105.0"
192.168.1.179 - - [19/Oct/2022:15:53:45 +0800] "GET /js HTTP/1.1" 302 154 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36"
192.168.1.179 - - [19/Oct/2022:15:54:08 +0800] "GET /js HTTP/1.1" 302 154 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/106.0.0.0 Safari/537.36"
192.168.1.167 - - [19/Oct/2022:16:35:14 +0800] "GET /test.log HTTP/1.1" 404 134 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:105.0) Gecko/20100101 Firefox/105.0"
192.168.1.179 - - [23/Nov/2022:19:08:31 +0800] "GET /ip HTTP/1.1" 200 13 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36"
```

配置轮询配置文件
```
vim /etc/logrotate.d/nginx

/var/log/nginx/*.log {
    daily
    compress
    delaycompress
    rotate 365
    missingok
    notifempty
    dateext
    sharedscripts
    postrotate
        if [ -f /run/nginx.pid ]; then
            # 重启nginx服务
            kill -USR1 `cat /run/nginx.pid`
        fi
    endscript
}
```
说明：
- `monthly` 每月运行一次。其它可用值为'daily'，'weekly'或者'yearly'
- `rotate 5` 切割后，保留最近的5次切割结果文件
- `compress` 在轮循任务完成后，已轮循的归档将使用gzip进行压缩
- `delaycompress` 总是与compress选项一起用，delaycompress选项指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行。这在你或任何软件仍然需要读取最新归档时很有用。
- `missingok` 在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误。
- `notifempty` 如果日志文件为空，轮循不会进行。
- `create 644 root root` 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件
- `postrotate`在所有其它指令完成后，postrotate和endscript里面指定的命令将被执行。在这种情况下，rsyslogd 进程将立即再次读取其配置并继续运行。
- `size 100k` 日志大小超过 100k 时, 进行切割
- `olddir /var/log/news/old` 切割后数据放入指定目录 /var/log/news/old
- `nocompress` 转储文件不压缩


配置完成后，载入/etc/lograte.d/nginx
```
# 即使轮循条件没有满足，我们也可以通过使用'-f'选项来强制logrotate轮循日志文件，'-v'参数提供了详细的输出
logrotate -vf /etc/logrotate.d/nginx

ll /var/log/nginx/
```
结果:
access.log
access.log-20230105

access.log-20230105没有变成压缩文件是因为设置了delaycompress，后面再观察下个轮询周期到了access.log-20230105就会变成access.log-20230105.gz文件，access.log会变成access.log-20230106，access.log是最新的空文件，`zcat access.log-20230105.gz`查看压缩内容。

logrotate自身的日志在`/var/lib/logrotate/`目录下

接下来分析日志：
```
cd /var/log/nginx
cp access.log-20230105.gz access.log-20230104.gz # 多制造一个副本

zcat access.log-*.gz |wc -l # 统计*.gz文件请求总数
zcat access.log-*.gz |awk -F' ' '{print $7}'|sort -nr|uniq -c # 统计每个接口的请求个数 
```
### CURL使用
```
# -s沉默输出，只输出返回值
curl myip.ipip.net -s|egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" # -o表只输出匹配到的，输出ip

# shell判断
result=`curl myip.ipip.net -s|egrep -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"`
[ -z $result ] && echo "empty" || echo "$result"

# -L 或 --location 跟随重定向，如经历一次301重定向到https
curl --location --request GET "http://httpbin.org/get?foo1=bar1&foo2=bar2"

# -X 或 --request 指定请求方法
curl --location --request GET "http://httpbin.org/get?foo1=bar1&foo2=bar2"

# -o 返回内容下载到指定文件名称文件
curl "http://httpbin.org/get?foo1=bar1&foo2=bar2" -o file

# -e 添加referer
curl -v -e "https://google.com?q=example" "http://httpbin.org/get?foo1=bar1&foo2=bar2"

# -H 添加header，-H也可写成 --header
curl -v "http://httpbin.org/get?foo1=bar1&foo2=bar2" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'

# -X POST -d 发送POST方法，-d也可写成--data
curl -v "http://httpbin.org/get?foo1=bar1&foo2=bar2" -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X POST -d '{"name": "dds"}'
```