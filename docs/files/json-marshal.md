# 生成JSON

很多接口都是以 JSON 字符串的形式输出的，Go语言中的json包提供了 Marshal 函数以实现json的转化。函数定义如下：

```text
func Marshal(v interface{}) ([]byte, error)
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

	s.Servers = append(s.Servers, Server{ServerName: "dazhairen.com", ServerIp: "127.0.0.1"})
	s.Servers = append(s.Servers, Server{"go.dazhairen.com", "127.0.0.1"})

	b, err := json.Marshal(s)
	if err != nil {
		panic(err)
	}

	fmt.Print(string(b))
}
```
输出：
```text
{"Servers":[{"ServerName":"dazhairen.com","ServerIp":"127.0.0.1"},{"ServerName":"go.dazhairen.com","ServerIp":"127.0.0.1"}]}
```


通过例子可以看到，输出中的字段首字母都是大写的。如果想要用小写的首字母，通过修改 struct 中的字段名为小写是不行的，因为只有导出的字段才能被输出。
如果要想调整json结构中的字段名为小写，或者其它名字，可以通过 struct tag 定义来实现。

示例：
```go
package main

import (
	"encoding/json"
	"fmt"
)

type Server struct {
	ServerName  string  `json:"serverName"`
	ServerIp    string  `json:"ip"`
}

type ServerSlice struct {
	Servers []Server
}

func main() {
	var s ServerSlice

	s.Servers = append(s.Servers, Server{ServerName: "dazhairen.com", ServerIp: "127.0.0.1"})
	s.Servers = append(s.Servers, Server{"go.dazhairen.com", "127.0.0.1"})

	b, err := json.Marshal(s)
	if err != nil {
		panic(err)
	}

	fmt.Print(string(b))
}
```
输出：
```text
{"Servers":[{"serverName":"dazhairen.com","ip":"127.0.0.1"},{"serverName":"go.dazhairen.com","ip":"127.0.0.1"}]}
```


json包的 Marshal 函数转化过程中需要注意几点：

- JSON 对象只支持string作为key，所以要编码一个map，那么必须是map\[string\]T（T可以是任意类型）

- Channel、complex 和 function 是不能被编码成 JSON 的

- 嵌套的数据是不能被编码的，不然可能会让JSON编码进入死循环

- 指针在编码的时候会输出指针指向的内容，而空指针会输出 null

