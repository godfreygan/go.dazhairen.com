# channel

channel是有类型的管道，可以用channel操作符 `<-` 对其发送或者接收值。

## 创建channel

一般定义格式：`var chanName chan ElementType`：
```text
var ch chan int
```

使用内置函数 make() 定义：
```text
ch := make(chan int)
```

注意：管道必须指定类型，比如这里是int，即往管道里传送的数据只能是int类型。当然可以是interface{}这种空接口方式，这样就可以传送各种数据类型了

```go
package main

import "fmt"

func main() {
	var ch chan int
	fmt.Printf("%T %v\n", ch, ch)
	ch2 := make(chan int)
	fmt.Printf("%T %v\n", ch2, ch2)
}
```
输出：
```text
chan int <nil>
chan int 0xc000080060
```


## 管道方向

```text
ch <- value     // Send value to channel ch.
value := <-ch   // Receive from ch, and assign value to value.
```

“箭头”就是数据流的方向

!!! note ""
    注意接受时候 "<-" 和 chan变量 之间没有空格。即使有空格也不会报错，最好还是不空格比较好


## 基本使用

```go
package main

import "fmt"

func foo(i int, c chan int) {
    c <- i
    fmt.Println("send:", i)
}

func main() {
    c := make(chan int)
    go foo(8, c)
    res := <-c
    fmt.Println("receive:", res)
}
```
输出：
```text
send: 8
receive: 8
```

发送和接收数据应当在并行线上，而不能是串行的，因为发送和接收都会阻塞，如果串行，就会死锁（就是一个一直阻塞在那等对端）。  
但go在执行时会报错，即使编译通过，比如：

```go
package main

func main() {
	ch := make(chan int)
	ch <- 0
	<-ch
}
```
报错：
```text
fatal error: all goroutines are asleep - deadlock!
```
