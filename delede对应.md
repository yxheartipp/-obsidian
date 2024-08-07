---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-08-08
sr-interval: 1
sr-ease: 250
---
error              code             status
STORE_ERR_INVALID_PARAM           0x36                          不可以删除

### delete_kb_dir
```text
| error | code | status |
|:------|:-------:|------:|
| STORE_ERR_DELETE_ROOT | 0x68 | 不可以删除根目录 |
| STORE_ERR_OPEN_DB | 0x30 | 默认db打开失败 |
| STORE_ERR_NO_EXIST | 0x59 | 当前知识库已经被删除 |
| STORE_ERR_DB_IS_USED | 0x58 | 当前文件夹正在被其他人使用 |
| STORE_ERR_PATH_NO_EXIST | 0x69 | 文件夹不存在 |
| STORE_ERR_DELETE_DIR | 0x67 | 其余删除失败情形 |
```
### delete_kb
```text
| error | code | status |
|:------|:-------:|------:|
| STORE_ERR_DELETE_DB | 0x686 | 删除失败 |
| STORE_ERR_OPEN_DB | 0x30 | 默认db打开失败 |
| STORE_ERR_NO_EXIST | 0x59 | 当前知识库已经被删除 |
| STORE_ERR_DB_IS_USED | 0x58 | 当前文件夹正在被其他人使用 |
```

### delete_file
```text
| error | code | status |
|:------|:-------:|------:|
| STORE_ERR_DELETE_DB | 0x686 | 删除失败 |
| STORE_ERR_OPEN_DB | 0x30 | 默认db打开失败 |
| STORE_ERR_NO_EXIST | 0x59 | 当前知识库已经被删除 |
| STORE_ERR_DB_IS_USED | 0x58 | 当前文件夹正在被其他人使用 |
```