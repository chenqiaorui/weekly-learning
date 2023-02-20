#### 安装
centos
```
# 依赖安装
yum -y install gcc tcl

gcc -v # gcc版本需要5.3以上

# 升级gcc
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

scl enable devtoolset-9 bash
# 需要注意的是 scl 命令启用只是临时的，退出 shell 或重启就会恢复原系统 gcc 版本。
# 如果要长期使用 gcc 9.3的话：
echo -e "\nsource /opt/rh/devtoolset-9/enable" >>/etc/profile

cd /opt/soft
wget http://download.redis.io/releases/redis-6.0.9.tar.gz
tar -zxvf redis-6.0.9.tar.gz

# 编译
cd redis-6.0.9 && make MALLOC=libc

# 安装
make PREFIX=/usr/local/redis install
cd /usr/local/redis
cp redis.conf 6379.conf

# 启动
./bin/redis-server 6379.conf

# 测试
./bin/redis-cli
127.0.0.1:6379> ping
PONG
```
#### 常用命令
redis-cli -h 127.0.0.1 -p 6379  -a "passwd"
set aa bb

#### redis.conf配置详解

#### redis执行lua脚本语法
cd /opt/redis-demo

编辑hello.lua
```
local msg = "Hello, world!"
return msg
```
执行：`redis-cli EVAL "$(cat Hello.lua)" 0`  // EVAL返回一个值。第一个参数是lua脚本，第二个参数是需要访问的redis键的数字号，我们简单的"Hello Script"不会访问任何键，所以我们使用0。

message demo check `https://github.com/Tinywan/lua-nginx-redis/blob/master/Redis/redis-lua.md`
