# PostgreSQL数据库

PostgreSQL是一个自由的对象-关系型数据库服务器，PostgreSQL和MySQL比较，它更加庞大，因为它是用来代替Oracle而设计的。

Go语言实现的支持PostgreSQL的驱动也很多，因为国外很多人在开发中使用了这个数据库：

- https://github.com/lib/pq 支持database/sql驱动

- https://github.com/jbarham/gopgsqldriver 支持database/sql驱动

- https://github.com/lxn/go-pgsql 支持database/sql驱动


代码示例：

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

func main() {
	db, err := sql.Open("postgres", "user=admin password=123456 dbname=test sslmode=disable")
	checkError(err)

	// 插入数据
	stmt, err := db.Prepare("INSERT INTO student (name, sex, age) VALUES ($1, $2, $3) RETURNING id")
	checkError(err)

	res, err := stmt.Exec("马六", "男", 20)
	checkError(err)

	var lastInsertId int
	err = db.QueryRow("INSERT INTO student (name, sex, age) VALUES ($1, $2, $3) returning id;", "陈七", "女", 16).Scan(&lastInsertId)
	checkError(err)
	fmt.Printf("最后插入的id为：%d", lastInsertId)

	// 更新数据
	stmt, err = db.Prepare("UPDATE student SET age=$1 WHERE id=$2")
	checkError(err)

	res, err = stmt.Exec(17, lastInsertId)
	checkError(err)

	affect, err := res.RowsAffected()
	checkError(err)
	fmt.Printf("影响行数%d\n", affect)

	// 查询数据
	rows, err := db.Query("SELECT * FROM student")
	checkError(err)

	for rows.Next() {
		var id      int
		var name    string
		var sex     string
		var age     int
		err = rows.Scan(&id, &name, &sex, &age)
		checkError(err)
		fmt.Printf("id=%d, name=%s, sex=%s, age=%d\n", id, name, sex, age)
	}

	// 删除数据
	stmt, err = db.Prepare("DELETE FROM student WHERE id=$1")
	checkError(err)

	res, err = stmt.Exec(lastInsertId)
	checkError(err)

	affect, err = res.RowsAffected()
	checkError(err)
	fmt.Printf("影响行数%d\n", affect)

	db.Close()
}

func checkError(err error) {
	if err != nil {
		panic(err)
	}
}
```

!!! note ""
    PostgreSQL不支持LastInsertId函数，因为PostgreSQL内部没有实现类似MySQL的自增ID返回。