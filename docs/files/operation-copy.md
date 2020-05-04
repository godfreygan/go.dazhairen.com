# 复制文件

示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 打开原始文件
	originalFile, err := os.Open("example.txt")
	if err != nil {
		panic(err)
	}
	
	defer originalFile.Close()
	
	// 创建新的文件作为目标文件
	newFile, err := os.Create("example_copy.txt")
	if err != nil {
		panic(err)
	}
	
	defer newFile.Close()
	
	// 从源文件中复制字节到目标文件
	bytesWritten, err := io.Copy(newFile, originalFile)
	if err != nil {
		panic(err)
	}
	fmt.Printf("文件已复制，大小为 %d byte\n", bytesWritten)
	
	// 将内容flush到硬盘中
	err = newFile.Sync()
	if err != nil {
		panic(err)
	}
}
```
输出：
```text
文件已复制，大小为 25 byte
```


注意：

Create() 执行之后需要执行 Close() 回收资源，一般使用defer。

io包的copy函数执行之后，可能还没保存到文件中，尽量使用 Sync() 函数进行同步保存到硬盘中
