# Seelog

Seelog 是用Go语言实现的一个日志系统，它提供了一些简单的函数来实现复杂的日志分配、过滤和格式化。具备以下特性：

- 拥有XML的动态配置，可以不用重新编译程序而动态加载配置信息

- 支持热更新，能够动态改变配置而不需要重启应用

- 支持多输出流，能够同时把日志输出到多种流中，例如文件流、网络流等

- 支持不同的日志输出，包括命令行输出、文件输出、缓存输出等

- 支持 log rotate 和 SMTP 邮件提醒


安装：
>go get -u github.com/cihub/seelog


使用示例：
```go
package  main

import (
	log "github.com/cihub/seelog"
)

func main() {
	defer log.Flush()
	log.Info("Hello go，hello Seelog !")
}
```
输出：
```text
1588670142720113000 [Info] Hello go，hello Seelog !
```
