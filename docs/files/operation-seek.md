# 跳转到文件指定位置

Go语言中，跳转函数为 Seek() ，用于文件中的指定位置。Seek() 函数的特点类似于鼠标光标的定位，指定位置之后可以执行复制、剪切、粘贴等一系列操作。

示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 打开原始文件
	file, err := os.Open("example.txt")
	if err != nil {
		panic(err)
	}

	defer file.Close()

	// 偏离位置，可以是正数也可以是负数
	var offset int64 = 10

	// offset的初始位置标记：0-文件开始位置，1-当前位置，2-文件结尾
	var whence int = 0
	newPosition, err := file.Seek(offset, whence)
	if err != nil {
		panic(err)
	}

	fmt.Printf("Move to %d：%d\n", offset, newPosition)

	// 从当前位置回退两个字节
	offset = -2
	whence = 1
	newPosition, err = file.Seek(offset, whence)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Move to %d：%d\n", offset, newPosition)

	// 获取当前位置
	offset = 0
	whence = 1
	currentPosition, err := file.Seek(offset, whence)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Current position：%d\n", currentPosition)

	// 转到文件开始处
	offset = 0
	whence = 0
	newPosition, err = file.Seek(offset, whence)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Position after seeking %d，%d：%d", offset, whence, newPosition)
}
```
输出：
```text
Move to 10：10
Move to -2：8
Current position：8
Position after seeking 0，0：0
```

