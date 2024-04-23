## golang类型转换

类型转换是将一种类型转换成另一种类型的变量

基本格式如下

```go
type_name(expression)
```

- type_name: 类型
- expression : 表达式

数值类型转换

1、整数转换成符点数

```go
var a int = 10
var b float64 = float64(a)
```

 2、字符串类型转换

```go
var str string = 10

var num int

num, _ = strconv.Atoi(str)
```

将字符串转换成整数

strconv.Atoi 有两个返回值 一个是返回转换后的整形值, 第二个是可能发生的错误, 我们使用_来忽略错误

3、整数转换成字符串

```go
str := strconv.Itoc(10)
```

4、 符点数转换成字符串

```go
str := strconv.FormatFloat(3.14, 'f', 2, 64)
```

5、 接口类型转换

接口类型转换有两种: 类型断言 和类型转换

类型断言用于将接口类型转换成指定类型, 其语法为: 

```go
value.(type)
 或者
value.(T)
```

其中 value 是接口类型的变量, type 和 T 是需要转换成的类型

如果类型断言成功, 他将返回一个转换后的值和一个bool 值, 表示转换成功

```go
package main

import "fmt"

func main() {
	var i interface{} = "Hello, World"
	str, ok := i.(string)
	if ok {
		fmt.Printf("'%s' is a string \n", str)
	} else {
		fmt.Println("conversion failed")
	}
}
```

将一个接口类型转换成另一个接口类型, 语法为

```GO
T(value)
```

- T : 接口类型
- value : 需要转换的值

> 在类型转换中, 我们必须保证要转换的值和我目标接口类型之间是兼容的, 否则编译器会报错.

 