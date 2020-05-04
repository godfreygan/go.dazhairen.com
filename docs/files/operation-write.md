# 写入函数

写入函数有下面几种：

```text
func (file *File) Write(b []byte) (n int, err Error)                // 写入byte类型的数据到文件中

func (file *File) WriteAt(b []byte, off int64) (n int, err Error)   // 在指定位置开始写入byte类型的数据

func (file *File) WriteString(s string) (ret int, err Error)        // 写入string类型到文件
```

示例：

```go
package main

import (
	"os"
)

func main() {
	filePath := "example.txt"

	// 以可写方式打开文件
	file, err := os.OpenFile(filePath, os.O_WRONLY | os.O_TRUNC, 0666)
	if err != nil {
		panic(err)
	}

	defer file.Close()

	// 将字节写入文件中
	file.Write([]byte("hello world, hello go! \r\n"))

	// 将字符串写入文件中
	file.WriteString("go language is best!")

	// 打印文件内容
	file, err = os.Open(filePath)
	if err != nil {
		panic(err)
	}

	buf := make([]byte, 1024)
	for {
		n, _ := file.Read(buf)
		if 0 == n {
			break
		}
		os.Stdout.Write(buf[:n])
	}
	file.Close()
}
```
输出：
```text
hello world, hello go! 
go language is best!
```

