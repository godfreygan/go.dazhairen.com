# Logrus

logrus 是用Go语言实现的一个日志系统，与标准库 log 完全兼容并且核心API很稳定，是当前使用最广泛的日志库。


安装：
> go get -u github.com/sirupsen/logrus


简单示例：
```go
package main

import (
	log "github.com/sirupsen/logrus"
)

func main() {
	log.WithFields(log.Fields{
		"language": "Golang",
	}).Info("Golang is best")
}
```
输出：
```text
INFO[0000] Golang is best                                language=Golang
```


使用logrus自定义日志处理，示例：
```go
package main

import (
	log "github.com/sirupsen/logrus"
	"os"
)

func init() {
	// 日志格式化为JSON，而不是默认的ACSII
	log.SetFormatter(&log.JSONFormatter{})

	// 输出stdout，而不是默认的stderr。也可以是一个文件
	log.SetOutput(os.Stdout)

	// 只记录严重或警告
	//log.SetLevel(log.WarnLevel)
}

func main() {
	log.WithFields(log.Fields{
		"name": "godfrey",
		"sex": "男",
		"age": 18,
	}).Info("godfrey is a man. He's 18 years old.")
	//.Warn("godfrey is a man. He's 18 years old.")
	//.Fatal("godfrey is a man. He's 18 years old.")

	// 通过日志语句重用字段
	contextLogger := log.WithFields(log.Fields{
		"common": "这是一个字段",
		"other": "这是其它的字段",
	})

	contextLogger.Info("此处会记录common和other字段")
}
```
输出：
```text
{"age":18,"level":"info","msg":"godfrey is a man. He's 18 years old.","name":"godfrey","sex":"男","time":"2019-05-05T10:57:24+08:00"}
{"common":"这是一个字段","level":"info","msg":"此处会记录common和other字段","other":"这是其它的字段","time":"2019-05-05T10:57:24+08:00"}
```
