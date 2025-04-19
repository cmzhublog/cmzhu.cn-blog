## golang 结构体



golang 中数组可以存储同一类数据, 结构体中可以针对不同的定义不同的数据类型; 例如

PeoPle类型:

	- Age: 年龄
	- Name : 名字

###  定义结构体

### 定义结构体

需要使用type 和 struct 语句来定义一个新的数据类型, 结构体中需要包含一个或者多个成员名称, 具体格式如下

```go
type struct_variable_type struct {
  member definition
  member definition
  ...
  member definition
  
}
```

 一旦定义了结构体, 就能用于变量声明

```go
variable_name := struct_variable_type {value1, ..., valuen}

或者
variable_name := struct_variable_type {key1:value1, ..., keyn:valuen}
```



例子:

```go
package main

import "fmt"

type People struct {
	Name string
	Age int
}

func main() {

	var cmzhu  = People { "cmzhu",  24 }

	fmt.Printf("名字: %s, 年龄: %d\n", cmzhu.Name, cmzhu.Age)
}

```

运行结果

```bash
$ go run struct_trading.go
名字: cmzhu, 年龄: 24
```

