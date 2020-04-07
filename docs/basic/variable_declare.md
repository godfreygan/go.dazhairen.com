# 变量

变量是Go语言中表示抽象值和存储计算结果的一种方式。
变量的值通过变量名来访问，Go语言的变量名命名规则和常量一样，由字母、数字、下划线组成，但名称首字母不能是数字。

## 声明
格式为：`var name type`。

var 是声明变量的关键字，name 是变量名，type 是变量的类型。

示例1：
```go
package main

import "fmt"

func main() {
    // 定义一个变量a为int型
    var a int
    
    // 定义一个变量b为bool型
    var b bool
    
    // 定义一个变量str为string型
    var str string
    
    // 定义两个变量a、b为int型
    var d,e int
    
    // 与常量一样，同时定义多个
    var (
        i,j int
        m bool
        name string
    )
    
    // 定义变量并赋值
    var h int = 100
    var x,y int = 10,20
    var name,age = "Godfrey",18

    fmt.Println(h, x, y, name, age)
}
```

示例2：
```go
package main

import "fmt"

/* 1. 这种全局定义的写法是错误的 */
// a := 1

/* 2. 全局定义使用这个方式 */
var a = 1

/* 前面已经定义了变量a，同一个代码块中不能再次定义 */
// a := 10

func main() {
    b := 2          // 4. 只能在函数体内部使用简写形式
    a = 100         // 5. 把10赋值给a是没有语法问题的
    fmt.Println(a, b)
    
    a := "hello"    // 6. 函数体内部属于另一个代码块，可以再次声明变量a，也可以改变变量类型
    fmt.Println(a)
    
    /* 7. 不能在调用之后定义变量，即不能使用未定义的变量 */
    // fmt.Println(d)
    // d := 4
    
    fmt.Println(c)
}

// 8. 全局定义不限定位置，在函数外即可
var c = 0
```

使用 := 可以快速创建一个函数内部的变量，包括初始化声明与复制两大部分。以下几点需要注意：

* 同一个代码块内不能多次声明同一个变量

* 函数内部可以使用全局定义的变量

* 函数体内属于独立代码块，可以再次声明变量，可以改变变量的类型，并且定义只在函数内部生效。

* 如果定义变量前就调用该变量，会出现编译错误

* 全局变量不限制位置，建议统一放在函数代码前面

* 变量声明时没有赋值，系统会赋予当前指定类型的“零值”。例如 int为0、float为0.0、bool为false、string为“”等


## 匿名变量

“_” 匿名变量，即丢弃数据不做处理，如下示例：

```go
package main

import "fmt"

func main() {
    tmp, _ := 7, 8
    fmt.Println(tmp)
    
    _, e, f := test()
    fmt.Println(e, f)
    
    a, b, _ := test()
    fmt.Println(a, b)
}

func test()(a, b, c int){
    return 1, 2, 3
}
```
