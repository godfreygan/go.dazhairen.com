# UDP Scoket

Go语言包中处理 UDP Socket 和 TCP Socket 不同的地方就是在服务器端处理多个客户端请求数据包的方式不同，UDP缺少对客户端连接请求的 Accept函数。

### 客户端
示例：
```go
package main

import (
	"fmt"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "用法：%s host:port", os.Args[0])
		os.Exit(1)
	}
	
	service := os.Args[1]
	udpAddr, err := net.ResolveUDPAddr("udp4", service)
	checkError(err)
	
	conn, err := net.DialUDP("udp", nil, udpAddr)
	checkError(err)
	
	_, err = conn.Write([]byte("anything"))
	checkError(err)
	
	var buf [512]byte
	n, err := conn.Read(buf[0:])
	checkError(err)
	
	fmt.Println(string(buf[0:n]))
	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "错误：", err.Error())
		os.Exit(1)
	}
}
```

### 服务器端
示例：
```go
package main

import (
	"fmt"
	"net"
	"os"
	"time"
)

func main() {
	service := ":8080"
	udpAddr, err := net.ResolveUDPAddr("udp4", service)
	checkError(err)
	
	conn, err := net.ListenUDP("udp", udpAddr)
	checkError(err)
	
	for {
		handleClient(conn)
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "错误：", err)
		os.Exit(1)
	}
}

func handleClient(conn *net.UDPConn) {
	var buf [512]byte
	_, addr, err := conn.ReadFromUDP(buf[0:])
	if err != nil {
		return 
	}
	
	daytime := time.Now().String()
	conn.WriteToUDP([]byte(daytime), addr)
}
```
