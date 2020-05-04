# 数据库

对web应用程序而言，核心所在就是数据库。可以这么说，数据库是灵魂。

Go语言没有内置的驱动支持任何的数据库，但是，定义了 database/sql 接口。


## database/sql 接口

Go语言与PHP不同的地方就是Go语言官方没有提供数据库驱动，而是为开发数据库驱动定义了一些标准接口，开发者可以根据定义的接口来开发相应的数据库驱动。
这样做的好处是，只要是按照标准接口开发的代码，以后需要做数据库的迁移时，不需要做任何修改。


下面，针对这些标准接口进行分析：

1）sql.Register

这个函数是用来注册数据库驱动的，第三方开发者开发数据库驱动时，都会实现 init函数 ，在 init函数 里面都会调用这个 Register(name string, driver driver.Driver) 来完成本驱动的注册。

如：
```text
// go-sqlite3
func init() {
    sql.Register("sqlite3", &SQLiteDriver{})
}

// mymysql
var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
func init() {
    Register("SET NAMES utf8")
    sql.Register("mymysql", &d)
}
```


此外，我们在开发中经常看到如下形式的代码：
```text
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)
```
前面已经讲过，"_"是Go语言用来忽略变量赋值的占位符，对于包引入来说，这是引入后面的包名而不直接使用这个包中定义的函数、变量等资源。
上面代码中，包在引入的时候会自动调用包的 init函数 以完成对包的初始化。而初始化函数里面存在注册这个数据库驱动的代码。所以，后续操作可以直接使用这个数据库驱动。


2）driver.Driver

Driver是一个数据库驱动的接口，它定义了一个方法 Open(name string)，这个方法返回一个数据库的 Conn接口。

```text
type Driver interface {
    Open(name string) (Conn, error)
}
```

!!! note ""
    Conn只能用来进行一次 goroutine 的操作，否则会报错。系统不知道某个操作到底是由哪个goroutine发起的，从而导致数据混乱。


3）driver.Conn

Conn 是一个数据库连接接口，如上面所说的，只能应用在一个goroutine里面。接口定义了如下方法：

```text
type Conn interface {
    Prepare(query string) (Stmt, error)
    Close() error
    Begin() (Tx, error)
}
```

Prepare() 方法返回与当前连接相关的执行SQL语句的准备状态，可以进行查询、删除、等操作。

Close() 方法是关闭当前的连接，执行释放连接拥有的资源等清理工作。

Begin() 方法返回一个代表事务处理的Tx，通过它可以进行查询、更新等操作，抑或对事务进行回滚、提交。


4）driver.Stmt

Stmt是一种准备好的状态，和 Conn 相关联，而且只能应用于一个 goroutine 中。

```text
type Stmt interface {
    Close() error
    NumInput() int
    Exec(args []value) (Result, error)
    Query(args [value]) (Rows, error)
}
```

Close() 方法用于关闭当前的连接状态，但如果当前正在执行query，query还是会有效返回rows数据。

NumInput() 方法返回当前预留参数的个数。

Exec() 方法执行Prepare准备好的SQL，传入参数后执行Update/Insert等操作时，返回Result数据。

Query() 方法执行Prepare准备好的SQL，传入参数后执行select操作，返回Rows结果集。


5）driver.Tx

事务处理一般就两个结果，提交或者回滚，数据库驱动里面也只需实现这两个方法：

```text
type Tx interface {
    Commit() error
    Rollback() error
}
```


6）driver.Execer

这不是 Conn 必须要实现的接口，如果这个接口没有定义，那么调用 DB.Exec，就会首先调用Prepare返回Stmt，然后执行 Stmt 的 Exec，最后关闭 Stmt。

```text
type Execer interface {
    Exec(query string, args []Value) (Result, error)
}
```


7）driver.Result

这个是执行 Update/Insert 等操作返回的结果接口定义：

```text
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```


8）driver.Rows

这个是执行查询返回的结果集接口定义：

```text
type Rows interface {
    Columns() []string
    Close() error
    Next(dest []Value) error
}
```

Columns() 方法返回所查询数据表的字段信息，是所查询的字段，而不是数据库里面的字段。

Close() 方法用来关闭迭代器。

Next() 方法用来返回下一条数据，把数据赋值给desc。


9）driver.RowsAffected

RowsAffected 是一个 int64 的别名，但是它实现了 Result 的接口：

```text
type RowsAffected int64
func (RowsAffected) LastInsertId() (int64, error)
func (v RowsAffected) RowsAffected() (int64, error)
```


10）driver.Value

Value 是一个空接口，可以容纳任何的数据。

```text
type Value interface {}
```


11）driver.ValueConverter

ValueConverter 接口定义了如何把一个普通的值转化成 driver.Value 的接口：

```text
type ValueConverter interface {
    ConvertValue(v interface{}) (Value, error)
}
```


12）driver.Valuer

Valuer 接口定义了返回一个 driver.Value 的方式：

```text
type Valuer interface {
    Value() (Value, error)
}
```