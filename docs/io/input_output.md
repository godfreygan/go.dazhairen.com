# 输入/输出

Go语言将 I/O 操作存在于下面几个包中：

- io，为 I/O 提供基本的接口，其中比较重要的两个接口是：Reader和Writer接口。

- io/ioutil，提供一些比较常用、方便的 I/O 操作函数。

- fmt，实现格式化 I/O ，类似C语言中的 printf和scanf。

- bufio，实现带缓冲的 I/O ，封装于 io.Reader 和 io.Writer 对象，能够一定程度减少大块数据读写带来的开销。


## io：基本I/O接口

### Reader接口

Reader接口定义如下：

```text
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

io.Reader接口只有一个Read方法，这个方法接收一个 byte 类型的切片，并返回两个值，一个是读入的字节数n，另一个是错误err。

io.Reader接口有如下规则：

1）Read方法最多读取len(n)字节的数据，并保存到p；

2）Read方法返回读取到字节数以及任何发生的错误信息；

3）读取的字节数n要满足 0<=n<=len(p)；

4）读取的字节数 n<len(p) 时，表示读取的数据不足以填满p，这时方法会立即返回，而不会继续等待更多的数据；

5）读取过程中遇到错误，会返回读取的字节数n以及相应的错误err；

6）在底层输入流结束时，方法会返回 n>0 字节，但是遇到错误可能会中断（EOF），也可以返回nil；

7）在第6种情况下，再次调用 Read 方法，会返回 (0, EOF)；

8）调用Read方法时，如果 n>0 时，优先处理读入的数据，然后再处理错误err，EOF也要这样处理；

9）Read方法不鼓励返回 n=0 并且 err=nil 的情况（即，返回字节数n为空时不推荐把err设置为nil）


代码示例：

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

// 读取信息 num:读取的字节数量
func ReadFrom(reader io.Reader, num int)([]byte, error){
	//创建一个缓冲切片
	p := make([]byte, num)
	n, err := reader.Read(p)
	if n > 0 {
		return p[:n], nil
	}
	return p, err
}

func main(){
	data, _ := ReadFrom(strings.NewReader("hello golang"), 12)
	fmt.Println(data)
}
```


### Writer接口

Writer接口定义如下：
```text
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

同样的，io.Writer 接口也有着严格的规范要求：

1）Write方法向底层数据流写入 len(p) 字节的数据，这些数据来源于切片p；

2）返回被写入的字节数n，其中 0<=n<=len(p)；

3）如果 n<len(p)，则必须返回一些非nil的err（和Read一样）；

4）如果中途出现错误，也要返回非nil的err；

5）Write方法绝对不能修改切片p和其中的数据


代码示例：

```go
package main

import "fmt"

type chanWriter struct {
	ch chan byte		// ch 实际上就是目标资源
}

func newChanWriter() *chanWriter {
	return &chanWriter{make(chan byte, 1024)}
}

func (w *chanWriter) Chan() <-chan byte {
	return w.ch
}

func (w *chanWriter) Write(p []byte) (int, error) {
	n := 0
	// 遍历输入数据，按字节写入目标资源
	for _, b := range p {
		w.ch <- b
		n++
	}
	return n, nil
}

func (w *chanWriter) Close() error {
	close(w.ch)
	return nil
}

func main() {
	writer := newChanWriter()
	go func() {
		defer writer.Close()
		writer.Write([]byte("Golang"))
		writer.Write([]byte("PHP"))
	}()
	for c := range writer.Chan() {
		fmt.Printf("%c", c)
	}
	fmt.Println()
}
```


## fmt：格式化I/O

fmt包实现了格式化I/O函数，类似于C语言中的printf和scanf。格式占位符衍生于C语言，但在Go语言中更为简单。


### Print序列函数

Print序列函数包括：Fprint/Fprintf/Fprintln/Sprint/Sprintf/Sprintln/Print/Printf/Println。

Fprint/Fprintf/Fprintln 函数的第一个参数接收一个 io.Writer 类型，会将内容输出到 io.Writer 中去。
Print/Printf/Println 函数是将内容输出到标准输出中。
Sprint/Sprintf/Sprintln 函数是格式化内容为string类型，而并不是输出到某处，需要格式化字符串并返回时，可以用Stdout函数。

在这三组函数中，Sprintf/Fprintf/Printf 函数通过指定的格式输出或格式化内容；Sprint/Fprint/Print 函数只是使用默认的格式输出或格式化内容；Sprintln/Fprintln/Println 函数使用默认的格式输出或格式化内容，同时会在最后加上"换行符"。

代码示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	//Print 系列
	fmt.Printf("Printf: %s\n", "按格式打印，空格换行需自行添加")
	fmt.Print("Print: ", "默认格式打印，非字符串自动加空格，换行需自行添加\n")
	fmt.Println("Println:", "默认格式打印，所有元素自动加空格，自动添加换行")

	//Sprint系列
	str := fmt.Sprintf("Sprintf: %s\n", "按格式打印，空格换行需自行添加")
	fmt.Print(str)
	str = fmt.Sprint("Sprint: ", "默认格式打印，非字符串自动加空格，换行需自行添加\n")
	fmt.Print(str)
	str = fmt.Sprintln("Sprintln:", "默认格式打印，所有元素自动加空格，自动添加换行")
	fmt.Print(str)

	//Fprint系列
	fmt.Fprintf(os.Stdout, "Fprintf: %s\n", "按格式打印，空格换行需自行添加")
	fmt.Fprint(os.Stdout, "Fprint: ", "默认格式打印，非字符串自动加空格，换行需自行添加\n")
	fmt.Fprintln(os.Stdout, "Fprintln:", "默认格式打印，所有元素自动加空格，自动添加换行")
}
```

输出：
```text
Printf: 按格式打印，空格换行需自行添加
Print: 默认格式打印，非字符串自动加空格，换行需自行添加
Println: 默认格式打印，所有元素自动加空格，自动添加换行
Sprintf: 按格式打印，空格换行需自行添加
Sprint: 默认格式打印，非字符串自动加空格，换行需自行添加
Sprintln: 默认格式打印，所有元素自动加空格，自动添加换行
Fprintf: 按格式打印，空格换行需自行添加
Fprint: 默认格式打印，非字符串自动加空格，换行需自行添加
Fprintln: 默认格式打印，所有元素自动加空格，自动添加换行
```


### Scan序列函数

该序列函数和Print序列函数相对应，包括：Fscan/Fscanf/Fscanln/Sscan/Sscanf/Sscanln/Scan/Scanf/Scanln。

Fscan/Fscanf/Fscanln 函数的第一个参数接收一个 io.Reader 类型，从其中读取内容并赋值给相应的实参。
Scan/Scanf/Scanln 函数是从标准输入获取内容。
Sscan/Sscanf/Sscanln 函数是从字符串中获取内容。

代码示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	//Scanf系列
	var a, b, c int
	fmt.Scanf("%d", &a) 				// 输入时用任意空格或tab隔开，不能换行
	fmt.Printf("ouput 1 => %d\n", a)
	fmt.Scanf("%d", &a) 				// 输入时要用逗号隔开，要跟引号内保持一致,其他字符均可
	fmt.Printf("ouput 2 => %d\n", a)
	fmt.Scan(&a) 							// 输入时用任意空格、TAB或者回车隔开，直到达到参数数量为止
	fmt.Printf("ouput 3 => %d\n", a)
	fmt.Scanln(&a) 							// 输入时用任意空格、TAB隔开，不能用换行，换行即终止
	fmt.Printf("ouput 4 => %d\n", a)

	input := "12\n\n25\n30"
	fmt.Sscanf(input, "%d\n\n%d\n%d", &a, &b, &c) 	//input与%d串保持一致，若%d之间无任何字符，则input之间可为任意空格或TAB
	fmt.Printf("ouput 5 => \n%d\n%d\n%d\n", a, b, c)
	fmt.Sscan(input, &a, &b, &c) 							//input中可用任意空格、tab、回车间隔，直到到达参数数量为止
	fmt.Printf("ouput 6 => \n%d\n%d\n%d\n", a, b, c)
	input2 := "23 45 67"
	fmt.Sscanln(input2, &a, &b, &c) 						//input中只能用任意空格、tab间隔，不能换行，换行即终止
	fmt.Printf("ouput 7 => \n%d\n%d\n%d\n", a, b, c)

	fmt.Fscanf(os.Stdin, "%d", &a) 					//输入时要用逗号隔开，要跟引号内保持一致,其他字符均可。无间隔时默认输入时用任意空格或tab隔开，不能换行
	fmt.Printf("ouput 8 => %d\n", a)
	fmt.Fscan(os.Stdin, &a) 								//输入时用任意空格、TAB或者回车隔开，直到达到参数数量为止
	fmt.Printf("ouput 9 => %d\n", a)
	fmt.Fscanln(os.Stdin, &a) 								//输入时用任意空格、TAB隔开，不能用换行，换行即终止
	fmt.Printf("ouput 10 => %d\n", a)
}
```

输入输出：
```text
输入：10
ouput 1 => 10
输入：11
ouput 2 => 11
输入：12
ouput 3 => 12
输入：13
ouput 4 => 13
ouput 5 => 
12
25
30
ouput 6 => 
12
25
30
ouput 7 => 
23
45
67
输入：14
ouput 8 => 14
输入：15
ouput 9 => 15
输入：16
ouput 10 => 16
```


### Stringer接口

Stringer 接口定义如下：

```text
type Stringer interface {
    String() string
}
```
根据Go语言中实现接口的定义，一个类型只要有 String() 方法，就说它实现了 Stringer接口。如果格式化输出某种类型的值，只要它实现了 String() 方法，那么就会调用 String() 方法进行处理。

示例1：
```go
package main

import "fmt"

type People struct {
	name    string
	sex     string
	age     int
}

func (p *People) String() string {
	return fmt.Sprintf("name：%s，sex：%s，age：%d", p.name, p.sex, p.age)
}

func main() {
	var p1 *People = &People{name: "张三", sex: "男", age: 18}
	fmt.Printf("%s\n", p1)
	fmt.Println(p1)
	fmt.Printf("%v", p1)
}
```

输出：
```text
name：张三，sex：男，age：18
name：张三，sex：男，age：18
name：张三，sex：男，age：18
```


示例2：
```go
package main

import "fmt"

type People struct {
	name    string
	sex     string
	age     int
}

func (p People) String() string {
	return fmt.Sprintf("name：%s，sex：%s，age：%d", p.name, p.sex, p.age)
}

func main() {
	var p1 People = People{name: "李四", sex: "女", age: 16}
	fmt.Printf("%s\n", p1)
	fmt.Println(p1)
	fmt.Printf("%v", p1)
}
```

输出：
```text
name：李四，sex：女，age：16
name：李四，sex：女，age：16
name：李四，sex：女，age：16
```

!!! note ""
    Go语言中的Stringer接口和Java中的T哦String方法类似！


### Formatter接口

Formatter接口定义如下：

```text
type Formatter interface {
    Format(f State, c rune)
}
```
Formatter接口由带有定制的格式化占位符的值所实现。Format的实现可调用 Sprintf 或 Fprintf(f) 等函数来生成输出。

也就是说，我们可以通过实现 Formatter接口 来自定义输出格式（自定义占位符）。

示例：
```go
package main

import "fmt"

type People struct {
	name    string
	sex     string
	age     int
}

func (p *People) String() string {
	return fmt.Sprintf("name：%s，sex：%s，age：%d\n", p.name, p.sex, p.age)
}

func (p *People) Format(f fmt.State, c rune) {
	if c == 'L' {
		f.Write([]byte(p.String()))
		f.Write([]byte("这是People自定义的输出格式"))
	} else {
		f.Write([]byte(fmt.Sprintln(p.String())))
	}
}

func main() {
	var p *People = &People{"王五", "男", 20}
	fmt.Printf("%L", p)
}
```

输出：
```text
name：王五，sex：男，age：20
这是People自定义的输出格式
```


### GoStringer接口

GoStringer接口的定义如下：

```text
type GoStringer interface {
    GoString() string
}
```
该接口定义了类型的Go语言语法类型。用于打印（Printf）格式化占位符为 %#v 的值。

示例：
```go
package main

import (
	"fmt"
	"strconv"
)

type People struct {
	name    string
	sex     string
	age     int
}

type Student struct {
	People
}

func (s *Student) GoString() string {
	return "&Student{姓名："+ s.name +"，性别："+ s.sex +"，年龄："+ strconv.Itoa(s.age) +"}"
}

func main() {
	p := &People{"张三", "男", 10}
	fmt.Printf("%#v", p)

	fmt.Println()

	s := &Student{People{"李四", "女", 8}}
	fmt.Printf("%#v", s)
}
```

输出：
```text
&main.People{name:"张三", sex:"男", age:10}
&Student{姓名：李四，性别：女，年龄：8}
```
一般情况下不需要实现该接口。


## 文本处理

Go语言标准库有几个包专门用于处理文本：

- strings：字符串操作，这个包提供了很多操作字符串的简单函数，通常一般的字符串操作需求都可以在这个包中找到

- bytes：byte slice 便利操作，在Go语言中string是内置类型，同时它与普通的slice类型有着相似的性质，均可以进行切片操作

- strconv：字符串和基本数据类型之间转换，因为在Go语言中，没有隐式类型转换，字符串类型和int、float、bool等类型之间不能直接使用基本数据类型那套方法转换，需要使用这个包的方法实现转换

- regexp：正则表达式，文本字符串处理离不开正则表达式，这个提供了正则表达式的功能

- unicode：Unicode编码、UTF-8/16编码
