# 读写权限判断

示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	filePath := "example.txt"

	// 测试写权限
	file, err := os.OpenFile(filePath, os.O_WRONLY, 0666)
	if err != nil && os.IsPermission(err) {
		fmt.Println("error：没有写入权限")
	} else if err != nil && os.IsNotExist(err) {
		fmt.Println("error：文件不存在")
	} else if err != nil {
		fmt.Println(err)
	}
	file.Close()

	// 测试读权限
	file, err = os.OpenFile(filePath, os.O_RDONLY, 0666)
	if err != nil && os.IsPermission(err) {
		fmt.Println("error：没有读取权限")
	} else if err != nil && os.IsNotExist(err) {
		fmt.Println("error：文件不存在")
	} else if err != nil {
		fmt.Println(err)
	}
	file.Close()
}
```
根据不同情况会输出不同内容，如：
```text
error：没有写入权限
error：没有读取权限
```

!!! note ""
    文件没有读写权限会返回error，并且文件不存在也会返回error，所以需要检查error的信息来确定到底是哪个错误导致的。