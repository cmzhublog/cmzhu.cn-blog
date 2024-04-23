## golang 接口

golang 提供了另外的一种数据类型 -- 接口， 它能够将共性的方法定义在一起， 任何其他类型之遥实现了这些方法， 就是实现了这个接口

接口可以将不同类型绑定到一组公共方法上， 从而实现多态和灵活的设计

golang 中的接口是隐式实现的， 如果一个类型， 实现了一个接口定义的所有的方法， 那么它就自动的实现了接口， 因此， 我们可以通过将接口座位参数来实现不同类型的调用， 从而实现多态

例子

```go
/* 定义接口 */

type interface_name interface {
  method_name1 [return_type]
  method_name2 [return_type]
  method_name3 [return_type]
  method_name4 [return_type]
  ....
}

type struct_name struct {
  
}

/* 实现接口方法 */
func (struct_name_variable struct_name) method_name1() [return_type]{
  /* 方法实现 */
}
...
```

代码例子

```go
package main

import (
	"fmt"
)

type Phone interface {
  call()
}

type NokiaPhone struct {
  
}

func (nokiaPhone NokiaPhone) call() {
  fmt.Println("I am Nokia, I can call you!")
}

type IPhone struct() {
  
} 

func (iPhone IPhone) call() {
  fmt.Println("I am iPhone, I can call you!")
}

func main() {
  var phone Phone
  phone = new(NokiaPhone)
  phone.call()
  
  phone = new(IPhone)
  phone.call()
}
```

