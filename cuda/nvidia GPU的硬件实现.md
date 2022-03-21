#<center>Nvidia GPU的硬件实现(Hardware implementation)<center>

Nvidia GPU 的构建中包含一系列的可伸缩的多线程的流多处理器(streaming multiprocessers, SM),当一个cuda程序中的主机代码(host)调用一个kernel grid时,grid中的blocks将根据可用执行空间被列举和分配到多个多处理器上.block中的线程们在一个流多处理器上并行执行,grid中的block们也可以在一个流多处理器上并行执行.当线程块(thread blocks)们执行完后,新的blocks将会被发送到流多处理器上.

一个流多处理器被设计用于数百个线程的并行执行.为了管理这么多个线程,引入了一个独立框架SIMT(single instruction, multiple-thread).在线程内的instruction们组成流水线以便于指令级的并行,类似于硬件多线程上的多线程并行.

## SIMT 框架
多处理器创建,管理,规划并执行一个32个线程组成的组(group)叫做warps.组成warp的独立线程在相同的程序地址同时启动.但是这些线程有独立的地址计数器和状态寄存器,因此可以并独立执行分支.warp术语来自于纺织物(第一个线程技术).一个half-warp可以是第一个或者第二个half-warp.一个quartor-warp可以是第1\2\3\4个quartor-warp.

当一个多处理器执行被分配一个或多个blocks时,会将它们分到warps中,每个warp由warp scheduler调度执行.每个block通过相同的方式分配到warps中,一个warp中的线程们都是连续的,有着递增的thread ID.第一个warp中包含ID为0 的线程.

## 硬件多线程(hardware multithreading)
在warp的整个生命周期中多处理器执行的每一个warp的上下文(程序计数器,寄存器,等)维持在芯片上(chip).在因此从一个上下文切换到另一个上下文没有开销.在每个指令执行的时候,warp调度器会选择一个线程们已经准备好执行下一条指令的warp(一个活动的warp)并将指令发送给warp中的线程们.

特别的,每一个多处理器有一组32位的寄存器被划分到多个warp中,同时一个并行的数据缓存或共享内存被划分到一个块中的线程们.

多处理器上blocks和warp的数量依赖于kernel所需的寄存器数量和共享内存和多处理器上可用的寄存器数量和共享内存.每个多处理器都有一个最大存放的blocks和warp数量.如果多处理器没有足够的寄存器和可用共享空间用于寄存一个block,那么所需执行的kernel调用将会失败.

一个block中的warps数量:
$$\text{ceil}(\frac{T}{W_{size}},1)$$

- $T$是block中的线程数;
- $W_{size}$是warp尺寸,这里等于32;
- $\text{ceil(x,y)}$等于$x$约等于最接近$y$的乘倍数;
