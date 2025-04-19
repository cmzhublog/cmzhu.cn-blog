## 常量

常量是一个简单值的标识符, 在程序运行时, 不会被修改的值

常量可以为 布尔型、 数字型、字符串型

常量的定义格式

```go
const identifier [type] = value 
```

可以省略类型, 因为编译器可以通过变量的值来推断其类型

- 显示类型定义 : const b string = "abc"
- 隐式类型定义: const b = "abc"

多个相同类型的定义可以简写为

```go
const a, b string = "abc", "bcd"
```

演示常量的应用

例子

```go
// const trading 

package main

import "fmt"

func main() {
	const LENGTH int = 10 
	const WIDTH int = 20 
	var area int
	const a, b, c = 1, 2, 3

	area = LENGTH * WIDTH
	fmt.Printf("面积为: %d \n", area)
	fmt.Println()
	fmt.Println(a, b, c)
}

```

运行结果

```bash
$ go run const_trading.go
面积为: 200 

1 2 3
```

常量还可以用作枚举

```go
const (
  right = 1 
  wrong = 0
  middle = 2
)
```

常量可以使用 len(), cap(), unsafe.Sizeof() 函数来计算表达式的值. 常量表达式中, 函数必须是内置函数, 否则编译不过



例子

```go
// const trading 

package main

import "unsafe"

const (
	a = "abc"
	b = len(a)
	c = unsafe.Sizeof(a)
)

func main() {
	println(a, b, c)
}

```

运行结果

```bash
$ go run const_trading.go
abc 3 16
```

### iota

- iota : 特殊常量, 可以认为是可以被编译器修改的的常量
- ioat 在const 关键字出现时被重置成0 (const 内部的第一行之前,) const 中每增加一行常量声明, 都会使得iota 计数一次(iota 可以理解为const 语句中的行索引)

```go
const (
	a = iota
  b = iota
  c = iota
)
```

第一个iota 等于0 , 每当iota 在新的一行被使用时, 他的值都会自动加1; 所以a = 0, b=1, c=2; 简写如下

```go
const (
	a = iota
  b
  c
)
```

例子

```go
// const trading 

package main

import "fmt"

func main() {
	const (
		a = iota
		b
		c
		d = "ha"
		e
		f = 100
		g
		h = iota
		i
	)
	fmt.Println(a,b,c,d,e,f,g,h,i)

}

```

运行结果为

```bash
$ go run const_trading.go
0 1 2 ha ha 100 100 7 8
```

