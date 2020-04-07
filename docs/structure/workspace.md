# 工程结构


## GOROOT结构

\$GOROOT作为Go语言环境的根目录，有如下内容：

```shell
$ cd $GOROOT && ls -la
-rw-r--r--    1 godfrey  staff   1339 11  1 06:56 CONTRIBUTING.md
-rw-r--r--    1 godfrey  staff  84339 11  1 06:56 CONTRIBUTORS
-rw-r--r--    1 godfrey  staff   1303 11  1 06:56 PATENTS
-rw-r--r--    1 godfrey  staff    397 11  1 06:56 SECURITY.md
-rw-r--r--    1 godfrey  staff      8 11  1 06:56 VERSION
drwxr-xr-x   19 godfrey  staff    608 11  1 06:56 api
drwxr-xr-x    5 godfrey  staff    160 11  1 06:56 bin
drwxr-xr-x   50 godfrey  staff   1600 11  1 06:56 doc
-rw-r--r--    1 godfrey  staff   5686 11  1 06:56 favicon.ico
drwxr-xr-x    3 godfrey  staff     96 11  1 06:56 lib
drwxr-xr-x   16 godfrey  staff    512 11  1 06:56 misc
drwxr-xr-x    6 godfrey  staff    192 11  1 06:56 pkg
-rw-r--r--    1 godfrey  staff     26 11  1 06:56 robots.txt
drwxr-xr-x   71 godfrey  staff   2272 11  1 06:56 src
drwxr-xr-x  327 godfrey  staff  10464 11  1 06:56 test
```

* api文件夹：存放了公开的变量、常量、函数等API的列表，用于Go语言API检索

* bin文件夹：主要用于存储标准命令执行文件，例如默认的go、godoc、gofmt三件套

* pkg文件夹：用于存放编译Go语言标准库生成的文件，这个文件夹里面的文件根据不同系统和结构会有所不同，在\$GOROOT这个路径中的pkg一般只有标准库的编译缓存文件，这些文件编译Go程序时起到很大的作用

* src文件夹：存放的是Go语言自己的源代码（包括标准库），阅读源码时就是看这里的内容



## GOPATH结构

如果把 \$GOROOT 看作一个安装目录，那么 $GOPATH 就是一个工作目录，开发工作就是在这个工作目录（workspace）里面进行的。

与PHP语言不同的是，Go语言只需要一个工作目录就可以了，在这个工作目录下可以分不同的项目不同的包来进行开发和运行。

在这个目录里面一般有3个文件夹，如下所示：

```shell
$ cd $GOPATH && ls -la
drwxr-xr-x   7 godfrey  staff  224  2 12 12:46 bin
drwxr-xr-x   3 godfrey  staff   96  2 11 13:44 pkg
drwxr-xr-x  10 godfrey  staff  320  2 12 00:16 src
```

* bin文件夹：存放 go install 生成的可执行文件，前面把 \$GOPATH/bin 路径加入到PATH环境变量后就可以全局任意位置使用这个文件夹中的可执行文件。

* pkg文件夹：存放go编译生成的文件

* src文件夹：存放我们开发的Go项目的源代码，不同工程项目的代码以包名区分。其中，src中的包名或项目名称习惯以 domain/project 为格式进行命名。


workspace大体结构为 workspace、repository、package：
```text
workspace：可包含多个版本控制的repository

	repository：可包含多个package

		package：可包含多个go源码文件
```

!!! note ""
	\$GOPATH 也是可以有很多个的，多个以冒号分隔。但是，go get 下载下来的文件只会存放在第一个目录里面。





