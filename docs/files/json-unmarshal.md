# 解析JSON

解析JSON有两种方法，一种是解析到结构体，另一种是解析到接口。
第一种是要在知道JSON数据的结构的前提下采取的方案。如果不知道被解析的数据的格式，则应该采用解析到接口的方案。


## 解析到结构体

Go语言的json包提供了 Unmarshal 函数，通过该函数就可以实现json的解析。函数定义如下：

```text
func Unmarshal(data []byte, v interface{}) error
```

示例：
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Server struct {
	ServerName  string
	ServerIp    string
}

type ServerSlice struct {
	Servers []Server
}

func main() {
	var s ServerSlice
	str := `{"servers": [{"serverName": "dazhairen.com", "serverIp": "127.0.0.1"}, {"serverName": "go.dazhairen.com", "serverIp": "127.0.0.1"}]}`
	json.Unmarshal([]byte(str), &s)
	fmt.Print(s)
}
```
输出：
```text
{[{dazhairen.com 127.0.0.1} {go.dazhairen.com 127.0.0.1}]}
```


解析 JSON 到 struct 的时候遵循以下规则：

- 首先查找 struct 中的 tag 是否含有 XML 相匹配的字段

- 其次查找 struct 中的字段名是否与 XML 中的字段相匹配

- 最后查找 struct 中除了首字母之外的其它大小写不敏感的导出字段

!!! note ""
    Go语言的json包要求struct定义中的所有字段必须是可导出的（首字母大写）。


## 解析到接口

使用Bitly公司开源的 simplejson 包，在处理未知结构体的 JSON 时比官方包更加方便。

示例：
```go
package main

import (
	"fmt"
	"github.com/bitly/go-simplejson"
)

type Server struct {
	ServerName  string
	ServerIp    string
}

type ServerSlice struct {
	Servers []Server
}

func main() {
	str := `{"servers": [{"serverName": "dazhairen.com", "serverIp": "127.0.0.1"}], "dbServer": {"host": "127.0.0.1", "database": "test", "username": "root", "password": "root"}}`

	json, err := simplejson.NewJson([]byte(str))
	if err != nil {
		panic(err)
	}

	serverArr, _ := json.Get("servers").Array()
	fmt.Println(serverArr)

	dbHost := json.Get("dbServer").Get("host").MustString()
	fmt.Println("database host：", dbHost)

	dbName := json.Get("dbServer").Get("database").MustString()
	fmt.Println("database：", dbName)
}
```
输出：
```text
[map[serverIp:127.0.0.1 serverName:dazhairen.com]]
database host： 127.0.0.1
database：test
```