# 函数进阶


## 参数传递机制

Go语言的参数传递分为“值传递”和“引用传递”。

GO语言默认按值传递，传递的是参数的副本，函数接收参数副本后，使用变量的过程中可能对副本的值进行更改，但不会影响原来的变量。  
换句话说，调用函数时修改参数的值，不会影响原来的参数的值，因为数值的变化只作用在副本上。

如果要让函数直接修改参数的值，而不是对参数的副本进行修改，就需要将参数的地址（变量名前面添加&符号，比如 &variable ）传递给函数，这就是按“引用传递”，此时传递给函数的是一个指针。

如果把指针传递给函数，指针的值（一个地址）就会被复制，但指针的值指向的地址上的那个值不会被复制（被复制的是指针，但是两个指针指向同一个实际的值）。  
如此，修改这个指针的值，实际上意味着这个值所指向的地址上的值被修改了（指针也是变量，有自己的地址和值，指针的值指向一个变量的地址）。

示例：
```go
package main

import "fmt"

func main() {
    x := 3
    fmt.Println("x=", x, ", &x=", &x)
    
    y := add(x)                     // 执行add函数，但实际上修改的是x的副本
    fmt.Println("x=", x, ", y=", y) // 输出的x还是原来的值，改变的是副本y的值
    
    z := addP(&x)                   // 执行addP函数，实际上修改的是x的值
    fmt.Println("x=", x, ", z=", z) // 输出的x已经被修改
    fmt.Println("&x=", &x)          // x地址还是原来的
}

func add(a int) int {
    a++
    return a
}

func addP(a *int) int {
    *a++
    return *a
}
```
输出：
```text
x= 3 , &x= 0xc000018060
x= 3 , y= 4
x= 4 , z= 4
&x= 0xc000018060
```

传指针的好处：

1） 传指针使得多个函数能操作同一个对象

2） 传指针比较轻量级，毕竟只传内存地址，可以用指针传递体积大的结构体。如果传递值，实际上是创建副本，而创建副本会花费比较多的系统开销（内存和时间）。函数调用时，切片（slice）、字典（map）、接口（interface）、通道（channel）这些类型都是默认使用引用传递（没有显示的指出指针）。

3） 传递指针给函数不但可以节省内存，而且赋予了函数直接修改外部变量的能力，所以被修改的变量不再需要使用return返回。

!!! note ""
    在实际开发中传递一个指针很容易引发一些不确定的事，需小心那些可以改变外部变量的函数，开发时需完善注释。


## defer与跟踪

### defer延迟语句

defer语句会将其后跟随的语句进行延迟处理，在defer归属的函数即将返回时，将延迟处理的语句按defer的逆序进行处理，类似栈的操作。
在进行一些IO操作的时候，如果遇到错误，便需要提前返回，而返回之前应该关闭相应的资源，否则容易造成资源泄漏等问题。
defer语句可以很好的处理此类问题。

打开一个资源一般这样写：
```go
func ReadWrite() bool {
    file.Open("file")
    // todo 处理逻辑
    
    if aFailure {           // 遇到状况a，需要提前返回
        file.Close()
        return false
    } else if bFailure {    // 遇到状况b，需要提前返回
        file.Close()
        return false
    }
    file.Close()
    return true
}
```

上面例子中存在很多重复代码。使用defer语句：
```go
func ReadWrite() bool {
    file.Open("file")
    defer file.Close()  // 打开和关闭写在一起，方便管理，还不容易遗漏
    
    // todo 处理逻辑
    
    if aFailure {
        return false
    } else id bFailure {
        return false
    }
    return true
}
```

完整示例：
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    i, p := a()
    fmt.Println("return:", i, p, time.Now())
}

func a() (i int, p *int) {
    //i++
    defer func(i int) {
        fmt.Println("defer3:", i, &i, time.Now())
    }(i)
    
    defer fmt.Println("defer2:", i, &i, time.Now())
    
    defer func() {
        fmt.Println("defer1:", i, &i, time.Now())
    }()
    
    i++
    func() {
        fmt.Println("print1:", i, &i, time.Now())
    }()
    
    fmt.Println("print2:", i, &i, time.Now())
    return i, &i
}
```
输出：
```text
print1: 1 0xc000018060 2020-04-10 00:01:11.473682 +0800 CST m=+0.000088653
print2: 1 0xc000018060 2020-04-10 00:01:11.473885 +0800 CST m=+0.000290746
defer1: 1 0xc000018060 2020-04-10 00:01:11.47389 +0800 CST m=+0.000295873
defer2: 0 0xc000018060 2020-04-10 00:01:11.47368 +0800 CST m=+0.000086777
defer3: 0 0xc000018148 2020-04-10 00:01:11.473896 +0800 CST m=+0.000301887
return: 1 0xc000018060 2020-04-10 00:01:11.473898 +0800 CST m=+0.000304719
```
!!! note ""
    defer、return、返回值三者的执行逻辑是：defer最先执行一些收尾工作；然后return执行，return负责将结果写入返回值中；最后函数携带当前返回值退出。


### 跟踪

defer还经常用于代码执行跟踪，具体用途为在进入和离开某个函数时打印相关的消息，常用于log。

示例：
```go
package main

import "fmt"

func main() {
    b()
}

func a() {
    defer un(trace("a"))
    fmt.Println("a的逻辑代码")
}

func b() {
    defer un(trace("b"))
    fmt.Println("b的逻辑代码")
    a()
}

func trace(s string) string {
    fmt.Println("开始执行", s)
    return s
}

func un(s string) {
    fmt.Println("结束执行", s)
}
```
输出：
```text
开始执行 b
b的逻辑代码
开始执行 a
a的逻辑代码
结束执行 a
结束执行 b
```


## 错误与恢复

错误处理是编程语言必须慎重考虑的一个环节。
主流语言如Java、PHP、C++等都引入了错误处理的概念，比如异常（exception）的概念和try-catch关键字等。  
Go语言并没有像Java那样提供异常机制，所以Go语言不能抛出异常，取而代之的是panic和recover机制。


### error

Go语言通过error接口实现错误处理的标准模式，该接口的定义方式如下：
```go
type PathError struct {
    Op      string
    Path    string
    Err     string
}

func (e *PathError) Error() string {
    return e.Op +" "+ e.Path +" "+ e.Err.Error()
}

func Stat(name string) (fi FileInfo, err error) {
    var stat syscall.Stat_t
    err = syscall.Stat(name, &stat)
    if err != nil {
        return nil, &PathError{"stat", name, err}
    }
    return fileInfoFromStat(&stat, name), nil
}
```


### panic

panic（恐慌）是一个内建函数，可以中断原有的控制流程，进入一个令人“恐慌”的流程中。  
当函数调用panic时，函数的执行就会被中断，但是函数中的defer会正常执行，然后回到调用它的地方。  
在调用函数的地方，panic会继续蔓延，继续向外围扩散，直到panic的goroutine中所有调用的函数返回，此时程序才会退出。  
错误信息将被报告，包括在调用panic()函数时传入的参数，这个过程称为错误处理流程。

恐慌可以直接调用panic产生，也可以由运行时错误产生，例如访问越界的数组。  
区别使用panic和error两种方式：导致关键流程出现不可修复性错误的情况使用panic，其它情况使用error。

```go
package main

func main() {
    defer func() {
        panic("defer panic")
    }()
    panic("test panic")
}
```


### recover

recover是一个内建的函数，可以让进入令人恐慌的流程中的goroutine恢复过来。  
如果当前的goroutine陷入恐慌，调用recover可以捕获到panic的输入值，并且恢复正常的执行。
由于recover()函数用于终止错误处理流程，所以一般情况下，recover尽在defer语句的“函数”中有效，以便有效截取错误处理流程。

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // genErr()
    throwsPanic(genErr)
}

func genErr() {
    fmt.Println(time.Now(), "正常的语句")
    defer func() {
        fmt.Println(time.Now(), "defer正常的语句")
        panic("第二个错误")
    }()
    panic("第一个错误")
}

func throwsPanic(f func()) (b bool) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println(time.Now(), "捕获到的异常：", r)
            b = true
        }
    }()
    f()
    return 
}
```
输出：
```text
2020-04-10 00:02:13.697739 +0800 CST m=+0.000073265 正常的语句
2020-04-10 00:02:13.697933 +0800 CST m=+0.000266626 defer正常的语句
2020-04-10 00:02:13.697937 +0800 CST m=+0.000270989 捕获到的异常： 第二个错误
```

上述例子中，虽然“第一个错误”早就发生了，但是在recover()函数中看不到“第一个错误”的捕获时间，因为recover()只会捕获最后一个错误，而且捕获的时机是在函数最后面，不影响“第一个错误”之后的defer语句的执行。