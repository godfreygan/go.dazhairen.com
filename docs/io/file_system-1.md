# os：系统功能实现

os包 提供了不依赖平台的操作系统函数接口。尽管错误处理是 Go语言 风格，但设计是 UNIX风格 。所以，失败的调用会返回 error 而非 错误码。


## os常用的导出函数

```text
func Hostname() (name string, err error)        // Hostname 返回内核提供的主机名

func Environ() []string                         // Ecviron 返回表示环境变量的格式为 "key=value" 的字符串的切片

func Getenv(key string) string                  // Getenv 检索并返回名为key的环境变量的值

func Getpid() int                               // Getpid 返回调用者所在进程的进程ID

func Exit(code int)                             // Exit 让当前程序以给出的状态码退出（一般的，状态码0表示成功，非0表示出错）。程序会终止，defer函数不会被执行

func Stat(name string) (fi FileInfo, err error) // 获取文件信息

func Getwd() (dir string, err error)            // Getwd 返回一个对应工作目录的根路径

func Mkdir(name string, perm FileMode) error    // 使用指定的权限和名称创建一个目录

func MkdirAll(name string, perm FileMode) error // 使用指定的权限和名称创建一个目录，包括任何必要的上级目录，并返回nil，否则返回错误

func Remove(name string) error                  // 删除name指定的文件和目录

func TempDir() string                           // 返回一个用于保管临时文件的默认目录

var Args []string                               // Args 保管了命令行参数，第一个是程序名
```

代码示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 预定义变量，保存命令行参数
	fmt.Printf("预定义变量：%v\n", os.Args)

	// 获取hostname
	hostname, err := os.Hostname()
	fmt.Printf("hostname：%v，err：%v\n", hostname, err)

	// 获取进程ID
	fmt.Printf("进程ID：%v\n", os.Getpid())

	// 获取全部环境变量
	env := os.Environ()
	fmt.Println("环境变量：")
	for _, v := range env {
		fmt.Printf("%v\n", v)
	}

	// 终止程序
	//os.Exit(1)

	// 获取一条环境变量
	fmt.Printf("PATH的环境变量值：%v\n", os.Getenv("PATH"))

	// 获取当前目录
	dir, err := os.Getwd()
	fmt.Printf("当前目录是：%v，err：%v\n", dir, err)

	// 创建目录
	dir1 := dir +"/new_file"
	err = os.Mkdir(dir1, 0755)
	if err != nil {
		fmt.Printf("创建目录失败：%v\n", err)
	} else {
		fmt.Printf("创建目录成功：%v\n", dir1)
	}

	// 创建多级目录
	dir2 := dir +"/p1/p2"
	err = os.MkdirAll(dir2, 0755)
	if err != nil {
		fmt.Printf("创建多级目录失败：%v\n", err)
	} else {
		fmt.Printf("创建多级目录成功：%v\n", dir2)
	}

	// 删除目录
	err = os.Remove(dir1)
	if err != nil {
		fmt.Printf("删除目录失败：%v\n", err)
	} else {
		fmt.Printf("删除目录成功：%v\n", dir1)
	}

	// 删除多层级目录
	dir3 := dir +"/p1"
	err = os.RemoveAll(dir3)
	if err != nil {
		fmt.Printf("删除多层级目录失败：%v\n", err)
	} else {
		fmt.Printf("删除多层级目录成功：%v\n", dir3)
	}

	// 创建临时目录
	tmp_dir := os.TempDir()
	fmt.Printf("创建临时目录成功：%v\n", tmp_dir)
}
```

输出：
```text
预定义变量：[/private/var/folders/9m/zvn5nvnn2wndr_0jd8jsq7v80000gn/T/___go_build_test_go]
hostname：godfreydeMacBook-Pro.local，err：<nil>
进程ID：26601
环境变量：
PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/godfrey/data/www/go/bin:/usr/local/Cellar/go/1.13.8/libexec/bin
JETBRAINS_LICENSE_SERVER=http://fls.jetbrains-agent.com
SHELL=/bin/zsh
OLDPWD=/Applications/GoLand.app/Contents/bin
TERM=xterm-256color
GOPATH=/Users/godfrey/data/www/go
USER=godfrey
GOROOT=/usr/local/Cellar/go/1.13.8/libexec
TMPDIR=/var/folders/9m/zvn5nvnn2wndr_0jd8jsq7v80000gn/T/
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.6Odlf3WJrZ/Listeners
XPC_FLAGS=0x0
VERSIONER_PYTHON_VERSION=2.7
__CF_USER_TEXT_ENCODING=0x1F5:0x19:0x34
LOGNAME=godfrey
LC_CTYPE=zh_CN.UTF-8
XPC_SERVICE_NAME=com.jetbrains.goland.2248
PWD=/Users/godfrey/data/www/private/go
HOME=/Users/godfrey
PATH的环境变量值：/usr/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/godfrey/data/www/go/bin:/usr/local/Cellar/go/1.13.8/libexec/bin
当前目录是：/Users/godfrey/data/www/private/go，err：<nil>
创建目录成功：/Users/godfrey/data/www/private/go/new_file
创建多级目录成功：/Users/godfrey/data/www/private/go/p1/p2
删除目录成功：/Users/godfrey/data/www/private/go/new_file
删除多层级目录成功：/Users/godfrey/data/www/private/go/p1
创建临时目录成功：/var/folders/9m/zvn5nvnn2wndr_0jd8jsq7v80000gn/T/
```


## File结构体

```text
func Create(name string) (file *File, err error)            // Create 采用0666模式（任何人可读可写不可执行）创建一个名为name的文件，如果文件已存在则覆盖

func Open(name string) (file *File, err error)              // Open 用于打开一个文件用于读取。如果操作成功，返回文件对象的方法用于读取数据，文件只读

func (f *File) Stat() (fi FileInfo, err error)              // Stat 返回描述文件 f 的FileInfo类型值

func (f *File) Readdir(n int) (fi []FileInfo, err error)    // Readdir 读取目录f的内容，返回一个有n个成员的[]FileInfo，这些FileInfo是被Lstat返回的，采用目录顺序

func (f *File) Read(b []byte) (n int, err error)            // Read 从f中读取最多len(b)字节数据并写入b

func (f *File) WriteString(s string) (ret int, err error)   // WriteString 向文件f中写入字符串

func (f *File) Sync() (err error)                           // Sync 递交文件的当前内容进行稳定的存储。简单来说，就是将写入数据从内存中保存到硬盘中

func (f *File) Close() error                                // Close 关闭文件f
```


代码示例：

```go
package main

import (
	"fmt"
	"os"
	"time"
)

func main() {
	// 获取当前目录
	dir, err := os.Getwd()
	fmt.Printf("当前目录是：%v，err：%v\n", dir, err)

	file := dir +"/new_file"
	var fh *os.File

	fi, _ := os.Stat(file)
	if fi == nil {
		fh, _ = os.Create(file)                         // 不存在就创建
	} else {
		fh, _ = os.OpenFile(file, os.O_RDWR, 0666)      // 存在就打开
	}
	defer fh.Close()

	w := []byte("hello go "+ time.Now().String())
	n, err := fh.Write(w)
	fmt.Printf("写入字节数：%d，err：%v\n", n, err)

	// 设置下次读写位置
	ret, err := fh.Seek(0, 0)
	fmt.Printf("当前文件指针位置：%d，err：%v\n", ret, err)

	b := make([]byte, 128)
	n, err = fh.Read(b)
	fmt.Printf("读取到的字节数：%d，err：%v，本文内容：%s\n", n, err, string(b))
}
```

输出：
```text
当前目录是：/Users/godfrey/data/www/private/go，err：<nil>
写入字节数：60，err：<nil>
当前文件指针位置：0，err：<nil>
读取到的字节数：60，err：<nil>，本文内容：hello go 2019-04-17 00:23:19.689965 +0800 CST m=+0.000157653
```


## FileInfo结构体

FileInfo 用来描述一个文件对象。接口定义如下：

```text
type FileInfo interface {
    Name() string
    Size() int64
    Mode() FileMode
    ModTime() time.Time
    IsDir() bool
    Sys() interface{}
}
```

```text
func Stat(name string) (fi FileInfo, err error)     // Stat 返回一个描述文件的FileInfo。如果指定的文件对象是一个符号链接，返回的FileInfo描述该符号链接指向文件信息，同时会尝试跳转该链接

func Lstat(name string) (fi FileInfo, err error)    // Lstat 返回描述文件对象的FileInfo。如果指定的文件对象是一个符号链接，返回的FileInfo描述该符号链接的信息，但不会跳转该链接
```


如上面示例中：
```text
fi, _ := os.Stat(file)
if fi == nil {
    fh, _ = os.Create(file)                         // 不存在就创建
} else {
    fh, _ = os.OpenFile(file, os.O_RDWR, 0666)      // 存在就打开
}
```
