---
alias: []
Type: 
tags: ["#review"]
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


基于Promises和Future简化了异步编程的逻辑，这两个特性将传统的生产者（Promises）和消费者（使用futrue的任何用户）分离。wu'l