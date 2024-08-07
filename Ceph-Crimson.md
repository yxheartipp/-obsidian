---
aliases: 
Type: 
tags:
  - "#review"
sr-due: 2024-06-13
sr-interval: 1
sr-ease: 250
---
## 为什么需要异步编程框架？
随着硬件的高速发展（SSD, 10G/40G network, NUMA)。软件开发人员的技能还停留在传统的开发模式上，这样程序已经逐渐开始限制了新硬件性能的发挥：

当硬件发展到了一定阶段之后，多核在逐渐增加，但是应用很难利用多核的全部性能。

这是因为在传统的多进程/线程的编程模式中，锁和上下文信息交换占据了很大一部分开销。由于资源(file, memory)的竞争, 进程/线程不得不阻塞等待。据实验测试，一个高并发的应用，20%~70%的时间可能耗在无谓的锁等待上。

数据分配在一个核上，可能复制和使用在别的核上例如一个网卡的中断程序运行在一个core上，而后续的数据包的处理可能迁移到别的core上，这样CPU的cache line频繁的miss，造成性能的penalty。
## 异步编程的起源
最近一些年，异步编程开始流行起来。各种语言从Python，JaveScript，Go到C++纷纷开始支持异步编程。一个server是异步的，那么它本质上是事件驱动的（event-driven）。通常只有一个thread，这个thread就是一个迭代循环执行。每次迭代它都要轮询（poll）有没有新的事件（event）要处理，如果有，可以调用相应的已经注册好的具体事件处理函数。

事件的处理是run-to-completion，也即意味着main thread不会处理下一个even，除非上一个event已经处理完成。这些event可以对应网络socket连接，存储disk I/O和计时器等。这样整个系统没有thread休眠和锁造成的休眠或者忙等待，所有的组成构件都在不停地运转，系统性能达到最优。当然，对于Seastar来说，是每个core一个这样thread，它称之为engine。
## 异步编程的挑战
复杂性：代码不再像之前同步编程时可读性那么好。许许多多从简单到复杂的回调函数嵌入（递归嵌入）到各种代码分支中，大部分时候，程序员阅读所见的代码并不一定能在逻辑上顺序执行到，执行的时机取决于一些状态值。

要妥善安排这些代码之间的关系（event 和event hander）并非易事。为了减少这种异步带来的复杂性，Seastar实现了future/promise对象，用来管理这些异步操作。这样基于回调函数的编程就变成了基于future/promise的编程。

非阻塞模式：因为每个core上只有一个thread在运行，这个thread不能被阻塞。所以它不能直接或者间接调用任何可能阻塞的系统调用，也不能调用锁接口以防止死锁。

## Seastar异步编程基石
### Future
Future代表一个值可能未定的计算结果，这个结果可能现在不能马上得到，需要等待到将来某个时间点。这种future可以是网络传输的一个缓存，定时器的到期，磁盘写的完成等，它可以是任意一个需要等待的结果。一般我们把一个异步函数的返回值作为一个future，这个future最终向调用者提供结果。

例如我们可以用future read来表示读取磁盘文件的结果，这个结果是一个int值，这个read函数没有任何等待，立马返回给我们一个future. 调用者调用future.available()检查值是否可用，一旦可用，就用future.get获取相应的值。
```
seastar::future<> f() 
{ 
	std::cout << "Sleeping... " << std::flush; 
	using namespace std::chrono_literals; 
	return seastar::sleep(1s).then([] { std::cout << "Done.\n"; });
}
```

这个示例程序提供了一个`future`将在 1 秒后变为可用。
### Promise
Promise顾名思义承诺，代表一个异步函数，这个异步函数返回future，并承诺在将来的某个时间点给future赋值。接口promise.get_Future获取对应的future，promise.set_value（T）给对应的future赋值。


基于Promises和Future简化了异步编程的逻辑，这两个特性将传统的生产者（Promises）和消费者（使用futrue的任何用户）分离。无论是在future被消费之前履行Promises，还是相反，都不会改变代码的结果。

```
future<int> get();   // promises an int will be produced eventually
future<> put(int)    // promises to store an int

void f() {
    get().then([] (int value) {
        put(value + 1).then([] {
            std::cout << "value stored successfully\n";
        });
    });
}
```

## Ceph中的Seastar
虽然BlueStore在设计时考虑了与SSD及 NVMeSSD闪 存的适配，但其对新硬件或混合存储的支持不佳。除此之外，BlueStore的设计也存在一些问题，如数据和元数据被存储在不同的位置，元数据结构和IO逻辑都较复杂，在IO较小的情况下可能存在双写问题，同时元数据占用内存较大。

Ceph社区提出了一种新的磁盘布局方式 SeaStore，该布局在较高层次的驱动上进行垃圾回收。其基本思想是将设备空间分为多个空闲段，每个段的大小为100MB 到10GB，所有数据顺序地被流式传输到设备的段上，在删除数据时仅做标记不进行垃圾回收，当段中的利用率降低至某个利用率阈值时，会将其中的数据移到另一个段中。

### Seastore的设计目标
- 基于NVMe SSD，没有过多考虑PEME和HDD的情况。对于NVMe SSD，考虑使用SPDK的user-space driver
- 使用Seastar 编程模型，实现run-to-completion和 sharded memory/process（share-nothing）模型避免锁竞争
- 基于Seastar的messenger实现，使用DPDK技术，完成数据读写的Zero(or minimal）数据拷贝


由于Flash设备的特性，重写时必须先要进行擦除操作。垃圾回收擦除时，并不清楚哪些数据有效，哪些数据无效（除非显式调用discard线程），但是文件系统层是知道这一点的。所以Ceph希望将垃圾回收功能可以提到SeaStore来做。SeaStore的设计思路主要有以下几点。

- SeaStore的逻辑段（segment）应该与硬件segment（Flash擦除单位）对齐。
- SeaStar是每个线程一个CPU核，所以将底层按照CPU核进行分段。

当空间使用率达到设定上限时，就会进行回收。当segment完全回收后，就会调用discard线程通知硬件进行擦除。尽可能保证逻辑段与物理段对齐，避免逻辑段无有效数据，但是底层物理段存在有效数据，那么就会造成额外的读/写操作。同时由于discard带来的消耗，需要尽量平滑处理回收工作，减少对正常读/写的影响。


Crimson 项目希望通过 ​`​shared-nothing​`​​ 设计和 ​`​run-to-completion​`​ 模型来解决 CPU 可扩展性问题。该设计的重点是强制每个内核或 CPU 运行一个固定线程并在用户空间中分配非阻塞任务。因为请求以及它们的资源可以被分配到各个核心，所以它们可以在同一个核心中被处理，直到处理完成。理想情况下，我们不再需要所有的锁和上下文切换，因为每个正在运行的非阻塞任务都使用到 CPU，一直到它完成任务。没有其他线程可以在同一时间抢占任务。如果不需要与数据路径中的其他分片通信，理想情况下，性能将随着内核数量线性扩展，直到 IO 设备达到其极限。这种设计非常适合 Ceph OSD，因为在 OSD 层面，所有 IO 都已经被 PG 分片了。

综合考虑现实条件，在相同的随机 4KB RBD 工作负载下，在没有复制的情况下，通过将传统和 Crimson OSD 与 BlueStore 后端进行比较来验证 ​`​single-shard run-to-completion​`​​。两个 OSD 都分配了 2 个 CPU 资源。Crimson OSD 很特别，因为 Seastar 需要一个独占 CPU 核心来运行 ​`​single-shard​`​ OSD 逻辑。这意味着 BlueStore 线程必须固定到另一个核心，引入 AlienStore 来弥合 Seastar 线程和 BlueStore 线程之间的边界，并在两个边界之间提交 IO 任务。相比之下，传统 OSD 没有限制使用分配的 2 个 CPU。

性能结果显示，使用 BlueStore 时，Crimson OSD 的随机读取性能大约提高了 25%，随机写入情况下的 IOPS 大约比传统 OSD 高 24%。进一步的分析显示，在随机写的情况下，CPU 的利用率很低，因为大约 20% 的 CPU 被消耗在频繁的查询中，这表明 Crimson OSD 应该不是是当前的瓶颈。
![[Pasted image 20240612095944.png]]

Crimson OSD 提交和完成 IO 任务，以及在 Seastar 和 BlueStore 线程之间进行同步，也有额外的开销。因此，我们针对 MemStore 后台重复了同一组实验，两个 OSD 都分配了 1 个 CPU。如下图所示，Crimson OSD 在随机读取中提供了大约 70% 的 IOPS，在随机写入中比 传统 OSD 高 25%，这与之前实验中的结论一致，即 Crimson OSD 可以做得更好。

![[Pasted image 20240612100002.png]]

Crimson 支持三种 ObjectStore 后端：AlienStore、CyanStore 和 SeaStore。AlienStore 提供与 BlueStore 的向后兼容性。CyanStore 是用于测试的虚拟后端，由易失性内存实现。SeaStore 是一种新的对象存储，专为 Crimson OSD 设计，采用 ​`​shared-nothing ​`​设计。根据后端的具体目标，实现多分片支持的路径是不同的。

AlienStore 是 Seastar 线程中的一个简单代理，用于与使用 POSIX 线程的 BlueStore 进行通信。对于多个 OSD 分片没有特别的工作要做，因为 IO 任务通信同步了。BlueStore 中没有为 Crimson 定制其他内容，因为不可能真正将 BlueStore 扩展到 shared-nothing 设计，因为它依赖于第 三 方 RocksDB 项目，而 RocksDB 仍然是线程的。但是，在 Crimson 能够拿出一个足够优化和足够稳定的原生存储后端解决方案（SeaStore）之前，合理的开销来换取复杂的存储后端解决方案是可以接受的。