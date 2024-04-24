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