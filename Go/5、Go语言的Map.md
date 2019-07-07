上一节中我们介绍了切片的相关内容，这一节我们看看另一种数据结构——Map，其实这个和 Java 中的 HashMap 类似，都是基于键值对存储的无序集合，有相关概念学习起来就很简单了。

## Map的定义和初始化
Map 的定义格式是 `map[keyType]valueType`，表示键是一个 `keyType` 类型，值是一个 `valueType` 类型。和切片一样，Map 也是引用类型，支持 make 创建和字面量创建。
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
插入一个元素很简单，直接使用中括号读取，删除元素使用 `delete` 函数来删除

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
这里很简单，继续往下考虑，如果我们插入一条同键名的记录呢？或者删除一条不存在的语句又会怎么样？接着往下看

```go
package main

import "fmt"

func main() {
    m := map[string]int{
        "xiaoming": 18,
        "xiaohong": 16,
        "tubashu":  10,
    }
    m["tubashu"] = 20       // 追加一条同键名记录会覆盖原有的
    fmt.Println(m)
}
----------
map[tubashu:12 xiaohong:16 xiaoming:18]
```
通过代码我们发现追加一条同键名记录会覆盖原有的记录，而读取一条不存在的记录什么反应都没有？！
这个时候我们可以通过标识位 `ok` 来确定

```go
package main

import "fmt"

func main() {
    m := map[string]int{
        "xiaoming": 18,
        "xiaohong": 16,
        "tubashu":  10,
    }
    test, ok := m["test"]
    fmt.Println(test, ok)

    m["test"] = 100
    test, ok = m["test"]
    fmt.Println(test, ok)
}
----------
0 false
100 true
```
第一次读取键值 `test` 时候 map 中并没有，所以返回对应的零值，第二个参数为 `false`，
当我们插入了 `test` 时，就可以正确返回数据了。

## Map的遍历
还是以刚刚的例子，通过 `for range` 来遍历一下
```go
package main

import "fmt"

func main() {
    m := map[string]int{
        "xiaoming": 18,
        "xiaohong": 16,
        "tubashu":  10,
    }
    for key, value := range m {
        fmt.Println(key, value)
    }
}
-----------
xiaoming 18
xiaohong 16
tubashu 10
```


## 本章小结
Go 语言中 map 是一种常见的数据结构，我们通过键值可以快速的索引到某个值；上面我们介绍了 map 的定义和初始化以及添加删除记录的操作，其实忘了说 `无序` 了，这个大家可以自己尝试一下，每次遍历一个 map 输出得到的结果都不一定相同的哦~


## 导航
+ 上一节：[Go语言的切片](./4、Go语言的切片.md)
+ 下一节：[Go语言的结构体](./6、Go语言的结构体.md)