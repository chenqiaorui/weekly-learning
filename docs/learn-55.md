#### golang安装或升级

# 升级
yum upgrade go

go version
#### goctl 安装

```
# Go.16+
GOPROXY=https://goproxy.cn/,direct go install github.com/zeromicro/go-zero/tools/goctl@latest

# 查看版本
goctl -v
```
#### Go Module设置
```
#Go Module是Golang管理依赖性的方式，像Java中的Maven
# 开启GO111MODULE
go env -w GO111MODULE="on"

# 设置GOPROXY
go env -w GOPROXY=https://goproxy.cn
```
#### protoc & protoc-gen-go安装
```
goctl env check -i -f --verbose

ln -s /root/go/bin/goctl /usr/bin/goctl
```
#### 构建go-mall
```
$ mkdir go-mall
$ cd go-mall
$ go mod init go-mall
```
#### 创建user rpc
```
ln -s /root/go/bin/protoc /usr/bin/protoc

ln -s /root/go/bin/protoc-gen-go /usr/bn/protoc-gen-go

ln -s /root/go/bin/protoc-gen-go /usr/bin/protoc-gen-go

ln -s /root/go/bin/protoc-gen-go-grpc /usr/bin/protoc-gen-go-grpc

mkdir -p app/user/cmd/rpc
```
编辑app/user/cmd/rpc/user.proto
```
syntax = "proto3";

package user;
  
// protoc-gen-go 版本大于1.4.0, proto文件需要加上go_package,否则无法生成
option go_package = "./user";

message IdRequest {
    string id = 1;
}
  
message UserResponse {
    // 用户id
    string id = 1;
    // 用户名称
    string name = 2;
    // 用户性别
    string gender = 3;
}
  
service User {
    rpc getUser(IdRequest) returns(UserResponse);
}
```

生成代码
```
goctl rpc protoc user.proto --go_out=./types --go-grpc_out=./types --zrpc_out=.
```
填充业务逻辑
```
编辑internal/logic/getuserlogic.go


func (l *GetUserLogic) GetUser(in *user.IdRequest) (*user.UserResponse, error) {
    return &user.UserResponse{
            Id:   "1",
            Name: "test",
    }, nil
}
```
#### 创建order api服务
```
mkdir -p app/order/cmd/api

cd app/order/cmd/api

vim order.api

type(
  OrderReq {
    Id string `path:"id"`
  }

  OrderReply {
    Id string `json:"id"`
    Name string `json:"name"`
  }
)

service order {
  @handler getOrder
  get /api/order/get/:id (OrderReq) returns (OrderReply)
}
```
#### 生成order服务
```
goctl api go -api order.api -dir .

# 添加user rpc配置
vim internal/config/config.go

package config

import (
    "github.com/zeromicro/go-zero/zrpc"
    "github.com/zeromicro/go-zero/rest"
)

type Config struct {
    rest.RestConf
    UserRpc zrpc.RpcClientConf
}

# 添加yaml配置
vim etc/order.yaml

Name: order
Host: 0.0.0.0
Port: 8888
UserRpc:
  Etcd:
    Hosts:
    - 127.0.0.1:2379
    Key: user.rpc

# 完善服务依赖
vim internal/svc/servicecontext.go

package svc

import (
        "go-mall/app/order/cmd/api/internal/config"
        "go-mall/app/user/cmd/rpc/userclient"

        "github.com/zeromicro/go-zero/zrpc"
)

type ServiceContext struct {
        Config config.Config
        UserRpc userclient.User
}

func NewServiceContext(c config.Config) *ServiceContext {
        return &ServiceContext{
                Config: c,
                UserRpc: userclient.NewUser(zrpc.MustNewClient(c.UserRpc)),
        }
}

# 给 getorderlogic 添加业务逻辑
vim internal/logic/getorderlogic.go

package logic

import (
        "context"
        "errors"

        "go-mall/app/order/cmd/api/internal/svc"
        "go-mall/app/order/cmd/api/internal/types"
        "go-mall/app/user/cmd/rpc/types/user"
        
        "github.com/zeromicro/go-zero/core/logx"
)

type GetOrderLogic struct {
        logx.Logger
        ctx    context.Context
        svcCtx *svc.ServiceContext
}

func NewGetOrderLogic(ctx context.Context, svcCtx *svc.ServiceContext) *GetOrderLogic {
        return &GetOrderLogic{
                Logger: logx.WithContext(ctx),
                ctx:    ctx,
                svcCtx: svcCtx,
        }
}

func (l *GetOrderLogic) GetOrder(req *types.OrderReq) (resp *types.OrderReply, err error) {
        // todo: add your logic here and delete this line
        user, err := l.svcCtx.UserRpc.GetUser(l.ctx, &user.IdRequest{
            Id: "1",
        })
        if err != nil {
            return nil, err
        }

        if user.Name != "test" {
            return nil, errors.New("用户不存在")
        }
        return &types.OrderReply{
            Id:   req.Id,
            Name: "test order",
        }, nil

}

# 启动etcd
etcd &

# 切换到go-mall目录，下载依赖
go mod tidy

# 启动 user rpc
cd app/user/cmd/rpc/
go run user.go -f etc/user.yaml

# 启动 order api
cd app/order/cmd/api
go run order.go -f etc/order.yaml

# 访问order api服务
curl -i -X GET http://localhost:8888/api/order/get/1

```