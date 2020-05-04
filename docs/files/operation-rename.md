# 重命名与移动

在Go语言中，重命名和移动文件都是同样的原理，方法如下：

```text
func Rename(oldpath, newpath string) error
```


代码示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	oldPath := "test.txt"
	newPath := "example.txt"
	err := os.Rename(oldPath, newPath)
	if err != nil {
		fmt.Println(err)
	}

	oldPath2 := "hello.txt"
	newPath2 := "a/hello.txt"
	err = os.Rename(oldPath2, newPath2)
	if err != nil {
		fmt.Println(err)
	}
}
```
