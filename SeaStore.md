---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-06-12
sr-interval: 1
sr-ease: 250
---

虽然BlueStore在设计时考虑了与SSD及 NVMeSSD闪 存的适配，但其对新硬件或混合存储的支持不佳。除此之外，BlueStore的设计也存在一些问题，如数据和元数据被存储在不同的位置，元数据结构和IO逻辑都较复杂，在IO较小的情况下可能存在双写问题，同时元数据占用内存较大。

  

闪存盘在写入前需要将原有数据擦除。现有的NVMe设备并未记录哪些地址在写入前需要被擦除，因此导致设备内部的垃圾回收效率低下。原则上，异步的垃圾回收可以提高写入效率，若不修改磁盘布局则垃圾回收的粒度较小，但 是实际上该操作在设备和中间层实现的效果并不佳。针对闪存盘的特点，Ceph社区提出了一种新的磁盘布局方式 SeaStore，该布局在较高层次的驱动上进行垃圾回收。其基本思想是将设备空间分为多个空闲段，每个段的大小为100MB 到10GB，所有数据顺序地被流式传输到设备的段上，在删除数据时仅做标记不进行垃圾回收，当段中的利用率降低至某个利用率阈值时，会将其中的数据移到另一个段中。

Seastore的设计目标

- 基于NVMe SSD，没有过多考虑PEME和HDD的情况。对于NVMe SSD，考虑使用SPDK的user-space driver
- 使用Seastar 编程模型，实现run-to-completion和 sharded memory/process（share-nothing）模型避免锁竞争
- 基于Seastar的messenger实现，使用DPDK技术，完成数据读写的Zero(or minimal）数据拷贝

由于Flash设备的特性，重写时必须先要进行擦除操作。垃圾回收擦除时，并不清楚哪些数据有效，哪些数据无效（除非显式调用discard线程），但是文件系统层是知道这一点的。所以Ceph希望将垃圾回收功能可以提到SeaStore来做。SeaStore的设计思路主要有以下几点。

- SeaStore的逻辑段（segment）应该与硬件segment（Flash擦除单位）对齐。
- SeaStar是每个线程一个CPU核，所以将底层按照CPU核进行分段。

当空间使用率达到设定上限时，就会进行回收。当segment完全回收后，就会调用discard线程通知硬件进行擦除。尽可能保证逻辑段与物理段对齐，避免逻辑段无有效数据，但是底层物理段存在有效数据，那么就会造成额外的读/写操作。同时由于discard带来的消耗，需要尽量平滑处理回收工作，减少对正常读/写的影响。
![[Pasted image 20240611175521.png]]


ZNS Device导致存储实际上变成了一系列的Segment的空间，或者Blob空间。数据只能以Append的方式存储到这些Segment中。

一个Seatore的写操作包括：

- Data Block的写入
- Inode对应的Extents的更新
- LBA tree的更新
- BackRef Tree的更新
- Inode的写入

这些都作为Transaction的形式一次IO写入Journal，同时在内存中更新相应的Btree就返回。Btree的Checkpoint的更新也是以追加的方式写入新的segment中。

Seastore的写入操作看似涉及很多btree，但是实际就两次IO的操作：1) journal的写入，这可以通过batch聚合多个Transaction完成。2）btree的更新都是在内存中完成，然后在后台定期执行checkpoint操作顺序持久化到SSD上。

如果Crash发生，就replay journal完成数据更新。