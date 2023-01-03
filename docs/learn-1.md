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