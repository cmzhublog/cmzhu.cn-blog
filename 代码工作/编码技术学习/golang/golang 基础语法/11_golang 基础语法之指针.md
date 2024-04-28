## Golang 指针

 go 语言指针可以用来更加简单的执行任务

变量是使用方便的占位符, 用于引用计算机内存地址

golang 语言取地址符是 &, 放到一个变量前就会返回相应变量的地址

示例代码	

```go
package main

import "fmt"

func main() {
	var pointer int = 100

	fmt.Printf("变量 pointer 的内存地址是: %x\n", &pointer)

}
```

运行结果

```bash
$ go run pointer_trading.go
变量 pointer 的内存地址是: 14000126008
```



### 什么是指针

- 指针变量指向一个值的内存地址
- 使用指针时需要声明, 格式如下

```go
var var_name *var_type
```

例子:

```go
var ip *int
var str *string
```

### 如何使用指针

使用流程

- 定义指针变量
- 为指针变量赋值
- 访问指针变量中指向的地址

在指针类型前面加上`*` 前缀 来获取指针所指向的内容

```go
package main

import "fmt"

func main() {
	var pointer int = 100
	var tmp *int

	tmp = &pointer


	fmt.Printf("变量 pointer 的内存地址是: %x\n", &pointer)
	fmt.Printf("变量 *tmp 的存储指针地址是: %x\n", tmp)
	fmt.Printf("变量 *tmp 的指向的值是: %d\n", *tmp)

}
```

运行结果

```bash
$ go run pointer_trading.go
变量 pointer 的内存地址是: 14000126008
变量 *tmp 的存储指针地址是: 14000126008
变量 *tmp 的指向的值是: 100
```

###  golang 空指针



当一个指针被定义后, 没有分配任何变量时, 它的值是nil;

nil 指针也叫空指针

指针变量通常缩写为ptr

```go

package main

import "fmt"

func main() {
	var tmp *int

	fmt.Printf("变量 tmp 的指向的值是: %x\n", tmp)

}
```

输出结果

```bash
$ go run pointer_trading.go
变量 tmp 的指向的值是: 0
```

判断空指针

```go
if(ptr != nil)     /* ptr 不是空指针 */
if(ptr == nil)    /* ptr 是空指针 */
```

### 指针数组

```go
package main

import "fmt"

const MAX int = 3

func main() {
	a := []int{1,20,300}
	var i int

	for  i=0; i< MAX; i++ {
		fmt.Printf("a[%d] = %d\n", i, a[i])
	}
```

运行结果

```bash
$  go run pointer_trading.go
a[0] = 1
a[1] = 20
a[2] = 300
```

当有一种情况时, 我们需要保存数组, 这样就需要用到指针

声明指针类型数组如下

```go
var ptr [MAX]*int
```

Ptr 为整型的指针数组, 因此每一个原属都指向了一个值, 一下示例将三个证书存储在指针数组中



```go
package main

import "fmt"

const MAX int = 3

func main() {
	a := []int{1, 20, 300}
	var i int
	var ptr [MAX]*int
	for i=0; i<MAX; i++ {
		ptr[i] = &a[i]
	}
	for i = 0; i < MAX; i++ {
		fmt.Printf("a[%d] = %d\n", i, *ptr[i])
	}
}
```

运行结果为

```bash
$ go run pointer_trading.go
a[0] = 1
a[1] = 20
a[2] = 300
```

### golang 指向指针的指针(双指针)

指针变量存放的是另一个指针变量的地址, 称这个指针变量为指向指针变量

![img](./11_golang%20%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95%E4%B9%8B%E6%8C%87%E9%92%88.assets/pointer_to_pointer.jpg)

定义方式如下

```go
var ptr **int
```

指向指针变量需要在前面加上2个*

例子: 

```go
package main

import "fmt"

const MAX int = 3

func main() {
	var a int = 100
	var ptr *int
	var pptr **int

	ptr = &a
	pptr = &ptr
  
	fmt.Printf("a 的值是: %d\n", a)
	fmt.Printf("ptr 指向的地址是: %x\n", ptr)
	fmt.Printf("pptr 指向的地址是: %x\n", pptr)
	fmt.Printf("ptr 指向的值是: %d\n", *ptr)
	fmt.Printf("pptr 指向的值是: %d\n", **pptr)

}
```

### go 指针作为函数参数

go 语言允许向函数传递指针, 只需要在函数定义上设置成指针即可

```go
package main

import "fmt"

const MAX int = 3

func swap (x *int, y *int) {
	var tmp int
	tmp = *x
	*x = *y
	*y = tmp
}

func main() {

	var a, b int = 10, 20 

	swap(&a, &b)

	fmt.Printf("a 的值是: %d\n", a)
	fmt.Printf("b 的值是: %d\n", b)

}
```

