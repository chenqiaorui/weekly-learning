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
参考：https://github.com/inancgumus/learngo

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

#### 条件表达
```
# 编辑main.go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello!")

	// Statements change the execution flow
	// Especially the control flow statements like `if`
	if 5 > 1 {
		fmt.Println("bigger")
	}
}
运行：go run main.go
```
#### 引用库
`cd /opt/go-demo/package`

编辑main.go
```
package main

import (
	"fmt" // You should replace this with your username

	"github.com/inancgumus/learngo/05-write-your-first-library-package/exercise/solution/golang"
)

func main() {
	fmt.Println(golang.Version())
}

go mod init package
go mod tidy
go run main.go
```

#### 变量
```
package main

import "fmt"

func main() {
	fmt.Println(42, 8500, 344433, -2323)
	fmt.Println(3.14, 6.28, -42.)
	fmt.Println(true, false)
	fmt.Println("Hi! I'm Inanc!")
	fmt.Println("Merhaba, adım İnanç!")
}
```
#### 打印
```
package main

import "fmt"

func main() {
    total := 12

    fmt.Println("total: ", total)
    fmt.Printf("total: %d\n", total) \\ 整型

    brand = "Google"
	fmt.Printf("%q\n", brand)   \\ 字符串
}
```
#### 打印类型
```
package main

import "fmt"

func main() {
	// I'm using multiple declarations instead of singular
	var (
		speed int
		heat  float64
		off   bool
		brand string
	)

	fmt.Printf("%T\n", speed)
	fmt.Printf("%T\n", heat)
	fmt.Printf("%T\n", off)
	fmt.Printf("%T\n", brand)
}
```
#### 算法操作
```
package main

import "fmt"

func main() {
	fmt.Println(5 % 2) // 1
    fmt.Println(5 / 2) // 2

    // addition operators
	fmt.Println(1 + 2.5) //3.5
	fmt.Println(2 - 3)  // -1
}
```
#### 常量定义 
```
package main

import "fmt"

func main() {
    const a int = 100
    b := 200
    fmt.Printf("%da is %db\n", a, b)
}
```
#### switch
```
package main

import "fmt"

func main() {
	city := "beijing"

	switch city {
	case "beijing":
		fmt.Println("beijing")
	default:
        fmt.Println("where am i?")
    }
}
```
#### 循环
```
package main

import "fmt"

func main() {
    var sum int

	for i := 1; i < 10; i++ {
        sum += i
    }
    fmt.Println(sum)
}
```
#### 数组
https://github1s.com/inancgumus/learngo/blob/master/14-arrays/01-whats-an-array/main.go

#### go搭建一个web服务器
参考：https://github.com/astaxie/build-web-application-with-golang

http包建立web服务器
cd /opt/go-demo/web-go

go mod init web-go

编辑main.go
```
package main

import (
	"fmt"
	"net/http"
	"strings"
	"log"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	r.ParseForm()  //解析参数，默认是不会解析的
	fmt.Println(r.Form)  //这些信息是输出到服务器端的打印信息
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val:", strings.Join(v, ""))
	}
	fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
}

func main() {
	http.HandleFunc("/", sayhelloName) //设置访问的路由
	err := http.ListenAndServe(":9090", nil) //设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

# 运行：go run main.go
# 访问：http://localhost:9090
```
#### go get 连接timeout
go env -w GOPROXY=https://goproxy.cn,direct  # 设置代理即可。

#### 什么是Protobuf
```
数据描述语言。

# 语法
# 在.proto文件中定义Message
syntax = "proto3"; // 指定使用proto3语法。必须是文件中非空非注释行的第一行。

message SearchRequest {      // SearchRequest，定义了3个带有类型的字段，
  string query = 1;          // 唯一编号用来在二进制消息体中识别字段
  int32 page_number = 2;     // 
  int32 result_per_page = 3;
}

// 单个文件可以定义多个message
message M2   ...略

说明：使用Protocol buffer编译器编译.proto文件后生成.pb.go文件
# 定义Service
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);  // 接收SearchRequest，返回SearchResponse
}
```
####  编译器protoc编译.proto文件示例
```
# 安装protoc编译器
cd /opt/protoc-demo

wget https://github.com/protocolbuffers/protobuf/releases/download/v22.0/protoc-22.0-linux-x86_64.zip

# 解压
unzip -d protoc protoc-22.0-linux-x86_64.zip

ln -s /opt/protoc-demo/protoc/bin/protoc /usr/bin/protoc

protoc --version

# 官方的protoc编译器中并不支持Go语言，需要安装一个插件才能生成Go代码
go mod init protoc-demo

go get github.com/golang/protobuf/protoc-gen-go

ln -s /root/go/bin/protoc-gen-go /usr/bin/protoc-gen-go # protoc-gen-go位置可根据find / -name protoc-gen-go 查得 

# 编辑request.protoc
syntax = "proto3"; // 指定使用proto3语法。必须是文件中非空非注释行的第一行。

option go_package="/main"; // 指定包名，必须

message SearchRequest {
  string query = 1;          
  int32 page_number = 2;     
  int32 result_per_page = 3;
}

# 编译
protoc --proto_path=. --go_out=. request.proto

说明：
--proto_path 表request.proto所处位置
--go_out=. 表生成的.pb.go文件输出到当前目录，又因为request.proto里面定义了包名，最终是生成main/request.pb.go
```