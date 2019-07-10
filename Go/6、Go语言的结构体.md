Go 语言中的结构体是一种类似于 Java 中类的概念，我们可以在结构体里面声明一些属性和字段。结构体在 Go 中有着非常重要的位置，它可以是基础类型、切片、map、结构体等类型的数据构成的集合。由于结构体是 **值类型**，所以可以使用 new 函数来创建。

![来源-码洞](https://mmbiz.qpic.cn/mmbiz_png/bGribGtYC3mIF98WxZkyOgbtbkiblaQgy6v1DrribArRBAoETvdCiaKVtO0ovyxJfaNDqTRPSJmT1bP0KfgGvgTiciaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


## 结构体的定义与使用

我们声明一个 `Person`
```go
type Person struct {
    name string
    age  int
}
--------------
这里我们通过 `type` XXX `struct` 声明一个结构体
结构体内定义两个字段，分别是名称和类型
```

上面我们定义了一个结构体类型，但是并不能直接用，需要我们给结构体初始化
```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    per := Person{
        name: "xiaoming",
        age:  12,			// 逗号不可以省略
    }
    // 还可以通过 per.name 形式赋值
    // per := Person{}
    // per.name = "xiaoming"
    // per.age = 12
    fmt.Println(per)
}
-------------
{xiaoming 12}
```
这里我们通过字面量的形式给结构体进行了初始化，这里如果不指定初始值，会得到默认类型的零值。

## 结构体在函数间传递
结构体和数组类似，是值传递，我们把一个数组或者结构体作为参数传给函数时，函数会复制一个副本，为了提高性能，我们一般不会把数组直接传递给函数，而是传递数组的引用——切片。我们先看直接传递一个结构体会怎么样，然后在对比一下传递结构体指针又会怎么样

```go
package main

import "fmt"

type person struct {
    name string
    age  int
}

func main() {
    per := person{
        name: "ming",
        age:  20,
    }
    fmt.Println("main ", per)
    A(per)
    fmt.Println("main after", per)
}

func A(per person) {
    per.age = 18
    fmt.Println("A ", per)
}
-------------
main  {ming 20}
A  {ming 18}
main after {ming 20}
```

通过打印输出我们发现传递一个结构体只是复制了一个结构体副本，对传入的结构体修改并不会影响原有的结构体，接着我们传入一个指针结构体试试

```go
package main

import "fmt"

type person struct {
    name string
    age  int
}

func main() {
    per := person{
        name: "ming",
        age:  20,
    }
    fmt.Println("main ", per)
    B(&per)
    fmt.Println("main after", per)
}

func B(per *person) {
    per.age = 16
    fmt.Println("B ", per)
}
-------------
main  {ming 20}
B  &{ming 16}
main after {ming 16}
```
我们看到在 B 函数中传入了一个结构体指针，函数对结构体所做的修改同样会影响到指针所指向的结构体。这正说明了结构体传参是 **值传递**，复制了一个副本，而指针结构体会影响到原有结构体，注意这两种的区别，选择哪一种看我们自身需求，需要改变原有结构体就需要传递指针，需要一个新的内容返回并且不影响原有结构体我们则传递结构体副本即可。