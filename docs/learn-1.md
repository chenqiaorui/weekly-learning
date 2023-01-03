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