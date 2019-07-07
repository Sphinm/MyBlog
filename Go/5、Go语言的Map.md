上一节中我们介绍了切片的相关内容，这一节我们看看另一种数据结构——Map，其实这个和 Java 中的 HashMap 类似，都是基于键值对存储的无序集合，有相关概念学习起来就很简单了。

## Map的定义和初始化
Map 的定义格式是 `map[keyType]valueType`，表示键是一个 `keyType` 类型，值是一个 `valueType` 类型。和切片一样，Map 也是引用类型，支持 make 创建和字面量创建。
```go
package main

import "fmt"

func main() {
    m := make(map[string]int)
    fmt.Println(m)
}
----------
map[]
```
我们创建了一个键类型为 `string` 的，值类型为 `int` 的 `map`，这个时候发现什么都没有，需要手动初始化，我们通过字面量创建试试

```go
package main

import "fmt"

func main() {
    m := map[string]int{
        "xiaoming": 18,
        "xiaohong": 16,
        "tubashu":  10,     // 注意这里的逗号不可以省略，否则会报错
    }
    fmt.Println(m)
}
-------------
map[tubashu:10 xiaohong:16 xiaoming:18]
```
通过以上两种方式我们都可以创建一个 `map`，在 [effective_go](https://go-zh.org/doc/effective_go.html#%E6%98%A0%E5%B0%84) 中讲到，map 的键可以支持多种类型，注意不支持切片类型，因为切片类型不支持相等性判断，什么是相等性判断呢？
这里可以理解为支持 `==` 运算符判断的类型均可以，所以我们的引用类型就不可以了，比如切片、函数以及含有切片的结构体等；对于 map 的值就没有任何限制了。

## 插入或者删除一个元素
插入一个元素很简单，直接使用中括号读取，删除元素使用 `delete` 函数来删除

```go
package main

import "fmt"

func main() {
    m := map[string]int{
        "xiaoming": 18,
        "xiaohong": 16,
        "tubashu":  10,         // 注意这里的逗号不可以省略，否则会报错
    }
    m["gogo"] = 12              // 插入一条语句
    fmt.Println(m)

    delete(m, "xiaoming")       // 删除元素
}
-------------
map[gogo:12 tubashu:10 xiaohong:16 xiaoming:18]
map[gogo:12 tubashu:10 xiaoming:18]
```