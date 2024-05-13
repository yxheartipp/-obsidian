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