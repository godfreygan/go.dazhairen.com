# SQLite数据库

SQLite是一个开源的嵌入式关系型数据库，实现自包容、零配置、支持事务的SQL数据库引擎。其特点是高度便携、使用方便、结构紧凑、高效、可靠。

Go语言支持SQLite的驱动也比较多，但是很多都不支持 database/sql 接口：

- https://github.com/mattn/go-sqlite3 支持database/sql接口

- https://github.com/feyeleanor/gosqlite3 不支持database/sql接口

- https://github.com/phf/go-sqlite3 不支持database/sql接口


数据库设计：
```text
CREATE TABLE `student` (
  `id` int unsigned NOT NULL AUTO_INCREMENT COMMENT '自增序号',
  `name` varchar(20) NOT NULL COMMENT '姓名',
  `sex` varchar(20) NOT NULL COMMENT '性别',
  `age` int unsigned NOT NULL DEFAULT 0 COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='学生表';

CREATE TABLE `course` (
  `id` int unsigned NOT NULL AUTO_INCREMENT COMMENT '自增序号',
  `student_id` int NOT NULL COMMENT '学生id',
  `week` varchar(20) NOT NULL COMMENT '星期：周一、周二、、、',
  `course` varchar(50) NOT NULL COMMENT '科目：语文、数学',
  `am` varchar(20) DEFAULT '' COMMENT '上午是否有课：有课、null',
  `pm` varchar(20) DEFAULT '' COMMENT '下午是否有课：有课、null',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='课程表';
```

代码示例：

```go
package main

import (
	"database/sql"
	"fmt"
	"time"
	_ "github.com/mattn/go-sqlite3"
)

func main() {
	db, err := sql.Open("sqlite3", "./demo.db")
	checkError(err)
	
	// 插入数据
	stmt, err := db.Prepare("INSERT INTO student (name, sex, age) VALUES (?, ?, ?)")
	checkError(err)
	
	res, err := stmt.Exec("李四", "女", 17)
	checkError(err)
	
	id, err := res.LastInsertId()
	checkError(err)
	fmt.Printf("新增记录的id为：%d\n", id)
	
	// 更新数据
	stmt, err = db.Prepare("UPDATE student SET name=? WHERE id=?")
	checkError(err)
	
	res, err = stmt.Exec("王五", id)
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