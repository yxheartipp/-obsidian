---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-05-14
sr-interval: 1
sr-ease: 250
---

sqlite3 *sql

sqlite3_open_v2

sqlite3_stmt *stmt

sqlite3_prepare_v2

sqlite3_step(stmt) == SQLITE_ROW

sqlite3_close_v2

int sqlite3_exec(   
  sqlite3*, /* 数据库  \* /
  const char \* sql, / \* 要计算的 SQL \* /
  int (\*callback)(void*,int,char**,char**), /* 回调函数 \*/
  void \*, /* 回调的第一个参数 \*/
  char \*\*errmsg /\* 错误信息写在这里 \*/ );


在SQLite中最主要的两个对象是，database_connection和 prepared_statement。database_connection对象是由sqlite3_open() 接口函数创建并返回的，在应用程序使用任何其他SQLite接口函数之 前，必须先调用该函数以便获得database_connnection对象，在随后 的其他APIs调用中，都需要该对象作为输入参数以完成相应的工作。 至于prepare_statement，我们可以简单的将它视为编译后的SQL语 句，因此，所有和SQL语句执行相关的函数也都需要该对象作为输入 参数以完成指定的SQL操作。

ATTACH命令在一个 连接中方便的访问多个数据库。