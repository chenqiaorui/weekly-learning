#### 一、日志切割
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
创建一个10M的塞满随机二进制数据的文件
```
touch /var/log/log-file
head -c 10M < /dev/urandom > /var/log/log-file 
```

配置轮询配置文件
```
vim /etc/logrotate.d/log-file

/var/log/log-file {
    monthly
    rotate 5
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
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


配置完成后，载入/etc/lograte.d/log-file
```
# 即使轮循条件没有满足，我们也可以通过使用'-f'选项来强制logrotate轮循日志文件，'-v'参数提供了详细的输出
logrotate -vf /etc/logrotate.d/log-file 

ll /var/log/log-file*
```
结果:
-rw-r----- 1 root root   0 Nov 16 14:57 log-file
-rw-r--r-- 1 root root   3 Nov 16 14:57 log-file.1

log-file.1没有变成压缩文件是因为设置了delaycompress，后面再观察下个轮询周期到了就会变成log-file.2.gz文件

logrotate自身的日志在`/var/lib/logrotate/`目录下


#### 三、logrotate生产应用
nginx日志轮询
```
vi /etc/logrotate.d/nginx

/usr/local/nginx/logs/*.log {
    daily
    compress
    delaycompress
    rotate 365
    missingok
    notifempty
    dateext
    sharedscripts
    postrotate
      if [ -f /usr/local/nginx/logs/nginx.pid ]; then
          kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
      fi
    endscript
}
```
接着执行
```
logrotate /etc/logrotate.d/nginx # 载入配置，因为没有达到daily的轮询条件，要到第二天才能看到轮询结果。可'logrotate -vf /etc/logrotate.d/nginx' 强制执行
```

说明：
- `dateext` 表示以日期命名轮询文件后缀，如access.log变成access.log-20211116
- `sharedscripts` 意味着 postrotate 脚本将只运行一次（在旧日志被压缩之后），而不是为每个轮询的日志运行一次

- USR1通常被用来告知应用程序重载配置文件；例如，向Apache HTTP服务器发送一个USR1信号将导致以下步骤的发生：停止接受新的连接，等待当前连接停止，重新载入配置文件，重新打开日志文件，重启服务器，从而实现相对平滑的不关机的更改。

- kill -HUP pid 或者 killall -HUP pName，其中pid指进程id，pName为进程名称。
如果想要更改配置而不需停止并重新启动服务，可以使用上面两个命令。在对配置文件作必要的更改后，发出该命令以动态更新服务配置。根据约定，当你发送一个挂起信号(信号1或HUP)时，大多数服务器进程(所有常用的进程)都会进行复位操作并重新加载它们的配置文件。