# 文件/文件夹状态

查看文件/文件夹状态的方法如下：

func Stat(name string) (FileInfo, error)

如果没有错误将返回描述该文件的FileInfo。


示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	fileInfo, err := os.Stat("os_create.txt")
	if err != nil {
		panic(err)
	}

	fmt.Println("文件名称：", fileInfo.Name())
	fmt.Println("文件大小：", fileInfo.Size())
	fmt.Println("文件权限", fileInfo.Mode())
	fmt.Println("最后修改时间：", fileInfo.ModTime())
	fmt.Println("是否是文件夹：", fileInfo.IsDir())
	fmt.Printf("系统接口类型：%T\n", fileInfo.Sys())
	fmt.Printf("系统信息：%+v", fileInfo.Sys())
}
```
输出：
```text
文件名称： os_create.txt
文件大小： 7
文件权限 -rw-r--r--
最后修改时间： 2019-04-19 16:18:42.47947227 +0800 CST
是否是文件夹： false
系统接口类型：*syscall.Stat_t
系统信息：&{Dev:16777220 Mode:33188 Nlink:1 Ino:12270048 Uid:501 Gid:20 Rdev:0 Pad_cgo_0:[0 0 0 0] Atimespec:{Sec:1587284323 Nsec:760702025} Mtimespec:{Sec:1587284322 Nsec:479472270} Ctimespec:{Sec:15872 Nsec:479472270} Birthtimespec:{Sec:1587284113 Nsec:997478909} Size:7 Blocks:8 Blksize:4096 Flags:0 Gen:0 Lspare:0 Qspare:[0 0]}
```