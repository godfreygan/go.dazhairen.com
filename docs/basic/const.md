# 常量

常量是指编译期间就已知，且在执行过程中不会改变的固定值。可定义为数值（整型、浮点型和复数类型）、布尔值或字符串等类型。格式为：`const name [type] = value`

```go
const PI = 3.1415926
const NAME string = "张三"
```

如果省略常量的类型，那么编译器会根据值来反推类型。此外，还可以使用关键字 iota 实现枚举，如下：
```go
package main

import "fmt"

const (
    a = iota                    // a = 0
    b                           // b = 1，隐式使用 iota 关键字，实际等同于 b = iota
    c                           // c = 2，实际等同于 c = iota
    d, e, f = iota, iota, iota  // d = 3，e = 3，f = 3，同一行值相同，此处不能只写一个iota
    g = iota                    // g = 4
    h = "h"                     // h = "h"，单独赋值，iota 依旧递增为5，只是这里没有用到
    i                           // i = "h"，默认使用上面的赋值，iota 依旧递增为6
    j = iota                    // j = 7
)

const z = iota                  // 每个单独定义的 const 常量中，iota 都会重置，此时 z = 0

func main() {
    fmt.Println(a, b, c, d, e, f, g, h, i, j, z)
}
```
输出：
```text
0 1 2 3 3 3 4 h h 7 0
```

说明：每一个const定义的第一个常量被默认设置为0，显式设置为其它值除外。
后续的常量默认设置为它上面那个常量的值，若前面那个常量的值为iota，则它也被设置为iota。
因为iota有递增效果。


!!! note ""
    iota也可以用在表达式中，如：`iota + 50`
