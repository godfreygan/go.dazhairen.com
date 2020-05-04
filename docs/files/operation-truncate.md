# 裁剪

Go语言还提供了一个裁剪函数用于处理文件大小。方法定义如下：

```text
func Truncate(name string, size int64) error
```

name 为所要裁剪的文件（路径）；size 为裁剪大小，也就是字节数。

示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	err := os.Truncate("example.txt", 100)
	if err != nil {
		fmt.Println(err)
	} else {
	    fmt.Println("裁剪文件成功！！！")
	}
}
```
输出：
```text
裁剪文件成功！！！
```


额外说明：

- 如果文件里面的内容本就少于指定裁剪的字节数，则文件原始内容会继续保留，剩余的字节以 null 字节填充

- 如果文件里面的内容超过指定裁剪的字节数，则超过部分将会被裁剪丢弃

- 如果指定裁剪的字节数为 0，则表示清空整个内容
