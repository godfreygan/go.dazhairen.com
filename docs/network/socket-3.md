# Dial()函数

Go语言中 Socket编程 的API都在 net包 中，GO语言提供了 Dial函数 来连接服务器，使用Listen监听，Accept接收连接，所以Go语言的网络编程和其它编程语言一样（PHP、Java）一样有着相似的API。

Go语言标准库对传统的 Socket编程 过程进行了抽象和封装，无论是使用什么协议，建立什么形式的连接，都只需调用net.Dial（）即可。

Dial() 函数，几种常见协议的调用方式：

!!! note "TCP链接"

    conn, err := net.Dial("tcp", "127.0.0.1:8080")

!!! note "UDP链接"

    conn, err := net.Dial("udp", "127.0.0.1:8080")

!!! note "ICMP链接（使用协议名称）"

    conn, err := net.Dial("ip4:icmp", "dazhairen.com")

!!! note "ICMP链接（使用协议编号）"

    conn, err := net.Dial("ip4:1", "127.0.0.1")

当下，Dial()函数 支持如下几种网络协议：TCP、TCP4（仅限ipv4）、TCP6（仅限ipv6）、UDP、UDP4（仅限ipv4）、UDP6（仅限ipv6）、IP、IP4（仅限ipv4）、IP6（仅限ipv6）等。


在建立好链接后，就可以进行数据的发送和接收。发送数据，使用conn的 Write()，接收数据，使用conn的 Read()。

示例：
```go
package main

import (
	"bytes"
	"fmt"
	"io"
	"net"
	"os"
)

func main() {
	if len(os.Args) != 2 {
		fmt.Println("用法：", os.Args[0], "hostname")
		os.Exit(1)
	}
	
	service := os.Args[1]
	
	conn, err := net.Dial("ip4:icmp", service)
	checkError(err)
	
	var msg [512]byte
	msg[0] = 8
	msg[1] = 0
	msg[2] = 0
	msg[3] = 0
	msg[4] = 0
	msg[5] = 13
	msg[6] = 0
	msg[7] = 37
	len := 8
	check := checkNum(msg[0:len])
	msg[2] = byte(check >> 8)
	msg[3] = byte(check & 255)
	
	_, err = conn.Write(msg[0:len])
	checkError(err)
	
	_, err = conn.Read(msg[0:])
	checkError(err)
	
	fmt.Println("取得响应")
	if msg[5] == 13 {
		fmt.Println("Identifier匹配")
	}
	if msg[7] == 37 {
		fmt.Println("Sequence匹配")
	}
	os.Exit(0)
}

func checkNum(msg []byte) uint16 {
	sum := 0
	
	for n := 1; n < len(msg) - 1; n+=2 {
		sum += int(msg[n]) * 256 + int(msg[n+1])
	}
	
	sum = (sum >> 16) + (sum & 0xffff)
	sum += (sum >> 16)
	var answer uint16 = uint16(^sum)
	return answer
}

func checkError(err error) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "错误：%s", err.Error())
		os.Exit(1)
	}
}

func readFully(conn net.Conn) ([]byte, error) {
	defer conn.Close()
	result := bytes.NewBuffer(nil)
	var buf [512]byte
	for {
		n, err := conn.Read(buf[0:])
		result.Write(buf[0:n])
		if err != nil {
			if err == io.EOF {
				break
			}
			return nil, err
		}
	}
	return result.Bytes(), nil
}
```
