# 打包与解压

Go语言中的 archive 包提供了打包文件和解包文件的方法。


## 打包文件


操作步骤说明：

1）生成打包后的目标文件

2）获取要打包的文件集

3）往目标文件里写文件


示例：
```go
package main

import (
	"archive/tar"
	"fmt"
	"io"
	"os"
)

func main() {
	dist := "a.tar"
	err := TestTar([]string{"test.go", "test2.go"}, dist)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("打包成功")
	}
}

func TestTar(src []string, dist string) error {
	// 创建tar文件
	fw, err := os.Create(dist)
	if err != nil {
		return err
	}

	defer fw.Close()

	// 通过fw创建一个tar.Writer
	tw := tar.NewWriter(fw)

	// 如果关闭失败，可能会造成tar不完整
	defer func(){
		if err := tw.Close(); err != nil {
			fmt.Println(err)
		}
	}()

	for _, fileName := range src {
		fi, err := os.Stat(fileName)
		if err != nil {
			fmt.Println(err)
			continue
		}

		fih, err := tar.FileInfoHeader(fi, "")

		// 将tar的文件信息fih写入到tw
		err = tw.WriteHeader(fih)
		if err != nil {
			return err
		}

		// 将文件数据写入
		fs, err := os.Open(fileName)
		if err != nil {
			return err
		}

		if _, err := io.Copy(tw, fs); err != nil {
			return err
		}

		fs.Close()
	}
	return nil
}
```
输出，同时会在指定目录产生tar文件：
```text
打包成功
```


## 解包文件


操作步骤说明：

1）打开tar文件

2）遍历tar中的文件信息

3）创建文件、写入、保存、关闭文件


示例：
```go
package main

import (
	"archive/tar"
	"fmt"
	"io"
	"os"
)

func main() {
	srcFile := "a.tar"
	if err := TestUnTar(srcFile); err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("解包成功")
	}
}

func TestUnTar(srcFile string) error {
	// 打开tar包
	fr, err := os.Open(srcFile)
	if err != nil {
		return err
	}

	// 延迟语句关闭文件
	defer fr.Close()

	tr := tar.NewReader(fr)

	for {
		fih, err := tr.Next()
		if err != nil {
			if err == io.EOF {
				break
			} else {
				fmt.Println(err)
				continue
			}
		}

		// 读取文件信息
		fi := fih.FileInfo()

		// 创建一个空文件，用来写入解包后的数据
		newFileName := "bk_"+ fi.Name()
		fw, err := os.Create(newFileName)
		if err != nil {
			fmt.Println(err)
			continue
		}

		// 写入内容
		_, err = io.Copy(fw, tr)
		if err != nil {
			fmt.Println(err)
			continue
		}

		// 修改权限并关闭
		os.Chmod(newFileName, fi.Mode().Perm())
		fw.Close()
	}
	return nil
}
```
输出，同时会在指定目录中生成bk_系列文件：
```text
解包成功
```
