![可爱的土拨鼠.png](https://upload-images.jianshu.io/upload_images/1394028-95d6eaf99111984f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

## 前言
在我们接触一门新的语言的时候，并不着急马上去学习它的语法，应该从一个全局的视角来看看我们接下来要学的语言时一个什么样的语言，值不值得我们去学，它和我们现有的技术栈有什么区别，是否能够解决技术痛点，以上这几点是我觉得学习之前先要了解的。

Go 语言目前还是比较新的语言，它是一种自带并发、具有垃圾回收以及快速编译的静态类型语言，它结合了解释型语言的游刃有余，动态类型语言的开发效率，以及静态类型的安全性。同 Java 相比，Go 语言有很多不同的地方，比如一开始的公有变量和私有变量的规范，Go 语言中变量大写就是公有变量，小写就是私有变量（error除外），Go 内置了线程和队列，实现起来比 Java 简单很多，再比如 Go 的异常处理没有一大推 try/catch，等等就不一列举。

Go 语言是 Google 团队的大佬精心打造出来的，成员之一就有大名鼎鼎的 Unix 操作系统的创造者 Ken Thompson，C 语言就是他和已经过世的李奇一起发明的。接下来让我们一起走进 Go 语言的世界吧！

## 1、环境配置
在我们学习一门新的语言时，通常我们会先写一个 Hello World，Go 语言的 Hello World 如下：
```go
package main

import "fmt"

func main() {
  fmt.Println("hello world!")
}
```

看起来是不是很简单？我们使用命令 
```go
go run main.go
```
就会得到我们想要的 hello world! 这个时候我已经迫不及待的想要自己试试了，首先我们需要按照 Go 语言的安装包然后再配置一下 GOPATH 环境变量，其实这里的 GOPATH 环境变量也就是你的 workspace。

### 1.1 Go 语言安装
我们根据自己的机器操作系统选择相应的开发工具包，比如你的是Windows 64位的，就选择 windows-amd64 的工具包；是Linux 32位的就选择 linux-386 的工具包。可以自己查看下自己的操作系统，然后选择，我这边选择 Mac 版本的下载就可以了。

> Mac 安装 Go

Mac 下安装可以直接用 homebrew 或者 [官网下载](https://golang.org/dl/)
```go
brew install go
```
Mac 会默认给你配置好 GOPATH，如果不喜欢可以自己修改，我的目录是 `` /Users/sumin/workspace/dev/golang ``，接下来我们配置环境变量，在终端输入 `` vim .bash_profile ``，然后将一下添加进去后保存退出，并执行 ``source .bash_profile``
```go
export GOPATH="/Users/sumin/workspace/dev/golang"
export GOBIN="${GOPATH}/bin"
export PATH="${PATH}:${GOPATH}:${GOPATH}/bin"
```
 然后终端输入 
```go
go version
```
输出对应的版本号表示安装成功
```go
go version go1.11.1 darwin/amd64
```

以上是针对 Mac 的环境安装教程，Linux 和 Mac 都是基于 Unix ，命令这一块基本差不多，不过 Linux 更 geek 一些，更偏向于命令行操作，比如下载下来的压缩包需要手动解压，Linux 中环境变量的配置和 Mac 稍有不同，Linux 下又两个文件可以配置，其中 /etc/profile 是针对所有用户都有效的；$HOME/.profile 是针对当前用户有效的，可以根据自己的情况选择。

> Windows 安装 Go

首先还是从官网下载相应系统的包，解压到自己选的目录下，我的是 `` F:\Go\ ``， 然后开始配置环境变量，如图所示

![image.png](https://upload-images.jianshu.io/upload_images/1394028-27ef30a046fd2c71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在系统变量中新建 GOROOT，变量值是我们刚刚安装的路径；
修改PATH系统变量，在变量值最后添加 ``%GOROOT%\bin`` 路径;
最后配置工作目录 GOPATH，这是我们存放源码的地方，如图是我选择的路径，这个随个人喜好。

 同样终端输入 
```go
go version
```

输出对应的版本号表示安装成功
```go
go version go1.11.2 windows/amd64
```

接下来设置设置我们 GOPATH 下的 src 文件夹下新建如下文件夹
```
├── bin
├── pkg
└── src
```
+ bin 文件夹存放 go install 生成的可执行文件，`当我们把 GOPATH/bin 加入到 PATH 后，我们可以再终端的任意位置直接执行 go 文件了`
+ pkg 文件夹是存在go编译生成的文件
+ src存放的是我们的go源代码，不同工程项目的代码以包名区分


当我们在终端输入 `` go env `` 查看编译环境:

![image.png](https://upload-images.jianshu.io/upload_images/1394028-5cdf1edea3a6259f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中有几个重要的参数需要我们注意：

+ GOOS ：目标操作系统，当前我的系统是 amd64 位，这个值根据你的操作系统决定
+ GOBIN : 存放可执行文件的目录的绝对路径
+ GOEXE ： 当前系统可执行文件的后缀
+ GOPATH ： 工作区目录的绝对路径，就是我刚刚设置的工作路径
+ GOROOT ： Go语言的安装目录的绝对路径

### 1.2 Go 命令
接下来我们看一下 Go 有哪些命令

![Go 命令.png](https://upload-images.jianshu.io/upload_images/1394028-bf10b393bfd80ea9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

简单的过一下，我们常用的一般也就几条，如
```go
go build     用于编译代码
go run       并运行 Go 程序
go doc       查看 Go 文档，但是需要安装相关插件
go version   查看 Go 当前的版本
go clean     移除当前源码包和关联源码包里面编译生成的文件
go fmt       格式化 Go 文件，gofmt -w -l src 可以格式化整个项目
go get       获取远程包，该命令内部执行了两步，第一步是下载源码包，第二步是执行go install
go install   生成可执行文件
...
```

详细命令可参考官方文档 [Here](https://go-zh.org/cmd/go/)

## 2、Go Package

Go 语言同 Java 一样都是有包的概念，这也是一种模块化的方式组织代码，在 Go 语言中，包也是类似的概念，它是把我们的 Go 文件组织起来，可以方便进行归类、复用等目的。比如 Go 内置的 net 包

```go
net
├── http
├── internal
├── mail
├── rpc
├── smtp
├── testdata
├── textproto
└── url
```

以上是 net 包的一个目录结构，net 本身是一个包，net 目录下的 http 又是一个包。从这个大家可以看到，Go 语言的包其实就是我们计算机里的目录，或者叫文件夹，通过它们进行目录结构和文件组织，Go 只是对目录名字做了一个翻译，叫``"包"``而已。比如这里的 net 包其实就是 net 目录，http 包其实就是 http 目录，这也是 Go 语言中的一个命名习惯，包名和文件所在的目录名是一样的。

### 2.1 包的命名

> 首先我们在前面的 Hello World 中声明了一个 main 包，这就和 Java 的 main 函数一个意思，告诉程序我是入口函数，这里 Go 会把声明了 main 的包编译为一个可执行程序，并且同一个项目里只有一个入口文件。

Go 语言中的包名一般遵循小写、和所在目录同名且不应使用下划线或驼峰记法的原则。当我们只有一个项目的时候我们可以直接把文件写在 src 目录下，如果我们有多个项目同时存在，我们就要通过包组织我们的代码了，官网是推荐使用你个人的 github.com，这一点同 Java 中类似。

这里再看下上面的 Hello World 程序
+ 首先我们引入了 ``package main``，这里 package 是一个关键字，定义一个包，和 Java 里的 package 一样，也是模块化的关键。
+ main 包是一个特殊的包名，表示程序执行的入口，而不是一个库。
+  import 也是一个关键字，表示要引入的包，和 Java 的 import 关键字一样，引入后才可以使用它。
+ fmt 是一个包名，这里表示要引入 fmt 这个包，这样我们就可以使用它的函数了。
+ Println 是 fmt 包里的函数，和 Java 里的 system.out.println 作用类似，这里注意 Println 是大写的，至于为啥呢，我们接下来再说。

### 2.2 包的导入
要想使用一个包，必须先导入它才可以使用，Go 语言提供了 import 关键字来导入一个包，这个关键字告诉 Go 编译器到磁盘的哪里去找要想导入的包，所以导入的包必须是一个全路径的包，也就是包所在的位置。``注意：引用了必须使用，否则会报错。``

```go
import "fmt"
```

在我们的 Hello World 中我们打印了一段话，引入了 ``fmt`` 包，如果引入多个我们可以如下使用

```go
import "fmt"
import "github.com/sumin/test"  // 远程包引入

// 推荐下面的写法
import (
    "fmt"
    "github.com/sumin/test"
)
```
### 2.3 包的别名
在我们开发过程中有可能会出现重复的情况（尽量避免），这个时候我们可以给其中一个设置别名使用，在包的左侧引入，编译器会自动识别该别名。

```go
package main

import (
    "fmt"
    sufmt "github.com/sumin/fmt"
)

func main() {
    fmt.Println()
    sufmt.Println()
}
```

还有上面那个问题，Go 语言规定导入的包必须引用，这样确实避免了我们引用很多无用的包增加代码量，对于维护简直友好度MAX！但是有的时候我们需要导入一个包执行包里的 init 函数而不想引用它，这个时候可以使用 Go 内置的空白标识符 ``_``，还是拿上面的代码举例

```go
package main

import (
    "fmt"
    _ "github.com/sumin/fmt"
)

func main() {
    fmt.Println()
}
```

## 3、 编辑器
工欲善其事必先利其器，我们使用一个顺手的编辑器开发也会省心不少。不过 Go 不怎么挑剔编辑器，它本身采用的是 UTF-8 的文本文件存放源代码，所以原则上你可以使用任何文本编辑器。

对于我这种新手来说 Jetbrains idea + Go 插件最省心，当然也有专门支持 Go 语言的 GoLand [下载地址]([https://www.jetbrains.com/go/](https://www.jetbrains.com/go/)
)

同时也推荐使用 vscode，内置 Go 全家桶，轻便简洁，主要是好看，不过需要手动下载 Go 插件。

## 本章小结
第一篇关于 Go 的准备工作就到这里吧，大概讲了的 Go 开发安装配置、环境搭建、Go 相关命令，后面还介绍了 Go 的安装和包管理以及编辑器这一块，接下来会继续学习 Go 的基础语法和库，加油~

## 导航
+ 下一节：[Go语言的变量与类型](./1、Go语言的变量与类型.md)