# 结构体

Go语言通过用自定义的方式形成新的类型，结构体是类型中带有成员的复合类型。  
Go语言使用结构体和结构体成员来描述实体和实体对应的属性。
结构体成员是由一系列成员变量构成，这些成员变量也称为“字段”。它们有如下特性：

* 字段拥有自己的类型和值

* 字段名必须唯一

* 字段的类型也可以是结构体，甚至是结构所在结构体的类型

Go语言中，没有“类”的概念，也不支持“类”的继承等面向对象的概念。Go语言中结构体的内嵌配合接口比面向对象具有更高的扩展性和灵活性。


## 声明与赋值

结构体定义格式：
```
type structName struct {
    field1 type1
    field2 type2
    ...
}
```

示例：
```go
package main
import "fmt"

type myStruct struct {
    i1  int
    f1  float32
    str string
}

func main() {
    ms := new(myStruct)
    ms.i1 = 10
    ms.f1 = 15.5
    ms.str = "Google"
    fmt.Printf("int: %d\n", ms.i1)
    fmt.Printf("float: %f\n", ms.f1)
    fmt.Printf("string: %s\n", ms.str)
    fmt.Println(ms)
}
```
输出：
```text
int: 10
float: 15.500000
string: Google
&{10 15.5 Google}
```


## 结构体传参

结构体可以作为参数传递给函数，同样有形式参数传递和指针参数传递两种形式。

示例：
```go
package main

import "fmt"

type Employee struct {
    Id      int
    Name    string
    Address string
    Phone   string
}

func main() {
    var employee Employee
    employee.Id = 10001
    employee.Name = "Tom"
    employee.Address = "US"
    employee.Phone = "18842685441"
    
    fmt.Printf("形式传参之前，employee Id: %d\n", employee.Id)
    operateEmployee1(employee)
    fmt.Printf("形式传参之后，employee Id: %d\n", employee.Id)
    
    fmt.Printf("指针传参之前, employee Id: %d\n", employee.Id)
    operateEmployee2(&employee)
    fmt.Printf("指针传参之后, employee Id: %d\n", employee.Id)
}

// 形式传参
func operateEmployee1(employee Employee) {
    employee.Id = 10002
}

// 指针传参
func operateEmployee2(employee *Employee) {
    employee.Id = 10002
}
```
输出：
```text
形式传参之前，employee Id: 10001
形式传参之后，employee Id: 10001
指针传参之前, employee Id: 10001
指针传参之后, employee Id: 10002
```


## 带标签的结构体

结构体中的字段不仅有名字与类型，还有一个可选的附属于字段的标签（tag）：用于文档说明或者开发过程中的特殊标记，它的值是一串字符串。  
标签内容只有reflect包能获取。

示例：
```go
package main

import (
    "fmt"
    "reflect"
)

type TagType struct {
    Name    string  "姓名"
    Age     int     "年龄"
    Height  int     "身高"
    Weight  int     "体重"
}

func main() {
    tt := TagType{"张三", 18, 175, 120}
    for i := 0; i < 4; i++ {
        refTag(tt, i)
    }
}

/*
在一个变量上调用reflect.TypeOf()可以获得变量的正确类型
如果变量是一个结构体类型，就可以通过Field来索引结构体的字段，然后就可以使用Tag属性
*/
func refTag(tt TagType, ix int) {
    ttType := reflect.TypeOf(tt)
    ixField := ttType.Field(ix)
    fmt.Printf("%v\n", ixField.Tag)
}
```
输出：
```text
姓名
年龄
身高
体重
```


## 匿名字段

结构体可以包含一个或多个匿名（或内嵌）字段，即这些字段没有显示的名字，只有字段的类型是必需的，此时类型就是字段的名字。  
匿名字段本身可以是一个结构体类型，即结构体可以包含内嵌结构体。

示例：
```go
package main

import "fmt"

type One struct {
    in1 int
    in2 int
}

type Two struct {
    b int
    c float32
    int     // 匿名字段
    One     // 匿名字段
}

func main() {
    two := new(Two)
    two.b = 6
    two.c = 7.5
    two.int = 100
    two.in1 = 5
    two.in2 = 10
    
    fmt.Printf("two.b is: %d\n", two.b)
    fmt.Printf("two.c is: %f\n", two.c)
    fmt.Printf("two.int is: %d\n", two.int)
    fmt.Printf("two.in1 is: %d\n", two.in1)
    fmt.Printf("two.in2 is: %d\n", two.in2)
    
    // 使用结构体字面量
    two2 := Two{8, 9.9, 55, One{6, 9}}
    fmt.Println("two2 is: ", two2)
}
```
输出：
```text
two.b is: 6
two.c is: 7.500000
two.int is: 100
two.in1 is: 5
two.in2 is: 10
two2 is:  {8 9.9 55 {6 9}}
```


## 内嵌结构体

Go语言中的继承是通过内嵌或组合来实现的，在Go语言中，组合比继承更受欢迎。

示例：
```go
package main

import "fmt"

type A struct {
    ax, ay int
}

type B struct {
    A
    bx, by float32
}

func main() {
    b := B{A{1, 2}, 3.0, 4.0}
    fmt.Println(b.ax, b.ay, b.bx, b.by)
    fmt.Println(b.A)
}
```
输出：
```text
1 2 3 4
{1 2}
```


## 命名冲突

当两个字段拥有相同的名字（例如继承其它结构体）时会产生冲突，一般情况有两种：

1） 外层字段的名字覆盖内层字段的名字，但是两者的内存空间都会保留，这提供了一种重载字段或方法的方式。

2） 相同的名字在同层次结构体中出现了重复，并且这个字段被程序调用了，这将导致程序错误（字段不被调用不会报错，但有隐患）。这种情况只能由开发者自己修正。

示例：
```go
package main

import "fmt"

type A struct {
	a int
}

type B struct {
	a int
}

type C struct {
	A
	B
}

func main() {
	c := &C{}
	// 字段名冲突，将会报错，ambiguous selector c.a
	// c.a = 1
	c.A.a = 1		// 针对某个内嵌类的某个字段进行赋值
	fmt.Println(c)
}
```
输出：
```text
&{{1} {0}}
```