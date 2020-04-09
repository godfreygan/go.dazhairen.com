# 数据类型扩展

除前面介绍的几种类型，Go语言还有更复杂的数据类型，以及数据类型的操作

## 强制类型转换

类型转换用于将一种数据类型的变量转换为另外一种类型的变量。

格式：`type_name(expression)`

在做强制类型转换时，需注意数据长度被截断而发生的数据精度损失（如将浮点数转成整型）与精度溢出（值超过转换的目标类型的值范围时）的问题。

如下，将整型转换成浮点型并计算结果，将结果赋值给浮点型变量：
```go
package main

import "fmt"

func main() {
    sum := 20
    count := 6
    mean := float32(sum) / float32(count)
    mean2 := sum / count
    fmt.Printf("mean的值为：%f\n", mean)
    fmt.Printf("mean2的值为：%d\n", mean2)
}
```
输出：
```text
mean的值为：3.333333
mean2的值为：3
```


## 自定义类型

自定义类型，其实是与结构体有关的。后面将会详细说明。

```go
type people struct {
    age     int8
    height  float32
    weight  float32
    sex     string
}
```


## 类型别名

类型别名主要用于解决代码升级、迁移中存在的类型兼容性问题。

格式：`type TypeAlias = Type`。

TypeAlias只是Type的别名，本质上TypeAlias和Type是同一个类型，可以理解成孩子的小名，上学后的学名，上英语课时候的英文名，这些名字都指向他本人。

!!! note ""
    注意：Go语言中，类型别名和原类型是两种类型，不能直接操作，需要先进行类型转换。也就是上面所说的，乳名、学名、英文名都指向某个人，但是每个名字所代表的意义是不一样的，即各个名字互不影响。


```go
package main

import "fmt"

// 将NewInt定义为int的别名
type NewInt int

// 将int取一个别名叫IntAlias
type IntAlias = int

func main() {
    // 将a声明为NewInt类型
    var a NewInt
    // 查看a的类型名
    fmt.Printf("a的类型为：%T\n", a)
    
    // 将a2声明为IntAlias类型
    var a2 IntAlias
    // 查看a2的类型名
    fmt.Printf("a2的类型为：%T\n", a2)
    
    a = 100
    var a3 int = 200
    //z := a + a3   // 编译报错，不同类型
    fmt.Printf("a的类型为：%T，a3的类型为：%T\n", a, a3)
}
```
输出：
```text
a的类型为：main.NewInt
a2的类型为：int
a的类型为：main.NewInt，a3的类型为：int
```


## 指针

指针是一个变量，其值是另一个变量的地址，即指针变量指向内存地址。
每个内存位置都有其定义的地址，可以使用&运算符来访问它，这个运算符表示内存中的地址。格式：`var name *type`

```go
package main

import "fmt"

func main() {
	a := 20
	ap := &a
	fmt.Printf("a的地址：%x\n", &a)
	fmt.Printf("ap的地址：%x\n", &ap)
	fmt.Printf("ap指向的地址：%x\n", ap)
	fmt.Printf("a的值：%d，*ap的值：%d\n", a, *ap)
}
```
输出：
```text
a的地址：c0000a8008
ap的地址：c0000a2018
ap指向的地址：c0000a8008
a的值：20，*ap的值：20
```


在定义好指针变量后，编译器为其分配一个nil值，防止指针没有确切的地址分配。
此外还可定义指针的指针，即指针指向另一个指针，格式：`var ptr* *int`

```go
package main

import "fmt"

func main() {
    var a *int      // 定义一个a指针，指向nil
    ap := &a        // 空指针a的指针
    fmt.Printf("a-->nil：%x\n", a)
    fmt.Printf("a-->nil：%x\n", &a)
    fmt.Printf("ap-->a：%x\n", ap)
    fmt.Printf("ap-->a-->nil：%x\n", *ap)
    fmt.Printf("&ap-->ap：%x\n", &ap)
    /**
    说明：
    ap指向a，也就是a的地址，所以ap == &a
    a指向0(int)，而ap指向a的地址，所以 *ap == a
    ap作为指针，本身也存在地址，&ap取的就是ap本身的地址
    */
    
    fmt.Printf("\n")
    
    b := 10
    bp := &b
    bpp := &bp
    fmt.Printf("b：%d\n", b)         // 说明1
    fmt.Printf("&b：%x\n", &b)       // 说明2
    fmt.Printf("bp：%x\n", bp)       // 说明3
    fmt.Printf("*bp：%d\n", *bp)     // 说明4
    fmt.Printf("&bp：%x\n", &bp)     // 说明5
    fmt.Printf("bpp：%x\n", bpp)     // 说明6
    fmt.Printf("*bpp：%x\n", *bpp)   // 说明7
    fmt.Printf("**bpp：%d\n", **bpp) // 说明8
    
    /**
    说明：
    1. b变量存储10，所以 b=10
    2. &b，获取变量b的地址，所以 &b=c210000018
    3. bp存储变量b的地址，所以 bp=c210000018
    4. bp存储变量b的地址，而*bp指向变量b，所以 *bp=10
    5. &bp，获取bp的地址，所以 &bp=c210000020
    6. bpp存储bp的地址，所以 bpp=c210000020
    7. bpp存储bp的地址，所以*bpp指向变量bp，又bp存储b的地址，所以 *bpp=c210000018
    8. *bpp指向bp，*bp指向b，故**bpp也指向b，所以 **bpp=10
    */
}
```
输出：
```text
a-->nil：0
a-->nil：c00000e028
ap-->a：c00000e028
ap-->a-->nil：0
&ap-->ap：c00000e030

b：10
&b：c000018070
bp：c000018070
*bp：10
&bp：c00000e040
bpp：c00000e040
*bpp：c000018070
**bpp：10
```
