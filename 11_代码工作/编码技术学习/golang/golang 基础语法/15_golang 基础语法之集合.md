## golang 语言 map 

- Map 是一种无序的键值对的集合
- Map 最重要的是通过key 来快速检索到值, key 类似于索引, 指向数据的值
- Map 也是一种集合, 因此可以像迭代数组那样去迭代他. 不过, map 是无序的, 遍历map 时返回的键值对的顺序不是确定的
- 在获取键值时, 如果键不存在, 则返回该类型的零值, int 返回0, string 返回“”.
- Map 属于引用类型, 如果将map 传递给宁一个函数或者另一个变量, 他们是指向同一个数据结构, 因此map 的修改会影响到所有引用他的变量



### 定义一个map

可以使用make 函数或者使用map 关键字来定义Map

```go
map_variable := make(map[KeyTyope]ValueType, initialCapacity)
```



- keyTpye 是键的类型
- ValueType 是值的类型
- initialCapacity 是Map 的容量, 不指定go 会根据实际情况选择一个合适的值

```go
map1 := make(map[int]int, 10)

map2 := make(map[int]string)
```



也可以根据字面量创建map

```go
map2 := map[string]int {
  "cmzhu": 30,
  "htj": 20
}
```

如何获取元素呢?

```go
v1 := map2["cmzhu"]
v2, ok := map2["test"] // 如果ok 的值为false , 则表示该key 不存在
```

修改元素

```go
map2["cmzhu"] = 20
```

获取map 的长度

```go
len := len(map2)
```

遍历map 

```go

for key, value := range map2 {
  fmt.Printf("map2[%d] = %d", key, value)
}
```

删除元素

```go
delete(map2, "cmzhu")
```

例子 - 基于go 实现简单的hashMap, 未做key 值的校验

```go
package main

import (
	"fmt"
)

type HashMap struct {
	key string
	value string
	hashCode int
	next *HashMap
}

var table [16](*HashMap)


func initTable() {
	for i := range table {
		table[i] = &HashMap{"","",i,nil}
	}
}

func getInstance() [16](*HashMap) {
	if table[0] == nil {
		initTable()
	}
	return table
}

func genHashCode(k string) int {
	if len(k) == 0 {
		return 0
	}
	var hashCode int = 0 
	var lastIndex int = len(k) - 1

	for i := range k {
		if i == lastIndex {
			hashCode += int(k[i])
			break
		}
		hashCode += (hashCode + int(k[i])) * 31
	}

	return hashCode
}

func indexTable(hashCode int) int {
	return hashCode % 16 
}

func indexNode(hashCode int) int {
	return hashCode >> 4
}

func put(k string, v string) string {
	var hashCode = genHashCode(k)
	var thisNode = HashMap{k, v, hashCode, nil}

	var tableIndex = indexTable(hashCode)
	var nodeIndex  = indexNode(hashCode)

	var headPtr [16](*HashMap) = getInstance()
	var headNode = headPtr[tableIndex]

	if (*headNode).key == "" {
		*headNode = thisNode
		return ""
	}

	var lastNode *HashMap = headNode
	var nextNode *HashMap = (*headNode).next

	for nextNode != nil && (indexNode((*nextNode).hashCode) < nodeIndex) {
		lastNode = nextNode
		nextNode = (*nextNode).next
	}

	if (*lastNode).hashCode == thisNode.hashCode {
		var oldValue string = lastNode.value
		return oldValue
	}

	if lastNode.hashCode < thisNode.hashCode {
		thisNode.next = &thisNode
	}

	if nextNode != nil {
		thisNode.next = nextNode
	}
	return ""

}

func get(k string) string {
	var hashCode = genHashCode(k)
	var tableIndex = indexTable(hashCode)

	var headPtr [16](*HashMap) = getInstance()
	var node *HashMap = headPtr[tableIndex]

	if (*node).key == k {
		return (*node).value
	}

	for (*node).next != nil {
		if k == (*node).key {
			return (*node).value
		}
		node = (*node).next
	}
	return ""
}

func main() {
	getInstance()
	put("a", "a_put")
	put("b", "b_put")
	fmt.Println(get("a"))
	fmt.Println(get("b"))
	put("p","p_put")
	fmt.Println(get("p"))
}

```

运行结果

```bash
$ go run  newhashMap_trading.go
a_put
b_put
p_put
```

