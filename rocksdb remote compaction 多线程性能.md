---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-06-15
sr-interval: 1
sr-ease: 250
---
## 测试命令_bluefs
### 填入
```
sudo ./db_bench --benchmarks=fillseq --dev=/dev/nvme0n2 --num=2000000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30  --max_write_buffer_number=2 --statistics=0 --value_size=4096 --key_size=20
```
###  覆盖写
```
sudo ./db_bench --benchmarks=overwrite --use_existing_db=1 --dev=/dev/nvme0n2 --num=1000000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=non
e --db=db --level0_file_num_compaction_trigger=4 --max_background_jobs=3 --num_levels=6 --value_size=4096 --key_size=20 --remote_compact=1 --seed=6
```

## 测试命令_ext4
### 填入
```
sudo ./db_bench --benchmarks=fillseq --dev=/dev/nvme0n2 --num=2000000 --compression_type=none --db=/mnt2/db --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30  --max_write_buffer_number=2 --statistics=0 --value_size=4096 --key_size=20
```
### 覆盖写
```
sudo ./db_bench --benchmarks=overwrite --use_existing_db=1 --dev=/dev/nvme0n2 --num=1000000 --compression_type=none --db=/mnt2/db --level0_file_num_compaction_trigger=4 --max_background_jobs=4 --num_levels=6 --value_size=4096 --key_size=20  --seed=6
```

## 情况分析
对**i5_ext4** 本地compaction, **i5_bluefs** remote_compaction 和盘上**Bluefs** remote_compaction。分别进行了2，3，4**最大后台线程**的覆盖写测试，大批量的触发remote_compaction。
速度最快的是使用i5_bluefs 进行remote_compaction
### i5_bluefs remote_compaction
80万数据覆盖写
### 2线程
- 39785 ops/sec
- 156.2 MB/s
![[h1.png]]
### 3线程
- 47151 ops/sec
- 185.1 MB/s
![[h3.png]]
### 4线程
- 26057 ops/sec
- 102.3 MB/s
![[i5_host 4.png]]
### i5_ext4 local_compaction
### 2线程
- 14365 ops/sec
- 56.4 MB/s
![[100万2_thread_i5.png]]
### 3线程
- 21786 ops/sec
- 85.5 MB/s
![[100万3_thread_i5.png]]
### 4线程
- 15951 ops/sec
- 62.6 MB/s
![[100万4_thread_i5.png]]
### 盘上bluefs remote_compaction
### 2线程
- 10908 ops/sec
- 42.8 MB/s
![[100万2_thread_4K.png]]
### 3线程
- 11066 ops/sec
- 43.4 MB/s
![[100万3_thread.png]]
### 4线程
- 10992 ops/sec
- 43.1 MB/s
![[100万4_thread_4K.png]]
