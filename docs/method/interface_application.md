# 接口使用

不同的结构体同时实现了同一个接口，这是Go语言的"多态"。

!!! note ""
    一个类型如果拥有一个接口需要的所有方法，那么这个类型就实现了这个接口。

示例：
```go
package main

import "fmt"

type Shaper interface {
    Area() float64
}

type Square struct {
    side float64
}

func (sq *Square) Area() float64 {
    return sq.side * sq.side
}

func main() {
    sq := new(Square)
    sq.side = 5
    
    // 定义areaIntf
    //areaIntf := sq
    
    // 更简洁，不需要分开定义：
    //areaIntf := Shaper(sq)
    
    // 甚至这样：
    areaIntf := sq
    fmt.Printf("面积为：%f\n", areaIntf.Area())
}
```
如上，程序定义了一个结构体 Square 和一个接口 Shaper ，接口有一个方法Area()。在main()方法中创建了一个 Square 的实例。在主程序外边定义了一个接收者类型是Square方法的Area()，用来计算正方形的面积，结构体Square实现了接口Shaper。
所以可以将一个Square类型的变量赋值给一个接口类型的变量：areaIntf = sq。

如果Square没有实现Area()方法，那么编译器将会提示报错：
> cannot use sq (type \*Square) as type Shaper in assignment:
> \*Square does not implement Shaper (missing Area method)

如果Shaper有另外一个方法Perimeter()，但是Square没有实现它，即使没有人在Square实例上调用这个方法，编译器也会给出上面的错误。


扩展上面的例子：
```go
package main

import "fmt"

type Shaper interface {
    Area() float64
}

type Square struct {
    side float64
}

func (sq *Square) Area() float64 {
    return sq.side * sq.side
}

type Rectangle struct {
    length, width float64
}

func (r Rectangle) Area() float64 {
    return r.length * r.width
}

func main() {
    r := Rectangle{5, 3}
    q := &Square{5}         // 传入的是一个指针
    
    // shapes := []Shaper{Shaper(r), Shaper(q)}
    
    // 更简洁的写法：
    shapes := []Shaper{r, q}
    
    // 遍历shapes
    for n, _ := range shapes {
        fmt.Println("形状参数：", shapes[n])
        fmt.Println("形状面积是：", shapes[n].Area())
    }
}
```

示例2：
```go
package main

import "fmt"

type stockPosition struct {
    ticker      string
    sharePrice  float64
    count       float64
}

// 获取stock的价格
func (s stockPosition) getValue() float64 {
    return s.sharePrice * s.count
}

type car struct {
    make    string
    model   string
    price   float64
}

// 获取car的价格
func (c car) getValue() float64 {
    return c.price
}

// 定义具有价值的不同事物的接口
type valuable interface {
    getValue()  float64
}

func showValue(asset valuable) {
    fmt.Printf("资产的价值是：%f\n", asset.getValue())
}

func main() {
    var o valuable = stockPosition{"goods", 199.90, 3}
    showValue(o)
    o = car{"BMW", "M3", 66666}
    showValue(o)
}
```
