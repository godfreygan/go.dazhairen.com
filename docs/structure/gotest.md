# 测试

计算机编程中，单元测试又称为模块测试，是针对程序模块来进行正确性检验的测试工作。

testing是Go语言的一个package，它提供自动化测试功能。通过 go test 命令能够自动执行下面形式的函数：
```go
func TestXxx(t *testing.T)
```
其中，Xxx 可以是任何字母、数字、字符串，但是 Xxx 的首字母不能是小写字母。

规则如下：

1. 文件以`_test.go`结尾

2. `import "testing"`

3. 函数名为`TestXXX`，并且parameter为`t *testing.T`

使用示例：

新建文件 `a_test.go` 作为要测试的代码，内容如下：
```go
package main

import "testing"

func TestHelloWorld(t *testing.T) {
    t.Log("hello world")
}

func TestA(t *testing.T) {
    t.Log("A")
}
```

执行测试：

```shell
go test a_test.go
```

指定 TestA 进行测试

```shell
go test -run TestA a_test.go
```


输出如下，表示测试通过

```text
ok  	command-line-arguments	0.006s
```

!!! note ""
	推荐使用`go test -v`，输出更详细

