<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [cuda 性能指标(performance guidelines)](#cuda-性能指标performance-guidelines)
  - [1. 总体性能优化策略(Overall Performance Optimization Strategies)](#1-总体性能优化策略overall-performance-optimization-strategies)
  - [2. 利用率最大化](#2-利用率最大化)
    - [2.1. 占用计算器](#21-占用计算器)
  - [3. 内存吞吐量最大化](#3-内存吞吐量最大化)
    - [3.1. host 和 device 之间的数据传输](#31-host-和-device-之间的数据传输)
    - [3.2. device memory 访问](#32-device-memory-访问)
  - [4. 指令吞吐量最大化](#4-指令吞吐量最大化)
    - [4.1. 算术指令](#41-算术指令)
    - [4.2. 控制流指令](#42-控制流指令)
    - [4.3. 同步指令(Synchronization instruction)](#43-同步指令synchronization-instruction)

<!-- /code_chunk_output -->

# cuda 性能指标(performance guidelines)

## 1. 总体性能优化策略(Overall Performance Optimization Strategies)

性能优化围绕三个基本策略:

- 最大化并行执行达到利用率最大;
- 优化内存使用方式达到内存吞吐量最大;
- 优化指令使用方式达到指令吞吐量最大;

## 2. 利用率最大化

为了最大限度地提高利用率，应用程序的结构应该尽可能地并行性，并有效地将这种并行性映射到系统的各个组件，以使它们在大部分时间保持忙碌.

对于给定的内核调用，驻留在每个多处理器上的 block 和 warp 的数量取决于调用 kernel 的运行配置、多处理器的内存资源以及硬件多线程中描述的内核的资源需求.

一个 kernel 调用所需的寄存器数量显著影响多处理器上存放的 warp 数量.
block 中的 thread 数量最好是 warp 尺寸的乘倍数,防止出现不饱和的 warp 浪费了计算资源.比如对于 32 的 warp 尺寸,block 中的线程数应该取 32,64,96,128,....

### 2.1. 占用计算器

有一些 API 函数可以辅助程序员根据寄存器数量和共享内存选择 thread block 尺寸.

- cudaOccupancyMaxActiveBlocksPerMultiprocessor 这个 API 函数提供了基于 block 尺寸和一个 kernel 所用的共享内存在推断多处理器上的 block 占用,该函数根据每个多处理器的并发线程块数量进而得到占用情况。
- cudaOccupancyMaxPotentialBlockSize 和 cudaOccupancyMaxPotentialBlockSizeVariableSMem 启发性的计算出多处理器水平最大占用时的 kernel 运行配置(<<<numblocks,numthreads>>>).

下面的代码实例计算出了 MyKernel 核函数实际活动时单多处理器的 warp 数量和其最大支持 warp 数量的比值.

```c++
#include<cuda_runtime.h>
#include<iostream>
// Device code
__global__ void MyKernel(int *d, int *a, int *b)
{
    int idx = threadIdx.x + blockIdx.x * blockDim.x;
    d[idx] = a[idx] * b[idx];
}
// Host code
int main()
{
    int numBlocks;
    int blockSize = 32;
    // Occupancy in terms of active blocks
    // These variables are used to convert occupancy to warps
    int device;
    cudaDeviceProp prop;
    int activeWarps;
    int maxWarps;
    cudaGetDevice(&device);
    cudaGetDeviceProperties(&prop, device);
    cudaOccupancyMaxActiveBlocksPerMultiprocessor(
        &numBlocks,
        MyKernel,
        blockSize,
        0);
    activeWarps = numBlocks * blockSize / prop.warpSize;
    maxWarps = prop.maxThreadsPerMultiProcessor / prop.warpSize;
    std::cout << "Occupancy: " << (double)activeWarps / maxWarps * 100 << "%" << std::endl;
}
```

## 3. 内存吞吐量最大化

应用内存吞吐量最大化的第一步是要将低带宽的数据传输(data transfer)最小化.
此外,尽量减少 device 和 GPU 的 global memory 之间的数据传输,使用 device 上面的 on-chip memory:共享内存,L1 缓存,L2 缓存(计算兼容性 2.0 以上),纹理缓存,常量缓存.

共享内存相当于用户管理的缓存:由应用程序显式的分配和访问.一个典型的编程模式是将 device memory 分段输入到共享内存中.简而言之,对于一个 block 中的 thread:

- 从设备内存中加载数据到共享内存;
- 同步 block 中的所有线程使每一个线程可以安全的从不同线程填充的共享内存的位置读取;
- 处理共享内存数据;
- 如果需要更新共享内存的结果,再次同步;
- 将结果写入设备内存.

### 3.1. host 和 device 之间的数据传输

应用需要竭力减少 host 和 device 之间的数据传输.一种方式是通过将更多的代码从 host 迁移到 device,尽管这样可能会是 device 的 kernel 的计算并行度减少,但是临时数据结构可以在 device 上创建,执行,销毁,这个过程不需要映射到 host 或者传到 host memory.

另一种方式将小的传输数据 batch 到一起形成大的数据再传输的性能优于独立的数据传输.

在前端总线系统中,使用 page-locked memory 进行 host 和 device 之间的数据传输能达到更高的效率.

另外,在使用 mapped page-locked memory 时不需要在 device 另外分配一块空间,也不用显式的进行内存拷贝.数据的传输是每次 kernel 访问内存是自动完成的,为了获得最大的性能，这些内存访问必须与全局内存访问合并在一起.

在一些 CPU 和 GPU 一体化的系统中,host memory 和 device memory 是一体的.内存拷贝是不需要的,需要使用映射叶锁定内存代替.

### 3.2. device memory 访问

一个访问可寻址内存(全局内存,本地内存,共享内存,常量或纹理内存)的指令可能需要重新发行多次,取决于内存地址贯穿 warp 中的线程的分布.
(省略)

## 4. 指令吞吐量最大化

应用为了最大化指令吞吐量应当:

- 最小化低吞吐量的算数指令的调用,这包括在不影响最终结果的情况下以精度换速度.例如用固有函数代替通用函数,单精度代替双精度,将非正态化的数据刷新为 0;
- 最小化由控制流程的指令引起的分歧 warps;
- 减少指令的数量,例如尽量优化掉同步点(synchronization point)使用限制点(\_\_restrict\_\_).

### 4.1. 算术指令

**单精度浮点型除法**
固有函数\_\_fdividef(x,y)提供了比除法运算符更快的单精度浮点型除法.
**单精度浮点型平方根倒数**
为了保留 IEEE-754 语义,编译器只有在倒数和平方根都是近似值时可以将 1.0/sqrtf()优化为 rsqrtf()(如 -prec-div=false 和 prec-sqrt=false).因此,建议直接调用 rsqrtf()在需要的地方.
**单精度浮点型平方根**
单精度浮点平方根实现为倒数平方根后面跟的是倒数而不是倒数的平方根后面跟的是乘法所以它给出了 0 和 ∞ 的正确结果.

### 4.2. 控制流指令

控制流指令(if,swtich,for,do,while)可以显著的影响有效的指令吞吐量通过将 warp 中的多个线程执行路径分离.如果发生了这种情况,不同的执行路径需要顺序化执行,这将增加 warp 执行的指令的总数.

在控制流依赖于 warp 中的指定线程 ID 的情况下为了获得最好的性能,控制条件被改写使得分离的 warps 数量最小化.

### 4.3. 同步指令(Synchronization instruction)

|    计算能力    | 吞吐量(operations per clock cycle) |
| :------------: | :--------------------------------: |
|      3.x       |                128                 |
|      6.0       |                 32                 |
| 5.x 6.1 和 6.2 |                 64                 |
|      7.x       |                 16                 |
