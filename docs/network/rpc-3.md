# HTTP RPC

实现基于HTTP协议的RPC，数据编码采用Gob编码。

服务器端代码示例：

```go
package main

import (
	"errors"
	"fmt"
	"net/http"
	"net/rpc"
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
	if err != nil {
		fmt.Println(err.Error())
	}
	
	rpc.HandleHTTP()

	err = http.ListenAndServe(":8080", nil)
	if err != nil {
		fmt.Println(err.Error())
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
		fmt.Printf("用法：%s server\n", os.Args[0])
		os.Exit(1)
	}

	serverAddress := os.Args[1]

	client, err := rpc.DialHTTP("tcp", serverAddress +":8080")
	if err != nil {
		log.Fatal("Dial错误", err)
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
	fmt.Printf("Arith：%d/%d=%d...%d\n", args.A, args.B, quot.Quo, quot.Rem)
}
```
