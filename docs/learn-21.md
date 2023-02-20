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

```
- GOBIN // 存放可执行文件的目录的绝对路径。
- GOARCH // 执行环境计算架构
- GOEXE // 可执行文件后缀
- GOHOSTOS // 执行环境操作系统
- GOPATH // 工作区目录的绝对路径。我们需要显式的设置环境变量GOPATH。如果有多个工作区，那么多个工作区的绝对路径之间需要用分隔符分隔。在windows操作系统下，这个分隔符为“;”。在其它操作系统下，这个分隔符为“:”。注意，GOPATH的值不能与GOROOT的值相同。

- GOROOT // Go语言的安装目录的绝对路径。GOROOT会是我们在安装Go语言时第一个碰到Go语言环境变量。它的值指明了Go语言的安装目录的绝对路径。但是，只有在非默认情况下我们才需要显式的设置环境变量GOROOT。这里所说的默认情况是指：在Windows操作系统下我们把Go语言安装到c:\Go目录下，或者在其它操作系统下我们把Go语言安装到/usr/local/go目录下。另外，当我们不是通过二进制分发包来安装Go语言的时候，也不需要设置环境变量GOROOT的值。比如，在Windows操作系统下，我们可以使用MSI软件包文件来安装Go语言
```
- `go install` 编译和安装文件及其依赖包，一定程度上等价于`go build`，但 `go install`可以指定编译后可执行文件的位置。

- `go get` 安装指定依赖。如 `go get -u github.com/chromedp/chromedp`。`-u`表示只会从网络上下载本地不存在的代码包，而不会更新已有的代码包。

- `go run` 编译和运行源码文件，不需要生成可执行文件即可执行。如`go run main.go`
