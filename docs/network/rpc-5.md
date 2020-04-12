# JSON RPC

JSON RPC 是数据编码采用JSON，而不是Gob编码。

服务器端代码示例：

```go
package main

import (
	"errors"
	"fmt"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
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

func (t *Arith) Divide(args *Args, quot *Quotient) error {
	if args.B == 0 {
		return errors.New("除数不能为0")
	}
	quot.Quo = args.A / args.B
	quot.Rem = args.A % args.B
	return nil
}

func main() {
	arith := new(Arith)
	rpc.Register(arith)

	tcpAddr, err := net.ResolveTCPAddr("tcp", ":8080")
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	for {
		conn, err := listener.Accept()
		if err != nil {
			continue
		}
		jsonrpc.ServeConn(conn)
	}
}

func checkError(err error) {
	if err != nil {
		fmt.Println("错误：", err.Error())
		os.Exit(1)
	}
}
```

客户端示例代码：

```go
package main

import (
	"fmt"
	"log"
	"net/rpc/jsonrpc"
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
    	fmt.Printf("用法：%s server:port", os.Args[0])
    	os.Exit(1)
    }
    service := os.Args[1]
    
    client, err := jsonrpc.Dial("tcp", service)
    if err != nil {
    	log.Fatal("Dial错误：", err)
    }
    
    args := Args{17, 8}
    var reply int
    err = client.Call("Arith.Multiply", args, &reply)
    if err != nil {
    	log.Fatal("arith错误：", err)
    }
    fmt.Printf("Arith：%d*%d=%d\n", args.A, args.B, reply)
    
    var quot Quotient
    err = client.Call("Arith.Divide", args, &quot)
    if err != nil {
    	log.Fatal("arith错误：", err)
    }
    fmt.Printf("Arith：%d/%d=%d...%d", args.A, args.B, quot.Quo, quot.Rem)
}
```
