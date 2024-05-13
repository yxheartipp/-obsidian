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
从上面三幅图中，可以看出。使用用户态IO的remote_compact 使用的CPU利用率最小，最大使用也仅使用了12%的CPU存储，同时CPU降速与使用pread接口的remote_compact相比更低，说明等待remote_compaction完成时间更短，盘上compact完成时间有所缩短。
### 磁盘IO对比
#### remote_compact
![[R_W_remote.png]]
#### remote_compact_user_nvme
![[remote_4K_user_IO.png]]
#### host_compact
![[R_W_host.png]]
#### 磁盘IO总结

