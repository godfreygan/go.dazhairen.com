# RPC接口

Go语言的 net/rpc 很灵活，在数据传输前后实现了编码解码器的接口定义。因此，我们可以自定义数据的传输方式以及RPC服务器端与客户端的交互行为。

RPC提供的编码解码器接口如下：
```text
type ClientCodec interface {
    WtiteRequest(*Request, interface{}) error
    ReadResponseHeader(*Response) error
    ReadResponseBody(interface{}) error
    Close() error
}

type ServerCodec interface {
    ReadRequestHeader(*Request) error
    ReadResponseBody(interface{}) error
    WriteResponse(*Response, interface{}) error
    Close() error
}
```

说明：

接口ClientCodec定义了RPC客户端如何在一个RPC会话中发送请求和读取响应。  
客户端程序通过 WriteRequest() 方法将一个请求写入 RPC 连接中，并通过 ReadResponseHeader() 和 ReadResponseBody() 读取服务器端的响应信息。
当整个流程执行完毕后，再通过 Close() 方法关闭连接。

接口ServerCodec定义了RPC服务器端如何在一个RPC会话中接收请求和发送响应。
服务器端程序通过 ReadRequestHeader() 和 ReadRequestBody() 方法从一个 RPC 连接中读取请求信息，然后通过 WriteResponse() 方法向这个连接中的 RPC 客户端发送响应。
当整个流程执行完毕后，通过 Close() 方法关闭连接。


通过实现这两个接口，可以自定义数据传输前后的编码解码方式，不再局限于 Gob。同时，还可以自定义RPC服务器端和客户端的交互行为。
