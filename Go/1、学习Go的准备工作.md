![可爱的土拨鼠.png](https://upload-images.jianshu.io/upload_images/1394028-95d6eaf99111984f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

## 前言
在我们接触一门新的语言的时候，并不着急马上去学习它的语法，应该从一个全局的视角来看看我们接下来要学的语言时一个什么样的语言，值不值得我们去学，它和我们现有的技术栈有什么区别，是否能够解决技术痛点，以上这几点是我觉得学习之前先要了解的。

Go 语言目前还是比较新的语言，它是一种自带并发、具有垃圾回收以及快速编译的静态类型语言，它结合了解释型语言的游刃有余，动态类型语言的开发效率，以及静态类型的安全性。同 Java 相比，Go语言有很多不同的地方，比如一开始的公有变量和私有变量的规范，Go语言中变量大写就是公有变量，小写就是私有变量（error除外），Go 内置了线程和队列，实现起来比 Java 简单很多，再比如 Go 的异常处理没有一大推try/catch，等等就不一列举。

Go 语言是 Google 团队的大佬精心打造出来的，成员之一就有大名鼎鼎的 Unix 操作系统的创造者 Ken Thompson，C 语言就是他和已经过世的李奇一起发明的。接下来让我们一起走进 Go 语言的世界吧！

### 1、环境配置
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

#### 1.1 Go 语言安装
我们根据自己的机器操作系统选择相应的开发工具包，比如你的是Windows 64位的，就选择windows-amd64的工具包；是Linux 32位的就选择linux-386的工具包。可以自己查看下自己的操作系统，然后选择，我这边选择 Mac 版本的下载就可以了。

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
接下来设置设置我们 GOPATH 下的 src 文件夹下新建如下文件夹
```
├── bin
├── pkg
└── src
```
+ bin 文件夹存放 go install 生成的可执行文件，当我们把 GOPATH/bin 加入到 PATH 后，我们可以再终端的任意位置直接执行 go 文件了
+ pkg 文件夹是存在go编译生成的文件
+ src存放的是我们的go源代码，不同工程项目的代码以包名区分

#### 1.2 Go 命令
首先我们看一下 Go 有哪些命令

![Go 命令.png](https://upload-images.jianshu.io/upload_images/1394028-bf10b393bfd80ea9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

简单的过一下，我们常用的一般也就几条，如
```go
go build     用于编译代码
go run       并运行 Go 程序
go doc       查看 Go 文档，但是需要安装相关插件
go env       查看当前 Go 的环境变量
go version   查看 Go 当前的版本
go clean     移除当前源码包和关联源码包里面编译生成的文件
go fmt       格式化 Go 文件，gofmt -w -l src 可以格式化整个项目
go get       获取远程包，该命令内部执行了两步，第一步是下载源码包，第二步是执行go install
go install   生成可执行文件
...
```

#### 1.3 Go Package
当我们只有一个项目的时候我们可以直接把文件写在 src 目录下，如果我们有多个项目同时存在，我们就要通过包组织我们的代码了，官网是推荐使用你个人的 github.com，这样就不会有重复了。

这里再看下上面的 Hello World 程序
+ 首先我们引入了 ``package main``，这里 package 是一个关键字，定义一个包，和 Java 里的 package 一样，也是模块化的关键。
+ main包是一个特殊的包名，它表示当前是一个可执行程序，而不是一个库。
+  import 也是一个关键字，表示要引入的包，和 Java 的 import 关键字一样，引入后才可以使用它。
+ fmt 是一个包名，这里表示要引入 fmt 这个包，这样我们就可以使用它的函数了。
+  main 函数是主函数，表示程序执行的入口，Java 也有同名函数，但是多了一个 String[] 类型的参数。
+ Println 是 fmt 包里的函数，和 Java 里的 system.out.println 作用类似，这里注意 Println 是大写的，至于为啥呢，我们接下来再说。

#### 1.4 编辑器
工欲善其事必先利其器，我们使用一个顺手的编辑器开发也会省心不少。不过 Go 不怎么挑剔编辑器，它本身采用的是 UTF-8 的文本文件存放源代码，所以原则上你可以使用任何文本编辑器。

对于我这种新手来说 Jetbrains idea + Go 插件最省心，当然也有专门支持 Go 语言的 GoLand [下载地址]([https://www.jetbrains.com/go/](https://www.jetbrains.com/go/)
)

同时也推荐使用 vscode，内置 Go 全家桶，轻便简洁，主要是好看，不过需要手动下载 Go 插件。

### 小结
第一篇就到这里吧，很多东西没说完整，只是提了一个大概的 Go 开发环境配置，主要是介绍了 Go 的安装和包管理和常用的 Go 命令以及部分编辑器这一块，接下来会继续学习 Go 的基础语法和库，加油~