#### etcd介绍
- 键值对存储，支持key监控

#### 概念
- Peer：对同一个etcd集群中另外一个Member的称呼。

#### 使用
```
# 设置
etcdctl put /testdir/testkey "Hello world"

# 获取
etcdctl get /testdir/testkey

etcdctl get --prefix /testdir

# 删除单key
etcdctl del /testdir/testkey

# 监听
etcdctl watch testkey
etcdctl watch -i # 监听多个键

# 设置过期：租约，100s
$ etcdctl lease grant 100
lease 694d71ddacfda227 granted with TTL(100s)

# 附加键foo 到租约，100s后foo被删除
$ etcdctl put --lease=694d71ddacfda227 foo bar

# 撤销租约，但绑定的foo也将被删除
etcdctl lease revoke 694d71ddacfda227

# 刷新租约
etcdctl lease keep-alive 694d71ddacfda227

# 查看租约现状
etcdctl lease timetolive 694d71ddacfda22c
```

### etcd v3 基于grpc通信

#### 1.proto3是结构化数据序列，类xml

proto3 定义了客户端和服务端的交互数据格式，是Protocol Buffer 2 的升级版。

#### 2.proto3 语法
编辑 request.proto 文件
```
syntax = "proto3"; // 指定语法

// 定义消息体
message Request { 
    string page_num = 2;
}
```
#### 3. .proto文件编译后
- for Golang，生成.pg.go文件

#### 4.小结
etcd v3的api接口在grpc服务中定义。