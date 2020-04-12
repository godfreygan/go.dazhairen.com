# Go语言支持IP类型

Go语言中的 net包 提供了很多类型、函数和方法用于网络编程。

IP定义方式：`type IP []byte`

ParseIP(s string)IP，把IPv4或者IPv6的地址转化为IP类型。

示例：
```go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	str := "127.0.0.1"
	addr := net.ParseIP(str)
	if addr == nil {
		fmt.Println("无效的IP地址")
	} else {
		fmt.Println("IP的地址是", addr.String())
		fmt.Println("IP的mask是", addr.DefaultMask())
	}
	os.Exit(0)
}
```