# 每周学习第 2 期

这里记录过去一周，我看到的值得学习的东西，每周六发布。
## APISIX网关使用
### 安装配置

```
git clone https://github.com/apache/apisix-docker.git
cd apisix-docker/example

# 修改conf/config.yaml
admin_key
  -
    name: "admin"
    key: newsupersecurekey  # 请修改 key 的值

docker-compose up -d # 修改docker-compose单个服务的配置，使用本命令可以生效
chmod 777 apisix_log # 如果因文件权限问题启动不了服务，可临时修改文件权限，启动后进到容器内部，查看容器内目录uid和gid(执行`id`查看)。创建apisix用户`useradd apisix`，更改宿主机相应目录 chown -R apisix:apisix apisix_log

# 访问管理面板
http://ip:9000 admin/admin

# 访问admin接口，即/apisix/route这些接口
curl http://127.0.0.1:9180/apisix/admin/routes?api_key=apisix -i

# 发布upstream
curl "http://127.0.0.1:9180/apisix/admin/upstreams/1" \
-H "X-API-KEY: apisix" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "httpbin.org:80": 1
  }
}'

# 发布路由
curl "http://127.0.0.1:9180/apisix/admin/routes/1" \
-H "X-API-KEY: apisix" -X PUT -d '
{
  "methods": ["GET"],
  "host": "example.com",
  "uri": "/anything/*",
  "upstream_id": "1"
}'

# 访问
curl -i -X GET "http://127.0.0.1:9080/anything/foo?arg=10" -H "Host: example.com"

# 插件：proxy-rewrite，重写路由
# 将/cookie/xx 重写成 /xx
"plugins": {
    "proxy-rewrite": {
      "regex_uri": [
        "^/cookie/(.*)",
        "/$1"
      ],
      "scheme": "http"
    }
}

# 当同一plugin同时绑定在route，consumer，Service时，只有一个生效，优先级： Consumer > Route > Plugin Config > Service。每一个插件的内部执行过程：rewrite阶段、access、before_proxy、header过滤、body过滤、log。

# 通过限制访问频率来保护接口
# plugin: limit-conn，限制并行数
# plugin: limit-req，限制请求数量
# plugin: limit-count，限制单位时间内的请求数量，如限制60s内只能请求2次，超过则返回503。
```
说明：

- `./conf/config-default.yaml`为默认配置，与代码强绑定，不修改；自定义配置可修改./conf/config.yaml
- 不修改`./conf/nginx.conf`，APISIX启动会根据`config.yaml`重置`nginx.conf`

### 计算机启动经历的事情
ROM(read-only memory) -> BIOS(basic-input-output-System)是一段程序: 进行硬件自检，显示内存磁盘等信息；接着指定启动顺序：决定使用哪块磁盘 -> 磁盘第一个扇区512bytes，存着主引导记录(Master boot record，简称MBR)，MBR分为1：调用系统的机器码；2：分区表(Partition table)；3：主引导记录签名（0x55和0xAA），表示该磁盘可以作为启动盘。分区表又可以分为主分区。 -> 主分区第一个扇区存着卷引导记录(volume boot record，即VBR，它记录着操作系统位置) 。

上面是windows系统的步骤。linux的启动步骤：

BIOS决定使用哪块磁盘后，交给grub启动界面 -> 控制权转到操作系统 -> 加载/boot/kernel -> 运行/sbin/init生成init进程，pid为1，其他进程都是子进程。 -> init进程加载网络程序、/sbin/login进行登录。

init进程启动后 -> 根据运行级别运行程序/etc/rc*.d(如/etc/rc1.d) -> 命令行或ssh登录(先加载/etc/profile，再加载~/.bash_profile、~/.bash_login、~/.profile。需要注意的是，这三个文件只要有一个存在，就不再读入后面的文件了。比如，要是 ~/.bash_profile 存在，就不会再读入后面两个文件了。)

### Nginx之nginx.conf配置文件解析
```
user  nobody; # 启动nginx子进程用户
worker_processes  auto; # 启动子进程个数，一般等于cpu个数
worker_cpu_affinity auto; # 子进程绑定cpu
worker_rlimit_nofile 51200; # 单进程可打开的最大文件数

#error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
        use epoll;
        worker_connections  51200; # 单进程能并发处理的请求数
        multi_accept on; # 开启进程能否同时处理并发请求，默认关闭
}

http {

    include       /etc/nginx/mime.types; # 设置Content-type，返回给浏览器让其正确渲染
    default_type  application/octet-stream; # 设置Content-type为字节流，返回给浏览器后默认下载

    sendfile        on; # 提高文件的传输速率
    tcp_nopush      on; # 在sendfile打开的状态下才会生效，将数据存到缓冲区，存满后再发，减少网络开销
    tcp_nodelay     on; # 有数据就发
    keepalive_timeout 65s; # 客户端与nginx连接时长，设置太短可能会导致文件上传失败

    open_file_cache            max=102400 inactive=20s; # 为102400个元素设置缓存，过期时间是20s
    open_file_cache_valid      30s; # 30s后去查看缓存文件是否有效
    open_file_cache_min_uses   1; # 20s内访问次数小于1则从缓存中删除

    client_header_buffer_size   4k; # 请求头大小超过4k，则参考large_client_header_buffers
    large_client_header_buffers 4 128k; # 每个header不超128k，全部header不超4*128k
    client_max_body_size        2048m; # 请求body超2048m返回413错误。
    client_body_buffer_size     256k; # body小于256k会存于缓存，大于256k则参考client_max_body_size大小，存到临时文件。

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" $request_time '
                      '$upstream_response_time $upstream_addr "$host"';
                       
    gzip  on; # nginx将文件压缩传给客户端，并在响应头添加Content-Encoding：gzip
    gzip_min_length  1k; # 超过1k文件就压缩
    gzip_buffers     4 16k; # 申请4个缓存空间，每个为16k
    gzip_http_version 1.0;
    gzip_comp_level 2; # 压缩级别越低，压缩效果越不明显，但快
    gzip_types text/plain  text/css application/json application/javascript application/x-javascript text/javascript text/xml application/xml application/rss+xml  application/atom+xml application/rdf+xml; # 哪些文件被压缩
    gzip_vary on; # 添加响应头Vary: Accept-Encoding，告知浏览器文件被压缩
    gzip_proxied        expired no-cache no-store private auth; # 是否对后端(代理服务器)返回的内容进行压缩
    gzip_disable        "MSIE [1-6]\."; # 排除不支持gzip的浏览器版本

    #server_tag off;
    #server_info off;
    server_tokens off; # 关闭在响应头显示nginx版本

    access_log  /var/log/nginx/access.log  main;

    include /etc/nginx/conf.d/*.conf;
}

```
### Python库工具jc
能够将普通文本输出转换成json数据
```
# 安装
pip3 install jc

# 使用
dig example.com | jc --dig

# 获取ip
dig example.com | jc --dig | jq -r '.[].answer[].data'

参考：https://kellyjonbrazil.github.io/jc/
```

### CSS训练
```
# 练习网站
https://codepen.io/pen?editors=1111

# 练习课堂
https://web.dev/learn/css/box-model/

# 示例
<p>xxx</p>

p {
    width: 50px; # 宽度
    height: 50px; # 高度
    padding: 50px; # 内边距
    boder: 1px solid; # 1px边框宽度，solid实线 
}
```

### 前端构建工具Webpack
```
# 概念
构建就是将所有源码转换成可执行的HTML、CSS、JS代码。简单讲就是把项目打包成可执行文件。
- 如typescript转换成Javascript、SCSS转换成CSS；
- 文件优化，如压缩js、html、css、图片；
- 模块合并
- 自动刷新构建到浏览器运行
- 代码校验

# 构建用到的工具：npm

参考：http://webpack.wuhaolin.cn/
```
### centos7安装docker-compose
```
# 下载docker-compose
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.15.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose 

# 赋权
chmod +x /usr/local/bin/docker-compose

# 查看版本
docker-compose version
```