---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-05-14
sr-interval: 1
sr-ease: 250
---

# 测试命令
1. 顺序写八十万
sudo ./db_bench --benchmarks=fillseq --use_existing_db=0 --dev=/dev/nvme0n2 --num=800000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=2 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --report_file=./fillseq_remote_800000.csv --remote_compact=1
2. 覆盖写八十万
sudo ./db_bench --benchmarks=overwrite --use_existing_db=1 --dev=/dev/nvme0n2 --num=800000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=2 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --report_file=./overwrite_remote_800000.csv --remote_compact=1
使用remote_compact是否为1来区分remote还是host进行compaction
### CPU利用率对比
#### remote_compact
![[cpu_remote 1.png]]
#### remote_compact_user_nvme
![[remote_4K_user_CPU.png]]
#### host_compact
![[cpu_host 1.png]]
#### cpu利用率总结
从上面三幅图中，可以看出。使用用户态IO的remote_compact 使用的CPU利用率最小，最大使用也仅使用了12%的CPU存储，同时CPU降速与使用pread接口的remote_compact相比更低，说明等待remote_compaction完成时间更短，盘上compact完成时间有所缩短。在更低的cpu使用率上完成了工作，也即主机侧cpu只负责compact调度数据写入，compact全部计算由盘上小Linux系统完成。
### 磁盘IO对比
#### remote_compact
![[R_W_remote.png]]
#### remote_compact_user_nvme
![[remote_4K_user_IO.png]]
#### host_compact
![[io_compact.png]]
#### 磁盘IO总结
从上述三幅图片中可以看出，使用user_nvme的remotecompact可以实现完整的offload，主机侧没有对盘上系统做出大量的读取运算操作，出现的微量读取是完成compact后进行的校验工作。使用user_nvme的remotecompact仍然出现了一次明显的掉速，是在等待remote_compact成功返回的阶段出现的。
### 整体QPS对比
#### remote_compact
![[Pasted image 20240424184706.png]]
#### remote_compact_user_nvme
![[qps_user_nvme.png]]
#### host_compact
![[Pasted image 20240424184803.png]]
#### QPS总结
因为使用pread, pwrite时，remote_compact有一部分计算工作已经由主机侧完成了，所以在20s-40s时，remote_compact有一定的提升。但是使用user_nvme的remote_compact任然在整体时间上优于使用pread,pwrite的版本。但是通过qps的时间曲线可以看出。主机侧在10s-15s仅用5s就完成了compact的一系列计算操作，而remote_compact需要接近30s才可以完成一条compact操作，计算时间较长。
## 总结
### 整体用时
### remote_compact_user_nvme
![[remote_user_4K.png]]
### host_compact
![[overwrite_host.png]]

使用user_nvme接口的remote_compact 有效的降低了由主机侧进行compact操作产生的负载。更高的等待用时，还是因为盘上的计算效率而不是磁盘的写入速度。采用同时写入和读取的bench_mark可能更明显的能体现出优势。