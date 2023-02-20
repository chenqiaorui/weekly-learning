参考：https://github.com/inancgumus/learngo

#### package
编辑main.go
```
package main
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
