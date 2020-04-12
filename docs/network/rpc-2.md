# Go RPC

Go语言标准包已经提供了对RPC的支持，支持三个级别的RPC：TCP、HTTP、JSONRPC。Go语言的RPC包内部采用Gob来编码，因此它只支持Go语言开发的服务器与客户端之间的交互。

Go RPC 的函数只有符合一定要求才能被远程访问，不然会被忽略。要求如下：

- 函数必须是导出（首字母大写）的

- 必须要有两个导出类型的参数

- 第一个参数是接收的参数，第二个参数是返回给客户端的参数，且第二个参数必须是指针

- 函数还要有一个error返回值

示例：
```text
func (e *E) MethodName(argType E1, replyType *E2) error 
```

!!! note ""
    上面示例中，如果是TCP、HTTP方式，则E、E1和E2类型必须要能被 encoding/gob 包编码解码。
