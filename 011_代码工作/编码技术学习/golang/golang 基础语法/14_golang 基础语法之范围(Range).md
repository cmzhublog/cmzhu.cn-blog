## golang 范围(Range)

go 语言中, range 关键字用于迭代for 循环中的迭代数组, 切片, 通道, 集合等元素, 在数组和切片中, 返回元素的缩影和索引对应的值, 返回状态为 key-value 的状态

```go
for key, value := range oldMap {
  newMap[key] = value
}
```

如果只想要key 可以按照一下语法

```go
for key := range oldMap {
 ... 
}
```

也可以写成

```go
for key, _ := range oldMap {
  ...
}
```

如果只需要value , 需要写成

```go
for _,value := range oldMap {
  ...
}
```

例子

```go
package main

import "fmt"

func main() {
	var pow = []int{1,2,4,8,16,32,64,128}

	for key, value := range pow {
		fmt.Printf("2**%d = %d\n", key, value)
	}
}
```

输出结果:

```bash
$ go run  range_trading.go
2**0 = 1
2**1 = 2
2**2 = 4
2**3 = 8
2**4 = 16
2**5 = 32
2**6 = 64
2**7 = 128
```

