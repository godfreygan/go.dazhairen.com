# TCP Socket

网络请求：
作为客户端来说，通过向远端某台机器的某个端口发送一个请求，然后得到所监听的远端机器的响应信息。
作为服务器来说，需要把服务绑定到某个指定端口，并且在此端口上监听，当有客户端发来请求是，读取信息并处理请求，最后返回响应内容。

### TCP客户端
Go语言通过 net包 中的DialTCP函数来建立一个TCP连接，并返回一个TCPConn类型的对象，当连接建立时，服务器端也创建一个同类型的对象。此时，客户端和服务器端通过各自拥有的TCPConn对象进行数据交换。

示例：
```go
package main

import (
	"fmt"
	"io/ioutil"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "用法：%s hostname：port", os.Args[0])
		os.Exit(1)
	}
	
	service := os.Args[1]
	tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
	checkError(err)
	
	conn, err := net.DialTCP("tcp", nil, tcpAddr)
	checkError(err)
	
	_, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
	checkError(err)
	
	result, err := ioutil.ReadAll(conn)
	checkError(err)
	
	fmt.Println(string(result))
	os.Exit(0)
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "错误：%s", err.Error())
		os.Exit(1)
	}
}
```


### TCP服务器端

通过 net包 创建一个服务器端程序，在服务器端绑定到一个指定的端口，并监听该端口。当有客户端请求到达时，就可以接收来自客户端发来的请求。

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
	address := net.TCPAddr{
		IP:     net.ParseIP("127.0.0.1"),
		Port:   8000,
	}
	
	// 创建TCP4服务器端监听器
	listener, err := net.ListenTCP("tcp4", &address)
	if err != nil {
		log.Fatal(err)
	}
	
	for {
		conn, err := listener.AcceptTCP()
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println("远程地址：", conn.RemoteAddr())
		go echo(conn)
	}
}

func echo(conn *net.TCPConn) {
	// 5秒请求一次
	tick := time.Tick(5 * time.Second)
	for now := range tick {
		n, err := conn.Write([]byte(now.String()))
		if err != nil {
			log.Println(err)
			conn.Close()
			return 
		}
		fmt.Printf("send %d bytes to %s\n", n, conn.RemoteAddr())
	}
}
```