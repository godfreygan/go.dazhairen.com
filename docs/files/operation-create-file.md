# 新建文件

新建文件可以通过两个方法来实现：

1）func Create(name string) (file *File, err error)

根据提供的文件名创建新的文件，返回一个文件对象，默认权限是 0666（读写）。


2）func NewFile(fd uintptr, name string) *File

根据文件描述符创建相应的文件，返回一个文件对象。


示例1：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	newFile, err := os.Create("os_create.txt")

	// 创建失败，直接 panic
	if err != nil {
		panic(err)
	}

	// 延迟语句关闭句柄
	defer func() {
		if err := newFile.Close(); err != nil {
			fmt.Println("关闭文件失败")
		}
	}()

	// 打印newFile
	fmt.Println(newFile)
}
```


示例2：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	newFile := os.NewFile(1, "os_newfile.txt")
	fmt.Println(newFile)
}
```
输出：
```text
&{0xc0000a4120}
```
