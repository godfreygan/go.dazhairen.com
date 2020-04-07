# 字符串处理


## 字符串连接

可以使用 "+" 符号连接字符串

示例：
```go
package main

import "fmt"

func main() {
	s := "abcd世界"
	str1 := s[4:] + "你好"
	fmt.Println(str1)
	
	str2 := "你好" + "Go语言"
	fmt.Println(str2)
}
```


## 字符串遍历

字符串的遍历和数组差不多，也可以 for 和 for-range。

```go
package main

import "fmt"

func main() {
	str := "你好Go语言"
	
	// 因字符编码而出现乱码
	for i := 0; i < len(str); i++ {
		fmt.Printf("%c", str[i])
	}
	
	fmt.Println()
	
	// 按完成字符遍历
	for _, val := range str {
		fmt.Printf("%c", val)
	}
}
```


## 字符串修改

Go语言中，字符串内容不可修改，也不能使用 str[i] 这种方式修改字符串。如果一定要修改，那么可以将字符串的内容复制到另一个可写的变量中再修改。

```go
package main

import "fmt"

func main() {
	str := "你好 世界"
	b := []byte(str)
	b[6] = ','
	fmt.Printf("%s\n", str)
	fmt.Printf("%s\n", b)
}
```


## strings包

### 包含判断

判断一个字符串中是否有相应的某个子字符串。

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Hello World"
	fmt.Printf("字符串 %s 中的前缀是否包含: %s ?\n", str, "He")
	fmt.Println(strings.HasPrefix(str, "He"))
	
	fmt.Printf("字符串 %s 中的后缀是否包含：%s ?\n", str, "rld")
	fmt.Println(strings.HasSuffix(str, "rld"))
	
	fmt.Printf("字符串 %s 中是否包含：%s ？\n", str, "Wo")
	fmt.Println(strings.Contains(str, "Wo"))
	
	fmt.Printf("字符串 %s 中是否包含：%s 或 %s ？\n", str, "abcd", "e & f")
	fmt.Println(strings.ContainsAny("abcd", "a & g"))
}
```

### 索引

Go语言中，字符串中的字符都有一个索引值，我们要操作字符串，必须获取在字符串中的索引值。在strings包中Index就可以返回字符串的第一个字符的索引值，如果不存在相应的字符串则返回-1。

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "this is my golang demo"
	fmt.Printf("%s 的位置是？\n", "my")
	fmt.Println(strings.Index(str, "my"))
	
	fmt.Printf("%s 的第一个单词的位置是？\n", "is")
	fmt.Println(strings.Index(str, "is"))
	
	fmt.Printf("%s 的最后一个单词的位置是？\n", "is")
	fmt.Println(strings.LastIndex(str, "is"))
	
	fmt.Printf("%s 的位置是？\n", "php")
	fmt.Println(strings.Index(str, "php"))
}
```

### 替换

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "hello php, php is the best!"
	old := "php"
	new := "go"
	n := 1
	fmt.Println(strings.Replace(str, old, new, n))
}
```

### 统计

#### 出现频率

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	str := "Golang is the best, It look so cool!"
	var matchOne = "o"
	fmt.Println(strings.Count(str, matchOne))
	fmt.Println(strings.Count(str, "oo"))
}
```

#### 字符数量

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
	str := "你好世界"
	
	//fmt.Println(len(str))         // 非英文，不能直接取字符串的长度
	
	fmt.Println(len([]rune(str)))
	
	fmt.Println(utf8.RuneCountInString(str))
}
```

### 大小写转换

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
    var orig = "hello world!"
    var lower string
    var upper string

    fmt.Printf("%s\n", orig)
    
    lower = strings.ToLower(orig)
    fmt.Printf("%s\n", lower)
    
    upper = strings.ToUpper(orig)
    fmt.Printf("%s\n", upper)
}
```

### 剪切

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str string = " !!! Hello World !!! "
	
	fmt.Println(strings.Trim(str, "! "))
	fmt.Println(strings.Trim(str, " ! "))
	fmt.Println(strings.Trim(str, "!"))
	
	fmt.Println()
	
	fmt.Println(strings.TrimLeft(str, "! "))
	fmt.Println(strings.TrimLeft(str, " ! "))
	fmt.Println(strings.TrimLeft(str, "!"))
	
	fmt.Println()
	
	fmt.Println(strings.TrimSpace("世界 \t 你好 \n Go语言"))
}
```

### 分割

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	fmt.Println(strings.Split("a,b,c,d", ","))
	
	fmt.Println(strings.Split("the day the mountain the people the dog", "the "))
    
    fmt.Println(strings.Split("tom", ""))
}
```

### 插入字符

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str string = "chinese is the best language in the world"
	
	strSli := strings.Fields(str)

	fmt.Println(strSli)
	
	fmt.Println()
	
	for _, val := range strSli {
		fmt.Printf("%s ", val)
	}
	
	fmt.Println()
	fmt.Println()
	
	str2 := strings.Join(strSli, "、")
	fmt.Println(str2)
}
```


## strconv包

主要用于字符串与其它基本类型的转换。

### ParseInt

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	v32 := "-354634382"
	if s, err := strconv.ParseInt(v32, 10, 32); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}

	if s, err := strconv.ParseInt(v32, 16, 32); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}

	v64 := "-3546343826724305832"
	if s, err := strconv.ParseInt(v64, 10, 64); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}

	if s, err := strconv.ParseInt(v64, 16, 64); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}
}
```

### ParseUint

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	v32 := "354634382"
	if s, err := strconv.ParseUint(v32, 10, 32); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}

	if s, err := strconv.ParseUint(v32, 16, 32); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}

	v64 := "3546343826724305832"
	if s, err := strconv.ParseUint(v64, 10, 64); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}

	if s, err := strconv.ParseUint(v64, 16, 64); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}
}
```

### Atoi

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	var str string = "100"
	if s, err := strconv.Atoi(str); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%T, %v\n", s, s)
	}
}
```
