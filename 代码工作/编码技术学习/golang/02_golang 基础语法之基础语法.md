## golang 基础语法

Go 程序可以由多个基础语法组成, 可以是关键字、标识符、 常量、字符串、 符号.

```go
fmt.Println("Hello, world !!!")
```

6 个标记分别是

```go
fmt
.
Println
(
"Hello,world !!!"
)
```

### 行分割符

golang 语句中, 不需要像C 一样使用`;` 结尾, 因为这些结尾符将由golang 编译器自动完成, 如果打算将多个语句写在同一行, 则需要以`;` 分开

例子:

```go
fmt.Println("Hello")
fmt.Println("World")
```

### 注释

和C 语言相同

- // 表示单行注释
- /**/ 表示多行注释

### 标识符

和C 语言类似, 标识符命名使用数字, 字母和下划线组成

### 字符串连接

1、 golang 字符串连接使用 `+` 完成

例子

```go
// string  字符串 设置
// str_trading.go
package main

import "fmt"

func main() {
	fmt.Println("hello" + "world")

}
```

```bash
$ go run str_trading.go 
helloworld
```

### 关键字

Golang 会使用到的有25 个关键字

| break    | default     | func   | interface | select |
| -------- | ----------- | ------ | --------- | ------ |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

除了上述的关键字外, Golang 还有36 个预定义标识符:

| append | bool    | byte    | cap     | close  | complex | complex64 | complex128 | uint16  |
| ------ | ------- | ------- | ------- | ------ | ------- | --------- | ---------- | ------- |
| copy   | false   | float32 | float64 | imag   | int     | int8      | int16      | uint32  |
| int32  | int64   | iota    | len     | make   | new     | nil       | panic      | uint64  |
| print  | println | real    | recover | string | true    | uint      | uint8      | uintptr |

### go 语言的空格

golang 的空格用于标识符, 关键字, 预算符和表达式, 提高代码的可读性

使用方式和python 基本类似

```go
fruit = apples + oranges; 

result := add(2, 3)
```

### 格式化字符串

golang 中使用 fmt.Sprintg 和 fmt.Printf 格式化字符串并赋值给新串

- Sprintf 根据格式化的参数生成格式化的字符串并返回该字符串
- Printf 根据格式化的参数生成格式化字符串并写入标准输出

```bash
// string  字符串 设置
package main

import "fmt"

func main() {
	// %d 表示整型数字, %s 表示字符串
	var stockcode=3
	var enddate="2024-04-07"
	var url="Code=%d&endDate=%s"
	var target_url=fmt.Sprintf(url, stockcode, enddate)
	fmt.Println(target_url)

}

```

运行结果如下

```bash
$ go run str_trading.go
helloworld
Code=3&endDate=2024-04-07
```

使用Printf 例子

```go
// string  字符串 设置
package main

import "fmt"

func main() {
	// %d 表示整型数字, %s 表示字符串
	var stockcode=3
	var enddate="2024-04-07"
	var url="Code=%d&endDate=%s"
	fmt.Printf(url, stockcode, enddate)

}
```

执行例子

```bash
 go run str_trading.go
Code=3&endDate=2024-04-07
```

