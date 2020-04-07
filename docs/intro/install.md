# Go语言的开发环境部署

Go语言支持 Linux、MacOS、Windows等平台。

与Java等编程语言一样，安装Go语言开发环境需要设置全局的操作系统环境变量（除非使用包管理工具直接安装）。

主要系统级别的环境变量有如下这些：

* \$GOROOT：表示Go语言环境在计算机上的安装位置，它的值可以是任意你喜欢的位置（方便管理为宜），这个变量只有一个值，值的内容必须是绝对路径。

* \$GOPATH：这是Go语言的工作目录，可以有多个，类似工作空间的概念。一般不建议将 \$GOROOT 和 \$GOPATH 设置为用一个目录。


下面将详细介绍在各个平台的安装方法：

* [1. 在Windows平台安装](/intro/windows-install)
* [2. 在Linux平台安装](/intro/linux-install)
* [3. 在MacOS平台安装](/intro/macos-install)