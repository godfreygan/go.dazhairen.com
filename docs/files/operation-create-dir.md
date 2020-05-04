# 新建文件夹

新建文件夹的方法有两种：

1）func Mkdir(name string, perm FileMode) error

创建名为name的目录，权限设置为perm。该方法只能逐个文件夹来创建，如果name表示多个多个文件夹（如 a/b/c），将会报错。


2）func MkdirAll(path string, perm FileMode) error

根据path创建多层级目录。如 /usr/data/godfreygan/go。


```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 使用Mkdir创建一个目录名为 a
	err := os.Mkdir("a", 0777)
	if err != nil {
		fmt.Println(err)
	}

	// 使用Mkdir创建多个目录 a/b/c
	err = os.Mkdir("a/b/c", 0777)
	if err != nil {
		fmt.Println(err)
	}

	// 创建多层级的目录 a/b/c
	err = os.MkdirAll("a/b/c", 0777)
	if err != nil {
		fmt.Println(err)
	}
}
```
输出：
```text
mkdir a/b/c: no such file or directory
```
