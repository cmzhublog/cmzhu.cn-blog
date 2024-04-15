## 切片(Slice)

go 切片是对数组的抽象, go 数组长度是不能改变的, 在特定场景中的, 这种集合就不太合适, go 提供了一种林火, 功能强悍的内置类型切片技术(动态数组), 和数组相比, 切片的长度是不固定的, 可以追加元素, 在追加时可能使切片容量增大



### 定义切片

1、 可以声明一个未指定大小的数组来声明切片

```go
var identifier []type
```

切片不需要说明长度

可以使用make 来创建切片

```go
var slice1 []type = make([]type, len)
简写
slice1 := make([]type, len)
```



也可以指定容量, 其中capacity 为可选参数

```go
make([]T, length, capacity)
```

len 是数组的长度, 也是切片的初始长度

### 初始化切片

直接初始化切片,  [] 表示切片类型, {1,2,3} 初始化的值依次是 1,2,3 其中 cap=len=3

```go
s := [] int {1, 2, 3}
```

初始化切片s, 是数组arr 的引用

```go
s = arr[:]
```

将arr 从下标startindex 到 endindex -1 的元素, 创建成一个新的切片

```go
s := arr[startindex: endindex]
```

从下表为startindex一直到最后一个元素

```go
s := arr[startindex:]
```

通过切片s 初始化切片s1

```go
s1 := s[startindex:endindex]
```

通过内置的make() 初始化切片s, []int 表示元素类型为 int 的切片

```go
s := make([]int, len, cap)
```

### Len()和 cap()

切片是可索引的, 并且可以由len()方法获取长度

切片提供了计算容量的cap() 方法,可以测量切片最长可以达到多少

例子

```go
package main

import "fmt"

func main() {
   var numbers []int

   printSlice(numbers)

   if(numbers == nil){
      fmt.Printf("切片是空的")
   }
}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```

 
