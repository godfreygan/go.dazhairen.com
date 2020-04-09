# 反射

反射是用程序检查代码中所拥有的接口尤其是类型的一种能力，这是元编程的一种形式。反射可以在运行时检查类型和变量，例如它的大小、方法和动态调用这些方法。这对于没有源代码的包尤其有用。

!!! note ""
    反射是一个强大的工具，但也存在一定的隐患，除非真的有必要，否则应当避免使用或小心使用。


## 方法和类型的反射

变量的最基本信息就是类型和值：反射包的Type用来表示一个Go语言类型，反射包的Value为Go值提供了反射接口。

两个简单的方法：reflect.TypeOf 和 reflect.ValueOf，返回被检查对象的类型和值。  
例如，x被定义为 var x float64 = 3.14，那么 reflect.TypeOf(x) 返回 float64，reflect.ValueOf(x) 返回 <float64 Value>。

实际上，反射通过检查一个接口的值，把变量转换成空接口。从函数签名可以明显看出来：
>func TypeOf(i interface{}) Type
>func ValueOf(i interface{}) Value

示例：
```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type: ", reflect.TypeOf(x))
    
    v := reflect.ValueOf(x)
    fmt.Println("value: ", v)
    
    fmt.Println("kind: ", v.Kind())
    fmt.Println("value: ", v.Float())
    fmt.Println(v.Interface())
    fmt.Printf("value is %5.2e\n", v.Interface())
    
    y := v.Interface().(float64)
    fmt.Println(y)
}
```
输出：
```text
type:  float64
value:  3.4
kind:  float64
value:  3.4
3.4
value is 3.40e+00
3.4
```


## 通过反射修改设置值

想要把一个值做修改，Value有一些方法可以完成此操作，但是必须小心使用，如下面例子中 v.SetFloat(3.1415)，这将产生一个错误：reflect.Value.SetFloat using unaddressable value。
示例：
```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.1415
    v := reflect.ValueOf(x)
    
    // 设置一个值：
    // v.SetFloat(3.1415)       // 将产生错误：Panic: reflect.Value.SetFloat using unaddressable value
    
    fmt.Println("v：", v.CanSet())
    
    v = reflect.ValueOf(&x) // 注意：取x的地址
    fmt.Println("v的类型：", v.Type())
    fmt.Println("v：", v.CanSet())
    
    v = v.Elem()
    fmt.Println("v的Elem是：", v)
    v.SetFloat(3.1415)  // 编译通过
    fmt.Println(v.Interface())
    fmt.Println(v)
}
```
输出：
```text
v： false
v的类型： *float64
v： false
v的Elem是： 3.1415
3.1415
3.1415
```
上述代码中，错误的原因是v不是可设置的（并不是说值不可寻址），是否可设置是值的一个属性，并且不是所有的反射值都有这个属性：可以使用CanSet()方法测试是否可设置。


## 反射结构

有时需要反射一个结构类型：NumField() 方法返回结构内的字段数量；通过一个for循环用索引取得每个字段的值 Filed(i) 。
示例：
```go
package main

import (
    "fmt"
    "reflect"
)

type NotknowType struct {
    s1, s2, s3 string
}

func (n NotknowType) String() string {
    return n.s1 +"-"+ n.s2 +"-"+ n.s3
}

var secret interface{} = NotknowType{"Google", "Go", "Code"}

func main() {
    value := reflect.ValueOf(secret)
    secretType := reflect.TypeOf(secret)
    // 也可以这样写
    // secretType := value.Type()
    
    fmt.Println(secretType)
    secretKind := value.Kind()
    fmt.Println(secretKind)
    
    for i := 0; i < value.NumField(); i++ {
        fmt.Printf("Filed %d: %v\n", i, value.Field(i))
    }
    
    // 调用第一个方法，即 string()
    results := value.Method(0).Call(nil)
    fmt.Println(results)
}
```
输出：
```text
main.NotknowType
struct
Filed 0: Google
Filed 1: Go
Filed 2: Code
[Google-Go-Code]
```


## Printf和反射

在Go语言的标准库中，前面所述的功能都被大量地使用。比如，fmt包中的Printf（以及其它格式化输出函数）都会使用反射来分析它的...参数。  
Printf的函数声明为：`func Printf(format string, args ... interface{}) (n int, err error)`

Printf中的...参数为空接口类型，Printf使用反射包来解析这个参数列表，所以，Printf能够知道它每个参数的类型。

示例：
```go
package main

import (
    "os"
    "strconv"
)

type Stringer interface {
    String() string
}

type Celsius float64

func (c Celsius) String() string {
    return strconv.FormatFloat(float64(c), 'f', 1, 64) + " 度"
}

type Day int

var dayName = []string{"星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日"}

func (day Day) String() string {
    return dayName[day]
}

func print(args ...interface{}) {
    for i, arg := range args {
        if i > 0 {
            os.Stdout.WriteString(" ")
        }
        
        switch a := arg.(type) {
        case Stringer:  os.Stdout.WriteString(a.String())
        case int:       os.Stdout.WriteString(strconv.Itoa(a))
        case string:    os.Stdout.WriteString(a)
        default:        os.Stdout.WriteString("???")
        }
    }
}

func main() {
    print(Day(1), "温度是", Celsius(18.36))
}
```
输出：
```text
星期二 温度是 18.4 度
```