---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-04-25
sr-interval: 1
sr-ease: 250
---

## 单线程remote_compact 
### 测试命令
1. 顺序写八十万
sudo ./db_bench --benchmarks=fillseq --use_existing_db=0 --dev=/dev/nvme0n2 --num=800000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=2 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --report_file=./fillseq_remote_800000.csv --remote_compact=1
2. 覆盖写八十万
sudo ./db_bench --benchmarks=overwrite --use_existing_db=1 --dev=/dev/nvme0n2 --num=800000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=2 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --report_file=./overwrite_remote_800000.csv --remote_compact=1
### CPU利用率图
![[cpu_remote.png]]
单线程compact的CPU利用率最高只能达到16%
### 磁盘速度
![[R_W_remote.png]]
八十万4K数据写入最高的数据速度达到了350MB/S，出现了两次读取是因为remote_compaction出现了两次失败导致的。
### QPS图
![[Pasted image 20240424184706.png]]

## 单线程host_compact 
### 测试命令
1. 顺序写八十万
sudo ./db_bench --benchmarks=fillseq --use_existing_db=0 --dev=/dev/nvme0n2 --num=800000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=2 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --report_file=./fillseq_800000.csv
2. 覆盖写八十万
sudo ./db_bench --benchmarks=overwrite --use_existing_db=1 --dev=/dev/nvme0n2 --num=800000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=2 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --report_file=./overwrite_800000.csv
### CPU利用率图
![[cpu_host.png]]
单线程的CPU利用达到了30%以上，较remote的16%有明显上升。可以得出结论remote_compact合理的降低了CPU负载
### 磁盘速度
![[R_W_host.png]]
单线程主机侧覆盖写的cpu利用率达到了500MB/s以上，大于remote_Compact的磁盘速度，但是一直有读取。
### QPS图
![[Pasted image 20240424184803.png]]
## 结果分析
从上图中可以看出一下几点：
1. remote_compact 可以减轻主机侧cpu使用，但目前remote_compact的并行性测试等并没有完善，无法得出在多线程使用情况下remote_compact对主机侧的减负情况。
2. 盘上的cpu计算速度，不如主机侧cpu计算速度。受限与盘上的计算速度，在最终QPS上无法与主机侧compact做比较