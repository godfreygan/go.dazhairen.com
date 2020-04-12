# TCP RPC

实现基于TCP协议的RPC，数据编码采用Gob编码。

服务器端代码示例：

```go
package main

import (
	"errors"
	"fmt"
	"net"
	"net/rpc"
	"os"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
	*reply = args.A * args.B
	return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
	if args.B == 0 {
		return errors.New("除数不能为0")
	}
	quo.Quo = args.A / args.B
	quo.Rem = args.A % args.B
	return nil
}

func main() {
	arith := new(Arith)
	err := rpc.Register(arith)
	checkError(err)
	
	tcpAddr, err := net.ResolveTCPAddr("tcp", "8080")
	checkError(err)
	
	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)
	
	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		rpc.ServeConn(conn)
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Println("错误：", err)
		os.Exit(1)
	}
}
```

客户端代码示例：

```go
package main

import (
	"fmt"
	"log"
	"net/rpc"
	"os"
)

type Args struct {
	A, B int
}

type Quotient struct {
	Quo, Rem int
}

func main() {
	if len(os.Args) != 2 {
		fmt.Printf("用法：%s server:port\n", os.Args[0])
	    os.Exit(1)
	}
	
	service := os.Args[1]
	
	client, err := rpc.Dial("tcp", service)
	if err != nil {
		log.Fatal("Dial错误：", err)
	}
	
	args := Args{17, 8}
	var reply int
	err = client.Call("Arith.Multiply", args, &reply)
	if err != nil {
		log.Fatal("arith错误：", err)
	}
	
	fmt.Printf("Arith：%d*%d=%d", args.A, args.B, reply)
	
	var quot Quotient
	err = client.Call("Arith.Divide", args, &quot)
	if err != nil {
		log.Fatal("arith错误：", err)
	}
	fmt.Printf("Arith：%d/%d=%d...%d", args.A, args.B, quot.Quo, quot.Rem)
}
```