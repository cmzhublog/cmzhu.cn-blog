## golang 递归函数

递归, 就是在运行过程中调用自己

格式

```go
func recursion() {
  recursion()
}

func main() {
  recursion()
}
```



golang 支持递归, 在使用递归时, 开发者需要设置递归退出条件, 否则会陷入无限循环中

递归常用来解决数学问题, 比如计算阶乘, 生成斐波那契数列



阶乘

```go
package main

import (
	"fmt"
)

func Factorial( n uint64) (result uint64) {
	if n > 0 {
		result = n * Factorial(n-1)
		return result
	}
	return 1
}

func main() {
	var i int = 10
	fmt.Printf("%d 的阶乘是%d \n", i, Factorial(uint64(i)))
}

```



