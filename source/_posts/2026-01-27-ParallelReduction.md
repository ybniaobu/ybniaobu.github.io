---
title: 并行约归 Parallel Reduction
date: 2026-01-27 13:16:11
categories: 
  - [算法, Compute Shader]
tags:
  - Compute Shader
  - 算法
  - GPGPU
top_img: /images/black.jpg
cover: https://s2.loli.net/2026/01/27/jeC94fMAuYXOrDU.gif
mathjax: true
description: 本文章主要讲解如何在 Compute Shader 中更有效地进行约归操作，比如求和。
---

> 最近在做 **Reflection Probe (Cubemap) Normalization** 的时候，需要计算 Cubemap 对应的球谐系数，而球谐系数的计算需要遍历整个球面求和，之前在 [IBL 基于图像的光照（一）](https://ybniaobu.github.io/2024/07/09/2024-07-09-IBL_Basics1/) 文章中计算时，是在 CPU 中计算球谐系数的，仅仅重建 irradiance 的工作是在 GPU 完成的。然而在 CPU 中单线程遍历纹理纹素的效率相对较低，我就想到能不能使用 Compute Shader 并行采样纹理计算求和，于是就查阅到了 NVIDIA 的这篇文章：[Optimizing Parallel Reduction in CUDA](https://developer.download.nvidia.com/assets/cuda/files/reduction.pdf) ，下面的内容都是基于这篇文章写的。  
>   
> 另外，想了解 Reflection Probe Normalization 技术，可以查看 God of War 在 GDC 2019 的这篇演讲：[The Indirect Lighting Pipeline of God of War](https://media.gdcvault.com/gdc2019/presentations/Hobson_Josh_The_Indirect_Lighting.pdf) 。

# Parallel Reduction 简介
**归约操作 Reduction Operation** 是对集合中的元素进行某种累积计算，最终产生单个结果，例如求和、求最大值或最小值等，而**并行约归 Parallel Reduction** 就是利用 GPU 多线程的并行计算特性来对归约操作进行加速（当然 CPU 多线程也是可以的，只是 GPU 拥有更多核心）。整体算法的思路，就是在每个线程组当中，使用基于树结构的方法进行 reduce，每个线程组 reduce 一部分的原始数据数组。每个线程组计算出部分数据的和后，可以在 CPU 里进行最终的求和，也通过另开 kernel 用递归的方式完成计算，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2026/01/27/ZROQXIp4bwAuYBT.png" width = "80%" height = "80%" alt="图1 - Tree-based approach within each thread block。Avoid global sync by decomposing computation
into multiple recursive kernel invocations。Block 即线程组（CUDA 中的叫法）。"/>
</div>

在原文章中一共提供了 7 个 Parallel Reduction 算法，每个算法都是基于上一个算法进行优化的，并且运行、带宽 Bandwidth 效率逐渐提高，下面来一一讲解，并将 CUDA 代码都转换为了 Compute Shader 的代码。

# Reduction #1: Interleaved Addressing

    #pragma kernel InterleavedAddressing1
    #define THREAD_NUM 16

    StructuredBuffer<float> _Source;
    RWStructuredBuffer<float> _Result;

    groupshared float sharedMem[THREAD_NUM];

    [numthreads(THREAD_NUM, 1, 1)]
    void InterleavedAddressing1(uint3 id : SV_DispatchThreadID, uint3 groupThreadId : SV_GroupThreadID, uint3 groupId : SV_GroupID, uint groupIndex : SV_GroupIndex)
    {
        // each thread loads one element from global to shared mem
        sharedMem[groupIndex] = _Source[id.x];
        GroupMemoryBarrierWithGroupSync();

        // do reduction in shared mem
        for (uint s = 1; s < THREAD_NUM; s <<= 1)  // stride: 1, 2, 4, 8
        {
            if (groupIndex % (2 * s) == 0)
            {
                sharedMem[groupIndex] += sharedMem[groupIndex + s];
            }
            GroupMemoryBarrierWithGroupSync();
        }

        // write result for this block to global mem
        if (groupIndex == 0) _Result[groupId.x] = sharedMem[0];
    }

这个算法的每个线程组，在循环的每一次迭代中求两个数据的和，通过增加步长 stride 对上一次循环的结果再次进行两两相加，最终每个线程组的合计数额在 groupshared 数组的第一位，示意图如下： 

<div align="center">  
<img src="https://s2.loli.net/2026/01/27/bYKmWUDz8jx2ldV.png" width = "70%" height = "70%" alt="图2 - Reduction #1: Interleaved Addressing."/>
</div>

这个算法的问题在于取余操作较慢，并且循环中的 if 语句会生成大量的 **Warp divergence**（Warp 是 GPU 调度的基本单元，在同一个 Warp 内的线程，若执行过程中分别进入了不同的分支，同一个 Warp 内无法同时执行不同分支，就会导致阻塞并影响性能）。

# Reduction #2: Interleaved Addressing
没看错，还是 Interleaved Addressing。为了解决上述问题，可以将 for 循环中的内容替换为如下：  

    for (uint s = 1; s < THREAD_NUM; s <<= 1)  // stride: 1, 2, 4, 8
    {
        int index = 2 * s * groupIndex;
        
        if (index < THREAD_NUM)
        {
            sharedMem[index] += sharedMem[index + s];
        }
        GroupMemoryBarrierWithGroupSync();
    }

新的代码去掉了取余操作，然后 if 分支中线程是连续的，缓解了 highly divergent warps 的问题，示意图如下： 

<div align="center">  
<img src="https://s2.loli.net/2026/01/27/4LNBi2FnMIYpOZT.png" width = "70%" height = "70%" alt="图3 - Reduction #2: Interleaved Addressing."/>
</div>

然而仍然有个问题，就是 Shared Memory 的 **Bank Conflicts** 问题。Shared Memory 在设计的时候为了并行访问速度，给 Shared Memory 设计了 32 个 banks，每个 bank 里包含许多 word（每个 word 32 bit，4 byte），每个 bank 负责多少数据的访问取决于申请了多少的 Shared Memory 的大小。当同一个 warp 里的多个线程对同一个 bank 发出访问请求，只能进行串行访问，这就是 Bank Conflicts。

# Reduction #3: Sequential Addressing
这次是更改整个 for 循环：  

    for (uint s = THREAD_NUM / 2; s > 0; s >>= 1) // stride: 8, 4, 2, 1
    {
        if (groupIndex < s)
        {
            sharedMem[groupIndex] += sharedMem[groupIndex + s];
        }
        GroupMemoryBarrierWithGroupSync();
    }

这样就解决了 Bank Conflicts 问题，示意图如下：  

<div align="center">  
<img src="https://s2.loli.net/2026/01/27/2c6jsqxYJOUDVNP.png" width = "70%" height = "70%" alt="图4 - Reduction #3: Sequential Addressing."/>
</div>

接下来是闲置线程 Idle Threads 的问题，可以看到在第一次循环中，有一半的线程处于闲置状态，毕竟浪费。

# Reduction #4: First Add During Load
改进的思路是在拷贝 global memory 到 shared memory 的时候，就进行一次加法：  

    // each thread loads one element from global to shared mem
    sharedMem[groupIndex] = _Source[id.x];
    GroupMemoryBarrierWithGroupSync();

改为：  

    uint i = THREAD_NUM * 2 * groupId.x + groupIndex;
    sharedMem[groupIndex] = _Source[i] + _Source[i + THREAD_NUM];
    GroupMemoryBarrierWithGroupSync();

这里要注意的是**线程组的数量是要减半的！！！！**。即 groupId.x 的最大值发生了变化，每个线程组中的线程不变，但总的线程数减少了一半。接下来为了继续提升性能，而性能瓶颈在指令调度上，可以将 for 循环展开。

# Reduction #5: Unroll the Last Warp
一个 warp 中有 32 个线程，意味着，当 groupIndex <= 32 的时候，其实线程组并不需要等待其他线程（warp），即不需要 `GroupMemoryBarrierWithGroupSync()`，也不需要 if (groupIndex < s)，于是可以将循环的最后 6 次迭代展开：  

    for (uint s = THREAD_NUM / 2; s > 32; s >>= 1) 
    {
        if (groupIndex < s) 
        {
            sharedMem[groupIndex] += sharedMem[groupIndex + s];
        }
        GroupMemoryBarrierWithGroupSync();
    }
    
    if (groupIndex < 32)
    {
        sharedMem[groupIndex] += sharedMem[groupIndex + 32];
        sharedMem[groupIndex] += sharedMem[groupIndex + 16];
        sharedMem[groupIndex] += sharedMem[groupIndex + 8];
        sharedMem[groupIndex] += sharedMem[groupIndex + 4];
        sharedMem[groupIndex] += sharedMem[groupIndex + 2];
        sharedMem[groupIndex] += sharedMem[groupIndex + 1];
    }

顺着这个思路，我们其实可以将 for loop 都展开，只要我们知道迭代次数。

# Reduction #6: Completely Unrolled
完全展开后的代码：  

    if (THREAD_NUM >= 1024)
    {
        if (groupIndex < 512) { sharedMem[groupIndex] += sharedMem[groupIndex + 512];} 
        GroupMemoryBarrierWithGroupSync();
    }
    if (THREAD_NUM >= 512) 
    {
        if (groupIndex < 256) { sharedMem[groupIndex] += sharedMem[groupIndex + 256]; }
        GroupMemoryBarrierWithGroupSync();
    }
    if (THREAD_NUM >= 256) 
    {
        if (groupIndex < 128) { sharedMem[groupIndex] += sharedMem[groupIndex + 128]; } 
        GroupMemoryBarrierWithGroupSync();
    }
    if (THREAD_NUM >= 128) 
    {
        if (groupIndex <  64) { sharedMem[groupIndex] += sharedMem[groupIndex +  64]; } 
        GroupMemoryBarrierWithGroupSync();
    }
    
    if (groupIndex < 32)
    {
        sharedMem[groupIndex] += sharedMem[groupIndex + 32];
        sharedMem[groupIndex] += sharedMem[groupIndex + 16];
        sharedMem[groupIndex] += sharedMem[groupIndex + 8];
        sharedMem[groupIndex] += sharedMem[groupIndex + 4];
        sharedMem[groupIndex] += sharedMem[groupIndex + 2];
        sharedMem[groupIndex] += sharedMem[groupIndex + 1];
    }

# Reduction #7: Multiple Adds / Thread
这个 Reduction #7，我没太看明白，是对 Reduction #4: First Add During Load 的进一步改进，Reduction #4 只在 Load 阶段进行了一次加法，其实可以使用 loop 加更多次，但是我没太看懂代码，我感觉若是 compute shader 实现的话，应该是要改变 dispatch 的数量，自己上传 gridDim 的，我不太确定。下面就直接摘抄 CUDA 的代码了，原本 Reduction #4 的代码为：  

    unsigned int tid = threadIdx.x;
    unsigned int i = blockIdx.x*(blockDim.x*2) + threadIdx.x;
    sdata[tid] = g_idata[i] + g_idata[i+blockDim.x];
    __syncthreads();

替换为：  

    unsigned int tid = threadIdx.x;
    unsigned int i = blockIdx.x*(blockSize*2) + threadIdx.x;
    unsigned int gridSize = blockSize*2*gridDim.x;
    sdata[tid] = 0;

    while (i < n) {
        sdata[tid] += g_idata[i] + g_idata[i+blockSize];
        i += gridSize;
    }
    __syncthreads();

最终代码为：  

``` C++
template <unsigned int blockSize>
__device__ void warpReduce(volatile int *sdata, unsigned int tid) {
    if (blockSize >= 64) sdata[tid] += sdata[tid + 32];
    if (blockSize >= 32) sdata[tid] += sdata[tid + 16];
    if (blockSize >= 16) sdata[tid] += sdata[tid + 8];
    if (blockSize >= 8) sdata[tid] += sdata[tid + 4];
    if (blockSize >= 4) sdata[tid] += sdata[tid + 2];
    if (blockSize >= 2) sdata[tid] += sdata[tid + 1];
}

template <unsigned int blockSize>
__global__ void reduce6(int *g_idata, int *g_odata, unsigned int n) {
    extern __shared__ int sdata[];
    unsigned int tid = threadIdx.x;
    unsigned int i = blockIdx.x*(blockSize*2) + tid;
    unsigned int gridSize = blockSize*2*gridDim.x;
    sdata[tid] = 0;
    while (i < n) { sdata[tid] += g_idata[i] + g_idata[i+blockSize]; i += gridSize; }
    __syncthreads();
    if (blockSize >= 512) { if (tid < 256) { sdata[tid] += sdata[tid + 256]; } __syncthreads(); }
    if (blockSize >= 256) { if (tid < 128) { sdata[tid] += sdata[tid + 128]; } __syncthreads(); }
    if (blockSize >= 128) { if (tid < 64) { sdata[tid] += sdata[tid + 64]; } __syncthreads(); }
    if (tid < 32) warpReduce(sdata, tid);
    if (tid == 0) g_odata[blockIdx.x] = sdata[0];
}
```

# 总结
文章中给出了上述 7 种算法的性能对比测试，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2026/01/27/aZLYFQHbxtGWdnv.png" width = "60%" height = "60%" alt="图5 - Performance for 4M element reduction"/>
</div>

其实绝大部分情况下，我们不需要优化到这么牛逼的地步，特别是进行离线计算的时候，基本上 #2、#3 的算法够用了，代码也更漂亮一点。