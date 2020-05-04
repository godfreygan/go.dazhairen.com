# 删除

有创建文件，自然就有删除文件。方法定义如下：

```text
func Remove(name string) error
```

示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	err := os.Remove("example.txt")
	if err != nil {
		fmt.Println(err)
	} else {
	    fmt.Println("删除文件成功！！！")
	}
}
```
输出：
```text
删除文件成功！！！
```

!!! note ""
    删除文件前要确保文件没有被其它程序占用。如果在当前程序中该文件已被打开，则需要先关闭文件（Close()）。
