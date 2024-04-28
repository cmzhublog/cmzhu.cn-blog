## golang 循环语句



实际问题中的有规律的重复操作, 在程序中使用循环语句来实现, 流程图如下:

![img](./08_golang%20%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95%E4%B9%8B%E5%BE%AA%E7%8E%AF%E8%AF%AD%E5%8F%A5.assets/go-loops.svg)

golang 支持以下几种循环语句

| 循环类型                                                   | 描述                                 |
| :--------------------------------------------------------- | :----------------------------------- |
| [for 循环](https://www.runoob.com/go/go-for-loop.html)     | 重复执行语句块                       |
| [循环嵌套](https://www.runoob.com/go/go-nested-loops.html) | 在 for 循环中嵌套一个或多个 for 循环 |

### 循环控制语句

| 控制语句                                                     | 描述                                             |
| :----------------------------------------------------------- | :----------------------------------------------- |
| [break 语句](https://www.runoob.com/go/go-break-statement.html) | 经常用于中断当前 for 循环或跳出 switch 语句      |
| [continue 语句](https://www.runoob.com/go/go-continue-statement.html) | 跳过当前循环的剩余语句，然后继续进行下一轮循环。 |
| [goto 语句](https://www.runoob.com/go/go-goto-statement.html) | 将控制转移到被标记的语句。                       |

### 死循环

循环永不结束, 没有结束条件, 叫死循环

示例代码



```go
package main

import "fmt"

func main() {
    for true  {
        fmt.Printf("这是无限循环。\n");
    }
}
```



例子 goto

```go
package main

import "fmt"

func main() {
	var index int = 0

	LOOP : for  index <= 20  {
		if (index == 15 || index == 5) {
			index = index + 1
			goto LOOP
		} 
		fmt.Printf("index's value is : %d \n", index)
		index ++
	}
}

```

执行结果

```bash
$ go run condition_trading.go
index's value is : 0 
index's value is : 1 
index's value is : 2 
index's value is : 3 
index's value is : 4 
index's value is : 6 
index's value is : 7 
index's value is : 8 
index's value is : 9 
index's value is : 10 
index's value is : 11 
index's value is : 12 
index's value is : 13 
index's value is : 14 
index's value is : 16 
index's value is : 17 
index's value is : 18 
index's value is : 19 
index's value is : 20
```

for 循环例子

```go
package main

import "fmt"

func main() {
	for i:=0; i<100; i++ {
		fmt.Printf("i的值是 %d \n", i)
	}
}
```

### For-each range 循环

```go
package main

import "fmt"

func main() {
	strings := []string{"google", "runoob"}

	for i, s := range strings {
		fmt.Println(i, s)
	}

}
```

