# MySQL数据库

目前网站开发中使用最多数据库是MySQL，MySQL以免费、开源、使用方便为优势，成为了web开发的后端数据库存储引擎。

Go语言中支持MySQL的驱动比较多，有些是支持 database/sql 标准，而有些是采用自己实现接口。常用的有如下几种：

- https://github.com/go-sql-driver/mysql 支持database/sql

- https://github.com/ziutek/mymysql 支持database/sql，也支持自定义接口

- https://github.com/Philio/GoMySQL 不支持database/sql，自定义接口


数据库设计：
```text
CREATE TABLE `student` (
  `id` int unsigned NOT NULL AUTO_INCREMENT COMMENT '自增序号',
  `name` varchar(20) NOT NULL COMMENT '姓名',
  `sex` varchar(20) NOT NULL COMMENT '性别',
  `age` int unsigned NOT NULL DEFAULT 0 COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='学生表';
```

代码示例：

```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	db, err := sql.Open("mysql", "username:pwd@/test?charset=utf8")
	checkError(err)

	// 插入数据
	stmt, err := db.Prepare("INSERT student SET name=?,sex=?,age=?")
	checkError(err)

	res, err := stmt.Exec("张三", "男", 18)
	checkError(err)

	id, err := res.LastInsertId()
	checkError(err)
	fmt.Printf("新增记录的id为：%d\n", id)

	// 更新数据
	stmt, err = db.Prepare("UPDATE student SET age=? WHERE id=?")
	checkError(err)

	res, err = stmt.Exec(19, id)
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
	stmt, err = db.Prepare("DELETE FROM student WHERE id=?")
	checkError(err)

	res, err = stmt.Exec(id)

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

sql.Open() 函数用来打开一个注册过的数据库驱动，go-sql-driver中注册了MySQL这个数据库驱动，第二个参数是DSN（Data Source Name），它支持如下格式：
 * user@uinx(/path/to/socket)/dbname?charset=utf8
 * user:password@tcp(localhost:3306)/dbname?charset=utf8
 * user:password@/dbname
 * user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname

!!! note ""
    传入的参数都是 =? 对应的数据，这样做的方式可以在一定程度上防止SQL注入。
