# 结构体方法

Go方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。所以在Go语言里方法是一种特殊类型的函数。
接收者几乎可以是任意类型（接口、指针除外），包括结构体类型、函数类型，可以是int、bool、string或数组别名类型。
接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却必须是具体的实现。
接收者也不能是一个指针类型，但需要注意的是它可以是任何其它允许类型的指针。


## 方法的声明

定义方法的一般格式如下：
`func (recv receiver_type) methodName(parameter_list) (return_value_list){ }`
在方法名之前，func关键字之后的括号中指定接收者。
如果recv是接收者的一个实例，Method1是接收者类型的一个方法名，那么方法调用遵循传统的选择器符号：recv.Method1()。

```go
package main

import "fmt"

type TwoInts struct {
    a int
    b int
}

func (tn *TwoInts) AddThem() int {
    return tn.a + tn.b
}

func (tn *TwoInts) AddToParam(param int) int {
    return tn.a + tn.b + param
}

func main() {
    two1 := new(TwoInts)
    two1.a = 12
    two1.b = 10
    fmt.Printf("%v 和为：%d\n", two1, two1.AddThem())
    fmt.Printf("将它们添加到参数：%d\n", two1.AddToParam(30))
    
    two2 := TwoInts{3, 4}
    fmt.Printf("和为：%d\n", two2.AddThem())
}
```
输出：
```text
&{12 10} 和为：22
将它们添加到参数：52
和为：7
```

方法是函数，所以不允许方法重载，对于一个类型只能有一个给定名称的方法。  
但如果基于接收者类型，则允许重载，具有同样名字的方法可以在多个不同的接收者类型上存在。
```
func (a *denseMatrix) Add(b Matrix) Matrix
func (a *sparseMatrix) Add(b Matrix) Matrix
```


## 基于指针对象的方法

在结构体类型上可以定义两种方法，分别基于指针接收器和基于值接收器。值接收器意味着复制整个值到内存中，内存开销非常大，而基于指针的接收器仅仅需要一个指针大小的内存。
如果想要方法改变接收者的数据，就在接收者的指针类型上定义该方法。否则，就在普通的值类型上定义方法：

```go
package main

import "fmt"

type HttpResponse struct {
    status_code int
}

func (r *HttpResponse) validResponse() {
    r.status_code = 200
}

func (r HttpResponse) updateStatus() string {
    return fmt.Sprint(r)
}

func main() {
    var r1 HttpResponse     // r1是值
    r1.validResponse()
    fmt.Println(r1.updateStatus())
    
    r2 := new(HttpResponse) // r2是指针
    r2.validResponse()
    fmt.Println(r2.updateStatus())
}
```
输出：
```text
{200}
{200}
```
validResponse()接收一个指向HttpResponse的指针，并改变它内部的成员；updateStatus()复制HttpResponse的值并只输出HttpResponse的内容，如r1是值而r2是指针。
接着，在updateStatus()中改变接收者r的值，将会看到它可以正常编译，但是开始的r值没有改变。因为指针作为接收者不是必需。


## 方法和未导出字段

当类型被明确导出时，它的字段并没有被导出，例如 p.firstName 的写法就是错误的。  
想在另一个程序中修改或者只是读取一个类型的的未导出字段，可以通过面向对象语言一个众所周知的技术来完成：getter和setter方法。

person.go:
```go
package person

type Person struct {
    firstName string
    lastName string
}

func (p *Person) FirstName() string {
    return p.firstName
}

func (p *Person) SetFirstName(newName string) {
    p.firstName = newName
}
```

usePerson.go:
```go
package main

import (
    "./person"
    "fmt"
)

func main() {
    p := new(person.Person)
    // p.firstName  未定义返回错误
    // cannot refer to unexported field or method firstName
    // p.firstName = "Tom"
    
    p.setFirstName("Tom")
    fmt.Println(p.FirstName())      // 返回：Tom
}
```


## 嵌入类型的方法和继承

Go语言不是一门传统意义上的面向对象编程语言（Java、PHP等），Go语言无法在语言层面上直接实现类的继承和亚类的实现。  
但是由于Go语言提供了创建匿名结构体的方法，所以可以把匿名结构体嵌入具名结构体内部，这样具名结构体也会拥有其内部匿名结构体的那些方法，在效果上等同于面向对象编程中的类的继承。
示例：
```go
package main

import (
    "fmt"
    "math"
)

// 一个已知的具名结构体
type Point struct {
    x, y float64
}

func (p *Point) Abs() float64 {
    return math.Sqrt(p.x*p.x + p.y*p.y)
}

// NamePoint结构体内部包含匿名字段Point
type NamePoint struct {
    Point
    name string
}

func main() {
    n := &NamePoint{Point{3, 4}, "Go"}
    fmt.Println(n.Abs())
}
```
输出：
```text
5
```

将一个已存在类型的字段和方法注入另一个类型里即为内嵌，匿名字段上的方法“晋升”成为外层类型的方法。  
当然类型可以有只作用于本身实例而不作用于内嵌“父”类型上方法，可以覆写方法（像字段一样）。
如：
```
func (n *NamePoint) Abs() float64 {
    return n.Point.Abs() * 100
}
```


## 多重继承

在大部分面向对象语言中，是不允许多重继承的，因为这会导致编译器变得复杂，不过由于Go语言并没有类的概念，所谓继承其实是内嵌结构体，通过在类型中嵌入所有必要的父类型，可以简单地实现多重继承。  
Go语言的多重继承不支持多重嵌套（即父级类型内部不允许有匿名结构体字段）。

举个例子，一个类型 CameraPhone，通过它可以调用Call()函数，也可以调用TakeAPicture()函数，但是第一个方法属于类型Phone，第二个方法属于类型Camera。
```go
package main

import "fmt"

type Camera struct {}

func (c *Camera) TakeAPicture() string {
    return "拍照"
}

type Phone struct {}

func (p *Phone) Call() string {
    return "响铃"
}

type CameraPhone struct {
    Camera
    Phone
}

func main() {
    cp := new(CameraPhone)
    fmt.Println("拍照有多种功能：")
    fmt.Println("打开相机：", cp.TakeAPicture())
    fmt.Println("电话来电：", cp.Call())
}
```
输出：
```text
拍照手机有多种功能：
打开相机： 拍照
电话来电： 响铃
```