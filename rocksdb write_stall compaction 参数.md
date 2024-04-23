---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-04-24
sr-interval: 1
sr-ease: 250
---
## 顺序加入
sudo ./db_bench --benchmarks=fillseq --dev=/dev/nvme0n2 --num=80000000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --remote_compact=1 --allow_concurrent_memtable_write=false --level0_file_num_compaction_trigger=4 --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=8 --key_size=20 --value_size=4096 --num_levels=6 --subcompactions=1 --statistics=0 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --open_files=-1 --level_compaction_dynamic_level_bytes=true --soft_pending_compaction_bytes_limit=24696061952 --hard_pending_compaction_bytes_limit=49392123904 --min_level_to_compress=0 --threads=1


--allow_concurrent_memtable_write=false --level0_file_num_compaction_trigger=4 --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=8 --max_write_buffer_number=8 --db=/data1/bench_7.x --wal_dir=/data1/bench_7.x --num=800000000 --num_levels=8 --key_size=20 --value_size=400 --block_size=8192 --cache_size=51539607552 --cache_numshardbits=6 --compression_max_dict_bytes=0 --compression_ratio=0.5 --compression_type=none --bytes_per_sync=8388608 --cache_index_and_filter_blocks=1 --cache_high_pri_pool_ratio=0.5 --benchmark_write_rate_limit=0 --write_buffer_size=16777216 --target_file_size_base=16777216 --max_bytes_for_level_base=67108864 --verify_checksum=1 --delete_obsolete_files_period_micros=62914560 --max_bytes_for_level_multiplier=8 --statistics=0 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --histogram=1 --memtablerep=skip_list --bloom_bits=10 --open_files=-1 --subcompactions=1 --compaction_style=0 --min_level_to_compress=3 \ --level_compaction_dynamic_level_bytes=true \ --pin_l0_filter_and_index_blocks_in_cache=1 --soft_pending_compaction_bytes_limit=24696061952 --hard_pending_compaction_bytes_limit=49392123904 --min_level_to_compress=0 --use_existing_db=0 --sync=0 --threads=1 --memtablerep=vector --allow_concurrent_memtable_write=false --disable_wal=1 --seed=1642906118

  
sudo ./db_bench_7.8 --benchmarks=fillseq \ --allow_concurrent_memtable_write=false --level0_file_num_compaction_trigger=4 --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=8 --max_write_buffer_number=8 --db=/data1/bench_7.x --wal_dir=/data1/bench_7.x --num=800000000 --num_levels=8 --key_size=20 --value_size=400 --block_size=8192 --cache_size=51539607552 --cache_numshardbits=6 --compression_max_dict_bytes=0 --compression_ratio=0.5 --compression_type=none --bytes_per_sync=8388608 --cache_index_and_filter_blocks=1 --cache_high_pri_pool_ratio=0.5 --benchmark_write_rate_limit=0 --write_buffer_size=16777216 --target_file_size_base=16777216 --max_bytes_for_level_base=67108864 --verify_checksum=1 --delete_obsolete_files_period_micros=62914560 --max_bytes_for_level_multiplier=8 --statistics=0 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --histogram=1 --memtablerep=skip_list --bloom_bits=10 --open_files=-1 --subcompactions=1 --compaction_style=0 --min_level_to_compress=3 \ --level_compaction_dynamic_level_bytes=true \ --pin_l0_filter_and_index_blocks_in_cache=1 --soft_pending_compaction_bytes_limit=24696061952 --hard_pending_compaction_bytes_limit=49392123904 --min_level_to_compress=0 --use_existing_db=0 --sync=0 --threads=1 --memtablerep=vector --allow_concurrent_memtable_write=false --disable_wal=1 --seed=1642906118



sudo ./db_bench --benchmarks=fillseq --dev=/dev/nvme0n2 --num=8000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --remote_compact=1 --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=8 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_per_interval=1 --stats_interval_seconds=5


sudo ./db_bench --benchmarks=fillseq --dev=/dev/nvme0n2 --num=80000000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --remote_compact=1 --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=8 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --level_compaction_dynamic_level_bytes=true --soft_pending_compaction_bytes_limit=24696061952 --hard_pending_compaction_bytes_limit=49392123904 --threads=1 --max_background_jobs=8


sudo ./db_bench --benchmarks=fillseq --dev=/dev/nvme0n2 --num=80000000 --fs_uri=bluefs_rpc:///dev/nvme0n2//mount/host --compression_type=none --db=db --remote_compact=1 --level0_file_num_compaction_trigger=4 --allow_concurrent_memtable_write=false --level0_slowdown_writes_trigger=20 --level0_stop_writes_trigger=30 --max_background_jobs=2 --max_write_buffer_number=8 --num_levels=6 --subcompactions=1 --statistics=0 --value_size=4096 --key_size=20 --stats_per_interval=1 --stats_per_interval=1 --stats_interval_seconds=5 --report_interval_seconds=5 --level_compaction_dynamic_level_bytes=true --soft_pending_compaction_bytes_limit=24696061952 --hard_pending_compaction_bytes_limit=49392123904 --threads=1 --max_background_jobs=8 --open_files=-1



800000 operations;   32.5 MB/s with remote compaction


800000 operations;  116.9 MB/s without remote compaction



sudo ./db_bench --benchmarks="fillseq,stats" --dev=/dev/nvme1n1 --max_background_jobs=24 --num=10000000 --block_size=4096 --write_buffer_size=1073741824 --arena_block_size=16777216 --max_write_buffer_number=50 --batch_size=32 --compression_type=none --max_bytes_for_level_base=4294967296  --enable_pipelined_write=true --disable_auto_compactions=true --max_background_compactions=12 --max_background_flushes=8 --value_size=4096 --threads=8