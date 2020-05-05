# 日志记录

Go语言提供了一个简易的log包，短小精悍，可以非常轻松地实现日志打印记录功能。并且log支持并发操作。

示例：
```go
package main

import (
	"fmt"
	"log"
	"os"
	"time"
)

var mylogger *log.Logger

func main() {
	Ldate()

	Ltime()

	Lmicroseconds()

	Llongfile()

	Lshortfile()

	LUTC()

	Ldefault()

	Lstander()

	LprintInfo()

	deferPanic()

	// 测试程序终止
	//fatalln()

	mylogger = LogInfo()

	fmt.Printf("创建时前缀为：%s，创建时输出项属性值为：%d\n", mylogger.Prefix(), mylogger.Flags())

	// 设置输出前缀
	mylogger.SetPrefix("mylogger_")

	// 设置输出选项
	mylogger.SetFlags(log.Ldate | log.Ltime | log.Llongfile)

	fmt.Printf("修改后前缀为：%s，修改后输出项属性为：%d\n", mylogger.Prefix(), mylogger.Flags())

	// 输出日志
	mylogger.Output(2, "Output输出日志")

	// 格式化输出日志
	mylogger.Printf("我是%v方法在%d行内容为：%s\n", "Printf", 52, "底层是以fmt.Printf的方式处理的，相当于Info级别")

	// 开启这个注释，下面代码就不会继续走，并且程序停止
	//mylogger.Fatal("我是Fatal方法，我会停止程序，但不会抛出异常")

	// 调用业务层代码
	ServiceCode()

	mylogger.Printf("业务代码里的Panicln不会影响到我，因为他已经被处理干掉了，程序目前正常")

	// 切换输出屏幕中
	mylogger.SetOutput(os.Stdout)
	mylogger.Printf("切换屏幕进行日志输出")
}

func Ldate() {
	log.SetFlags(log.Ldate)
	log.Printf("这是%s格式\n", "Ldate")
}

func Ltime() {
	log.SetFlags(log.Ltime)
	log.Printf("这是%s格式\n", "Ltime")
}

func Lmicroseconds() {
	log.SetFlags(log.Lmicroseconds)
	log.Printf("这是%s格式\n", "Lmicroseconds")
}

func Llongfile() {
	log.SetFlags(log.Llongfile)
	log.Printf("这是%s格式\n", "Llongfile")
}

func Lshortfile() {
	log.SetFlags(log.Lshortfile)
	log.Printf("这是%s格式\n", "Lshortfile")
}

func LUTC() {
	log.SetFlags(log.Ldate | log.Ltime | log.Lmicroseconds | log.LUTC)
	log.Printf("这是%s格式\n", "LUTC")
}

func Ldefault() {
	log.Printf("这是%s格式\n", "Ldefault")
}

func Lstander() {
	log.SetFlags(log.Llongfile | log.Ldate | log.Ltime)
	log.Printf("这是%s格式\n", "标准日志")
}

// 信息打印格式
func LprintInfo() {
	list := []string{"Tom", "Jerry"}
	log.Print("Print array ", list, "\n")
	log.Println("Println array ", list)
	log.Printf("Printf array with item [%s, %s]", list[0], list[1])
}

// 抛出异常
func deferPanic() {
	defer func() {
		log.Println("--- first ---")

		if err := recover(); err != nil {
			log.Println(err)
		}
	}()

	log.Panicln("log for defer panic")

	defer func() {
		log.Println("--- second ---")
	}()
}

// 程序终止
func fatalln() {
	defer func() {
		log.Println("--- first ---")
	}()

	log.Fatalln("log for defer fatalln")
}

func LogInfo() *log.Logger {
	// 创建文件对象，日志的格式为 2006-01-02 15:04:05.log，固定写法
	timeString := time.Now().Format("2006-01-02")

	// 组装日志文件名
	filename := "./"+ timeString +".log"

	// 创建并打开文件
	logFile, err := os.OpenFile(filename, os.O_RDONLY | os.O_CREATE | os.O_APPEND, 0766)
	if err != nil {
		panic(err)
	}

	// 延迟语句关闭
	defer logFile.Close()

	// 创建一个Logger，参数1：日志写入的文件，参数2：每条日志的前缀，参数3：日志属性
	return log.New(logFile, "prefix_", log.Lshortfile)
}

func ServiceCode() {
	defer func() {
		if r := recover(); r != nil {
			// 用来捕捉panicln抛出的异常
			fmt.Printf("使用recover()捕获到的错误：%s\n", r)
		}
	}()

	mylogger.Panicln("Panicln方法，抛出异常信息，相当于Error级别")
}
```
输出：
```text
201905/05 这是Ldate格式
00:44:55 这是Ltime格式
00:44:55.030587 这是Lmicroseconds格式
/Users/godfrey/data/go.dazhairen.com/test.go:84: 这是Llongfile格式
test.go:89: 这是Lshortfile格式
201905/04 16:44:55.030604 这是LUTC格式
201905/04 16:44:55.030606 这是Ldefault格式
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:103: 这是标准日志格式
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:109: Print array [Tom Jerry]
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:110: Println array  [Tom Jerry]
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:111: Printf array with item [Tom, Jerry]
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:124: log for defer panic
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:117: --- first ---
201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:120: log for defer panic

创建时前缀为：prefix_，创建时输出项属性值为：16
修改后前缀为：mylogger_，修改后输出项属性为：11
使用recover()捕获到的错误：Panicln方法，抛出异常信息，相当于Error级别

mylogger_201905/05 00:44:55 /Users/godfrey/data/go.dazhairen.com/test.go:64: 切换屏幕进行日志输出
```


这些日志是基于fmt包的打印再结合panic之类的函数来进行一般的打印、抛出错误处理的。

如果想要把日志保存到文件，然后又能结合日志实现很多复杂的功能，可以使用第三方开发的日志系统 Logrus 或 Seelog，后面章节会讲到。