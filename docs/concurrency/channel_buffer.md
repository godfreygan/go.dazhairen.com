# 通道的缓存机制

channel可以带buffer：`make(chan TypeName, length)`，这里的length表示数量，而非具体的内存容积。

```text
make(chan string, 2)
```
说明：

- 表示可以容纳2个string的内容，比如"abcdefg"和"h"这样2个string，而不是指只能存"a"和"b"。

- 当不设置length时候即为buffer=0


特点：

- 往chan发送数据时候，若buffer没满的时候，则发送数据即刻成功，不会被阻塞。当buffer满了就会被阻塞

- 从chan接受数据时候，若buffer是空的则会被阻塞，当buffer不是空的时候则即刻完成，不会被阻塞


通道长度，示例：
```go
package main

import "fmt"

func main() {
	c := make(chan int, 100)

	fmt.Println("1.len:", len(c))

	for i := 0; i < 34; i++ {
		c <- 0
	}

	fmt.Println("2.len:", len(c))

	<-c

	<-c

	fmt.Println("3.len:", len(c))
}
```