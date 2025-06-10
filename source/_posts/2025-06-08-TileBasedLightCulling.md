---
title: Tile-Based Light Culling
date: 2025-06-08 13:03:10
categories: 
  - [图形学]
  - [unity, pipeline]
tags:
  - 图形学
  - 游戏开发
  - unity
top_img: /images/black.jpg
cover: https://s2.loli.net/2025/06/08/1hF5QplnZjxBvJS.gif
mathjax: true
description: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
---

> 本篇文章主要参考了这篇文章： https://www.cnblogs.com/KillerAery/p/17139487.html 、 https://github.com/wlgys8/SRPLearn/wiki/DeferredShading#34-depth-slice 、https://wickedengine.net/2018/01/optimizing-tile-based-light-culling/ ，并作出了一定的补充。其他参考文献如下：  
> ① DirectX 11 Rendering in Battlefield 3 : https://www.slideshare.net/slideshow/directx-11-rendering-in-battlefield-3/7207663 ;  
> ② A 2.5D Culling for Forward+ (SIGGRAPH ASIA 2012) ：https://www.slideshare.net/slideshow/a-25d-culling-for-forward-siggraph-asia-2012/34909590 ；  
> ③ Improve Tile-based Light Culling with Spherical-sliced Cone ：https://lxjk.github.io/2018/03/25/Improve-Tile-based-Light-Culling-with-Spherical-sliced-Cone.html ；  
> ④《GPU Pro 6》chapter VI - 1 ：Compute-Based Tiled Culling 以及 《GPU Pro 7》chapter II - 2 ：Fine Pruned Tiled Light Lists ；
> ⑤ Cull that cone! Improved cone/spotlight visibility tests for tiled and clustered lighting ：https://bartwronski.com/2017/04/13/cull-that-cone/ ；  
> ⑥ Improved Culling for Tiled and Clustered Rendering ：https://advances.realtimerendering.com/s2017/2017_Sig_Improved_Culling_final.pdf 。

# 多光源渲染优化技术简介
随着现代游戏渲染引擎对多光源的需求，传统的遍历所有光源的方式在性能上无法满足这个需求，即便是传统的 Deferred Rendering。传统 Deferred Rendering 相对传统 Forward Rendering 能够支持更多光源，是因为其将光照计算与场景几何体渲染进行了解耦，但是面临巨量灯光，仍然存在性能问题，毕竟还是要遍历全屏像素和每盏灯计算光照。Deferred Rendering 的 **Light Volume** 的方案能在一定程度上减少多光源渲染的性能问题，但在效率上仍然不如光照剔除技术，好处就是可以不使用 Compute Shader 从而支持更多平台。为了更好地支持大量光源，出现了很多光源剔除技术，主要有 **Tile-based Light Culling** 和 **Cluster-based Light Culling**，由于这些技术无论 Forward 还是 Deferred 渲染都可以使用，故也被称为 **Forward+** 或者 **Deferred+**。

## Light Volume 简介
**Light Volume** 是一个只能在 Deferred Rendering 中使用的技术，它通过绘制光照几何体的形式进行光照计算，这样子就可以只就几何体内部的像素进行光照计算，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/09/pE6kWNGjRMu8zLD.png" width = "30%" height = "30%" alt="图1 - 左：spot light；右：point light"/>
</div>

Light Volume 还有另外的形式，比如 Screen-Space Quads，就是灯光都用一个屏幕上的小方块来表示，传说中的那个游戏 5 (GTA 5) 的远处的小灯光都是用这种形式渲染的（近景用的应该是 Light Volume），如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/09/CEJpXMrmk9zhsKq.png" width = "100%" height = "100%" alt="图2 - 左：Light Quads；中：Light Quads After Depth Test；右：Scene"/>
</div>

> 关于 GTA 5 的整体渲染流程，可以看这篇文章：https://www.adriancourreges.com/blog/2015/11/02/gta-v-graphics-study/ 。我又看了一篇关于荒野大镖客 2 的文章，本地灯光仍然使用的是 Light Volume，但是渲染 Environment Map 使用了 Tile-based Light Culling，不知道这么选择的原因是什么，文章如下：https://imgeself.github.io/posts/2020-06-19-graphics-study-rdr2/ 。

常见的 Light Volume 绘制方式有两种：  
① **Double Pass Stencil & Depth Test**：这也是米哈游的崩坏星穹铁道的绘制方式，通过两个 Pass 分别绘制 Light Volume 的正面和背面。第一个 Pass 绘制正面，深度测试为 LEqual，模板测试为 ALWAYS，在模板和深度测试均通过的情况下写入模板值，这样子在 Light Volume 的正面后面的像素都会写入模板值。第二个 Pass 绘制背面，深度测试为 GEqual，模板测试为 EQUAL，这样子在 Light Volume 的背面的前面的像素，跟第一个 Pass 的交集，即为要计算 lighting 的像素。  
② **Depth Bounds Test**：大镖客 2 使用的就是这样方式，Depth Bounds Test 是一个硬件特性，现在的 PC 显卡应该都支持这一特性，即用户可以定义一个最大和最小深度值，和 fragment 的深度进行比较，在范围外即 discard 掉 fragment。可以在绘制每个光源的时候，在 CPU 计算它的 depth bound 并设置给 graphics pipeline。我好像没找到 Unity 的任何 API 是关于 Depth Bounds Test 的，不知道以后会不会支持。

## Tile-based Light Culling 简介
Tile-based Light Culling 是一个被不断改进过的技术，简单地来说，其就是将屏幕划分为多个 tile ，可能会影响到某个 tile 的光源会被记录在一个列表中。在执行渲染时，给定 tile 中的像素着色器会使用该 tile 对应的光源列表，来对表面进行着色，从而达到减少光照计算的目的，如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/09/oET8l1qGOufyJ4P.png" width = "60%" height = "60%" alt="图3 - Tile-based Light Culling"/>
</div>

前面的文章 [Unity Custom SRP 基础（八）](https://ybniaobu.github.io/2025/04/29/2025-04-29-CustomSRP8/#Simple-Tiled-Forward) 就实现过类似上面所说的版本，即拿光源在屏幕上的 AABB 包围盒和 tile 进行求交。这样做的弊端就是得到的光源列表非常粗糙，忽略了 Z 轴，如上图中的右图，红色线段为该 tile 中的最近最远距离，理论上来说光源 3 不会影响到该 tile，但是之前的粗糙剔除的方式无法将光源 3 正确剔除，这就引进了基于深度的剔除，即采样 Depth Texture，得到一个 tile 中的最大最小深度进行剔除，但也因此 Tile-based Light Culling 对透明物体光照渲染支持较差。

但即使根据 Z 轴剔除光源，在某些场景中，仍然做不到有效的光源剔除，比如森林等植被场景，会有很高比例的 tile 出现背景是遥远的山，同时包含了一片树叶，那么此时 Z 的范围是巨大的。因此有人提出了 **2.5D Culling**，即将 tile 沿着深度方向分割为 n 个单元格，然后创建一个包含 n 个 bit 的几何位掩码，在存在几何体的地方，对应的 bit 会被设置为 1，然后对所有的光源进行迭代，并为每个与 tile 视锥体重叠的光源都创建一个光源位掩码。这个几何位掩码和光源位掩码使用按位与（AND）操作，如果结果为零，那么说明这个光源不会影响 tile 中的任何几何物体，如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/09/mcTnR7tNiD4A3Px.png" width = "60%" height = "60%" alt="图4 - 2.5D Culling"/>
</div>

详细介绍见后面。

## Cluster-based Light Culling 简介
Tile-based 的光源分类使用的是一个基于 tile 的二维空间范围，以及几何物体的深度边界；而 Cluster-based 则将视锥体划分为一组三维单元格，称为**簇 cluster**，簇的划分是在整个视锥体上进行的，独立于场景中的几何形状，如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/09/eJoPiS7p2bhuWKr.png" width = "60%" height = "60%" alt="图5 - Cluster-based Light Culling"/>
</div>

光源会基于 cluster 来进行分类，并形成对应的光源列表，由于 cluster 并不依赖场景几何的 z-depth，故无论是透明的还是不透明的物体，都可以使用该物体的位置来检索相关的光源列表。本篇文章就不详细介绍该技术了，以后实现的时候再行阐述。


# Compute Shader 同步相关概念简介
这里略微介绍一下之前没怎么遇到过的 Compute Shader 的一些概念，方便后面 Tile-based Light Culling 的 Compute Shader 编写。

## GPU 架构简介
架构这里就简单介绍一下，详细请看 NVIDIA 的 Ada 显卡架构白皮书：https://images.nvidia.com/aem-dam/Solutions/geforce/ada/nvidia-ada-gpu-architecture.pdf 。Ada 架构应该是前几年的主力架构，最新架构应该是 Blackwell。

GPU 由多个**图像处理簇 Graphics Processing Clusters (GPCs)** 组成，GPCs 包括数个**纹理处理簇 Texture Processing Clusters (TPCs)**，TPCs 又包括数个**流式多处理器 Stream Multiprocessors (SMs)**，如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/10/lo7u3nYyTdEmfUP.jpg" width = "80%" height = "80%" alt="图6 - Ada GPU Architecture"/>
</div>

SMs 是理解 Compute Shader 的重点，SM 可以被分为多个 processing blocks，每个 processing block 包含一个**寄存器堆 Register File**、一个**L0 指令缓存 L0 instruction cache**、一个**线程束调度器 warp scheduler**、一个**指令分发单元 dispatch unit**、数个 **CUDA Cores**（基础并行计算单元，用于处理通用浮点/整数运算）和一个 **Tensor Core**（专用 AI/深度学习加速单元，针对矩阵运算优化）等等，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/10/27wjQ5LAXoKtd6g.jpg" width = "45%" height = "45%" alt="图7 - Ada Streaming Multiprocessor (SM)"/>
</div>

**线程束 warp** 是 SM 的基本执行单元，一个 warp 包含 32 个并行的**线程 Thread**。这也是为什么 compute shader 的 numthreads 要设置为 64 的倍数的原因，假如我们使用 numthreads 设置每个线程组只有 10 个线程，但是由于 SM 每次调度一个 Warp 就会执行 32 个线程，这就会造成有 22 个线程是不干活的，之所以是 64 是因为 AMD 显卡相对于 warp 概念的叫 **Wavefront**，由 64 个线程组成。

一个 SM 可同时处理多个**线程组 Thread Block/Group**（如 Ada SM 最多处理 32 个线程组），反之一个线程组也可能被拆分到多个 SM（不太常见）。线程组其实算是软件上的概念，compute shader 里叫 Thread Group，CUDA 编程里叫 Thread Block。

## Memory Types in GPU
下面介绍一下 GPU 里的内存类型，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/10/8r7YAf9XvHLFEzi.webp" width = "50%" height = "50%" alt="图8 - Memory Types in GPU"/>
</div>

简单地来说可以分为：  
①**本地内存 Local Memory**：每个线程拥有的局部内存；  
②**共享内存 Shared Memory**：每个线程组内所有线程共享的内存，即带有 `groupshared` 关键字的变量；  
③**全局内存 Global Memory**：GPU 中最大、最主要内存区域，所有线程组共享。有时也被称为 **Device Memory**，但严格来说不完全是同个意思；  
④**Texture Memory and Constant Memory** 这个就不多说了，应该非常熟悉了。

## Memory Barrier
由于 GPU 的并行特性，对共享资源的访问需要特别注意同步问题，以确保数据一致性以及避免**数据竞争 Data Race**，即**内存一致性 Memory Coherency**。这就涉及到了**内存屏障 Memory Barrier** 这个概念，诸如以下函数：`GroupMemoryBarrier()`、`DeviceMemoryBarrier()` 和 `AllMemoryBarrier()`，顾名思义，这三个对应的是 Shared Memory、Global Memory 以及所有 Memory。在线程调用 Memory Barrier 之后，才能确保共享内存的写入对其他线程可见，但 MemoryBarrier 系列函数无法保证线程同步，即它不等待所有线程都执行到此点。这就又涉及到了**执行屏障 Execution Barrier**，用于保证屏障前的代码已被组内所有线程执行完毕，对应 `GroupMemoryBarrierWithGroupSync()`、`DeviceMemoryBarrierWithGroupSync()`、`AllMemoryBarrierWithGroupSync()` 这三个函数，它们既是内存屏障也是执行屏障。总的来说，内存屏障保证的是内存可见性同步，执行屏障保证的是线程执行同步，滥用或忘记使用屏障会导致极其难以调试的数据竞争和错误结果。

还有额外的一点是，即便加了 DeviceMemoryBarrier 或者 AllMemoryBarrier，Global Memory 这个时候也只能保证在 Group 内部操作的可见性，默认情况下 Global Memory 资源的访问不保证跨线程组或 SM 的写入立即可见性或读取缓存一致性。GPU 可以利用缓存优化性能，但这可能导致不同线程看到的内存状态不一致。此时，需要用 `globallycoherent` 关键字修饰 Global Memory 资源，要求 HLSL 编译器对该特定变量的所有访问（读/写）都必须遵循严格的全局内存一致性。使用 globallycoherent 会带来额外的性能开销。

## Atomic Operations
内存一致性的问题还有一个内存操作顺序的问题，简单地来说，所有线程都在给一个共享变量加一，如果这个加一操作不是原子的，那么在 A 线程正在更新这个位置时，B 线程可能会同时访问这个位置，导致 B 线程得到的值是旧的、不正确的。而**原子操作 Atomic Operations** 提供了一种无锁、线程安全的方式来操作共享变量，可以说它是并行编程的基石，是实现跨线程安全访问共享资源的核心机制，确保了在并行环境下对同一内存地址的读写操作具有不可分割性（即操作执行过程中不会被其他线程中断）。

HLSL 中的原子操作函数有如下：`InterlockedAdd()`，`InterlockedAnd()`，`InterlockedCompareExchange()`，`InterlockedCompareStore()`，`InterlockedExchange()`，`InterlockedMax()`，`InterlockedMin()`，`InterlockedOr()`，`InterlockedXor()`。

## Wave Operations
**波形操作 Wave Operations** 则允许 Warp 或 Wave 内的线程之间进行通信，允许同一 Wave 内的线程直接使用共享寄存器交换数据，无需通过共享内存，从而显著提升性能和简化代码。Wave Operations 需要 Shader Model 6.0。这里我先就不详细介绍了，大概知道这么一个概念就行，以后实现效果遇到了，再详细学习如何使用。


# Tile-Based Light Culling
## 整体流程
CPU 中的操作我这里就不做记录了，根据需要的数据资源写就行，这里只摘录 GPU 中的流程。

## Depth Texture

## 球形相交测试
### Sphere-Frustum Test
### Cone Test
### Spherical-sliced Cone Test
## Spot Light 相交测试