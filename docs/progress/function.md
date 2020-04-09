# 函数

函数是包含用于执行某一任务或计算的一系列语句，对这一系列语句的进行包装，使其更便于阅读也使其更加简明、清晰、便捷。简单来说就是，封装代码、即时调用。

函数可以提高程序的模块性和代码的重复利用率。函数在程序中是一等公民，包含一下特点：

* 函数本身可以作为值进行传递

* 支持匿名函数和闭包

* 函数可以满足接口


## 声明

函数声明结构格式如下：
```go
func funcName(input1 type1, input2 type2)(output1 type1, output2 type2){
    // 逻辑代码
    return value1, value2   // 返回多值
}
```

示例：
```go
package main

import "fmt"

func main() {
    x := 1
    y := 2
    z := 3
    var maxNum int
    maxNum = max(x, y)
    fmt.Printf("max(%d, %d)=%d\n", x, y, maxNum)
    maxNum = max(x, z)
    fmt.Printf("max(%d, %d)=%d\n", x, z, maxNum)
    fmt.Printf("max(%d, %d)=%d\n", y, z, Max(y, z))
}

// max私有函数
func max(a, b int)(maxNum int) {
    if a > b {
        return a
    }
    return b
}

// Max公有函数
func Max(a, b int)(maxNum int) {
    maxNum = b
    if a > b {
        maxNum = a
    }
    return maxNum
}
```
输出：
```text
max(1, 2)=2
max(1, 3)=3
max(2, 3)=3
```
在上面例子中有两个功能相同的函数，其中一个为max，另一个是Max，这两个函数的区别在于外部是否能够调用。


## 多返回值

相比C、C++、Java和PHP，Go语言的函数支持多返回值，这也是Go语言的一大特性。这为判断一个函数是否正常执行提供了方便。

!!! note ""
    一个函数有多个返回值，如果不需要某些返回值，可以使用空白标识符"_"忽略掉。

```go
package main

import "fmt"

// SumAndMul 返回 a+b 和 a*b
func SumAndMul(a, b int) (int, int) {
	return a + b, a * b
}

func main() {
	x := 20
	y := 30
	xPlusy, xTimesY := SumAndMul(x, y)
	fmt.Println(xPlusy, xTimesY)
	
	m, _ := SumAndMul(8, 9)
	fmt.Println(m)
	
	_, n := SumAndMul(6, 4)
	fmt.Println(n)
}
```
输出：
```text
50 600
17
24
```


## 函数作为参数

函数也可作为参数，只要被调用函数的返回值个数、返回值类型和返回值的顺序与调用函数所需求的实参是一致的，就可以把这个被调用的函数当作其它函数的参数。

如下，f1需要3个参数，同时f2返回3个参数：

```go
func f1(a, b, c int)
func f2(a, b int)(int, int, int)
func f1(f2(a, b))
```

示例：
```go
package main

import "fmt"

func pipe(ff func() int) int {
    return ff()
}

// FormatFunc定义函数类型
type FormatFunc func(s string, x, y int) string

func format(ff FormatFunc, s string, x, y int) string {
    return ff(s, x, y)
}

func main() {
    s1 := pipe(func() int { return 100 })
    s2 := format(func(s string, x, y int) string {
        return fmt.Sprintf(s, x, y)
    }, "%d, %d", 10, 20)
    fmt.Println(s1, s2)
}
```
输出：
```text
100 10, 20
```


## 函数作为类型

函数也是一种变量，可以通过 type 来定义它，它的类型就是所有拥有相同参数与相同返回值的一种函数类型。
函数作为类型，最大的好处就是可以把这个类型的函数当作值来传递。

示例：
```go
package main

import "fmt"

// 是否奇数
func isOdd(v int) bool {
    if v%2 == 0 {
        return false
    }
    return true
}

// 是否偶数
func isEven(v int) bool {
    if v%2 == 0 {
        return true
    }
    return false
}

type boolFunc func(int) bool    // 声明一个函数类型

// 函数作为一个参数使用
func filter(slice []int, f boolFunc) []int {
    var result []int
    for _, value := range slice {
        if f(value) {
            result = append(result, value)
        }
    }
    return result
}

func main() {
    slice := []int{2, 1, 9, 6, 7, 3}
    fmt.Println("slice=", slice)
    odd := filter(slice, isOdd)     //  函数当作值来传递
    fmt.Println("odd：", odd)
    even := filter(slice, isEven)   //  函数当作值来传递
    fmt.Println("even：", even)
}
```
输出：
```text
slice= [2 1 9 6 7 3]
odd： [1 9 7 3]
even： [2 6]
```


## 可变参数

函数有着不确定数量的参数时，就需要用到可变参数。格式：`func foo(arg ...int) {}`

示例：
```go
package main

import "fmt"

func ageMinOrMax(m string, a ...int) int {
    if len(a) == 0 {
        return 0
    }
    if m == "max" {
        max := a[0]
        for _,v := range a {
            if v > max {
                max = v
            }
        }
        return max
    } else if m == "min" {
        min := a[0]
        for _,v := range a {
            if v < min {
                min = v
            }
        }
        return min
    } else {
        e := -1
        return e
    }
}

func main() {
    // 手动填写参数
    age := ageMinOrMax("min", 1, 3, 2, 0)
    fmt.Printf("年龄最小的是%d岁\n", age)
    
    // 数组作为参数
    ageArr := []int{7, 9, 3, 5, 1}
    age = ageMinOrMax("max", ageArr...)
    fmt.Printf("年龄最大的是%d岁\n", age)
}
```
输出：
```text
年龄最小的是0岁
年龄最大的是9岁
```


## 函数参数不支持默认值

Go语言追求"显式的表达，避免隐喻"。这其实是一种很好的行为，避免调用者不清楚默认值而导致一系列问题。
如果你一定要使用默认值，那么可变参数也许可以达到你想要的效果。

示例：
```go
package main

import "fmt"

func foo(names ...string) {
	if len(names) == 0 {
		names = []string{"Tom"}
	}

	for _, name := range names {
		fmt.Println(name)
	}
}

func main () {
	foo()

	fmt.Println()

	foo("Jerry", "Rose")
}
```
输出：
```text
Tom

Jerry
Rose
```


## 匿名函数

匿名函数是指不需要定义函数名的一种函数定义方式，由一个不带函数名的函数声明和函数体声明。

示例：
```go
package main

import "fmt"

func main() {
    func(num int) int {
        sum := 0
        for i := 1; i <= num; i++ {
            sum += i
        }
        fmt.Println(sum)
        return sum
    }(100)
}
```
输出：
```text
5050
```


## 闭包

匿名函数同样被称为闭包，

* 闭包对它作用域上部的变量可以进行修改，修改引用的变量会对变量进行实际修改

* 被捕获到闭包中的变量让闭包本身拥有了记忆效应，闭包中的逻辑可以修改闭包捕获的变量，变量会跟随闭包生命期一直存在，闭包本身就如同变量一样拥有了记忆效应

* 闭包的记忆效应被用于实现类似于设计模式中工厂模式的生成器

示例1：
```go
package main

import "fmt"

func main() {
    fmt.Printf("%d\n", Add()(3))
    
    fmt.Printf("%d\n", Add2(6)(3))
}

// Add无参函数，返回值是一个匿名函数，匿名函数返回一个int类型的值
func Add() func(b int) int {
    return func(b int) int {
        return b + 2
    }
}

// Add2无参函数，返回值是一个匿名函数，匿名函数返回一个int类型的值
func Add2(a int) func(b int) int {
    return func(b int) int {
        return a + b
    }
}
```
输出：
```text
5
9
```

示例2：
```go
package main

import "fmt"

func main() {
    var f = adder()
    
    fmt.Println(f(1))
    
    fmt.Println(f(2))
    
    fmt.Println(f(3))
}

func adder() func(int) int {
    var x int   // 闭包中的变量可以在闭包函数体内声明，也可以在外部函数声明
    return func(d int) int {
        fmt.Println("x=", x, " ,d=", d)
        x += d
        return x
    }
}
```
输出：
```text
x= 0  ,d= 1
1
x= 1  ,d= 2
3
x= 3  ,d= 3
6
```


## 递归函数

递归，就是在运行的过程中调用自己。
在使用递归时，需要设置退出条件，否则递归将陷入无限循环，可能会导致程序崩溃。

示例1，阶乘：
```go
package main

import "fmt"

func Factorial(n uint64)(result uint64) {
    if n > 0 {
        result = n * Factorial(n - 1)
        return result
    }
    return 1
}

func main() {
    var i int = 15
    fmt.Printf("%d的阶乘是%d\n", i, Factorial(uint64(i)))
}
```
输出：
```text
15的阶乘是1307674368000
```

示例2，斐波那契数列：
```go
package main

import "fmt"

func fibonacci(n int) int {
    if n < 2 {
        return n
    }
    return fibonacci(n-2) + fibonacci(n-1)
}

func main() {
    var i int
    for i = 0; i < 10; i++ {
        fmt.Printf("%d\t", fibonacci(i))
    }
}
```
输出：
```text
0       1       1       2       3       5       8       13      21      34
```


## 内置函数

内置函数，也就是Go语言自带的函数，不需要进行导入操作就可以直接使用。

常见的内置函数有：

- close：资源管道关闭操作

- len：返回某个数据类型的长度或数量（字符串、数组、切片、映射和管道）

- cap：容量，返回某个数据类型的最大容量（只适用于切片和映射）

- new：分配内存，用于值类型和用户自定义的类型

- make：分配内存，用于内置引用类型，如切片、映射、管道

- copy、append：用于复制和连接切片

- panic、recover：错误处理操作

- print、println：底层打印函数（实际开发中一般使用fmt包）

- complex、real、imag：创建和操作复数

