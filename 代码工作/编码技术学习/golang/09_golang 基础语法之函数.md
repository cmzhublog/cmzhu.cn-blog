## golang 函数

函数是基本代码块, 用于执行某一个任务

golang 至少需要一个main 函数

可以通过函数划分不同的功能

函数声明需要指定函数的名称, 返回值, 参数

### 函数的定义

```go
func function_name ([parameter list ]) [return type ]{
  函数体
}
```

函数定义解析：

- func：函数由 func 开始声明
- function_name：函数名称，参数列表和返回值类型构成了函数签名。
- parameter list：参数列表，参数就像一个占位符，当函数被调用时，你可以将值传递给参数，这个值被称为实际参数。参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
- return_types：返回类型，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types 不是必须的。
- 函数体：函数定义的代码集合。

例子, 返回两个传入参数的最大值;

```go
package main

import "fmt"

func max(num1, num2 int) int {
	if (num1 >= num2 ) {
		return num1
	} else {
		return num2
	}
}

func main() {
	fmt.Printf("%d 和 %d 两个数的最大值是: %d\n", 10, 20, max(10,20))
}
```

### 函数返回多个值

示例代码, 交换两个参数的位置

```go
package main

import "fmt"

func swap(s1, s2 string) (
	string,
	string) {
		return s2, s1
}

func main() {
	s1 := "hello"
	s2 := "world"
	s_swap_1, s_swap_2 := swap("hello","world")
	fmt.Printf("%s 和 %s 两个字符的交换后的结果是: %s, %s\n", s1, s2, s_swap_1, s_swap_2)
}
```

### 函数参数

函数如果使用参数，该变量可称为函数的形参。

形参就像定义在函数体内的局部变量。

调用函数，可以通过两种方式来传递参数：

| 传递类型                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [值传递](https://www.runoob.com/go/go-function-call-by-value.html) | 值传递是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。 |
| [引用传递](https://www.runoob.com/go/go-function-call-by-reference.html) | 引用传递是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。 |

默认情况下，Go 语言使用的是值传递，即在调用过程中不会影响到实际参数。

引用传递; 直接在函数中改变相应的值

```go
package main

import "fmt"

func swap(s1 *int, s2 *int) {
		var temp int
		temp = *s1
		*s1 = *s2
		*s2 = temp
}

func main() {
	s1 := 1
	s2 := 10
	swap(&s1, &s2)
	fmt.Printf("%d 和 %d \n", s1, s2)
}

```

