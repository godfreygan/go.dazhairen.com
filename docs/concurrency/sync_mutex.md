# 互斥锁: sync.Mutex

## 简介

用于主动控制 Mutex 类型的变量，或者将 Mutex 类型作为 struct 的元素的变量在同一时间只被一个 Routine 访问（即执行 Lock() 方法的代码块）。  
Mutex 带有2个方法：Lock() 和 Unlock()。

!!! note ""
    互斥锁不区分读和写，即无论是print打印还是写操作都是互斥的

示例：
```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mutex sync.Mutex
	fmt.Printf("%+v\n", mutex)

	mutex.Lock()
	fmt.Printf("%+v\n", mutex)

	mutex.Unlock()
	fmt.Printf("%+v\n", mutex)
}
```
输出：
```text
{state:0 sema:0}
{state:1 sema:0}
{state:0 sema:0}
```


## 使用

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string, id int) {
	c.mux.Lock()
	fmt.Printf("%d. Inc lock.\n", id)
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
	fmt.Printf("%d. Inc unlock.\n", id)
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	fmt.Println("Value lock.")
	// Lock so only one goroutine at a time can access the map c.v.
	defer fmt.Println("Value unlock.")
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 10; i++ {
		go c.Inc("somekey", i)
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```
输出：
```text
3. Inc lock.
3. Inc unlock.
1. Inc lock.
1. Inc unlock.
0. Inc lock.
0. Inc unlock.
6. Inc lock.
6. Inc unlock.
4. Inc lock.
4. Inc unlock.
5. Inc lock.
5. Inc unlock.
8. Inc lock.
8. Inc unlock.
7. Inc lock.
7. Inc unlock.
9. Inc lock.
9. Inc unlock.
2. Inc lock.
2. Inc unlock.
Value lock.
Value unlock.
10
```

说明：
- 每次输出结果也不太一样：0-9的顺序不一致

- 在锁定互斥锁之后紧接着就用 defer 语句来保证该互斥锁的及时解锁，因为 defer 是先进后出

- Inc 方法和 Value 方法的 receiver 是指针，因为Mutex锁对应的就是一个具体的变量（可以是struct，也可以是其他普通类型）


## 主动控制互斥锁

互斥锁的使用是主动控制互斥锁。比如即使一个 Routine 里用了 Lock()，但在另一个 Routine 可以不理会这个锁就能访问这个 struct，只需要不调用 Lock() 就行。

示例：
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	time.Sleep(3 * time.Second)
	c.v[key]++
	c.mux.Unlock()
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	go c.Inc("somekey")

	time.Sleep(1 * time.Second)
	c.v["somekey"]++
	fmt.Println(c.v["somekey"])

	time.Sleep(3 * time.Second)
	fmt.Println(c.v["somekey"])
}
```
输出：
```text
1
2
```

`c.v["somekey"]++`之前，是已经锁了，本应该等2-3秒，才能输出，但是从程序开始运行，只过了1秒就输出了，说明Lock只是一种人为的互斥，是一种协议，并不是强制。

不加锁的效果：
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string, id int) {
//	c.mux.Lock()
	fmt.Printf("%d. Inc lock.\n", id)
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
//	c.mux.Unlock()
	fmt.Printf("%d. Inc unlock.\n", id)
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	fmt.Println("Value lock.")
	// Lock so only one goroutine at a time can access the map c.v.
	defer fmt.Println("Value unlock.")
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 2; i++ {
		go c.Inc("somekey", i)
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```
输出：
```text
1. Inc lock.
1. Inc unlock.
0. Inc lock.
0. Inc unlock.
Value lock.
Value unlock.
2
```


## 已锁定的Mutex与特定的goroutine无关联

已经锁定的 Mutex 并不与特定的 goroutine 相关联，这样可以利用一个 goroutine 对其加锁，再利用其它 goroutine 对其解锁。
示例：
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type MyStruct struct {
	v   int
	mux sync.Mutex
}

func (s *MyStruct) Lock() {
	s.mux.Lock()
}

func (s *MyStruct) Unlock() {
	s.mux.Unlock()
}

func main() {
	s := MyStruct{v: 0}
	s.v = 1
	fmt.Printf("%+v\n", s)

	go s.Lock()
	time.Sleep(1 * time.Second)
	fmt.Printf("%+v\n", s)

	go s.Unlock()
	time.Sleep(1 * time.Second)
	fmt.Printf("%+v\n", s)
}
```
输出：
```text
{v:1 mux:{state:0 sema:0}}
{v:1 mux:{state:1 sema:0}}
{v:1 mux:{state:0 sema:0}}
```

上述代码中，可以一个routine里锁定，另一个routine里解锁。因为锁只和具体变量关联，和routine无关，只要这个变量是共享的，比如通过指针传递，或者全局变量都可以。

虽然互斥锁可以被直接的在多个 goroutine 之间共享，但还是强烈建议把对同一个互斥锁的成对的锁定和解锁操作放在同一个层次的代码块中。
