# 接口进阶


## 接口类型与约定

接口类型实际上是描述了一系列方法的集合，实现了这些方法的具体类型就是这个接口类型的实例。

io.Writer类型是用得比较多的一个接口，它提供了所有的类型写入 byte 的抽象，有文件类型、内存缓冲区、网络链接、HTTP客户端、压缩工具、哈希等。  
io包中还定义了其它的接口类型，io.Reader可以代表任意可读取 byte 的类型，io.Closer可以是任意可关闭的类型，如一个文件或网络链接：
```go
package io
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interfase {
    Close() error
}
```

有些接口类型，通过组合已有的接口来定义，如下：
```
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

### 动态类型

一个接口类型的变量（如 **varI** ）中可以包含任何类型的值，必须有一种方式来检测它的动态类型。通常可以使用 类型断言 来检测变量 **varI** 是否为某种数据类型 **T** 。
格式：`v := varI.(T)`

示例：
```go
package main

import (
	"fmt"
	"math"
)

type Square struct {
	side float64
}

type Circle struct {
	radius float64
}

type Shaper interface {
	Area() float64
}

func main() {
	var areaIntf Shaper

	sq1 := new(Square)
	sq1.side = 5

	areaIntf = sq1
	// 判断areaIntf的类型是否是Square
	if t, ok := areaIntf.(*Square); ok {
		fmt.Printf("areaIntf的类型是：%T\n", t)
	}

	if u, ok := areaIntf.(*Circle); ok {
		fmt.Printf("areaIntf的类型是：%T\n", u)
	} else {
		fmt.Println("areaIntf 不含类型为 Circle 的变量")
	}
}

func (sq *Square) Area() float64 {
	return sq.side * sq.side
}

func (ci *Circle) Area() float64 {
	return ci.radius * ci.radius * math.Pi
}
```
输出：
```text
areaIntf的类型是：*main.Square
areaIntf 不含类型为 Circle 的变量
```

### 类型判断

接口变量的类型也可以使用 type-switch 来检测。
示例：
```go
package main

import (
	"fmt"
	"math"
)

type Square struct {
	side float64
}

type Circle struct {
	radius float64
}

type Shaper interface {
	Area() float64
}

func main() {
	var areaIntf Shaper

	sq1 := new(Square)
	sq1.side = 5

	areaIntf = sq1
	switch t:= areaIntf.(type) {
    case *Square:
        fmt.Printf("Square类型的 %T 值为：%v\n", t, t)
    case *Circle:
        fmt.Printf("Circle类型的 %T 值为：%v\n", t, t)
    case nil:
        fmt.Printf("nil值\n")
    default:
        fmt.Printf("未知类型 %T\n", t)
	}
}

func (sq *Square) Area() float64 {
	return sq.side * sq.side
}

func (ci *Circle) Area() float64 {
	return ci.radius * ci.radius * math.Pi
}
```
输出：
```text
Square类型的 *main.Square 值为：&{5}
```


## 嵌套接口

一个接口可以包含一个或多个其它接口，这相当于直接将这些内嵌接口的方法列举在外层接口中。  
例如，接口File包含了ReadWriter和Lock的所有方法，它还有一个Close()方法：
```
type ReadWriter interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}
```


## 接口赋值

接口赋值在Go语言中分为如下两种情况：

1. 将对象实例赋值给接口

2. 将一个接口赋值给另一个接口



## 接口组合

和其它类型组合一样，Go语言同样支持接口组合。可以认为接口组合是类型匿名组合的一个特定场景，只不过接口只包含方法，而不包含任何成员变量。
```
// ReadWriter接口将基本的Read和Write方法组合起来
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type ReadWriter interface {
    Reader
    Writer
}

// 上述写法，等同于下面这个
type ReadWriter interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```