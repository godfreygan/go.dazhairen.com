# 映射

映射是一种数据结构，用来存储一系列无序的键值对，映射基于键来存储值，映射功能强大的地方是，能够基于键快速检索数据。键就像是索引一样指向与该键关联的值。


## 映射的实现和创建

映射也是一个数据集合，也可以使用类似处理数组和切片的方式来迭代映射中的元素。  
但映射是无序集合，所以即使以同样的顺序保存键值对，每次迭代映射时，元素顺序也可能不一样。无序的原因是映射的底层实现使用了散列表。
Go语言创建与初始化映射有很多种方法，使用内置的make函数或者使用映射字面量都是常用的方法。
```
// 创建一个映射，键的类型是string，值的类型是int
dict := make(map[string]int)

// 创建一个映射，键和值的类型都是string，使用键值对初始化映射
dict1 := map[string]string{"red": "#da1337", "orange": "#e95a22"}

// 创建一个映射，使用字符串切片作为键，将会报错
//dict2 := map[[]string]int{}
//invilid map key type []string 

// 创建一个映射，使用切片作为值
dict3 := map[string][]string{}
```
映射的键可以是任何值，这个值的类型并不限制，但是切片、函数以及包含切片的结构由于具有引用语义，均不能作为映射的键，否则会编译报错。


## 映射的使用

### 赋值
通过指定适当类型的键并给这个键赋一个值就完成了映射的 键值对 赋值：
```
// 创建一个空映射，用来存储颜色以及颜色对应的十六进制代码
colors := map[string]string{}
// 将red的代码加入到映射
colors["red"] = "#DA1337"
```

### 遍历
和迭代数组或切片一样，使用关键字range可以迭代映射里的所有值：
```go
package main

import "fmt"

func main() {
    // 创建一个映射，存储颜色以及颜色对应的十六进制代码
    colors := map[string]string{
        "AliceBlue": "#f0f8ff",
        "Coral": "#ff7f50",
        "DarkGray": "#a9a9a9",
        "ForestGreen": "#228b22",
    }
    // 显示映射里的所有颜色
    for key, value := range colors {
        fmt.Printf("Key: %s, Value: %s\n", key, value)
    }
}
```
输出：
```text
Key: ForestGreen, Value: #228b22
Key: AliceBlue, Value: #f0f8ff
Key: Coral, Value: #ff7f50
Key: DarkGray, Value: #a9a9a9
```

### 删除

Go语言提供了一个内置函数delete()用于删除容器内的元素：`delete(mymap, "1234")`
```go
package main

import "fmt"

func main() {
    colors := map[string]string{
        "AliceBlue": "#f0f8ff",
        "Coral": "#ff7f50",
        "DarkGray": "#a9a9a9",
        "ForestGreen": "#228b22",
    }
    fmt.Println(colors)
    
    delete(colors, "Coral")
    fmt.Println(colors)
}
```
输出：
```text
ap[AliceBlue:#f0f8ff Coral:#ff7f50 DarkGray:#a9a9a9 ForestGreen:#228b22]
map[AliceBlue:#f0f8ff DarkGray:#a9a9a9 ForestGreen:#228b22]
```


## 将映射传递给函数

在函数间传递映射并不会制造出该映射的一个副本。  
实际上，当传递映射给一个函数，并对这个映射做了修改时，所有对这个映射的引用都会察觉到这个修改：
```go
package main

import "fmt"

func main() {
    // 创建一个映射，存储颜色以及颜色对应的十六进制代码
    colors := map[string]string{
        "AliceBlue": "#f0f8ff",
        "Coral": "#ff7f50",
        "DarkGray": "#a9a9a9",
        "ForestGreen": "#228b22",
    }
    
    // 显示映射里的所有颜色
    for key, value := range colors {
        fmt.Printf("Key: %s, Value: %s\n", key, value)
    }
    
    // 调用函数来移除指定的键
    removeColor(colors, "Coral")
    
    // 显示映射里的所有颜色
    for key, value := range colors {
        fmt.Printf("Key: %s, Value: %s\n", key, value)
    }
}

// removeColor将指定映射里的键删除
func removeColor(colors map[string]string, key string) {
    delete(colors, key)
}
```
输出：
```text
Key: AliceBlue, Value: #f0f8ff
Key: Coral, Value: #ff7f50
Key: DarkGray, Value: #a9a9a9
Key: ForestGreen, Value: #228b22
Key: AliceBlue, Value: #f0f8ff
Key: DarkGray, Value: #a9a9a9
Key: ForestGreen, Value: #228b22
```