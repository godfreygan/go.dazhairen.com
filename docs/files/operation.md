# 文件操作

在任何系统中，文件都是必须的对象。UNIX的核心思想就是"万物皆文件"。对于编程而言，文件的操作一直是开发中经常遇到的问题。如创建目录、新建文件、文件编辑等。

os包中提供了大部分的文件操作函数，通过实现里面的接口，我们可以方便、快捷地对文件（夹）进行操作。

常用的操作函数有如下几个：

```text
func Mkdir(name string, perm FileMode) error        // 创建名为name的目录，权限设置为perm。如 0777

func MkdirAll(path string, perm FileMode) error     // 根据path创建多层级目录。如 /usr/data/godfreygan/go

func Remove(name string) error                      // 删除名为name的目文件/文件夹，如果目录下有文件或者其它目录时会报错

func RemoveAll(path string) error                   // 根据path删除多级子文件/文件夹，如果path是单个名称，那么该目录下的文件/文件夹全部删除

func Create(name string) (file *File, err error)    // 新建文件

func Stat(name string) (FileInfo, error)            // 查询文件/文件夹状态

func Rename(oldpath, newpath string) error          // 重命名或移动文件/文件夹

func Open(name string) (file *File, err error)      // 打开名为name的文件

func (file *File) Write(b []byte) (n int, err Error)// 写入byte类型的信息到文件
```

目录操作示例：
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 创建一个目录名为 goDir
	os.Mkdir("goDir", 0777)

	// 创建多层级的目录 goDir/a/b/c
	os.MkdirAll("goDir/a/b/c", 0777)

	// 删除一个指定的目录 goDir
	err := os.Remove("goDir")
	if err != nil {
		fmt.Println(err)
	}

	// 删除指定的目录以及它的子级
	err = os.RemoveAll("goDir")
	if err != nil {
		fmt.Println(err)
	}
}
```
输出：
```text
remove goDir: directory not empty
```
