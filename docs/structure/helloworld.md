# 第一个Go语言程序


1） 创建组织目录，如 go.local，然后建立一个项目名为 hello-world，最终目录路径为 \$GOPATH/src/go.local/hello-world/

```shell
$ mkdir -p $GOPATH/src/go.local/hello-world/
```


2） 在项目 hello-world 下新建一个文件，名为 main.go，里面的内容如下：

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello World")
}
```


3） 直接在当前目录运行，在命令行键入 go run main.go，会输出 “Hello World”

```shell
$ go run main.go
Hello World
```


说明一下：

上述程序分为三个部分，

第一部分是包所属，即 package main，package 用于定义当前代码所属的包，与 Java 里的 package 类似，作为模块化的标识。main 包是一个特殊的包名，它表示当前是一个可执行程序，而不是一个库。

第二部分就是 import，称为包的导入，Go语言要求只有用到的包才能导入，如果导入一个代码包又不使用，那么编译时会报错。

第三部分是程序的主体，func 是一个函数定义关键字，main 是程序的主函数，表示程序执行的入口。

## **GOPATH的目录结构变为**
```text
├─ bin
│   └─ helloworld
├─ pkg
└─ src
    └─ go.local
        └─ hello-world
            └─ hello.go
```
