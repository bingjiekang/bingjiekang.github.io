---
title: "Golang的Mysql方法"
layout: post
date: 2023-01-15
image: /assets/images/markdown.jpg
headerImage: false
tag:
- golang
category: 笔记
---

##	Golang的Mysql用法 


1. 必须导入的两个包（因为只需要下面那个包里的各个init()函数，所以不需要把整个包导入，只导入init函数，所以使用_忽视即可）

		"database/sql"
		_ "github.com/go-sql-driver/mysql"
		
		上面的mysql驱动中引入的就是mysql包中各个init()方法，无法通过包名来调用包中的其他函数。导入时，驱动的初始化函数会调用sql.Register将自己注册在database/sql包的全局变量sql.drivers中，以便以后通过sql.Open访问。
		
2. 初始化数据库的连接与释放
		
		Db, err := sql.Open("mysql", "root:12345678@tcp(localhost:3306)/Iris?charset=utf8")
		
		sql.Open()中的数据库连接串格式为："用户名:密码@tcp(IP:端口)/数据库?charset=utf8"。
		
		//验证连接
		if err := DB.Ping(); err != nil {
			log.Fatal(err)
		}
		返回的DB对象可以安全地被多个goroutine并发使用，并且维护其自己的空闲连接池。因此，Open函数应该仅被调用一次，很少需要关闭这个DB对象。
		
		// 释放与数据库的连接
		defer DB.Close()
		
3. Query和Exec的不同

		DB的类型为:*sql.DB，有了DB之后我们就可以执行CRUD操作。Go将数据库操作分为两类：Query与Exec。两者的区别在于前者会返回结果，而后者不会，后者经常用于插入、更新和删除操作。
		
		Query表示查询，它会从数据库获取查询结果（一系列行，可能为空）
		Exec表示执行语句，它不会返回行。
		
		还有两种常见的数据库操作模式：
		
		QueryRow表示只返回一行的查询，作为Query的一个常见特例。
		Prepare表示准备一个需要多次使用的语句，供后续执行用
		
4. 单行查询

		单行查询db.QueryRow()执行一次查询，并期望返回最多一行结果（即Row）。QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误。（如：未找到结果）
		
		func (db *DB) QueryRow(query string, args ...interface{}) *Row
		
		// 查询用户名是否存在
		func Select_user(DB *sql.DB, username string) bool {
			Data := "select username,password from User_information where username = " + "'" + username + "'"
			var mation SelectInfo
			// mation是SelectInfo结构体，用来接收查询到的username,password信息
			err := DB.QueryRow(Data).Scan(&mation.Username, &mation.Password)
			if err != nil {
				// log.Fatal(err)
				return false
			}
			return true
		}
		// 登陆验证信息
		type SelectInfo struct {
			Username string `db:"username"`
			Password string `db:"password"`
		}
		
5. 多行查询

		多行查询db.Query()执行一次查询，返回多行结果（即Rows），一般用于执行select命令。参数args表示query中的占位参数。
		
		func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
		// 查询多条数据示例
		func queryMultiRowDemo() {
			sqlStr := "select * from User_information"
			rows, err := DB.Query(sqlStr)
			if err != nil {
				fmt.Printf("query failed, err:%v\n", err)
				return
			}
			// 非常重要：关闭rows释放持有的数据库链接
			defer rows.Close()
		
			// 循环读取结果集中的数据
			for rows.Next() {
				var mation SelectInfo
				err := rows.Scan(&mation.Username, &mation.Password)
				if err != nil {
					fmt.Printf("scan failed, err:%v\n", err)
					return
				}
				fmt.Printf("username:%s password:%s \n", mation.Username, mation.Password)
			}
		}
		
6. 插入数据
		
		Exec执行一次命令（包括查询、删除、更新、插入等），返回的Result是对已执行的SQL命令的总结。参数args表示query中的占位参数。
		
		func (db *DB) Exec(query string, args ...interface{}) (Result, error)
		
		// 插入数据
		func Insert(DB *sql.DB, username string, password string, email string) bool {
			Data := "insert into User_information(username,password,email) values(?,?,?)"
			_, err := DB.Exec(Data, username, password, email)
			if err != nil {
				// log.Fatal(err)
				return false
			}
			theID, err := ret.LastInsertId() // 显示新插入数据的id
			if err != nil {
				fmt.Printf("get lastinsert ID failed, err:%v\n", err)
				return
			}
			fmt.Printf("insert success, the id is %d.\n", theID)
			// log.Fatal("插入成功")
			return true
		}
		
7. 更新数据

		// 更新数据
		func updateRowDemo() {
			sqlStr := "update User_information set password=? where username = ?"
			ret, err := DB.Exec(sqlStr, "12345678", "admin")
			if err != nil {
				fmt.Printf("update failed, err:%v\n", err)
				return
			}
			n, err := ret.RowsAffected() // 操作影响的行数
			if err != nil {
				fmt.Printf("get RowsAffected failed, err:%v\n", err)
				return
			}
			fmt.Printf("update success, affected rows:%d\n", n)
}

8. 删除数据

		// 删除数据
		func deleteRowDemo() {
			sqlStr := "delete from User_information where username = ?"
			ret, err := DB.Exec(sqlStr, "test1")
			if err != nil {
				fmt.Printf("delete failed, err:%v\n", err)
				return
			}
			n, err := ret.RowsAffected() // 显示受操作影响的行数
			if err != nil {
				fmt.Printf("get RowsAffected failed, err:%v\n", err)
				return
			}
			fmt.Printf("delete success, affected rows:%d\n", n)
		}
		
9. 预处理

		普通SQL语句执行过程：
		1、客户端对SQL语句进行占位符替换得到完整的SQL语句。
		2、客户端发送完整SQL语句到MySQL服务端
		3、MySQL服务端执行完整的SQL语句并将结果返回给客户端。
		
		预处理执行过程：
		1、把SQL语句分成两部分，命令部分与数据部分。
		2、先把命令部分发送给MySQL服务端，MySQL服务端进行SQL预处理。
		3、然后把数据部分发送给MySQL服务端，MySQL服务端对SQL语句进行占位符替换。
		4、MySQL服务端执行完整的SQL语句并将结果返回给客户端。
		
		为什么要预处理？
		1、优化MySQL服务器重复执行SQL的方法，可以提升服务器性能，提前让服务器编译，一次编译多次执行，节省后续编译的成本。
		2、避免SQL注入问题。
		
10. Go实现MySQL预处理

		database/sql中使用下面的Prepare方法来实现预处理操作.Prepare方法会先将sql语句发送给MySQL服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。
		
		func (db *DB) Prepare(query string) (*Stmt, error)
		
		type College struct{
			id int
			name string
		}
		
		// 预处理查询示例
		func prepareQueryDemo() {
			sqlStr := "select * from college where id > ?"
			stmt, err := DB.Prepare(sqlStr)
			if err != nil {
				fmt.Printf("prepare failed, err:%v\n", err)
				return
			}
			
			rows, err := stmt.Query(0)
			defer stmt.Close()
			if err != nil {
				fmt.Printf("query failed, err:%v\n", err)
				return
			}
			
			
			// 循环读取结果集中的数据
			for rows.Next() {
				var u College
				err := rows.Scan(&u.id, &u.name)
				if err != nil {
					fmt.Printf("scan failed, err:%v\n", err)
					return
				}
				fmt.Printf("id:%d name:%s\n", u.id, u.name)
			}
			defer rows.Close()
		}
		
11. 插入、更新、删除相似（插入示例）

		// 预处理插入示例
		func prepareInsertDemo() {
			sqlStr := "insert into college(name) values (?)"
			stmt, err := DB.Prepare(sqlStr)
			if err != nil {
				fmt.Printf("prepare failed, err:%v\n", err)
				return
			}
			
			// 插入
			_, err = stmt.Exec("小王子")
			defer stmt.Close()
			if err != nil {
				fmt.Printf("insert failed, err:%v\n", err)
				return
			}
			_, err = stmt.Exec("沙河娜扎")
			if err != nil {
				fmt.Printf("insert failed, err:%v\n", err)
				return
			}
			fmt.Println("insert success.")
			
			
12. sql注入问题

		我们任何时候都不应该自己拼接SQL语句！
		这里我们演示一个自行拼接SQL语句的示例：
		
		// sql注入示例
		func sqlInjectDemo(name string) {
			sqlStr := fmt.Sprintf("select * from college where name='%s'", name)
			fmt.Printf("SQL:%s\n", sqlStr)
			
			var u College
			err := DB.QueryRow(sqlStr).Scan(&u.id, &u.name)
			if err != nil {
				fmt.Printf("exec failed, err:%v\n", err)
				return
			}
			fmt.Printf("user:%#v\n", u)
		}
		
		此时以下输入字符串都可以引发SQL注入问题：
		sqlInjectDemo("xxx' or 1=1#")
		sqlInjectDemo("xxx' union select * from user #")
		
13. 事务

		在MySQL中只有使用了Innodb数据库引擎的数据库或表才支持事务。事务处理可以用来维护数据库的完整性，保证成批的SQL语句要么全部执行，要么全部不执行。
		
		通常事务必须满足4个条件（ACID）：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。
		
		事务相关方法：Go语言中使用以下三个方法实现MySQL中的事务操作。
		
		开始事务
		func (db *DB) Begin() (*Tx, error)
		
		提交事务
		func (tx *Tx) Commit() error
		
		回滚事务
		func (tx *Tx) Rollback() error
		
		示例：该事物操作能够确保两次更新操作要么同时成功要么同时失败，不会存在中间状态。
		
		// 事务操作示例
		func transactionDemo() {
			tx, err := DB.Begin() // 开启事务
			if err != nil {
				if tx != nil {
					tx.Rollback() // 回滚
				}
				fmt.Printf("begin trans failed, err:%v\n", err)
				return
			}
			sqlStr1 := "Update college set name='呜呜呜' where id=?"
			ret1, err := tx.Exec(sqlStr1, 2)
			if err != nil {
				tx.Rollback() // 回滚
				fmt.Printf("exec sql1 failed, err:%v\n", err)
				return
			}
			affRow1, err := ret1.RowsAffected()
			if err != nil {
				tx.Rollback() // 回滚
				fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
				return
			}
		
			sqlStr2 := "Update college set name='哈哈哈' where id=?"
			ret2, err := tx.Exec(sqlStr2, 3)
			if err != nil {
				tx.Rollback() // 回滚
				fmt.Printf("exec sql2 failed, err:%v\n", err)
				return
			}
			affRow2, err := ret2.RowsAffected()
			if err != nil {
				tx.Rollback() // 回滚
				fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
				return
			}
		
			fmt.Println(affRow1, affRow2)
			if affRow1 == 1 && affRow2 == 1 {
				fmt.Println("事务提交啦...")
				tx.Commit() // 提交事务
			} else {
				tx.Rollback()
				fmt.Println("事务回滚啦...")
			}
		
			fmt.Println("exec trans success!")
		}
		
	


		
		


			
