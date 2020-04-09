# 并发编程概念

Go语言里的并发指的是能让某个函数独立于其它函数运行的能力。当一个函数创建为协程（goroutine）时，Go语言会将其视为一个独立的工作单元，这个单元会被调度到可用的逻辑处理器上执行。

## 并发、并行与协程的概念

关于协程本身的概念，详见[进程、线程、协程](/other/concurrent/#_5)


## Go协程

> goroutine（go协程）是由Go runtime管理的轻量级线程。


## 启动Go协程

启动Go协程方法：只需要在调用函数时候在前面加上go即可，如:
```text
go Call("Hi, Tom")
```

**使用协程和不使用协程的区别**
示例：
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Printf("main function begin：%v\n", time.Now().Unix())

	//longWait()
	//shortWait()
	
	//使用协程：
	go longWait()
	go shortWait()
	
	fmt.Printf("main function sleep：%v\n", time.Now().Unix())
	time.Sleep(10 * 1e9)
	fmt.Printf("main function end：%v\n", time.Now().Unix())
}
func longWait() {
	fmt.Printf("longWait function begin：%v\n", time.Now().Unix())
	time.Sleep(5 * 1e9)
	fmt.Printf("longWait function end：%v\n", time.Now().Unix())
}

func shortWait() {
	fmt.Printf("shortWait function begin：%v\n", time.Now().Unix())
	time.Sleep(1e9)
	fmt.Printf("shortWait function end：%v\n", time.Now().Unix())
}
```
输出：
```text
main function begin：1586445831
main function sleep：1586445831
longWait function begin：1586445831
shortWait function begin：1586445831
shortWait function end：1586445832
longWait function end：1586445836
main function end：1586445841
```