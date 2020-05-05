# 压缩与解压

Go语言中的 compress 包提供了压缩文件和解压文件的方法。


## 压缩文件

操作步骤说明：

1）生成压缩后的目标文件

2）实例化gzip.writer

3）获取要打包的文件

4）往目标文件里写数据


示例：
```go
package main

import (
	"compress/gzip"
	"fmt"
	"os"
)

func main() {
	dist := "a.gzip"
	err := testGzip("test.go", dist)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("压缩成功")
	}
}

func testGzip(srcFile string, dist string) error {
	// 创建gzip文件
	outputFile, err := os.Create(dist)
	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	// 实例化新的gzip.Writer
	gzipWriter := gzip.NewWriter(outputFile)
	defer gzipWriter.Close()

	// 获取需要压缩的文件信息
	fr, err := os.Open(srcFile)
	if err != nil {
		return err
	}

	fi, err := fr.Stat()
	if err != nil {
		fmt.Println(err)
	}

	// 创建gzip.Header
	gzipWriter.Header.Name = fi.Name()

	// 读取文件信息
	buf := make([]byte, fi.Size())
	_, err = fr.Read(buf)
	if err != nil {
		return err
	}

	// 写入gzip包
	_, err = gzipWriter.Write(buf)
	if err != nil {
		return err
	}

	return nil
}
```
输出，同时会在指定位置产生gzip文件：
```text
压缩成功
```


## 解压文件


操作步骤说明：

1）打开gzip文件

2）创建gzip.Reader

3）读取gzip文件内容

4）写入新文件


示例：
```go
package main

import (
	"compress/gzip"
	"fmt"
	"os"
)

func main() {
	srcFile := "a.gzip"
	if err := testUnGzip(srcFile); err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("解压成功")
	}
}

func testUnGzip(dist string) error {
	// 打开gzip文件
	fr, err := os.Open(dist)
	if err != nil {
		return err
	}

	defer fr.Close()

	// 创建gzip.Reader
	gzipReader, err := gzip.NewReader(fr)
	if err != nil {
		return err
	}

	defer gzipReader.Close()

	// 读取文件内容
	buf := make([]byte, 1024 * 1024 * 10)
	n, err := gzipReader.Read(buf)

	// 将包中的文件数据写入新文件中
	fw, err := os.Create("bk_"+ gzipReader.Header.Name)
	if err != nil {
		return err
	}

	_, err = fw.Write(buf[:n])
	if err != nil {
		return err
	}

	return nil
}
```
输出，同时在指定位置中生成bk_xxx文件：
```text
解压成功
```
