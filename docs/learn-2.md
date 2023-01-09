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

### Systemd 练习
systemctl list-units # 查看所有运行单元
http://ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html