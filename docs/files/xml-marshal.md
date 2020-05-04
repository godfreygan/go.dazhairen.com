# 生成XML

xml包中提供了 Marshal 和 MarshalIndent 两个函数用于输出XML，这两个函数的主要区别是第二函数会增加前缀和缩进。函数定义如下：

```text
func Marshal(v interface{}) ([]byte, error)
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
```
两个函数中的第一个参数用来生成XML的结构定义类型数据，都是返回生成的XML数据流。

示例：
```go
package main

import (
	"encoding/xml"
	"fmt"
	"os"
)

type Servers struct {
	XMLName     xml.Name    `xml:"servers"`
	Version     string      `xml:"version,attr"`
	Servers     []server    `xml:"server"`
}

type server struct {
	ServerName  string  `xml:"serverName"`
	ServerIp    string  `xml:"serverIp"`
}

func main() {
	v := &Servers{Version: "2"}
	v.Servers = append(v.Servers, server{"dazhairen.com", "127.0.0.1"})
	v.Servers = append(v.Servers, server{"go.dazhairen.com", "127.0.0.1"})

	// 使用Marshal函数
	testMarshal(v)

	fmt.Printf("\n\n")

	// 使用MarshalIndent函数
	testMarshalIndent(v)
}

func testMarshal(v *Servers) {
	output, err := xml.Marshal(v)
	if err != nil {
		fmt.Println("error：", err)
	}
	os.Stdout.Write([]byte(xml.Header))
	os.Stdout.Write(output)
}

func testMarshalIndent(v *Servers) {
	output, err := xml.MarshalIndent(v, " ", "  ")
	if err != nil {
		fmt.Println("error：", err)
	}
	os.Stdout.Write([]byte(xml.Header))
	os.Stdout.Write(output)
}
```
输出：
```text
<?xml version="1.0" encoding="UTF-8"?>
<servers version="2"><server><serverName>dazhairen.com</serverName><serverIp>127.0.0.1</serverIp></server><server><serverName>go.dazhairen.com</serverName><serverIp>127.0.0.1</serverIp></server></servers>

<?xml version="1.0" encoding="UTF-8"?>
 <servers version="2">
   <server>
     <serverName>dazhairen.com</serverName>
     <serverIp>127.0.0.1</serverIp>
   </server>
   <server>
     <serverName>go.dazhairen.com</serverName>
     <serverIp>127.0.0.1</serverIp>
   </server>
 </servers>

```

!!! note ""
    Marshal 和 MarshalIndent 输出的信息都不会带 XML 头的，为了正确生成XML文件，必须使用xml包预定义的 Header 变量。

