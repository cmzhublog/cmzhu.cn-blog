## golang 基础语法学习

Go 语言的基础组成有以下几个部分：

- 包声明
- 引入包
- 函数
- 变量
- 语句 & 表达式
- 注释

使用go 完成hello_world 代码的编写

```go
// golang 实现hello world

// 指定包名
package main 

// 导入需要使用的包
import "fmt"

//定义程序入口 main 函数

func main() {
	fmt.Println("Helo, World!!")
}
// 尝试运行
/*
  go run hello_world.go

*/
// 编译运行
/*
   go build hello_world.go

*/
```

1、 在终端编译运行

```bash
$ go run hello_world.go 
Helo, World!!
```

