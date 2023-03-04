#### 背景
redis是基于内存存储的kv数据库，当redis服务异常的时候，首要分析redis是否存在大key或热key。借助工具：redis-rdb-tools可以清楚看到redis的键值大小等情况。

#### docker-compose 部署 redis
vi /opt/docker/docker-compose.yml
```
version: '3.5'
services:
  redis:
    image: redis:5
    restart: always
    container_name: redis
    ports:
        - "6379:6379"
    volumes:
        - "./redis/data:/data"
    command: redis-server --requirepass sxsEcUrE
```
部署redis：`docker-compose up -d`

#### 客户端工具
[Another Redis Desktop Manager](https://github.com/qishibo/AnotherRedisDesktopManager/releases)


##### 使用redis内存分析工具 -- redis-rdb-tools
```
# 安装libffi-devel
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel

# 下载python安装包
wget -P /tmp https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz

# 解压 & 编译 & 安装
mkdir -p /opt/python3 
tar -zxvf /tmp/Python-3.7.3.tgz -C /opt/python3
mkdir /usr/local/python3 
cd /opt/python3/Python-3.7.3 
./configure --prefix=/usr/local/python3
make && make install

# 建立软连接
ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3.7 /usr/bin/pip3
python -V # 查看版本

# 安装redis-rdb-tools
yum install python3-devel
pip3 install python-lzf
pip3 install rdbtools

# 进入redis容器，redis生成dump.rdb文件
[root@centos7 docker]# docker exec -ti redis bash
root@8a3338be9ec9:/data# redis-cli
127.0.0.1:6379> save
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth sxsEcUrE
OK
127.0.0.1:6379> set ss1 tt1
OK
127.0.0.1:6379> save
OK
127.0.0.1:6379> exit
root@8a3338be9ec9:/data# ls
dump.rdb

# 使用rdb-tools工具分析rdb文件
rdb -c memory /mnt/data/redis/dump.rdb >  /mnt/data/redis/memory.csv
```
说明：
```
可以看到，用工具转化成csv文件后，会划分成8个列，分别是：

- database：数据库编号
- type：数据类型
- key：键
- size_in_bytes：使用的内存：包括键，值和任何其他开销
- encoding：RDB编码类型
- num_elements：key中的value的个数
- len_largest_element：key中的value的长度
- expiry：过期值
```

#### rdb工具更多使用
```
# 导出内存字节排名前3的keys
rdb --command memory --largest 3 dump.rdb

# 导出rdb中的keys
rdb -c justkeys dump.rdb|uniq
```

参考：
https://blog.csdn.net/weixin_48380416/article/details/123995573

https://juejin.cn/post/7167161498840596510

https://github.com/sripathikrishnan/redis-rdb-tools