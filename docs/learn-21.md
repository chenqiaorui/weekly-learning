#### 源码安装go
环境：centos7

```
cd /opt/go

wget https://storage.googleapis.com/golang/go1.18.3.linux-amd64.tar.gz

tar -xzvf go1.18.3.linux-amd64.tar.gz

# 编辑 /etc/profile，添加：
export PATH=$PATH:/opt/go/go/bin

保存退出，执行 source /etc/profile

go version 
```

#### modd代码热部署
modd作用: 监听文件变化，热部署应用。

##### 示例
编辑modd.conf

```
# demo
# daemon +sigkill: ./data/demo -f app/demo.yaml
app/*.go {
    prep: go build -o data/demo -v app/main.go
    daemon +sigkill: ./data/demo
}
```

项目目录代码
app/main.go

```
package main

import "fmt"

func main() {
    fmt.Println("hello world")
}
```

执行modd: `modd -f modd.conf`，修改app/main.go，查看修改是否生效。

更多：https://github.com/cortesi/modd

#### 构建一个普通的go示例demo步骤
```
cd /opt/go-demo

# vi hello.go

package main

import "fmt"

func main() {
  fmt.Println("hello world");
}

执行: go run hello.go // hello world
```

##### go语法介绍
```
# := 的用法：左边有一个新变量，且变量不加var，把i赋值给变量t
var i = 5;
t := i;

# 包别名
import(
    ff "fmt"
)

ff.Println();

# 布尔值
var b bool
b = true

# 整型
var i,j int
var i init8 // 8位整型

# 数组
var arr [10]int

# 结构类型 struct

type s struct {
    X int
    Y int
}

# 字符串
var s string = "go"
```
#### go 命令介绍
- `go build` 编译源码文件以及它们的依赖包
比如，在`go build`后面不加任何代码文件，它将试图编译当前目录下对应的main.go。Windows系统会生成相应的.exe可执行文件。
- `go build -o main main.go` 指定编译后的可执行包名称为main

- `go env` 打印go语言环境变量，它的环境变量说明如下：
