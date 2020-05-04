# Cookie

Cookie，是网站为了辨别用户身份、进行session跟踪而存储在用户本地终端上的数据（通常是加密过的）。

Cookie机制是一种客户端机制，把用户数据保存在客户端；而Session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构来保存信息，每个网站访客都会被分配一个唯一标识，即SessionID。
SessionID要么经过URL传递，要么保存在客户端的Cookie中。

## 设置Cookie

Go语言通过 net/http包 中的 SetCookie 来设置Cookie：

```text
http.SetCookie(w ResponseWriter, cookie *Cookie)
```

w 表示需要写入的 response，cookie 是一个 struct，如下：

```text
type Cookie struct {
    Name        string
    Value       string
    Path        string
    Domain      string
    Expires     time.Time
    RawExpires  string
    MaxAge      int
    Secure      bool
    HttpOnly    bool
    Raw         string
    Unparsed    []string
}
```


## 读取Cookie

```text
cookie, _ := r.Cookies("username")
fmt.Fprint(w, cookie)
```

此外，还可以通过另一种方式读取：

```text
for _, cookie := range r.Cookies() {
    fmt.Fprint(w, cookie.Name)
}
```


完整代码示例：

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", set)
	http.HandleFunc("/get", get)
	http.ListenAndServe(":8080", nil)
}

func set(w http.ResponseWriter, req *http.Request) {
	http.SetCookie(w, &http.Cookie{
		Name: "username",
		Value: "admin",
	})
	fmt.Fprintln(w, "set cookie successful！")
}

func get(w http.ResponseWriter, req *http.Request) {
	c, err := req.Cookie("username")
	if err != nil {
		http.Error(w, http.StatusText(400), http.StatusBadRequest)
		return
	}
	fmt.Println(c)
	fmt.Fprintln(w, "cookie value：", c)
}
```

