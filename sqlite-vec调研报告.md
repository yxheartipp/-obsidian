---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-08-06
sr-interval: 1
sr-ease: 250
---
sqlite-vec是一个全c的由sqlite支持的扩展引擎。
优点：
1. 编译方便，在rk3588上可以直接编译出sqlite3-vec的so库，转移到2000D上可以直接编译运行
![[sqlite_vector 2000D.png]]
2. 采用汉明编码方式，减少了32倍的内存消耗，在损失5-10%的性能情况下，提升了10的速度，同时将直接提供了压缩接口可以直接应用于其他库中。
3. 