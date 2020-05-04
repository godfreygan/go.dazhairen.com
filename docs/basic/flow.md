# 流程控制

流程控制是每种编程语言控制逻辑走向和执行次序的重要部分，可以说是整个编程基础中的重要一环。
Go语言和其它编程语言有一些差别，Go语言没有do-while语句，不过for语句拥有更加广泛的含义与用途；另一方面switch语句也有一些差异，支持类型判断和初始化子语句等。


## 条件语句

### if-else判断

```go
package main

import "fmt"

func main() {
    a := 100
    if a < 20 {
        fmt.Println("a小于20")
    } else if a < 80 {
        fmt.Println("a小于80")
    } else {
        fmt.Println("a大于80")
    }
    fmt.Printf("a的值是：%d\n", a)
}
```
输出：
```text
a大于80
a的值是：100
```

## 选择语句

### switch语句

switch语句会根据初始化表达式得出一个值，然后根据case语句的条件，执行相应的代码块，最终返回特定内容。  
每个case被视为一种情况，只有当初始化语句的值符合case的条件语句时，case才会被执行。

```go
package main

import "fmt"

func main() {
    grade := "B"
    marks := 90
    switch marks {
    case 90:
        grade = "A"
    case 80:
        grade = "B"
    case 70,60:
        grade = "C"
    default:
        grade = "D"
    }
    fmt.Printf("你的成绩为%d，成绩等级为%s\n", marks, grade)
    
    switch {
    case grade == "A":
        fmt.Println("成绩优秀")
    case grade == "B":
        fmt.Println("表现良好")
    case grade == "C":
        fmt.Println("再接再厉")
    default:
        fmt.Println("成绩不合格")
    }
    fmt.Printf("你的成绩等级：%s\n", grade)
}
```
输出：
```text
你的成绩为90，成绩等级为A
成绩优秀
你的成绩等级：A
```


### select语句

select选择语句用于配合通道（channel）的读写操作，用于多个channel的并发读写操作。
switch是按顺序从上到下依次执行的，而select是随机选择一个case来判断，直到匹配到其中一个case。

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
有时会输出：
```text
第0次，from a
第1次，from b
第2次，from b
第3次，from a
第4次，from b
第5次，from b
第6次，from b
第7次，from a
第8次，from b
第9次，from b
```
上述，同时在a和b中选择，哪个有内容就从哪个读，由于channel的读写是阻塞操作，使用select语句可以避免单个channel的阻塞。


## 循环语句

### for的子语句

for语句可以根据指定的条件重复执行其内部的代码块，这个判断条件一般是由for关键字后面的子语句给出的。
for后面的三个子语句我们称为：初始化子语句、条件子语句和后置子语句，这三个不能颠倒顺序。
其中条件子语句可以放在循环内部，条件子语句会返回一个布尔值，true则执行代码块，false则跳出循环。

!!! note "注意"
for中的条件语句，如果没有写，或者没有中断情况，会造成程序死循环。开发人员应根据实际情况酌情开发。

```go
package main

import "fmt"

func main() {
    // for后跟三个子语句
    for i := 0; i < 5; i++ {
        fmt.Printf("i的值是：%d\n", i)
    }

    fmt.Printf("\n")

    // for后跟两个子语句
    for j := 0; j < 5; {
        j++
        fmt.Printf("j的值是：%d\n", j)
    }
    
    fmt.Printf("\n")

    // for后跟一个子语句
    a := 0
    b := 5
    for a < b {
        a++
        fmt.Printf("a的值是：%d\n", a)
    }
    
	fmt.Printf("\n")

	// for不跟子语句，在循环内设置条件和递增量
	m := 0
	n := 5
	for {
		m++
		if m > n {
			break
		}
		fmt.Printf("m的值是：%d\n", m)
	}
}
```
输出：
```text
i的值是：0
i的值是：1
i的值是：2
i的值是：3
i的值是：4

j的值是：1
j的值是：2
j的值是：3
j的值是：4
j的值是：5

a的值是：1
a的值是：2
a的值是：3
a的值是：4
a的值是：5

m的值是：1
m的值是：2
m的值是：3
m的值是：4
m的值是：5
```

### for-range语句

for-range是Go语言特有的一种迭代器，用于轮询数组、切片、字符串、map及通道（channel）。
格式：`for key,val := range coll {}`

```go
package main

import "fmt"

func main() {
    str := "abcz"
    for i, char := range str {
        fmt.Printf("字符串第%d个字符的值为%c\n", i, char)
    }
    
    // 忽略key
    for _, char := range str {
        fmt.Printf("字符的值为%c\n", i, char)
    }
    
    // 忽略val
    for i := range str {
        fmt.Println(i)
    }
}
```
输出：
```text
字符串第0个字符的值为a
字符串第1个字符的值为b
字符串第2个字符的值为c
字符串第3个字符的值为z
字符的值为a
字符的值为b
字符的值为c
字符的值为z
0
1
2
3
```


## 延迟语句

defer用于延迟调用指定函数，defer关键字只能出现在函数内部。defer有三大特点：

1. 只有当defer语句全部执行，defer所在函数才算真正结束执行

2. 当函数中有defer语句时，需要等待所有defer语句执行完毕，才会执行return语句

3. defer类似于栈操作，先进后出

因为defer的延迟特点，可以把defer语句用于回收资源、清理收尾等工作。不管回收代码放在哪里，反正都是最后执行。

```go
package main

import "fmt"

var i = 0
var j = 0

func print(i int) {
    fmt.Printf("print：%d\n", i)
}

func print2() {
    fmt.Printf("print2：%d\n", j)
}

func main() {
    // 循环压栈，并将参数传入方法体
    for ; i < 5; i++ {
        defer print(i)
    }
    
    defer fmt.Printf("\n")
    
    // 轮询之后，j变为5，此时再执行打印，会出现5，5，5，5，5
    for ; j < 5; j++ {
        defer print2()
    }
}
```
输出：
```text
print2：5
print2：5
print2：5
print2：5
print2：5

print：4
print：3
print：2
print：1
print：0
```


## 标签

Go语言有一个特殊的概念就是标签，可以给for、switch或select等流程控制代码块打上一个标签，配合标签标识符可以方便跳转到某一个地方继续执行，有助于提高编程效率。

!!! note ""
    注意：标签名称区分大小写，一般使用大写字母和数字组合。

```go
package main

import "fmt"

func main() {
    var i int
    for {
        fmt.Println(i)
        i++
        if i > 2 {
            goto BREAK
        }
    }
    
BREAK:
    fmt.Println("break")
}
```
输出：
```text
0
1
2
break
```
