---
alias: []
Type: 
tags: ["#review"]
sr-due: 2024-09-15
sr-interval: 1
sr-ease: 250
---

例如，微调 GPT3 将需要超过 8 个具有 3D 并行性的 DGX-2 节点（128 个 GPU），以仅适合训练模型，即使单个 DGX-2 节点（16-GPU）有足够的计算以合理的时间对其进行微调。

在本文中，我们从 3D 并行性向前迈进，并提出了 ZeRO-Infinity，这是一个能够解决上述所有大型模型训练挑战的新系统

这允许 ZeRO-Infinity 同时利用 CPU 和 NVMe 内存来支持有限 GPU 资源的大规模模型大小。此外，ZeRO-Infinity 还引入了一种新的 GPU 内存优化技术，称为以内存为中心的平铺，以支持非常大的单个层，否则即使一次一层，也不会适合 GPU 内存。随着无限卸载引擎和以内存为中心的平铺，ZeRO-Infinity不仅支持模型大小的下一个1000倍增长，而且使得GPU资源有限的数据科学家可以访问大型模型。

ZeRO-Infinity引入了一种新的数据分区策略，利用所有设备上的聚合内存带宽，我们称之为带宽中心分区，并将其与强大的通信重叠中心设计相结合，以及在无穷卸载引擎中实现高性能NVMe访问的优化。总之，ZeRO-Infinity 提供了出色的训练效率，尽管将数据卸载到 CPU 或 NVMe，但不受其有限带宽的影响。

ZeRO-Infinity 消除了手动模型代码重构的需要，即使通过易于启发的实现扩展到数万亿的参数，该实现自动化训练任意模型架构所需的所有通信和数据分区。

ZeRO-Infinity(第5、6节和第7节):一种新颖的DL训练系统技术，包括五种创新技术，以解决内存和带宽需求，以提供前所未有的模型规模，这是可访问的，易于使用，同时实现出色的训练效率:
i)无限卸载引擎，通过同时利用GPU、CPU和NVMe内存以及GPU和CPU计算，充分利用现代集群上的异构架构，
ii)以内存为中心的平铺来处理大量操作符，而不需要模型并行性，
iii)带宽为中心的分区，以利用所有并行设备上的聚合内存带宽，
iv)重叠中心设计，
v)易于启发的实现，以避免模型代码重构。


ZeRO-Infinity 设计了一种强大的卸载机制，称为无穷大卸载引擎，该机制可以将所有分区的模型状态卸载到 CPU 或 NVMe 内存，或者根据内存需求将它们保留在 GPU 上。

![[Pasted image 20240914141023.png]]

因此，通过将激活检查点卸载到 CPU 内存，ZeRO-Infinity 可以拟合具有数百万亿个参数的模型的激活检查点。

实际上，这极具挑战性，因为 CPU 内存比 GPU 内存带宽慢一个数量级，而 NVMe 带宽但比 CPU 内存带宽慢一个数量级。此外，从 GPU 读取和写入这些内存甚至更慢（见图 2b）。

这提出了两个问题：i) 对于大型模型，即使在 CPU 内存中，激活内存也会变得太大而无法拟合，并且 ii) 当扩展到数百或数千个 GPU 以实现有效收敛时，有效的批量大小变得太大。


DS_BUILD_OPS=1 DS_BUILD_AIO=1 DS_BUILD_GDS=0 DS_BUILD_EVOFORMER_ATTN=0  DS_BUILD_CUTLASS_OPS=0 DS_BUILD_RAGGED_DEVICE_OPS=0 pip install deepspeed --no-ca
che


_class_deepspeed.zero.TiledLinear(_in_features_, _out_features_, _bias=True_, _in_splits=1_, _out_splits=1_, _input_is_already_split=False_, _combine_out_splits=True_, _linear_cls=<class 'torch.nn.modules.linear.Linear'>_, _init_linear=None_, _**kwargs_)

**3.由于ZeRO-Infinity是建立在ZeRO-3之上的，因此在将优化器状态卸载到CPU内存时，它还可以利用聚合GPU和CPU内存带宽以及用于优化器步骤的聚合CPU计算**。然而，**使用NVMe卸载**，有必要**将数据从NVMe带到CPU内存中，然后以适合CPU内存的块的形式返回，以执行优化步骤，每次一个块**。

**ZeRO-Infinity有一个重叠引擎（** **overlap engine），它不仅将GPU-GPU通信与GPU计算重叠，还将NVMe与CPU、CPU与GPU通信重叠（，所有这些都在同一时间进行**。**重叠引擎有两个组成部分:i)一个动态预取器（dynamic prefetcher），用于在向前或向后传递中消耗参数之前，重叠重建参数所需的数据移动【在 FWD/BWD 时，要使用参数之前，并行执行参数的移动。实时跟踪 FWD/BWDW，构建每次迭代的算子序列的内部映射，跟踪算子序列的位置，并行加载即将执行算子的参数，如在执行算子的计算时，对即将执行的 3 个算子依次并行执行 NVMe → CPU，CPU → GPU 和 GPU → GPU。当 FWD/BWD 发生变化时，也可以更新算子序列映射。】;ii)一个通信和卸载重叠机制（communication and offload overlapping mechanism），用于并行执行梯度所需的数据移动和向后计算【并行执行 BWD 与梯度更新移动。类似上面，在计算 BWD 算子时，对已经执行的 2 个算子依次并行执行梯度更新与 GPU → CPU 梯度传输】。**


**ZeRO-Infinity中的动态预取器动态跟踪向前和向后的计算，为每次迭代构造算子序列的内部映射。在每次迭代期间，预取器跟踪它在算子序列中的位置，并预取未来算子所需的参数**。预取器知道三步通信过程，因此可以将一个参数的nc-transfer与其他参数的cgtransfer和gg-transfer重叠。例如，预取器在执行第i个算子之前，可以分别对i+ 3、i+2和i+1所需要的参数调用nc、cg和gg-transfer。注意，所有这些数据移动都可以与执行第i个运算符并行进行。此外，**ZeRO-Infinity可以在动态工作流的情况下更新算子序列映射，即使在迭代期间向前和向后传播发生变化时也允许适当的预取**。

类似地，**在后向传递中，ZeRO-Infinity 可以重叠第 (i+1)个 算子中参数的梯度的 reduce-scatter，同时将分区梯度从第 (i+ 2)个算子的梯度的 reduce-scatter 转移到 CPU 或 NVMe。凭借这种强大的以重叠为中心的设计，ZeRO-Infinity 隐藏了很大一部分数据移动，即使使用少量 GPU 进行训练，每个 GPU 的批量大小也很小**。

![[Pasted image 20240919144110.png]]

![[Pasted image 20240919144204.png]]

**在 ZeRO-3 中，模型每一层的参数都由一个独特的数据并行过程拥有**。**在训练期间，ZeRO-3 通过发出来自所有者进程的广播通信集合，确保操作员在执行前可以获得向前或向后传递所需的参数。在算子执行之后，ZeRO3 还删除了参数，因为它们不再需要，直到运算符的下一个前向和后向传递。此外，在训练的参数更新阶段，ZeRO-3 确保每个数据并行过程只更新与其拥有的参数相对应的优化器状态。因此，ZeRO-3 可以保持所有在整个训练过程中划分的模型状态，除了直接计算所需的参数**。

![[Pasted image 20240919145156.png]]


![[Pasted image 20240919145223.png]]


参数交换
![[Pasted image 20240919151503.png]]


优化器交换
![[Pasted image 20240919151604.png]]

梯度交换
![[Pasted image 20240919151632.png]]


 TORCH_CUDA_ARCH_LIST="8.6" DS_BUILD_OPS=1 pip install . --no-cache

using `ds` instead of `deepspeed` fixes this for me. documentation should be changed!