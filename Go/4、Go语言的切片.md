切片在 Go 语言中是一个非常重要的数据结构，很多场景里数组并不能满足我们的需求，比如我们开始不知道需要多大的数组的时候，我们就需要动态的数组，也就是我们今天讲的切片。

## 切片创建
首先看内置函数 make 如何创建，它需要提供三个参数：第一个是切片类型、切片长度和容量（底层数组的长度），其中第三个参数是可选的，如果不提供默认长度和容量是相等的，也就是创建一个满容切片。
```go
// make 
 var s1 []int = make([]int, 5, 8)
 var s2 []int = make([]int, 8) // 满容切片
 fmt.Println(s1)
 fmt.Println(s2)

-------------
[0 0 0 0 0]
[0 0 0 0 0 0 0 0]
```

使用 make 函数创建的切片内容是「零值切片」，也就是内部数组的元素都是零值，第二种方式是通过直接赋值的方式创建一个满容切片，可以通过内置函数 `len()` 获取切片长度(切片当前引用的数组片段的长度)，`cap()` 获取切片容量(当前切片的起始位置到当前切片所引用的数组末尾的长度)。
```go
var arr []int = []int{1, 2, 3, 4, 5} 
fmt.Println(arr, len(arr), cap(arr)) 
-------------
[1 2 3 4 5] 5 5
```

第三种方式可以通过已存在的数组或者切片再次声明一个新的切片，使用方式是 `array[i, j]`，这里注意是闭开区间，包含 `i` 但是不包含 `j`。

```go
// 首先声明一个长度为 10 的数组
var arr = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
// arr1 从底层第三个数据开始，到第五个结束
var arr1 = arr[2:5]      // [3 4 5] 
var arr2 = arr[4:8]      // [5 6 7 8]

// 我们从 arr2 获取 slice
slice1 := arr2[1:3]       // [6 7]
```
这里需要注意每一个切片的长度和容量变化，我们容易理解切片长度是怎么回事，那切片容量是怎么回事呢，其实它 ``指的是当前切片在数组的起始点到数组末尾的长度``，我们通过代码和图片来看一下
```go
fmt.Println("arr1", arr1, len(arr1), cap(arr1)) 
fmt.Println("arr2", arr2, len(arr2), cap(arr2)) 
fmt.Println("slice1", slice1, len(slice1), cap(slice1)) 

-----------
arr1 [3 4 5] 3 8        // 容量 8 是 3 到 10 的区间
arr2 [5 6 7 8] 4 6      // 容量 6 是 5 到 10 的区间
slice1 [6 7] 2 5        // 容量 5 是 6 到 10 的区间
```

此处它们共享底层数组，值得注意的是，切片支持 append 操作可以将新的内容追加到底层数组，也就是填充下面的灰色格子。如果格子满了，切片就需要扩容，底层的数组就会更换，这个我们下面再说。

![切片原理.png](https://mmbiz.qpic.cn/mmbiz_png/bGribGtYC3mJtc8O34mFE3jzWZ8xtVTeyathYpKrFkPEacEhouJ9xhNqMbsWH5uWOyEZlC2D4n8SIpMXwuq8FwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图中切片变量包含 **三个域**，分别是 ``指向底层数组的指针``，``切片长度``， ``切片的容量`` 三个概念。

> 在 Go 语言中切片也叫 slice，它是一个引用类型，指向一个底层数组。学过 Java 的同学可以把它看作 ArrayList，ArrayList 的内部实现也是一个数组。当数组容量不够需要扩容时，就会换新的数组，还需要将老数组的内容拷贝到新数组。ArrayList 内部有两个非常重要的属性 capacity 和 length。capacity 表示内部数组的总长度，length 表示当前已经使用的数组的长度。length 永远不能超过 capacity。

## 空切片和 nil切片
在切片这一节中，有两种特殊情况，空切片和 nil 切片，不同于 make 创建的零值切片，我们通过代码看一下
```go
func main() {
    var s1 []int                   // nil 切片
    var s2 []int = []int{}         // 空切片
    var s3 []int = make([]int, 0)  // 空切片
    var s4 []int = make([]int, 3)  // 零值切片
    fmt.Println(s1, s2, s3, s4)
    fmt.Println(len(s1), len(s2), len(s3), len(s4))
    fmt.Println(cap(s1), cap(s2), cap(s3), cap(s4))
}

-----------
[] [] [] [0 0 0]
0 0 0 3
0 0 0 3
```


## 切片赋值
切片的赋值是浅拷贝，拷贝的是变量的三个域，拷贝前后两个变量共享同一个底层数组，改变其中一个切片会影响另外一个切片内容。
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5, 6}
	s2 := s1
	
	s2[0] = 9
	fmt.Println(s1)
	fmt.Println(s2)
}
--------------
[9 2 3 4 5 6]
[9 2 3 4 5 6]
```

## 切片的遍历
同数组一样，切片支持 range 关键字遍历
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5, 6}
	for index,value := range s1 {
		fmt.Println(index, value)
    }
    // for i := 0; i < len(s1); i++ {
	// 	fmt.Printf("%d\n", s1[i])
	// }
}
-------------
0 1
1 2
2 3
3 4
4 5
5 6
```

## 切片追加append
前面我们提到切片可以通过追加操作改变切片长度生成一个新的切片变量，这就是我们要讲的 append，但是追加操作有一个问题就是有可能影响底层数组的长度，当追加的内容超出了底层数组，会导致底层数组扩容。
```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5}

    // 对满容的切片追加会分离底层数组
    // 这里注意的是先进行追加后判断是否满容
	s2 := append(s1, 6)
	fmt.Println("满容切片 ", s1, len(s1), cap(s1))
	fmt.Println("发生扩容 ", s2, len(s2), cap(s2))

	// 对非满容的切片进行追加会共享底层数组
	s3 := append(s2, 7)
	fmt.Println("s2 ", s2, len(s2), cap(s2))
	fmt.Println("s3 ", s3, len(s3), cap(s3))
}
-------------
满容切片  [1 2 3 4 5] 5 5
发生扩容  [1 2 3 4 5 6] 6 12
s2  [1 2 3 4 5 6] 6 12
s3  [1 2 3 4 5 6 7] 7 12
```

我们发现对满容切片进行追加底层数组进行扩容，而且长度也加倍了，追加生成的新切片指向新的底层数组，旧的切片还是指向为扩容的数组；还有一点注意的是新切片需要使用，否则会报错。

append 还支持同时追加多个值甚至追加整个切片

```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3, 4, 5}

	s2 := append(s1, 7, 8, 9, 10)
	fmt.Println("追加多个值: ", s2, len(s2), cap(s2))

	temp := []int{20, 21, 22, 23}

	s3 := append(s2, temp...)
	fmt.Println("追加整个切片 ", s3, len(s3), cap(s3))
}
-----------------
追加多个值:  [1 2 3 4 5 7 8 9 10] 9 12
追加整个切片  [1 2 3 4 5 7 8 9 10 20 21 22 23] 13 24
```

## 切片拷贝copy
在 Go 语言中内置了一个 copy 函数来进行深拷贝切片，copy 函数不会因为原切片和目标切片的长度问题而额外分配底层数组的内存，它只负责拷贝数组的内容，从原切片拷贝到目标切片，拷贝的量是原切片和目标切片长度的较小值 ———— min(len(src), len(dst))，函数返回的是拷贝的实际长度。我们来看一个例子

```go
package main

import "fmt"

func main() {
	var s = []int{1, 2, 3, 4, 5}

	var d = make([]int, 3, 6)
	var n = copy(d, s)      // n 表示返回拷贝的长度
	fmt.Println(n, d)
}
-------------
3 [1 2 3]
```

## 底层数组扩容算法
和 Java 的 ArrayList 一样，append 函数也支持动态扩容。当切片长度低于 **1024** 的时候，系统会分配给底层数组当前切片（满容）长度的 2 倍的空间；当切片长度大于等于 **1024** 的时候，系统只分配 1.25 倍的空间。

## 函数中切片传参
我们知道切片是3个字段构成的结构类型，所以在函数间以值的方式传递的时候，占用的内存非常小。在传递复制切片的时候，其底层数组不会被复制，也不会受影响，``复制只是复制的切片本身，不涉及底层数组。``

```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3, 4, 5}
	fmt.Printf("%p\n", &slice)
	modify(slice)
	fmt.Println(slice)
}

func modify(slice []int) {
    fmt.Printf("%p\n", &slice)
    slice[1] = 10
}
-----------------
0x40a0e0
0x40a0f0
[1 10 3 4 5]
```
我们发现传递到函数的切片地址不一样，说明切片在函数直接传递是复制了一个切片副本；只是当我们改变其中一个值得时候，才会改变原切片，说明共用一个底层数组。


## 本章小结
切片的内容是 Go 语言基础中很重要的一个内容，总算是啃完了，有的东西没讲，比如二维切片，在 Go 文档中有一处切片练习就是用到的二维切片，大家可以尝试一下画出有意思的图像出来。下一节开始 Go 语音的 Map，大概可以理解为 HashMap

PS：[切片练习](https://tour.go-zh.org/moretypes/18)

![image.png](https://upload-images.jianshu.io/upload_images/1394028-3213f262d0d0fa9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 导航
+ 上一节：[Go语言的数组](./3、Go语言的数组.md)
+ 下一节：[Go语言的切片](./5、Go语言的Map.md)