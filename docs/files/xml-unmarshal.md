# 解析XML

xml包 提供了 Unmarshal() 方法来解析XML文件。方法定义如下：

```text
func Unmarshal(data []byte, v interface{}) error
```
data 接收的是XML数据流，v 是需要输出的结构，可以转换成任意格式。

示例：
```xml
<?xml version="1.0" encoding="utf-8"?>

<servers version="2.0"> 
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

```go
package main

import (
	"encoding/xml"
	"fmt"
	"io/ioutil"
	"os"
)

type Recurlyservers struct {
	XMLName     xml.Name    `xml:"servers"`
	Version     string      `xml:"version,attr"`
	Servers     []server    `xml:"server"`
	Description string      `xml:",innerxml"`
}

type server struct {
	XMLName     xml.Name    `xml:"server"`
	ServerName  string      `xml:"serverName"`
	ServerIp    string      `xml:"serverIp"`
}

func main() {
	// 打开xml文件
	file, err := os.Open("server.xml")
	if err != nil {
		panic(err)
	}
	
	// 延迟语句关闭文件
	defer file.Close()
	
	// 读取文件内容
	data, err := ioutil.ReadAll(file)
	if err != nil {
		panic(err)
	}
	
	v := Recurlyservers{}
	err = xml.Unmarshal(data, &v)
	if err != nil {
		panic(err)
	}
	fmt.Print(v)
}
```
输出：
```text
{{ servers} 2.0 [{{ server} dazhairen.com 127.0.0.1} {{ server} go.dazhairen.com 127.0.0.1}] 
    <server>
        <serverName>dazhairen.com</serverName>
        <serverIp>127.0.0.1</serverIp>
    </server>
    <server>
        <serverName>go.dazhairen.com</serverName>
        <serverIp>127.0.0.1</serverIp>
    </server>
}
```

上面例子中，struct 多定义了一些类似于 xml:"ServerName" 这样的内容，这是[struct](/progress/struct/#_4)的一个特性，被称为 struct tag，是用来辅助反射的。
Unmarshal 解析的时候，XML 元素与 struct 字段的对应是有优先级顺序的，首先会读取 struct tag ，如果没有救护丢去对应的字段名。
注意：解析的时候tag、字段名、XML元素都是大小写敏感的，所以字段必须一一对应。


解析 XML 到 struct 的时候遵循以下规则：

- 若 struct 中的一个字段是 string 或者 []byte 类型且它的 tag 含有 ",innerxml"，Unmarshal 会将这个字段所对应的元素内所有内嵌的原始 XML 都加到这个字段上。如上面的 Description

- 若 struct 中的一个字段叫 XMLName，且类型为 xml.Name ，那么在解析的时候就会保存这个element的名字到该字段，如上面的 Servers

- 某个 struct 字段的tag定义中含有 XML 结构中的element的名称，那么解析的时候就会把相应的element值赋给该字段，如上面的 ServerName 和 ServerIp

- 若某个 struct 字段的tag定义中含有 ",attr"，那么解析的时候就会将该结构所对应element的与字段同名的属性的值赋给该字段，如上面的 Version

- 如果某个 struct 字段的tag定义形如 "a>b>c"，则解析的时候，会将XML结构a下面的b下面的c元素赋给该字段

- 如果某个 struct 字段的tag定义了 "-"，那么不会为该字段解析匹配任何XML数据

- 如果 struct 字段后面的tag定义了 ",any"，如果它的子元素在不满足其它规则时，就会匹配到这个字段

- 如果某个XML元素含有一条或者多条注释，那么这些注释将被累加到第一个tag含有 ",comments" 的字段上，这个字段的类型可能是 []byte 或 string ，如果没有这样的字段存在，那么注释将会被抛弃


!!! note ""
    Go语言的xml包要求struct定义中的所有字段必须是可导出的（首字母大写）。
