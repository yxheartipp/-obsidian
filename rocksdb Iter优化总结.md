---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-07-18
sr-interval: 1
sr-ease: 250
---
## 初始性能
直接使用ext4文件系统在盘上调用rocksdb，进行10万条数据的写入需要的耗时为19s左右。读取速率在**30MB/s**左右。在测试写入的过程中曾多次出现crc校验失败，导致的写入失败，同样代码在i9主机或rk3588主机上并不会出现这个问题
![[Snipaste_2024-07-08_13-50-50.png]]

## bluefs+rocksdb+user_nvme_io
在2000上使用bluefs+user_nvme_io 与ext版本采用相同的配置情况下耗时13s左右。读取速率在46MB/s左右，同样写入的过程中也多次出现了crc校验失败，后期采用在主机侧进行数据写入，盘上只负责读取的方式进行测试。
![[Snipaste_2024-07-12_13-53-32.png]]
通过优化参数等方式读取的时间可以进一步控制在10s左右。读取速率为60MB/s。

## rk3588性能摸测
在rk3588上使用perf对bulefs和ext4两种文件系统10条数据读写进行性能分析。
![[rocksdb_bluefs_rk3588_97.5MB.png]]
bluefs 在rk3588上需要6.6秒可以对10条数据进行迭代，读取速度为**97.5MB/s**.

与之相比ext4则只需要0.9s就可以完整读取，读取速度为**666.6MB/s**
![[rk3588 ext4.png]]

后经过与王俊讨论分析perf，发现大块的内存调用全部落在了page_cache上，清空系统的page_cache , 在相同参数配置下，ext4 文件系统的读取时间为7.41s左右，文件读取速率为**80.9MB/s**
![[ext4_after reboot.png]]

## 优化方向选择
通过rk3588上性能分析，要优化rocksdb的读取性能，就要忍受相对大的写放大，通过调大rocksdb的block_size，调大读取选项中的readhead_size。同时将bluefs允许的bluefs_max_prefetch从1_M调整到512M, 加大预读取的数据量。
最终在rk3588上10万条数据的读取耗时为3.6s，读取速率约为**162.1MB/s**。
![[Pasted image 20240717192430.png]]

## 最终优化结果
在2000D上采用bufferd_io的方式通过user_nvme进行读写由于与directIO的文件描述符，打开方式不同，会触发文件系统问题。等待新版本user_nvme提供后进行进一步匹配。
只才用direct_io的方式通过user_nvme+大buffer进行读写的方式在2000D上进行测试。
![[2000d_bluefs_big_cache+user_nvme_io.png]]
通过了多次的读测试，在2000D上10万条的读速度一般在6.7s左右。读取速率约为**88MB/s**

