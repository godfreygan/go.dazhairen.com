# channel结合select-case

## select-case

select-case 的组合可以使哪个管道就绪，就读取该管道数据并执行相应case的代码块。
select 会阻塞，直到条件分支中的某个可以继续执行，这时就会执行那个条件分支。当多个都准备好的时候，会随机选择一个。

```go
package main
import "fmt"
func main() {
    a := make(chan int, 1024)
    b := make(chan int, 1024)
    for i := 0; i < 10; i++ {
        fmt.Printf("第%d次，", i)
        a <- 1
        b <- 1
        select {
            case <- a:
                fmt.Println("from a")
            case <- b:
                fmt.Println("from b")
        }
    }
}
```


## select的随机性

```go
package main

import (
	"fmt"
	"time"
)

func receive(ch chan int) {
	for {
		<-ch
	}
}

func send(ch1, ch2, ch3 chan int) {
	for i := 0; i < 10; i++ {
		// sleep是为了保证所有的管道receiver都已阻塞等待数据
		time.Sleep(1000 * time.Millisecond)
		select {
		case ch1 <- i:
			fmt.Printf("send %d to ch1\n", i)
		case ch2 <- i:
			fmt.Printf("send %d to ch2\n", i)
		case ch3 <- i:
			fmt.Printf("send %d to ch3\n", i)
		}
	}
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	ch3 := make(chan int)
	go receive(ch1)
	go receive(ch2)
	go receive(ch3)
	send(ch1, ch2, ch3)
}
```


## select支持default

当所有case都阻塞的时候，执行default。可以在channel里使用select的default来告知channel已满。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	start := time.Now()
	var c1, c2 <-chan int
	select {
	case <-c1:
	case <-c2:
	default:
		fmt.Printf("In default after %v\n", time.Since(start))
	}
}
```
