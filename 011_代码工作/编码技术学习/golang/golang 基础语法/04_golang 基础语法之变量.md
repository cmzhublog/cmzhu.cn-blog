## 变量

golang 语言变量是由 字母 下划线 数字组成, 首字母不能为数字

声明方式一般使用var 关键字

```go
var identifier type 
```

如果需要一次声明多个变量, 可以如下

```GO
var identifier1, identifier2 type 
```

例子

```go
// identifier 使用·
// identifier_trading.go

package	main
import "fmt"

func main() {
	var a string = "test"
	fmt.Println(a)

	var b, c int = 1, 2 
	fmt.Println(b, c )

}
```

执行结果如下

```bash
$ go run identifier_trading.go 
test
1 2
```

### 变量声明

1、 指定变量类型, 如果没有初始化, 则默认为零值

```go
var v_name v_type 
v_name = value 
```

零值就是变量没有初始化时赋予的值

- 数字一般为 0 

- bool 一般为false
- 字符串一般为 ""

- 以下几种类型为 nil 

- ```go
  var a *int
  var a []int
  var a map[string] int
  var a chan int
  var a func(string) int
  var a error // error 是接口
  ```

  

例子

```go
package	main
import "fmt"

func main() {
	var i int
	var f float64
	var b bool
	var s string
  
	fmt.Printf("%v %v %v %q \n", i, f, b, s)
}
```

运行结果如下

```bash
$ go run identifier_trading.go
0 0 false ""
```



2、 根据不同的值自行判定变量类型

```go
var v_name = value
```

例子

```go

package	main
import "fmt"

func main() {
	var d = true
  
	fmt.Println(d)
}
```

运行结果

```bash
$ go run identifier_trading.go
true
```

3、 如果变量已经声明过了, 使用`:=` 声明变量, 就回产生编译错误, 格式

```go
v_name := value 
```

例子:

```go
var intVal int
intVal := 1 // 这里回产生编译错误, 因为 intVal 已经声明, 不需要重新声明
```

直接使用下面语句声明并赋值即可

```go
intVal := 1
```

`intVal := 1` 相当于

```go
var intVal int
intVal = 1
```



同时也可以将 var f string = "Runoob" 简写成 f := "Runoob"

```go
package	main
import "fmt"

func main() {
	f := "Runoob"
  
	fmt.Println(f)
}
```

### 多变量声明

```go
// 类型相同的多个变量

var vname1, vname2, vname3 type

vname1, vname2, vname3 = v1, v2, v3
vname1, vname2, vname3 := v1, v2, v3

// 因式分解关键字一般用于声明全局变量
var (
  vname1 v_type1
  vname2 v_type2
)
```

例子

```go
// identifier 使用·

package	main
import "fmt"

//声明变量
var x, y int
var ( // 用于声明全局变量
	a int
	b bool
)

var c, d int = 1, 2

var e, f = 123, "hello"

func main() {
	g, h := 123, "hello"
	fmt.Println(x, y, a, b, c, d, e, f, g, h)
}

```

运行结果

```bash
$ go run identifier_trading.go
0 0 0 false 1 2 123 hello 123 hello
```

### 值类型和引用类型

int, bool, float, string 这些类型都属于值类型, 使用这些类型的变量时直接指向存在内存中的值

![4.4.2_fig4.1](./04_golang%20%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95%E4%B9%8B%E5%8F%98%E9%87%8F.assets/4.4.2_fig4.1.jpeg)

当使用序号 = 将一个变量的值赋予给一个另一个变量时, 如 j = i, 实际上就是在 内存中将i 的值进行了拷贝

![4.4.2_fig4.2](./04_golang%20%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95%E4%B9%8B%E5%8F%98%E9%87%8F.assets/4.4.2_fig4.2.jpeg)

可以直接使用 `&i` 获取变量 i 的内存地址, 

值类型变量的值存储在堆中。

内存地址会根据机器的不同而有所不同，甚至相同的程序在不同的机器上执行后也会有不同的内存地址。因为每台机器可能有不同的存储器布局，并且位置分配也可能不同

更复杂的数据通常会需要使用多个字，这些数据一般使用引用类型保存。

一个引用类型的变量 r1 存储的是 r1 的值所在的内存地址（数字），或内存地址中第一个字所在的位置。

![4.4.2_fig4.3](./04_golang%20%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95%E4%B9%8B%E5%8F%98%E9%87%8F.assets/4.4.2_fig4.3.jpeg)

这个内存地址称之为指针，这个指针实际上也被存在另外的某一个值中。

同一个引用类型的指针指向的多个字可以是在连续的内存地址中（内存布局是连续的），这也是计算效率最高的一种存储形式；也可以将这些字分散存放在内存中，每个字都指示了下一个字所在的内存地址。

当使用赋值语句 r2 = r1 时，只有引用（地址）被复制。

如果 r1 的值被改变了，那么这个值的所有引用都会指向被修改后的内容，在这个例子中，r2 也会受到影响。