# 打开与关闭

通过以下两个方法来打开文件：

1）func Open(name string) (file *File, err error)

该方法打开一个名称为 name 的文件，但用的是只读方式，内部实现其实是调用了 OpenFile() 函数。


2）func OpenFile(name string, flag int, perm uint32) (file *File, err Error)

打开一个名称为 name 的文件，flag 是打开方式，包括只读、读写等，prem 是权限。

flag 属性说明如下：
```text
os.O_RDONLY     // 只读
os.O_WRONLY     // 只写
os.O_RDWR       // 读写
os.O_APPEND     // 追加
os.O_CREATE     // 不存在则先新建
os.O_TRUNC      // 文件打开时裁剪文件
os.O_EXCL       // 和O_CREATE一起使用，文件不能存在
os.O_SYNC       // 以同步I/O的方式打开
```

!!! note ""
    OpenFile() 中的第二个参数的属性可以单独使用，也可以组合使用：  
    os.O_CREATE|os.O_APPEND
    os.O_CREATE|os.O_TRUNC|os.O_WRONLY


示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 以只读方式打开
	file, err := os.Open("example.txt")
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(file)
	file.Close()

	// 打开文件，可以使用各种模式
	// 使用追加模式打开文件
	file, err = os.OpenFile("example.txt", os.O_APPEND, 0666)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(file)
	file.Close()

	// 组合使用
	file, err = os.OpenFile("example.txt", os.O_CREATE | os.O_APPEND, 0666)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(file)
	file.Close()
}
```
