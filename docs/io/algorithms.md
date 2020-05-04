# 数据结构与算法

数学方面的包主要是 math，用于实现数学相关的函数计算。其中，math 包含以下子包：

- math/big：大整数的高精度计算实现。有时在进行高精度运算的时候unint64已经无法满足需求，而此包可以满足实现

- math/cmplx：用于复数的基本函数操作

- math/rand：伪随机数生成器


Go语言在数据结构方面主要有三大相关的标准包：

- sort：包含基本的排序方法。支持切片数据排序，以及用户自定义数据集合排序

- index/suffixary：实现了后缀数组相关算法以支持许多常见的字符串操作

- container：实现了三个复杂的数据结构（堆、链表、环）。使用这三个数据结构时不必再从头开始写算法


## 排序

Go语言中实现了四种基本排序算法：插入排序、归并排序、堆排序、快速排序。

在对数据集合排序时，不必去考虑应当选择哪一种排序方法，只要实现了 sort.Interface 接口就可以对数据集合进行排序。接口定义如下：

```text
type Interface interface {
    Len()                   // 获取数据集合长度
    Less(i, j int) bool     // 比较两个元素的大小，i,j 为索引
    Swap(i, j int)          // 交换两个元素的位置，i,j 为索引
}
```
!!! note ""
    sort包 会根据实际数据自动选择高效的排序算法。也就是说，开发者可以不用知道是用插入排序、归并排序、还是堆排序。

Go语言为了对常用数据类型进行操作，sort包 提供了对 []int、[]float64和[]string 的完整支持，包括对切片进行排序、基本数据元素查找、判断切片是否已排好序、对排好序的数据集合进行逆序等。


### 数据集合排序


#### Sort()

数据集合实现了 sort.Interface 接口的三个方法后，即可调用该包的 Sort() 方法进行排序。Sort() 方法定义如下：

```text
func Sort(data Interface)
```
Sort() 方法唯一的参数就是待排序的数据集合。


代码示例：
```go
package main

import (
	"fmt"
	"sort"
)

type slice []int

// Len()：长度
func (s slice) Len() int {
	return len(s)
}

// Less()：由低到高排序
func (s slice) Less(i, j int) bool {
	return s[i] < s[j]
}

// Swap()：排序
func (s slice) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}

func main() {
	s := slice{10, 30, 28, 40, 90, 70}
	fmt.Println(s)

	sort.Sort(s)
	fmt.Println(s)

	// 判断是否已经排序
	fmt.Println("是否已经排好序？", sort.IsSorted(s))
}
```
输出：
```text
[10 30 28 40 90 70]
[10 28 30 40 70 90]
是否已经排好序？ true
```


#### Reverse()

sort包提供了 Reverse() 方法，可以将数据按 Less() 定义的排序方式逆序排序。方法定义如下：

```text
func Reverse(data Interface) Interface
```

Reverse() 内部实现如下：

```text
// 定义了一个reverse结构体，嵌入 Interface 接口
type reverse struct {
    Interface
}

// reverse 结构体的 Less() 方法拥有嵌入的 Less() 方法相反的行为
// Len() 和 Swap() 方法则会保持嵌入类型的方法行为
func (r reverse) Less(i, j int) bool {
    return r.Interface.Less(j, i)
}

// 返回新的实现 Interface 接口的数据类型
func Reverse(data Interface) Interface {
    return &reverse{data}
}
```

!!! note ""
    说到底，Reverse() 逆序的关键是在于 Less() 方法里面的判断。也就是说，调整Less()判断条件 等价于 使用Reverse()方法 !


示例1，调整 Less() 判断条件：
```go
package main

import (
	"fmt"
	"sort"
)

type slice []int

func (s slice) Len() int {
	return len(s)
}

func (s slice) Less(i, j int) bool {
	return s[i] >= s[j]
}

func (s slice) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}

func main() {
	s := slice{10, 30, 28, 40, 90, 70}
	fmt.Println(s)

	sort.Sort(s)
	fmt.Println(s)
}
```
输出：
```text
[10 30 28 40 90 70]
[90 70 40 30 28 10]
```

示例2，使用Reverse()方法：
```go
package main

import (
	"fmt"
	"sort"
)

type slice []int

func (s slice) Len() int {
	return len(s)
}

func (s slice) Less(i, j int) bool {
	return s[i] < s[j]
}

func (s slice) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}

func main() {
	s := slice{10, 30, 28, 40, 90, 70}
	fmt.Println(s)

	sort.Sort(sort.Reverse(s))
	fmt.Println(s)
}
```
输出：
```text
[10 30 28 40 90 70]
[90 70 40 30 28 10]
```


#### Search()

Search() 使用二分法来搜索某指定切片[0:n]，并返回能够使 f(i)=true 的最小的i值（0<=i<=n）。方法定义如下：

```text
func Search(n int, f func(int) bool) int
```

!!! note ""
    切片必须是已经升序排好序了的，否则可能找不到元素！

代码示例：
```go
package main

import (
	"fmt"
	"sort"
)

type slice []int

func (s slice) Len() int {
	return len(s)
}

func (s slice) Less(i, j int) bool {
	return s[i] < s[j]
}

func (s slice) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}

func main() {
	x := 11
	s := slice{6, 8, 10, 7, 3, 10, 15, 11, 19, 18}
	fmt.Printf("s：%v\n", s)
	pos := sort.Search(len(s), func(i int) bool {
		return s[i] >= x
	})
	if pos < len(s) && s[pos] == x {
		fmt.Printf("%d 在 s 中的位置为：%d\n", x, pos)
	} else {
		fmt.Println("s 中不包含元素：", x)
	}

	sort.Sort(s)
	fmt.Printf("s：%v\n", s)
	pos2 := sort.Search(len(s), func(i int) bool {
		return s[i] >= x
	})
	if pos2 < len(s) && s[pos2] == x {
		fmt.Printf("%d 在 s 中的位置为：%d\n", x, pos2)
	} else {
		fmt.Println("s 中不包含元素：", x)
	}
}
```
输出：
```text
s：[6 8 10 7 3 10 15 11 19 18]
s 中不包含元素： 11
s：[3 6 7 8 10 10 11 15 18 19]
11 在 s 中的位置为：6
```

猜数字游戏：
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	var s string
	fmt.Printf("选择一个从0到100的整数\n")
	answer := sort.Search(100, func(i int) bool {
		fmt.Printf("这个数是否小于等于 %d ? y-是，n-不是", i)
		fmt.Scanf("%s", &s)
		return s != "" && s[0] == 'y'
	})

	fmt.Printf("你选的数字是 %d 吧。", answer)
}
```


### 切片排序

sort包支持 []int、[]float64、[]string 三种内建数据类型切片的排序操作。我们不必再去实现相关的 Len()、Less()和Swap()方法。


#### []int 排序

sort包定义了一个 IntSlice类型 ，并且实现了 sort.Interface 接口，内部实现如下：

```text
type IntSlice []int

func (p IntSlice) Len() int { return len(p) }
func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p IntSlice) Swap(i, j int) { p[i], p[j] = p[j], p[i] }
// IntSlice 类型定义了Sort()函数，包装了sort.Sort()函数
func (p IntSlice) Sort() { Sort(p) }
// IntSlice 类型定义了SearchInts()函数，包装了sort.SearchInts()函数
func (p IntSlice) Search(x int) int { return SearchInts(p, x) }
```

代码示例：
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	s := []int{3, 2, 6, 9, 4, 8}
	fmt.Println("未排序：", s)
	
	// 升序
	sort.Ints(s)
	fmt.Println("升序：", s)
	
	// 降序
	sort.Sort(sort.Reverse(sort.IntSlice(s)))
	fmt.Println("降序：", s)
	
	// 查找 4 的位置
	x := 4
	s2 := []int{3, 2, 6, 9, 4, 8}
	sort.Ints(s2)
	pos := sort.SearchInts(s2, x)
	fmt.Printf("在 %v 中 %d 的位置是：%d", s2, x, pos)
}
```
输出：
```text
未排序： [3 2 6 9 4 8]
升序： [2 3 4 6 8 9]
降序： [9 8 6 4 3 2]
在 [2 3 4 6 8 9] 中 4 的位置是：2
```


#### []float64 排序

实现和 Ints 类似，Less() 方法有些许不同，在这里使用了 isNaN() 判断。内部实现如下：

```text
type Float64Slice []float64

func (p Float64Slice) Len() int { return len(p) }
func (p Float64Slice) Less(i, j int) bool { return p[i] < p[j] || isNaN(p[i]) || !isNaN(p[j]) }
func (p Float64Slice) Swap(i, j int) { p[i], p[j] = p[j], p[i] }
func (p Float64Slice) Sort() { Sort(p) }
func (p Float64Slice) Search(x float64) int { return SearchFloat64(p, x) }
```

!!! note ""
    isNaN() 和 math包中的 IsNaN() 实现相同，sort包 不使用math.IsNaN()，完全是基于包依赖性考虑。sort包实现不依赖任何包。

代码示例：
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	s := []float64{30, 20, 16, 19, 40, 18}
	fmt.Println("未排序：", s)

	// 升序
	sort.Float64s(s)
	fmt.Println("升序：", s)

	// 降序
	sort.Sort(sort.Reverse(sort.Float64Slice(s)))
	fmt.Println("降序：", s)

	// 查找 4 的位置
	var x float64 = 20
	s2 := []float64{30, 20, 16, 19, 40, 18}
	sort.Float64s(s2)
	pos := sort.SearchFloat64s(s2, x)
	fmt.Printf("在 %v 中 %f 的位置是：%d", s2, x, pos)
}
```
输出：
```text
未排序： [30 20 16 19 40 18]
升序： [16 18 19 20 30 40]
降序： [40 30 20 19 18 16]
在 [16 18 19 20 30 40] 中 20.000000 的位置是：3
```


#### []string 排序

两个string对象之间的大小比较是基于 "字典序" 的。实现与 Ints 类似，内部实现如下：

```text
type StringSlice []string

func (p StringSlice) Len() int { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int) { p[i], p[j] = p[j], p[i] }
func (p StringSlice) Sort() { Sort(p) }
func (p StringSlice) Search(x string) int { return SearchStrings(p, x) }
```

代码示例：
```go
package main

import (
	"fmt"
	"sort"
)

func main() {
	s := []string{"dell", "think pad", "mac pro", "ipad", "surface", "mac air"}
	fmt.Println("未排序：", s)
	
	// 升序
	sort.Strings(s)
	fmt.Println("升序：", s)
	
	// 降序
	sort.Sort(sort.Reverse(sort.StringSlice(s)))
	fmt.Println("降序：", s)
	
	// 查找 mac pro 的位置
	x := "mac pro"
	s2 := []string{"dell", "think pad", "mac pro", "ipad", "surface", "mac air"}
	sort.Strings(s2)
	pos := sort.SearchStrings(s2, x)
	fmt.Printf("在 %v 中 %s 的位置是：%d", s2, x, pos)
}
```
输出：
```text
未排序： [dell think pad mac pro ipad surface mac air]
升序： [dell ipad mac air mac pro surface think pad]
降序： [think pad surface mac pro mac air ipad dell]
在 [dell ipad mac air mac pro surface think pad] 中 mac pro 的位置是：3
```


## container

### 堆

堆 使用的数据结构是最小二叉树，即根结点比左边子树和右边子树的所有值都小。Go语言中的堆包只是实现了一个接口，接口定义如下：

```text
type Interface interface {
    sort.Interface
    Push(x interface{}) // 添加一个元素 x 并返回 Len()
    Pop() interface{}   // 移除并返回元素长度 Len()-1
}
```
通过接口定义可以看出，堆 继承自 sort.Interface，而 sort.Interface 需要实现三个方法：Len()、Less()、Swap()，加上 堆 接口定义的两个方法：Push()和Pop()。也就是说，定义一个堆就要实现五个方法。

代码示例：
```go
package main

import (
	"container/heap"
	"fmt"
)

type IntHeap []int

func (h IntHeap) Len() int {
	return len(h)
}

func (h IntHeap) Less(i, j int) bool {
	return h[i] < h[j]
}

func (h IntHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0:n-1]
	return x
}

func main() {
	ih := &IntHeap{2, 1, 5, 6}
	fmt.Println("未初始化：", ih)
	heap.Init(ih)
	fmt.Println("初始化后：", ih)

	heap.Push(ih, 9)
	fmt.Println("Push一个元素后：", ih)

	v := heap.Pop(ih)
	fmt.Printf("Pop一个元素：%d，剩余堆的值为：%v", v, ih)
}
```
输出：
```text
未初始化： &[2 1 5 6]
初始化后： &[1 2 5 6]
Push一个元素后： &[1 2 5 6 9]
Pop一个元素：1，剩余堆的值为：&[2 6 5 9]
```


### 链表

链表，就是一个有 prev 和 next 指针的数组，它维护两个type。链表不是接口，而是结构。结构定义如下：

```text
type Element struct {
    next, prev *Element     // 上一个元素和下一个元素
    list *List              // 元素所在的链表
    Value interface{}       // 元素
}

type List struct {
    root Element    // 链表的根元素
    len int         // 链表的长度
}
```

链表list对应的方法如下：
```text
type Element：
func (e *Element) Next() *Element
func (e *Element) Prev() *Element

type List：
func New() *List
func (l *List) Back() *Element  // 最后一个元素
func (l *List) Front() *Element // 第一个元素
func (l *List) Init() *List     // 链表初始化
func (l *List) InsertAfter(v interface{}, mark *Element) *Element   // 在某个元素后插入
func (l *List) InsertBefore(v interface{}, mark *Element) *Element  // 在某个元素前插入
func (l *List) Len() int                            // 链表长度
func (l *List) MoveAfter(e, mark *Element)          // 把e元素移动到mark之后
func (l *List) MoveBefore(e, mark *Element)         // 把e元素移动到mark之前
func (l *List) MoveToBack(e *Element)               // 把e元素移动到队列的末尾
func (l *List) MoveToFront(e *Element)              // 把e元素移动到队列的头部
func (l *List) PushBack(v interface{}) *ELement     // 在队列末尾插入元素
func (l *List) PushBackList(other *List)            // 在队列末尾连接新队列
func (l *List) PushFront(v interface{}) *Element    // 在队列头部插入元素
func (l *List) PushFrontList(other *List)           // 在队列头部插入新的队列
func (l *List) Remove(e *Element) interfaceP{}      // 删除某个元素
```

链表是先创建 list ，然后往 list 中插入值， list 就在内部创建一个 Element ，并在内部设置好 Element 的 next 和 prev 等。

代码示例：
```go
package main

import (
	"container/list"
	"fmt"
)

func main() {
	listEle := list.New()
	listEle.PushBack(1)
	listEle.PushBack(2)

	fmt.Printf("长度：%d\n", listEle.Len())
	fmt.Printf("第一个元素：%#v\n", listEle.Front())
	fmt.Printf("第二个元素：%#v\n", listEle.Front().Next())
}
```
输出：
```text
长度：2
第一个元素：&list.Element{next:(*list.Element)(0xc0000841e0), prev:(*list.Element)(0xc000084180), list:(*list.List)(0xc000084180), Value:1}
第二个元素：&list.Element{next:(*list.Element)(0xc000084180), prev:(*list.Element)(0xc0000841b0), list:(*list.List)(0xc000084180), Value:2}
```


### 环

环的结构比较特殊，环的尾部就是头部（有点像衔尾蛇），所以每个元素实际上就可以代表自身的这个环。环 不需要像 list 那样保持 list 和 element 两个结构，只需保持一个结构即可。
结构定义如下：

```text
type Ring struct {
    next, prev *Ring
    Value interface{}
}
```

环对应的方法如下：
```text
type Ring：
func New(n int) *Ring                       // 初始化环
func (r *Ring) Do(f func(interface{}))      // 循环环进行操作
func (r *Ring) Len() int                    // 获取环长度
func (r *Ring) Link(s *Ring) *Ring          // 连接另一个环
func (r *Ring) Move(n int) *Ring            // 指针从当前元素向后或向前移动，n代表移动步数（负数代表向前）
func (r *Ring) Next() *Ring                 // 当前元素的下一个元素
func (r *Ring) Prev() *Ring                 // 当前元素的前一个元素
func (r *Ring) Unlink(n int) *Ring          // 从当前元素开始，删除n个元素（不包含当前元素）
```

代码示例：
```go
package main

import (
	"container/ring"
	"fmt"
)

func main() {
	rLen := 3

	// 创建新的Ring
	r := ring.New(rLen)

	// 循环赋值
	for i := 0; i < rLen; i++ {
		r.Value = i
		r = r.Next()
	}

	fmt.Print("环的长度：", r.Len())

	printRing := func(v interface{}) {
		fmt.Print(v, " ")
	}

	// 循环输出环中的数据
	fmt.Printf("\n循环输出环中的数据：")
	r.Do(printRing)

	// 将r往前移动2个位置，即r+2
	result := r.Move(2)

	fmt.Printf("\n移动前：")
	r.Do(printRing)     // 打印删除 r+1 后的环

	fmt.Printf("\n前移2个位置后：")
	result.Do(printRing)// 打印删除的 r+1 元素

	another := ring.New(rLen)
	another.Value = 7
	another.Next().Value = 8    // 给another+1的元素进行赋值，也就是第二个元素
	another.Prev().Value = 9    // 给another-1的元素进行赋值，因为是环，-1 就是 末尾，也就是第三个元素

	fmt.Printf("\n打印another的数据：")
	another.Do(printRing)       // 打印another的数据

	// 将another插入到r的后面
	result = r.Link(another)
	fmt.Printf("\n插入another的环前：")
	r.Do(printRing)

	fmt.Printf("\n插入another的环后：")
	result.Do(printRing)

	// 删除r之后的两个元素
	result = r.Unlink(2)
	fmt.Printf("\nr的剩余元素：")
	r.Do(printRing)

	fmt.Printf("\nr后被删除的两个元素：")
	result.Do(printRing)
}
```
输出：
```text
环的长度：3
循环输出环中的数据：0 1 2 
移动前：0 1 2 
前移2个位置后：2 0 1 
打印another的数据：7 8 9 
插入another的环前：0 7 8 9 1 2 
插入another的环后：1 2 0 7 8 9 
r的剩余元素：0 9 1 2 
r后被删除的两个元素：7 8
```


模拟约瑟夫环：
```go
package main

import (
	"container/ring"
	"fmt"
)

type Player struct {
	position 	int		// 位置
	alive 		bool	// 是否存活：true-存活
}

func main() {
	// 设置玩家人数
	var playerCount int
	fmt.Printf("请输入玩家人数？\n")
	fmt.Scanf("%d", &playerCount)

	// 设置死亡编号
	var deadline int
	for {
		if deadline > 1 {
			break
		}
		fmt.Printf("请输入死亡编号？（>=%d）\n", 2)
		fmt.Scanf("%d", &deadline)
	}

	// 设置幸存人数
	var survivorCount int
	for {
		if survivorCount >= 1 && survivorCount < playerCount {
			break;
		}
		fmt.Printf("请输入最后的幸存者人数？（%d~%d）", 1, playerCount-1)
		fmt.Scanf("%d", &survivorCount)
	}

	// 创建游戏环
	r := ring.New(playerCount)

	// 设置玩家编号
	fmt.Printf("现在从1开始给玩家进行编号!\n")
	for i := 1; i <= playerCount; i++ {
		r.Value = &Player{i, true}
		r = r.Next()
	}

	// 设置开始报数的位置编号
	var startPos int
	for {
		if startPos >= 1 && startPos <= playerCount {
			break
		}
		fmt.Printf("请输入开始报数的玩家编号？（%d~%d）\n", 1, playerCount)
		fmt.Scanf("%d", &startPos)
	}

	// 如果不是从1开始，则向前移动环
	if startPos > 1 {
		r = r.Move(startPos - 1)
	}

	counter := 1		// 从1开始记录，用于判断是否到达deadline
	deadCount := 0		// 死亡统计
	peopleCount := playerCount - survivorCount  // 剩余人数

	// 循环报数，中者离场
	fmt.Printf("游戏开始：\n")
	for deadCount < peopleCount {
		// 报数转给下一个人
		r = r.Next()

		// 如果是活着的人，记录数值
		if r.Value.(*Player).alive == true {
			counter++
		}

		// 命中，直接死亡
		if counter == deadline {
			r.Value.(*Player).alive = false
			fmt.Printf("player %d died!\n", r.Value.(*Player).position)
			deadCount++
			counter = 0
		}
	}
}
```