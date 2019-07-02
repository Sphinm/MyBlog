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

第二种方式是通过直接赋值的方式创建一个满容切片，可以通过内置函数 `len()` 获取切片长度，`cap()` 获取底层数组容量
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

它们共享底层数组

![切片原理.png](https://mmbiz.qpic.cn/mmbiz_png/bGribGtYC3mJtc8O34mFE3jzWZ8xtVTeyathYpKrFkPEacEhouJ9xhNqMbsWH5uWOyEZlC2D4n8SIpMXwuq8FwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图中切片变量包含 **三个域**，分别是 ``指向底层数组的指针``，``切片长度``， ``切片的容量`` 三个概念。

在 Go 语言中切片也叫 slice，它是一个引用类型，指向一个底层数组。学过 Java 的同学可以把它看作 ArrayList，ArrayList 的内部实现也是一个数组。当数组容量不够需要扩容时，就会换新的数组，还需要将老数组的内容拷贝到新数组。ArrayList 内部有两个非常重要的属性 capacity 和 length。capacity 表示内部数组的总长度，length 表示当前已经使用的数组的长度。length 永远不能超过 capacity。