#### 一、centos7安装Nginx
##### 源码安装
- [所有nginx版本](http://nginx.org/download/)

wget下载tar.gz包
```
wget http://nginx.org/download/nginx-1.20.1.tar.gz

# 解压
tar xzf nginx-1.20.1.tar.gz
```

编译
```
cd nginx-1.20.1 && ./configure
```
注意：`安装报错误的话比如："C compiler cc is not found"，这个就是缺少编译环境，安装一下就可以了 yum -y install gcc make gcc-c++ openssl-devel`

安装
```
make && make install
```
说明：
- nignx会被安装到`/usr/local/nginx`目录下

nginx测试
```
$ which nginx
/usr/bin/nginx

nginx # 启动nginx
ps aux|grep nginx # 查看nginx进程
nginx -s stop # 停止nginx

nginx -v # nginx 版本
nginx -t # 测试配置文件
nginx -s reload # 重载配置

```

防火墙设置
```
systemctl status firewalld # 查看防火墙状态
```

设置放行端口，编辑`/etc/firewalld/zones/public.xml`文件，在</zone>前面加:

```
<zone>
  ...
  <port port="30000-30100" protocol="tcp"/>
  <port port="80" protocol="tcp"/>
</zone>
```
说明：
- 可以设置端口段或单个端口

配完文件后重载`firewall-cmd --reload`，查看被放行的端口
```
firewall-cmd --list-ports 
```

#### 二、配置文件
查看`/usr/local/nginx/conf/nginx.conf`文件：
```
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    include  vhost/*.conf;
}

```
说明：
- `worker_processes` - work进程数，一般根据cpu核数来设置，可以设为和CPU的数量一样
- `worker_connections` - 单个work进程可以连的并发连接数，连接数依赖于系统级别的进程可打开文件数，`ulimit -n`可查看
  
  `cat /proc/nginx_work_pid/limits` 里面的`Max open files`也可查询。

- `log_format` - 设置nginx日志格式
- `access_log` - 设置文件存储位置和应用的日志格式
- `sendfile` - 开启文件高效传输模式
- `keepalive_timeout` - 客户端和nginx之间建立的tcp长连接维持时间
- `gzip` - 开启gzip模块，文件被压缩传到客户端，优化传输效率，有效节省带宽。
- `listen 80` - 监听80端口
- `root   html;` - 相当于`root /usr/local/nginx/html`
- `index index.html` - 查找`/usr/local/nginx/html`目录下的index.html文件
- `include` - 添加配置文件

示例：在vhost下添加配置文件`test.conf`
```
server {
    listen 80;
    server_name 192.168.1.189;
    location /ricky/ {
	    return 200 "im ricky";
    }
}
```
注意：
- server_name指定为本机的ip而不是localhost，因为上一目录nginx.conf配置的localhost
- 指定访问前缀为/ricky/

检查并重载配置
```
nginx -t
nginx -s reload
```

访问：curl http://192.168.1.189:80/ricky/  
注意：
- 访问的时候`192.168.1.189` 不能写成localhost，不然匹配规则就跑到nginx.conf里面的server去了
- ricky后面的`/`不可以省略；如果想省略，可以把`location /ricky/`写成`location /ricky`
  
  `location /ricky`既会匹配到`http://192.168.1.189:80/ricky`的路由，也会匹配到`http://192.168.1.189:80/rickyadsafaf/a`

  总的来说，nginx先在所有配置文件中匹配端口->再到server_name->具体location(location之间又有优先级的区分)

#### 三、location优先级
优先级高到低
- `location = /uri` - 字符 `=` 表示精准匹配（精准匹配优先级最高）
- `location ^~ /uri` - 字符 `^` 表示以/test开头的路径，`~` 表区分大小写；但是 `~*` 表不区分大小写(带 `~*` 等修饰符优先级次之)
- `location /url` - 不带修饰符的前缀再次之
- `location /` - 最后是交给`/`通用匹配。

示例1：比较精准和带修饰符
```
server {
    listen 80;
    server_name 192.168.1.189;
    location = /ricky {
	    return 200 "精准";
    }

    location ^~ /ricky {
	    return 200 "修饰符";
    }
}

```
重载配置并访问：`curl http://192.168.1.189:80/ricky`，输出`精准`

示例2：比较带修饰符和不带修饰符
```
server {
    listen 80;
    server_name 192.168.1.189;
    location ^~ ricky {
	    return 200 "精准";
    }

    location / {
	    return 200 "通用";
    }
}

```
重载配置并访问：`curl http://192.168.1.189:80/ricky`，输出`精准`；访问不在/ricky规则内的就打出`通用`，如`curl http://192.168.1.189:80/ris`

#### nginx不需要后端就可以返回远程用户ip的配置
配置:
```
location /ip {
    default_type text/plain;
    return 200 $remote_addr;
}
```
访问
```
$ curl https://example.com/ip
192.168.1.187
```

返回json格式：
```
$ curl -s https://example.com/json_ip | jq
{
    "ip": "192.168.1.187"
}
```