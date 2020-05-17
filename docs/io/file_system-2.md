# path：兼容路径操作

path/filepath 包涉及路径操作时，路径分隔符使用 os.PathSeparator，不同系统的路径表示方式有所不同，Path 包能处理所有的文件路径，不管是什么系统。

!!! note ""
    路径操作函数并不会去校验路径是否真是存在。


## 解析路径名字符串

Dir() 和 Base() 函数将一个路径字符串分解成目录和文件名两部分（其实是和 UNIX 中的 dirname 和 basename 命令类似）。但是，如果路径以 "/" 结尾，Dir() 的行为和 dirname 就不太一致。

方法定义如下：

```text
func Dir(path string) string
func Base(path string) string
```

Dir()：  
返回路径中除去最后一个路径元素的部分，也就是该路径最后一个元素所在的目录。在使用 Split 去除最后一个元素后，会简化路径并去掉末尾的 "/"。如果路径是空字符串，会返回 "."，如果路径是 "/"，则返回 "/"，其它情况都不会返回以 "/" 结尾的路径。

Base()：  
返回路径的最后一个元素。在提取元素前会去掉末尾的 "/"。如果路径是空，同样会返回 "."，如果路径只有一个 "/"，同样会返回 "/"。


??? note "例1. 路径名为：/Users/godfrey/data/www/go/example.go"
    ```go
    package main

    import (
        "fmt"
        "path/filepath"
    )
    
    func main() {
        path := "/Users/godfrey/data/www/go/example.go"
    
        dir := filepath.Dir(path)
        fmt.Printf("Dir() 返回：%s\n", dir)
    
        base := filepath.Base(path)
        fmt.Printf("Base() 返回：%s\n", base)
    } 
    ```
    
    输出：
    ```text
    dir：/Users/godfrey/data/www/go
    base：example.go
    ```

??? note "例2：路径名为：/Users/godfrey/data/www/go/"
    ```go
    package main

    import (
        "fmt"
        "path/filepath"
    )
    
    func main() {
        path := "/Users/godfrey/data/www/go/"
    
        dir := filepath.Dir(path)
        fmt.Printf("dir：%s\n", dir)
    
        base := filepath.Base(path)
        fmt.Printf("base：%s\n", base)
    }
    ```
    
    输出：
    ```text
    dir：/Users/godfrey/data/www/go
    base：go
    ```

??? note "例3：路径名为空"
    ```go
    package main

    import (
        "fmt"
        "path/filepath"    
    )
    
    func main() {
        path := ""
    
        dir := filepath.Dir(path)
        fmt.Printf("dir：%s\n", dir)
    
        base := filepath.Base(path)
        fmt.Printf("base：%s\n", base)                
    }
    ```
    
    输出：
    ```text
    dir：.
    base：.
    ```

??? note "例4：路径名为：/"
    ```go
    package main

    import (
        "fmt"
        "path/filepath"    
    )
    
    func main() {
        path := "/"
        
        dir := filepath.Dir(path)
        fmt.Printf("dir：%s\n", dir)
    
        base := filepath.Base(path)
        fmt.Printf("base：%s\n", base)                    
    }
    ```
    
    输出：
    ```text
    dir：/
    base：/
    ```



此外，Ext() 可以获取文件的扩展名，函数定义如下：

```text
func Ext(path string) string
```

Ext() 函数返回 path 文件扩展名。扩展名是路径中最后一个从 "." 开始的部分，包括 "." 。如果该路径没有 "." ，会返回空字符串。

示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	path1 := "/Users/godfrey/data/www/go/example.go"
	fmt.Printf("文件扩展名为：%s\n", filepath.Ext(path1))
	
	path2 := "/Users/godfrey/data/www/go"
	fmt.Printf("文件扩展名为：%s\n", filepath.Ext(path2))
}
```
输出：
```text
文件扩展名为：.go
文件扩展名为：
```


## 相对路径和绝对路径


### IsAbs()

每个进程都会有当前工作目录，一般的相对路径就是针对进程当前工作目录而言的。当然，可以针对某个目录指定相对路径。

绝对路径，在 UNIX 中以 "/" 开始，在 Windows 中以某个盘符开始，比如 C:\User 等。IsAbs() 可以判断返回的路径是否是一个绝对路径。方法定义如下：

```text
func IsAbs(path string) bool
```

代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	path1 := "./a/b/c"
	fmt.Printf("路径：%s，是绝对路径：%v\n", path1, filepath.IsAbs(path1))
	
	path2 := "/a/b/c"
	fmt.Printf("路径：%s，是绝对路径：%v\n", path2, filepath.IsAbs(path2))
}
```
输出：
```text
路径：./a/b/c，是绝对路径：false
路径：/a/b/c，是绝对路径：true
```


### Abs()

Abs() 函数会返回 path 代表的绝对路径，如果输入参数 path 不是绝对路径，会加入当前工作目录来使其成为绝对路径。方法定义如下：

```text
func Abs(path string) (string, error)
```

因为硬链接的存在，不能保证返回的绝对路径是唯一指向该地址的绝对路径。在 os.Getwd 出错时，Abs() 会返回该错误。如果路径名长度太长超过系统限制，也会报错。

代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	path1 := "/a/b/c"
	absPath1, err := filepath.Abs(path1)
	if err != nil {
		panic(err)
	}
	fmt.Printf("路径：%s，绝对路径是：%s\n", path1, absPath1)

	path2 := "./a/b/c"
	absPath2, err := filepath.Abs(path2)
	if err != nil {
		panic(err)
	}
	fmt.Printf("路径：%s，绝对路径是：%s\n", path2, absPath2)
}
```
输出：
```text
路径：/a/b/c，绝对路径是：/a/b/c
路径：./a/b/c，绝对路径是：/Users/godfrey/data/www/private/go.dazhairen.com/a/b/c
```


### Rel()

Rel() 函数返回一个相对路径。方法定义如下：

```text
func Rel(basepath string, targetpath string) (string, error)
```

代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	basePath1 := "/a/b/c"
	targetPath1 := "/d/c"

	relPath, err := filepath.Rel(basePath1, targetPath1)
	if err != nil {
		panic(err)
	}
	fmt.Printf("basepath：%s，targetpath：%s，相对路径为：%s\n", basePath1, targetPath1, relPath)

	basePath2 := "/a/b/c"
	targetPath2 := "/a/b/c/d/example.go"

	relPath2, err := filepath.Rel(basePath2, targetPath2)
	if err != nil {
		panic(err)
	}
	fmt.Printf("basepath：%s，targetpath：%s，相对路径为：%s\n", basePath2, targetPath2, relPath2)
}
```
输出：
```text
basepath：/a/b/c，targetpath：/d/c，相对路径为：../../../d/c
basepath：/a/b/c，targetpath：/a/b/c/d/example.go，相对路径为：d/example.go
```


## 路径的切分和拼接


### Split()

对于一个常规的文件路径，可以通过 Split 函数得到它的目录路径和文件名，方法定义如下：

```text
func Split(path string) (dir, file string)
```

Split() 根据最后一个路径分隔符将路径 path 分隔为目录和文件名两部分（dir 和 file）。如果 path 中没有路径分隔符，函数返回值 dir 为空字符串，file 等于 path。如果 path 中最后一个字符是 "/"，则 dir 等于 path，file 为空字符串。
返回值满足 path = dir + file。另外，dir 为空时，最后一个字符必然是 "/"。

代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	path1 := "/a/b/c/d"
	dir1, file1 := filepath.Split(path1)
	fmt.Printf("path1：%q，dir1：%q，file1：%q\n", path1, dir1, file1)

	path2 := "/a/b/c/d/"
	dir2, file2 := filepath.Split(path2)
	fmt.Printf("path2：%q，dir2：%q，file2：%q\n", path2, dir2, file2)

	path3 := "a"
	dir3, file3 := filepath.Split(path3)
	fmt.Printf("path3：%q，dir3：%q，file3：%q\n", path3, dir3, file3)

	path4 := "/"
	dir4, file4 := filepath.Split(path4)
	fmt.Printf("path4：%q，dir4：%q，file4：%q\n", path4, dir4, file4)
}
```
输出：
```text
path1："/a/b/c/d"，dir1："/a/b/c/"，file1："d"
path2："/a/b/c/d/"，dir2："/a/b/c/d/"，file2：""
path3："a"，dir3：""，file3："a"
path4："/"，dir4："/"，file4：""
```


### Join()

将任意数量的路径元素合并为一个路径，忽略空元素，清理多余字符。方法定义如下：

```text
func Join(elem ...string) string
```

代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	path := filepath.Join("a", "///", "b", "", ":::", " ", `//c///d/////`)
	fmt.Printf("路径为：%s", path)
}
```
输出：
```text
路径为：a/b/:::/ /c/d
```


## 规整化路径

Clean() 函数通过单纯的词法操作返回和 path 代表同一地址的简短地址。方法定义如下：

```text
func Clean(path string) string
```

该函数会不断执行如下规则，直到不能再进行处理：

1）将连续的多个路径分隔符替换为单个路径分隔符。如 "///" 替换为 "/"

2）剔除每一个 "." 路径名元素（也就是当前目录），实际上是剔除 "./"。如 "./a/./b" 剔除后为 "a/b"

3）剔除每一个路径内的 ".." 路径名元素（也就是父级目录）和它前面的非 ".." 路径名元素。如 "/a/b/../c" 剔除后为 "/a/c"

4）剔除开始于跟路径的 ".." 路径名元素，也就是将路径开始处的 "/.." 替换为 "/"。如 "/../a/b/c" 剔除后为 "/a/b/c"

5）如果路径是空，则返回 "."，表示当前路径


代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	path1 := "/a///b//c"
	fmt.Printf("before clean：%q，after clean：%q\n", path1, filepath.Clean(path1))

	path2 := "./a/./b/./c"
	fmt.Printf("before clean：%q，after clean：%q\n", path2, filepath.Clean(path2))

	path3 := "/a/b/../c/../d"
	fmt.Printf("before clean：%q，after clean：%q\n", path3, filepath.Clean(path3))

	path4 := "/../a/b/c"
	fmt.Printf("before clean：%q，after clean：%q\n", path4, filepath.Clean(path4))

	path5 := ""
	fmt.Printf("before clean：%q，after clean：%q\n", path5, filepath.Clean(path5))
}
```
输出：
```text
before clean："/a///b//c"，after clean："/a/b/c"
before clean："./a/./b/./c"，after clean："a/b/c"
before clean："/a/b/../c/../d"，after clean："/a/d"
before clean："/../a/b/c"，after clean："/a/b/c"
before clean：""，after clean："."
```


## 文件路径匹配


###  Match()

Match() 判断 name 是否和指定的模式 pattern 完全匹配。方法定义如下：

```text
func Match(pattern, name string) (matched bool, err error)
```

匹配要求 pattern 必须和 name 完全匹配上，不只是子串。pattern 规则说明：

- 可以使用 "？" 匹配单个任意字符（不匹配路径分隔符）

- 可以使用 "*" 匹配0个或多个任意字符（不匹配路径分隔符）

- 可以使用 "[]" 匹配范围内的任意一个字符（可以包含路径分隔符）

- 可以使用 "[^]" 匹配范围外的任意一个字符（无需包含路径分隔符）

- "[]" 内可以使用 "-" 表示一个区间，如 "[a-z]" 表示 a-z 之间的任一字符

- 反斜杠 "\\" 用来匹配实际的字符，如 "\\*" 匹配 *，"\\a" 匹配 a等


代码示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	matched, err := filepath.Match(`???`, `abc`)
	fmt.Printf("line 1，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`???`, `abcd`)
	fmt.Printf("line 2，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`*`, `abc`)
	fmt.Printf("line 3，matched：%v，err：%v\n", matched, err)
	
	matched, err = filepath.Match(`*`, ``)
	fmt.Printf("line 4，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`a*`, `abc`)
	fmt.Printf("line 5，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`???\\???`, `abc\def`)
	fmt.Printf("line 6，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`???/???`, `abc/def`)
	fmt.Printf("line 7，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`/*/*/*/`, `/a/b/c/`)
	fmt.Printf("line 8，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`[aA][bB][cC]`, `aBc`)
	fmt.Printf("line 9，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`[^aA]*`, `abc`)
	fmt.Printf("line 10，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`[a-z]*`, `a+b`)
	fmt.Printf("line 11，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`\[*\]`, `[a+b]`)
	fmt.Printf("line 12，matched：%v，err：%v\n", matched, err)

	matched, err = filepath.Match(`[[\]]*[[\]]`, `[]`)
	fmt.Printf("line 13，matched：%v，err：%v\n", matched, err)
}
```
输出：
```text
line 1，matched：true，err：<nil>
line 2，matched：false，err：<nil>
line 3，matched：true，err：<nil>
line 4，matched：true，err：<nil>
line 5，matched：true，err：<nil>
line 6，matched：true，err：<nil>
line 7，matched：true，err：<nil>
line 8，matched：true，err：<nil>
line 9，matched：true，err：<nil>
line 10，matched：false，err：<nil>
line 11，matched：true，err：<nil>
line 12，matched：true，err：<nil>
line 13，matched：true，err：<nil>
```


### Glob()

Golb() 函数返回所有匹配了模式字符串 pattern 的文件列表，匹配不到文件则返回 nil。pattern的语法和 Match 函数相同。方法定义如下：

```text
func Glob(pattern string) (matches []string, err error)
```

!!! note ""
    Glob() 会忽略任何文件系统相关错误，如读目录引发的I/O错误。其它的错误和 Match() 一样，在pattern不合法时，返回 filepath.ErrBadPattern。

Glob() 常见的用法是用于读取某个目录下的相关文件，示例：
```go
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	// 列出 /Users/godfrey/data/go/ 目录下以go开头的所有项目（不区分大小写）
	list, err := filepath.Glob("/Users/godfrey/data/go/[gG][oO]*")
	if err != nil {
		fmt.Println(err)
	} else {
		for _, project := range list {
			fmt.Println(project)
		}
	}
}
```
输出：
```text
/Users/godfrey/data/go/go.dazhairen.com
/Users/godfrey/data/go/go-blog.dazhairen.com
```


## 遍历目录

在 filepath 包中，提供了 Walk() 函数用于遍历目录树。方法定义如下：

```text
func Walk(root string, walkFn WalkFunc) error
```

Walk() 函数会遍历 root 指定的目录下的文件树，对每一个该文件树中的目录和文件都会调用 walkFn 来进行处理，包括 root 本身。Walk() 函数不会遍历文件树中的 符号链接（快捷方式）文件包含的路径。

walkFn 的类型 WalkFunc 的定义如下：

```text
type WalkFunc func(path string, info os.FileInfo, err error) error
```

说明：

1）如果 WalkFunc 返回nil，则 Walk 函数继续遍历

2）如果 WalkFunc 返回 SkipDir，则 Walk 函数会跳过当前目录。如果当前遍历到的是文件，则同时跳过后续文件及子目录，继续遍历下一个目录

3）如果 WalkFunc 返回其它错误，则 Walk 函数会终止遍历过程

4）在 Walk 遍历过程中，如果遇到错误，会将错误通过 err 传递给 WalkFunc 函数，同时 Walk 会跳过出错的目录，继续处理后续流程


代码示例：
```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

// WalkFunc 函数：
// 列出含有 *.go 文件的目录（如果当前目录有，则跳过子目录）
func findGoDir(path string, info os.FileInfo, err error) error {
	ok, err := filepath.Match(`*.go`, info.Name())
	if ok {
		fmt.Println(filepath.Dir(path), info.Name())
		return filepath.SkipDir
	}
	return err
}

// WalkFunc 函数：
// 列出所有以 go 开头的目录
func showGoDir(path string, info os.FileInfo, err error) error {
	if info.IsDir() {
		ok, err := filepath.Match(`[gG][oO]*`, info.Name())
		if err != nil {
			return err
		}

		if ok {
			fmt.Println(filepath.Dir(path), info.Name())
		}
	}
	return nil
}

func main() {
	// 列出含有 *.go 文件的目录（如果当前目录有，则跳过子目录）
	err := filepath.Walk("/Users/godfrey/data/go/", findGoDir)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("=========================")

	err = filepath.Walk("/Users/godfrey/data/go/", showGoDir)
	if err != nil {
		fmt.Println(err)
	}
}
```
输出：
```text
/Users/godfrey/data/go/go.dazhairen.com test.go
=========================
/Users/godfrey/data/go go.dazhairen.com
```