# 作用域

作用域是声明常量、变量和函数在源码中的作用范围。

1） 函数外定义的变量称为全局变量，可以整个包或者外部包使用，使用时会优先考虑函数体内的变量。

2） 函数定义中的变量称为形式参数，只作用于函数体内。

3） 函数内定义的变量称为局部变量，只作用于函数体内。

!!! note ""
    在整个包内，可以存在相同变量名，但作用域不同的多个变量。但不建议这么做，容易混淆。

??? note "例1. 包级别的变量能否被函数使用?"
	```go
	package main

	import "fmt"
    
	var number int = 100
    
	func foo() {
	    fmt.Println(number)
	}
    
	func main() {
	    foo()
	}
	```

	输出
	```text
	100
	```

	答: 包级别的变量，也称为全局变量，可以被函数使用

??? note "例2. 包级别的函数有没有顺序要求?"
	```go
	package main

	import "fmt"
    
	var number int = 100
    
	func main() {
	    foo()
	}
    
	func foo() {
	    fmt.Println(number)
	}
	```

	输出
	```text
	100
	```

	答: 包级别的函数没有顺序要求，可以放在包内任意位置，不影响调用

??? note "例3. 一个函数里的变量能否被包级别的另一个函数使用?"
	```go
	package main

	import "fmt"
    
	func foo() {
	    fmt.Println(number)
	}
    
	func main() {
	    var number int = 100
	    foo()
	}
	```

	编译报错
	```text
	undefined: number
	```

	答: 一个函数里的变量不能被包级别的另一个函数使用

??? note "例4. 一个函数里的变量能否被内嵌的函数使用?"
	```go
	package main

	import "fmt"
    
	func main() {
	    var number int = 100
	    foo := func () {
	        fmt.Println(number)
	    }
	    foo()
	}
	```

	输出
	```text
	100
	```

	答: 一个函数里的变量可以被其内内嵌的函数使用

??? note "例5. 函数内变量的声明是否有顺序要求?"
	```go
	package main

	import "fmt"
    
	func main() {
	    foo := func () {
	        fmt.Println(number)
	    }
	    var number int = 100
	    foo()
	}
	```

	报错
	```text
	undefined: number
	```

	答: 非全局变量有顺序要求，使用前必须先定义

??? note "例6. 包级别的变量能否在函数内部的内嵌函数使用?"
	```go
    package main

    import "fmt"
    
	var number int = 100
    
	func main() {
	    foo := func () {
	        number = 200
	    }
	    foo()
	    fmt.Println(number)
	}
	```

	输出
	```text
	200
	```

	答: 可以使用，不然怎么叫全局变量

??? note "例7. 函数嵌套函数，是否可以使用外部定义的变量?"
	```go
    package main

    import "fmt"
    
	func main() {
	    number := 1
	    foo := func () {
	        bar := func () {
	            number = 2
	        }
	        bar()
	    }
	    foo()
	    fmt.Println(number)
	}
	```

	输出
	```text
	2
	```

	答: 可以使用外部变量，且会影响外部变量

??? note "例8. 是否存在不同作用域，但名字相同的变量?"
	```go
	package main

    import "fmt"
    
    func main() {
	    number := 1				// ①
        foo := func () {
            number = 2			// ②
            number := 3			// ③
            number = 4			// ④
            fmt.Println(number)
        }
        foo()
        fmt.Println(number)
    }
	```

	输出
	```text
	4
	2
	```

	答: 存在，
    ①在main函数定义number并初始化为1；
    ②在foo函数内，此前没有声明number，所以自动继承外部函数的number，并将其修改为2；
    ③在foo函数内，重新声明number，并初始化为3；
    ④在foo函数内，③已经定义过新的number，所以修改foo函数内number为4。


??? note "例9. 若一开始就声明同名的变量名，那么还能否使用外部同名变量?"
	```go
	package main

    import "fmt"
    
    func main() {
        number := 1				// ①
        foo := func () {
            number := 3			// ②
            number = 2			// ③
            fmt.Println(number)
        }
        foo()
        fmt.Println(number)
    }
	```

	输出
	```text
	2
	1
	```

	答: 若一开始就声明同名的变量名，则无法再引用外部同名变量

总结:

1. 包级别的函数，写上面或写下面都一样（包括和引用的变量，以及被引用的上层函数，比如变量number的声明写在foo函数声明后，或者foo函数写在main函数上面或者下面），而写在函数里（比如main函数）里的函数（函数值），则和顺序有关，比如在函数里面引用的变量如果是在这个函数上面声明的，则可以引用，如果是在函数下面声明的则提示undefined

2. 如果在函数内部声明一个在函数外部就已经存在的变量，则在函数的这个变量和函数的变量只是名字相同，但实际是2个变量


!!! note ""
    从上面例子可以看出，把函数放到包级别比较好，因为main函数里声明的变量无法直接被这个函数里引用，大大提高安全性！

