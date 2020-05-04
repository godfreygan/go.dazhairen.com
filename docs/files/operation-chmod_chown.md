# 权限操作

Go语言中，提供了修改文件权限和所有者的方法。类似于 Linux 下的 chmod 和 chown 命令，直接调用相应的函数即可。


示例：

```go
package main

import (
	"fmt"
	"os"
	"time"
)

func echoMode(filePath string) {
	fileInfo, err := os.Stat(filePath)
	if err != nil {
		if os.IsNotExist(err) {
			fmt.Println("file not exist")
		} else {
			fmt.Println(err)
		}
	}
	fmt.Println("当前权限为：", fileInfo.Mode())
}

func main() {
	filePath := "example.txt"

	// 查看文件权限
	echoMode(filePath)

	// 改变文件权限为 777 - 可读可写可执行
	time.Sleep(1 * time.Second)
	err := os.Chmod(filePath, 0777)
	if err != nil {
		fmt.Println(err)
	}

	// 查看文件权限
	echoMode(filePath)

	// 改变文件所有者
	time.Sleep(1 * time.Second)
	err = os.Chown(filePath, os.Getuid(), os.Getgid())
	if err != nil {
		fmt.Println(err)
	}
}

```
输出：
```text
当前权限为： ---xrwxrwx
当前权限为： -rwxrwxrwx
```
