---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-04-17
sr-interval: 1
sr-ease: 250
---
## write_stall 触发原因
根本原因就是由于来不及计算导致大量的文件存储在level0中
### 拒绝write_stall 参数
**no_slowdown**:  write options中的变量，默认为false，即如果需要stall或者stop就进行等待。如果设置为true，当需要进行stop或者是stall时，返回status为incomplete。适用于不能容忍阻塞等待的写入。需要注意的是，对于设置了no_slowdown的写请求，不能跟没有设置no_slowdown的请求进行joinbatchgroup。
### memtable数量过多
当memtable数量达到min-write-buffer-number-to-merge个时会触发flsush，Flush慢主要由于磁盘性能问题引起，当等待flush的memetable数量达到参数max-write-buffer-number时会完全停止写入。当max-write-buffer-number>3且等待flush的memetable数量>=参数max-write-buffer-number-1时会降低写入速度。
**解决方案**： 当由于memtable数量引起write stall时，内存充足的情况下可尝试调大max-write-buffer-number、max_background_jobs 、write_buffer_size 进行缓解。（我们使用的（I9cpu，所以内存较为充足，该参数可以调整很大）
### L0文件数量过多
当L0 sst文件数达到level0_slowdown_writes_trigger后会触发write stall 降低写入速度，当达到level0_stop_writes_trigger则完全停止写入。
### 待compact数据量过多
当需要compact的文件数量达到soft_pending_compaction_byte参数值时会触发write stall，降低写入速度，当达到hard_pending_compaction_byte时会完全停止写入.
**tikv**解决方案：
- 调大 soft-pending-compaction-bytes-limit 和 hard-pending-compaction-bytes-limit 参数，防止触发 write stall ，但是这个只是治标不治本，根本原因应该是 compaction 慢
- 确认用户是否有做过 compaction 限流，如果有，那需要放开一点限制看看
- 如果磁盘 IO 能力持续跟不上，建议扩容。
- 如果磁盘的吞吐达到了上限导致 write stall ，但是 CPU 资源比较充足，可以尝试采用压缩率更高的压缩算法来缓解磁盘压力，使用 CPU 资源换磁盘资源。
### 注意点：
1. 减慢/停止的触发和等待compaction的字节数限制是每个column family单独配置的，但是write stall 是应用到整个数据库的，这意味着如果一个column family触发write stall，整个数据库都会被stall。
### 重要参数
- **no_slowdown** ： writeOptions参数，如果no_slowdown不为0，那么不会sleep等待，会返回incomplete状态。用在一些延迟敏感的请求上。  
- **max_background_flushes**: flush线程数，用于too many memtables场景  
- **max_write_buffer_number**：同上
- **max_background_compactions**： LEVEL0 SST太多  
- **subcompact**：subcompaction线程的数量，不受background线程数量
- **write_buffer_size**： 改memtable大小  
- **slowdown_trigger**: 用于too many sst0  
- **stop_trigger**: 用于too many sst0
- **dynamic_level**：会导致频繁的触发compaction
- **rate_bytes_per_sec**：flush和compaction的总体限速阀，可以合理降低IO毛刺

## Rocksdb7.x的解决方案
[这篇文章](https://juejin.cn/post/7194404671014830117) 与rocksdb issue 9423:There are too many write stalls because对老版本rocksdb发生write_stall的具体原因进行了对比分析，发现出现性能抖动的主要原因是estimated_compaction_bytes的剧烈变化(增大），导致了QPS骤降为0:
![[Pasted image 20240416181219.png]]
文章中进一步进一步把L0 size与QPS关联起来：如下图，每当巨大的L0 size被compact下去(case 1)，size为0时，也遇到了write stall。
![[Pasted image 20240416181251.png]]
### 新版本
RocksDB 修复了 Compaction Pending Bytes 计算放大问题导致长时间 Write Stall 问题。减少了 Compaction 和写入的锁竞争，从而规避了 Compaction 期间阻塞写入问题利用动态target level size减少空间放大和写放大，由此可以得出一个结论：每层的最大容量限制也是触发write_stall 实验中需要考量的一个参数。在原有compaction流程中max_bytes_for_level_base和target_level_size的对比关系需要进一步分析。

## 测试方案
### 实验步骤
1. 数据填充：先push 一亿条4K的数据。
2. 数据复写：对上面的db overwrite，造成大量的compaction重叠。
3. 使用benchmark提供的report_file工具对写入QPS和CPU统计以及IO统计进行分析。
### 调整参数
1. 减小max_background_compactions增大subcompact。争取实现在主机侧用单个cpu的计算而2000侧多个subcompact并行计算，实现减负
2. 适当调整 soft-pending-compaction-bytes-limit 和 hard-pending-compaction-bytes-limit 参数观察远端是否能够成功的offload计算负担，如果可以在最终参数中调小参数多次在remote触发
3. 加大bluefs_compact_log_size，是否对数据由增强
4. 调大L0 file_size，观察remote_compact是否能够触发write_stall
5. 调整max_bytes_for_level_base和target_level_size的对比关系。分析是否对remote_compact性能有影响
6. 调整最大容量限制，观察对remote_compact的性能影响。




