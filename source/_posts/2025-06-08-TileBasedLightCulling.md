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
先说一下 Tile-Based Light Culling 的整体基本流程，我们要构建 tile 在视锥体 view frustum 下的包围体，包含上下左右四个平面。而基于深度的剔除，则通过采样深度贴图（必须要有 z-prepass）计算每个 tile 的最大最小深度，这样就组成了 tile 包围体的前后平面。拿这 6 个平面跟灯光包围体做相交测试，通过则记录下灯光索引。具体的 compute shader 中步骤如下：  
①将屏幕按照 16 x 16 进行分块，每个线程组的线程数量也是 16 x 16，每个线程组代表一个 tile；  
②初始化 groupshared 变量，主要包括每个 tile（线程组）中的最大最小深度、每个 tile 通过测试的灯光数量以及索引；  
③每个 tile（线程组）中的每个线程代表屏幕上的对应像素，获取 Depth Texture 的深度值，并用原子操作计算出每个 tile 的最大最小深度值，记录在该 tile（线程组）的 groupshared 变量里；  
④每个线程代表一盏灯光（故只支持 16 x 16 = 256 盏灯，超出 256 需要 for 循环）与当前 tile（线程组）做相交测试，通过则将灯光数量和索引记录到该 tile（线程组）的 groupshared 变量里；
⑤将记录灯光数量和索引 groupshared 变量传递到 RWStructuredBuffer 中，方便后面着色（灯光计算）时使用。

每个 tile（线程组）的 groupshared 变量，以及初始化如下：  

    #define THREAD_NUM_X 16 // equal to per tile size
    #define THREAD_NUM_Y 16

    groupshared uint tileMinDepthInt;
    groupshared uint tileMaxDepthInt;
    groupshared uint lightCountInTile;
    groupshared uint lightIndicesInTile[MAX_PUNCTUAL_LIGHT_COUNT];

    [numthreads(THREAD_NUM_X, THREAD_NUM_Y, 1)]
    void TiledLightCulling(uint3 id : SV_DispatchThreadID, uint3 groupThreadId : SV_GroupThreadID, uint3 groupId : SV_GroupID, uint groupIndex : SV_GroupIndex)
    {
        // initialize shared memory
        if (groupIndex == 0)
        {
            tileMinDepthInt = 0x7f7fffff;
            tileMaxDepthInt = 0;
            lightCountInTile = 0;
        }

        GroupMemoryBarrierWithGroupSync();
        ...
    }

`0x7f7fffff` 是 float 作为 uint 的最大值，因为原子操作只能比较 uint 或 int 需要把深度按位转换为 uint。最后别忘了组同步。

## Depth Intersect Test
### Load Depth Texture
这一步比较简单，就是每个线程使用 texture.load 读取 Depth Texture 深度值，并转换为 view space 下的线性深度，与 groupshared 变量进行 InterlockedMin 和 InterlockedMax 操作得到每个 tile 的最大最小深度：  

    void TiledLightCulling(...)
    {    
        // get the minimum and maximum depth of the tile
        bool inScreen = (int) id.x < _CameraBufferSize.z && (int) id.y < _CameraBufferSize.w;

        if (inScreen)
        {
            float depth = LOAD_TEXTURE2D_LOD(_CameraDepthTexture, id.xy, 0).r;
            float linearDepth = GetViewDepthFromDepthTexture(depth);
            uint z = asuint(linearDepth);
            InterlockedMin(tileMinDepthInt, z);
            InterlockedMax(tileMaxDepthInt, z);
        }

        GroupMemoryBarrierWithGroupSync();
    }

如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/12/jQ2xcw1EHb6B3lK.jpg" width = "30%" height = "30%" alt="图9 - Min Max Depth"/>
</div>
  
### Depth Test
然后就是和灯光进行深度相交测试了，测试也很简单：灯光最小深度小于 tile 最大深度，并且灯光最大深度大于 tile 最小深度，即相交，代码如下：  

    bool DepthIntersectTest(float3 lightPositionVS, float lightRange)
    {
        float tileDepthMin = asfloat(tileMinDepthInt);
        float tileDepthMax = asfloat(tileMaxDepthInt);

        float lightDepthMin = -lightPositionVS.z - lightRange;
        float lightDepthMax = -lightPositionVS.z + lightRange;
        
        return lightDepthMin <= tileDepthMax && lightDepthMax >= tileDepthMin;
    }

    bool IntersectTest(uint lightIndex, uint2 tileIndex)
    {
        if ((float) lightIndex >= GetPunctualLightCount()) return false;

        float3 lightPositionVS = TransformWorldToView(GetPunctualLightPosition(lightIndex));
        float lightRange = GetPunctualLightRange(lightIndex);

        if (!DepthIntersectTest(lightPositionVS, lightRange)) return false;
        ...
    }

`lightPositionVS.z` 之所以取反是因为 Unity 的 view matrix 是基于右手坐标系的，且摄像机面向 -z 轴方向。只进行深度相交测试，通过 debug 输出每个 tile 的灯光数量后，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/12/1EpankqobfOVmi2.jpg" width = "60%" height = "60%" alt="图10 - Depth Test"/>
</div>

可以看到底部 plane 边缘的 tile 因为深度范围很大，也判定为了相交。如果这样的情况发生在灯光范围内，就会少剔除掉很多灯光，这也是之前所说 2.5 culling 的原因。

## Sphere Intersect Test
我们这里先假设 spot light 跟 point light 一样也是球形范围进行剔除（spot light 的特殊处理后面再说），所以对于灯光球形包围盒，相交测试问题就是 tile 视锥体包围盒与球形包围盒求交。这个相交测试原理比较简单，就是灯光（球心）跟 tile 视锥体包围盒的上下左右 4 个平面（前后深度测试已经处理好了）的距离同时小于灯光范围（球半径）即可，这样子说不太严谨，后面会说原因。所以我们首先要计算这 4 个平面。

### Frustum Planes Calculation
因为我们要在 view space 做相交测试，所以需要知道每个 tile 在视锥体近裁切平面（或远裁切平面）的 4 个点的坐标，从而构建出 4 个平面，计算它们的法向，和灯光中心求距离。至于为什么 4 个点就可以构建 4 个平面，是因为 tile 视锥体一定和摄像机（view space 原点）相交，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/12/qDgaYMTuHWloVkC.png" width = "45%" height = "45%" alt="图11 - tile frustums in x-z plane"/>
</div>

我们只需要计算整个视锥体近裁切平面最左下角的 view space 坐标，以及每个 tile 在近裁切平面上 x 和 y 轴方向上的大小，就可以根据 tile index 来计算出每个 tile 的四个顶点了。view space 相对于 world space 只是平移加旋转，大小是不会变的，故计算 cameraNearPlaneLB（左下角坐标）和 tile 在近裁切平面下的大小是比较容易的，如下：  

``` C#
float nearPlaneZ = data.camera.nearClipPlane;
float nearPlaneHeight = Mathf.Tan(Mathf.Deg2Rad * data.camera.fieldOfView * 0.5f) * 2 * nearPlaneZ;
float nearPlaneWidth = data.camera.aspect * nearPlaneHeight;
passData.cameraNearPlaneLB = new Vector3(-nearPlaneWidth / 2, -nearPlaneHeight / 2, -nearPlaneZ);
passData.tileNearPlaneSize = new Vector2(YPipelineLightsData.k_TileSize * nearPlaneWidth / data.BufferSize.x, YPipelineLightsData.k_TileSize * nearPlaneHeight / data.BufferSize.y);
```

传递给 GPU 后就可以计算出每个 tile 在近裁切平面上的四个顶点了：  

    float3 tileCorners[4];
    tileCorners[0] = _CameraNearPlaneLB.xyz + tileIndex.x * float3(_TileNearPlaneSize.x, 0, 0) + tileIndex.y * float3(0, _TileNearPlaneSize.y, 0);
    tileCorners[1] = tileCorners[0] + float3(0, _TileNearPlaneSize.y, 0);
    tileCorners[2] = tileCorners[1] + float3(_TileNearPlaneSize.x, 0, 0);
    tileCorners[3] = tileCorners[0] + float3(_TileNearPlaneSize.x, 0, 0);

tileIndex 就是线程组 id，即 `uint3 groupId : SV_GroupID`。这样每两个点和原点就可以构建出一个平面，从而形成 tile 视锥体包围盒。

### Sphere-Frustum Test
有了 tile 的 4 个点以后，就可以取 2 个点做叉乘，计算出垂直于该平面的法向量，归一化后，和灯光原点坐标（view space 下）点乘，就可以计算出灯光距离平面的距离了。之所以点乘就可以计算出距离，是因为 4 个包围平面都会经过 view space 原点，所以原点到灯光位置的向量在平面的法向量上的投影就是距离，测试代码如下：  

    bool SidePlanesIntersect(float3 p1, float3 p2, float3 lightPositionVS, float lightRange)
    {
        float3 N = -normalize(cross(p1, p2));
        float distance = dot(N, lightPositionVS);
        return distance < lightRange;
    }

    bool SphereFrustumIntersectTest(uint lightIndex, uint2 tileIndex)
    {
        ...
        float3 tileCorners[4];
        ...

        bool sideIntersected0 = SidePlanesIntersect(tileCorners[0], tileCorners[1], lightPositionVS, lightRange);
        bool sideIntersected1 = SidePlanesIntersect(tileCorners[1], tileCorners[2], lightPositionVS, lightRange);
        bool sideIntersected2 = SidePlanesIntersect(tileCorners[2], tileCorners[3], lightPositionVS, lightRange);
        bool sideIntersected4 = SidePlanesIntersect(tileCorners[3], tileCorners[0], lightPositionVS, lightRange);
        return sideIntersected0 && sideIntersected1 && sideIntersected2 && sideIntersected4 && DepthIntersectTest(lightPositionVS, lightRange);
    }

之前说了灯光（球心）跟 tile 视锥体包围盒的上下左右 4 个平面的距离同时小于灯光范围（球半径），这个条件不太严谨是因为如下图所示，此时灯光距离 01 平面的距离大于灯光半径，那么这个 tile 就会被认定为不相交，这是不合理的。这里就需要一点小技巧，因为我们知道法向量的方向决定了点乘的正负，而法向量方向由叉乘顺序决定（这里注意一下，因为我们的点坐标都是在 view space 这个右手坐标系下，故叉乘是右手法则）。如果 4 个平面的法向量都朝外（即如下图），那么此时灯光距离 01 平面的距离就为负数，故一定小于半径，而 23 平面仍可以正常求交，这样这个 tile 就可以正确判断相交了。这也是 `SidePlanesIntersect()` 函数中，法向量 N 取负号的原因（当然交换顺序也可以），我先传递了点 0 后点 1，这样右手法则是朝内的，取反就朝外了。

<div align="center">  
<img src="https://s2.loli.net/2025/06/12/OwTnsS3LBFfJUm6.jpg" width = "35%" height = "35%" alt="图12 - Sphere-Frustum Test"/>
</div>

不取负号，或者对距离做绝对值的后果就是灯光边缘的 tile 相交会错误，可以直观地看下面这张图片（注释掉了 depth test 的结果）：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/12/31gZSoLm2OjWYnP.png" width = "70%" height = "70%" alt="图13 - 左：法向量朝内或者距离取绝对值后，灯光边缘的一些 tile 相交测试错误；右：法向量朝外"/>
</div>

### Cone Test

> 这里主要参考了这篇博客文章： https://lxjk.github.io/2018/03/25/Improve-Tile-based-Light-Culling-with-Spherical-sliced-Cone.html 

可以看到 Sphere-Frustum Test 的剔除精度仍然不够，会引入 false positives 的 tile，比如说上图中在灯光球形外的一些 tile。也就是说即使灯光不会真正和 tile 视锥体包围盒相交，也可能会被误判为相交，这主要是因为计算距离时，假设了视锥体包围盒每个面都是无限距离。如下图中，球体在红色区域，仍然通过 Sphere-Frustum Test：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/13/BySTK58D7ZYAXk3.png" width = "50%" height = "50%" alt="图14 - Sphere-Frustum Test introduces false positives"/>
</div>

为了减少误判区域，可以假设一个从原点（摄像机）出发的圆锥体包裹着 tile 视锥体，灯光也是同理，然后测试两个 cone 是否相交，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/13/YWHgPfK6iNA4IOl.png" width = "30%" height = "30%" alt="图15 - Cone Test。红色区域为 Sphere-Frustum Test 的 false positives，蓝色区域为 Cone Test 的 false positives。"/>
</div>

而 Cone Test 的具体操作则是比较角度，求出原点到灯光中心的向量和原点到 tile 中心的向量的夹角，若这个夹角比灯光 cone 的顶点角的一半加上 tile cone 的顶点角的一半要小，则相交，代码如下：    

    bool ConeIntersect(float3 tileCorners[4], float3 lightPositionVS, float lightRange)
    {
        float3 tileCenterVec = normalize(tileCorners[0] + tileCorners[1] + tileCorners[2] + tileCorners[3]);
        float tileCos = min(min(min(dot(tileCenterVec, normalize(tileCorners[0])), dot(tileCenterVec, normalize(tileCorners[1]))), dot(tileCenterVec, normalize(tileCorners[2]))), dot(tileCenterVec, normalize(tileCorners[3])));
        float tileSin = sqrt(1 - tileCos * tileCos);

        float lightDistSqr = dot(lightPositionVS, lightPositionVS);
        float lightDist = sqrt(lightDistSqr);
        float3 lightCenterVec = lightPositionVS / lightDist;
        float lightSin = clamp(lightRange / lightDist, 0.0, 1.0);
        float lightCos = sqrt(1 - lightSin * lightSin);
        
        float lightTileCos = dot(lightCenterVec, tileCenterVec);
        float sumCos = (lightRange > lightDist) ? -1.0 : (tileCos * lightCos - tileSin * lightSin);

        return lightTileCos >= sumCos;
    }

tile 的中心向量即 4 个点的平均，然后选择这 4 个向量与中心向量的夹角的最大值（cos 最小值）作为 tile cone 的顶点角的半角度。灯光 cone 的半顶点角的正弦值可以通过灯光半径除以灯光距离计算出来，之所以 clamp 是因为灯光半径会大于灯光距离，此时摄像机在灯光球体内部。摄像机在灯光球体内部时，博客选择让所有 tile 都通过测试，故 sumCos 设置为了 -1。至于 sumCos 为什么这么算是因为这个三角函数方程：$\,cos(A + B) = cos(A)cos(B) - sin(A)sin(B)\,$。比较的时候，两个中心的夹角要比合计角度小才算相交，所以 cos 值是大于。最后别忘了和 Depth Test 一起使用，最终效果大致如下：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/13/bWo5w7uKPYGR4nc.png" width = "60%" height = "60%" alt="图16 - 左：Cone Test；右：Cone Test & Depth Test"/>
</div>

可以看到，已经完美适配球形了，只不过剔除精度还是不够，但这次是因为 depth test 引起的。故博客中升级了 Cone Test，称为 Spherical-Sliced Cone Test。

### Spherical-Sliced Cone Test
这里不使用 min/max depth 进行测试，而使用 min/max distance to camera 代替 depth，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/13/HJ8jYZ526yT4kRc.png" width = "65%" height = "65%" alt="图17 - Spherical-Sliced Cone Test 示意图"/>
</div>

由上图可知，我们要比较的是 tile cone 和球体相交处的最小最大距离，于是要得到图中的 Diff Angle，这个 Diff Angle 可由 tile 中心向量和 light 中心向量的夹角（即上面计算的 sumCos）减去 tile cone 的顶点角的一半而得，唯一的问题是 Diff Angle 可能会变为负值，即灯光中心位于 tile 内部的时候，此时直接设置为 0。角度相减的三角函数公式有：$\,sin(A - B) = sinAcosB - cosAsinB\,$、$\,cos(A - B) = cosAcosB + sinAsinB\,$。

    float diffSin = clamp(lightTileSin * tileCos - lightTileCos * tileSin, 0.0, 1.0);
    float diffCos = (diffSin == 0.0) ? 1.0 : lightTileCos * tileCos + lightTileSin * tileSin;
    float lightTileDistOffset = sqrt(lightRadius * lightRadius - lightDistSqr * diffSin * diffSin);
    float lightTileDistBase = lightDist * diffCos;

    if (lightTileCos >= sumCos && lightTileDistBase - lightTileDistOffset <= maxTileDist && lightTileDistBase + lightTileDistOffset >= minTileDist)
    {
        // light intersect this tile
    }

然后原博客中并没有给出 min/maxTileDist 的计算方式，在最后代码总结中使用了 min/maxTileDepth 替代了 min/maxTileDist，这是不合理的，毕竟 Dist 是一个三维上的距离，depth 本质上是一个一维上的距离。我后来自己想了个方法计算 min/maxTileDist，即归一化后的 tile 的 4 个点向量跟 float3(0, 0, -1) 做点乘计算 cos 值，得到 cos 值的最大最小值，再用 min/maxTileDepth 除以 min/maxCos 得到 min/maxTileDist：  

    ...
    float3 tileCornerVec0 = normalize(tileCorners[0]);
    float3 tileCornerVec1 = normalize(tileCorners[1]);
    float3 tileCornerVec2 = normalize(tileCorners[2]);
    float3 tileCornerVec3 = normalize(tileCorners[3]);

    float tileDepthMin = asfloat(tileMinDepthInt);
    float tileDepthMax = asfloat(tileMaxDepthInt);
    float cosCornerMin = min(min(min(-tileCornerVec0.z, -tileCornerVec1.z), -tileCornerVec2.z), -tileCornerVec3.z);
    float cosCornerMax = max(max(max(-tileCornerVec0.z, -tileCornerVec1.z), -tileCornerVec2.z), -tileCornerVec3.z);
    float tileDistMin = tileDepthMin / cosCornerMax;
    float tileDistMax = tileDepthMax / cosCornerMin;

最后我还发现距离上的比较已经暗含了 Cone Test，也就是说不用 `lightTileCos >= sumCos` 了，结果是一样的： 

    return lightTileDistBase - lightTileDistOffset < tileDistMax && lightTileDistBase + lightTileDistOffset > tileDistMin;

最后效果如下，可以看到结果已经接近完美了：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/13/XrUksP4HVgCMJ1E.jpg" width = "30%" height = "30%" alt="图18 - Spherical-Sliced Cone Test"/>
</div>

### AABB Test
还有一种方案就是 AABB 测试，用一个 AABB 包围盒包围住 tile frustum 跟灯光球体求交。这里有个问题就是 depth 的范围可能会很大，会影响到求交的准确度，这里可以把 AABB 包围盒的最大深度设置为 tile 深度最大值和 light 深度最大值的较小值，最小深度设置为 tile 深度最小值和 light 深度最小值的较大值，因为包围盒的深浅超过了灯光的深浅的部分对求交来说没有意义的：  

    float minZ = max(tileDepthMin, lightDepthMin);
    float maxZ = min(tileDepthMax, lightDepthMax);

然后就可以求 xy 轴上的最大最小值了，从而计算出整个 AABB 的 min/max：  

    float nearPlaneScale = minZ / _ProjectionParams.y;
    float farPlaneScale = maxZ / _ProjectionParams.y;

    float2 tileCorner0 = _CameraNearPlaneLB.xy + float2(tileIndex.x * _TileNearPlaneSize.x, tileIndex.y * _TileNearPlaneSize.y);
    float2 tileCorner2 = tileCorner0 + float2(_TileNearPlaneSize.x, _TileNearPlaneSize.y);
    
    float3 tileMin = float3(min(tileCorner0 * nearPlaneScale, tileCorner0 * farPlaneScale), minZ);
    float3 tileMax = float3(max(tileCorner2 * nearPlaneScale, tileCorner2 * farPlaneScale), maxZ);

求 minXY/maxXY 要注意一下，maxXY 不一定是在 tileCorner2 被远平面缩放后的位置，X 和 Y 的最大值有可能一个在近平面，一个在远平面，画一下 tile frustum 的远近平面在 XY 平面的 4 个象限的不同情况就可以理解了，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/14/xj5GTFIeqtziuC2.jpg" width = "45%" height = "45%" alt="图19 - 求 minXY/maxXY"/>
</div>

然后就是相交测试了，可以通过 clamp 来找到离灯光中心最近的 AABB 包围盒上的点：  

    float3 closestPoint = clamp(lightPositionVS, tileMin, tileMax);

然后该点和灯光中心的距离比灯光半径小即相交：  

    float3 diff = closestPoint - lightPositionVS;
    float sqrDiff = dot(diff, diff);
    float sqrLightRange = lightRange * lightRange;
    return sqrDiff <= sqrLightRange && lightDepthMin <= tileDepthMax && lightDepthMax >= tileDepthMin;

还是要同时做 depth test 的，不然会有一些全是 skybox 的 tile 会判定为相交。最后效果如下：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/14/MflzIYHCBgq2JFe.jpg" width = "30%" height = "30%" alt="图20 - AABB Test"/>
</div>

从上图的结果上来看，看起来跟 Spherical-Sliced Cone Test 的效果差不多，但我测试下来感觉 AABB 测试的 false positive 会更大一点，特别是当 tile 的深度范围比较大时。AABB 的 false positive 范围大致如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/14/SagyG4tLl8IAkKQ.png" width = "60%" height = "60%" alt="图21 - 左图： Sphere-Frustum Test 的 false positive；AABB Test 的 false positive"/>
</div>

但上述无论哪种方式，在深度不连续或过长的情况下的剔除效果都不佳，毕竟都依赖于最近最远深度，这就引出了 2.5D culling 的方法。但在之前，先解决一下 Spot Light 的剔除问题。

## Cull Spot Light

> 以下主要参考了这位大神的文章： https://bartwronski.com/2017/04/13/cull-that-cone/ 。

### Spot Light as Sphere
首先是一个最简单的方法，就是用一个能包裹 Spot Light 的最小球体来作为包围体，如果用 light range/radius 作为包围球体的半径，就跟点光源一样，很明显这样的球体是偏大的，不是最小的能包裹的球体，具体如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/17/ZlIDSRj7nWFpvxy.gif" width = "40%" height = "40%" alt="图22 - Spot Light Bounding Sphere"/>
</div>

很明显我们需要区分两种情况，一种大于 90 度，一种小于。上图中的三角形可能会让我们对 spot light 的实际影响范围产生一定的误解，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/17/pkL95nhwvMYoxgq.png" width = "60%" height = "60%" alt="图23 - Spot Light Bounding Sphere"/>
</div>

实际的 spot light 影响范围是 OAB 这个扇形而不是三角形，我们要求的有两个东西，一是 Bounding Sphere 的半径 r，二是 Bounding Sphere 的圆心 C 的位置。首先 spot light 的 outer angle 小于 90 度的情况，即 $\,\theta\,$ 小于 45 度的情况：  

$$ cos 2 \theta \cdot r + r = cos \theta \cdot range $$
$$ (2cos^2 \theta - 1)r + r = cos \theta \cdot range $$
$$ r = \cfrac {range} {2 cos \theta} $$

outer angle 大于 90 度就是 $\,r = sin \theta \cdot range\,$。圆心 C 可以通过 Spot Direction 和 Light Position 共同获取，很容易推导，就不多说了。因为跟 Point Light Sphere 一样，只需要传递圆心位置和半径，所以可以直接在 CPU 里计算好，代码如下：

``` C#
if (visibleLight.spotAngle >= 90.0f)
{
    data.lightsData.lightsCullingInputInfos[punctualLightCount] = position + cosOuterAngle * visibleLight.range * -spotDirection;
    data.lightsData.lightsCullingInputInfos[punctualLightCount].w = Mathf.Sin(Mathf.Deg2Rad * 0.5f * visibleLight.spotAngle) * visibleLight.range;
}
else
{
    float boundRadius = visibleLight.range / (cosOuterAngle * 2.0f);
    data.lightsData.lightsCullingInputInfos[punctualLightCount] = position + boundRadius * -spotDirection;
    data.lightsData.lightsCullingInputInfos[punctualLightCount].w = boundRadius;
}
```

这里注意一下，之前计算的 spotDirection 是指向灯光的方向，这里要反过来。然后 spotDirection 是通过灯光的 localToWorld 矩阵的第三列获取的，localToWorld 矩阵只有旋转和位移，故第三列的前三位一定是单位向量，所以无需归一化。实现效果就不展示了，可想而知剔除效果是有限的。

### Tile as Sphere vs Cone
这个方法是将 tile 视作 sphere，和 spot light 的锥形体进行求交。对于 Tile-based Light Culling 来说，由于 tile 的深度范围可能会很大，将 tile 视作 sphere 可能会产生过大的半径，所以相对于 Cluster-based Light Culling 会引入更多的 false positive，但是仍然相对于其他 spot light 剔除方法来说能做到更加精确的剔除。这个方法的思路如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/19/EZCtzH75LXa93YS.gif" width = "30%" height = "30%" alt="图24 - Tile as Sphere vs Cone"/>
</div>

可以将 cone 的朝向分为三种情况求交，一是头部最接近 tile sphere 时，二是尾部最接近时，三是侧边最接近时。头部和尾部的情况计算都比较简单，我们先看侧边，如下图：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/19/ApNtBSg2xiZjDVM.jpg" width = "40%" height = "40%" alt="图25 - Tile as Sphere vs Cone"/>
</div>

由图可知，tile sphere 与 cone 侧边最接近时，OD 距离要大于 sphere radius 才不相交。OA 向量 v 很好计算，AB 方向为 spot light 方向，v 与 spot light 方向点乘可得到 v1 的长度，由此可得 v2 长度。然后 $\, OC = v2 - v1 \cdot tan \theta \,$，再乘上 $\,cos \theta\,$ 得到 OD 距离。代码如下：  

    bool SpotConeIntersectTest(float3 tileMin, float3 tileMax, float3 lightPositionVS, float lightRange, float3 spotLightDir, float angle)
    {
        float3 tileCenter = 0.5 * (tileMin + tileMax);
        float3 extent = tileMax - tileCenter;
        float tileRadius = sqrt(dot(extent, extent));
        float3 V = tileCenter - lightPositionVS;
        float VLenSqr = dot(V, V);
        float V1Len = dot(V, spotLightDir);

        float distanceClosestPoint = cos(angle) * sqrt(VLenSqr - V1Len * V1Len) - V1Len * sin(angle);

        bool angleCull = distanceClosestPoint > tileRadius;
        bool frontCull = V1Len > tileRadius + lightRange;
        bool backCull = V1Len < -tileRadius;
        return !(angleCull || frontCull || backCull);
    }

注意一下 spotLightDir 要转换至观察空间，左手右手要和 tile、lightPosition 保持一致，且是远离光源的方向。V1Len 因为是点乘求出来的，所以有方向，我们可以根据方向来判断 tile sphere 跟 cone 的头部还是尾部更近。头部更近时的测试是上面的 backCull，尾部更近的测试是上面的 frontCull，应该很容易理解，就不多说了。我实际测试下来，这个测试最好能够结合普通的 AABB Test 以及 Depth Test，否则会引入较多的 false positive，结合的效果如下：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/19/sCPcDq3BFgGTX6V.jpg" width = "70%" height = "70%" alt="图26 - Spot Light Culling Result"/>
</div>

可以看到，除了深度范围较大的 tile，剔除的精度已经非常高了，下面开始解决深度问题。


# 2.5D Culling

> 这里出自 https://www.slideshare.net/slideshow/a-25d-culling-for-forward-siggraph-asia-2012/34909590 、https://pastebin.com/h7yiUYTD 、https://extremeistan.wordpress.com/2014/04/18/implementing-2-5d-culling-for-tiled-deferred-rendering-in-opencl/comment-page-1/ 、https://wickedengine.net/2018/01/optimizing-tile-based-light-culling/。

## 2.5D Culling 基本实现

2.5D Culling 的主要思路就是对深度进行分割，将 min/max depth 分割为 32 份，大致如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/20/QDM56o7OadGYhzx.jpg" width = "60%" height = "60%" alt="图27 - 2.5D Culling 示意图"/>
</div>

具体操作步骤有两步：  
①在 depth 上进行区域划分，然后根据 tile 内每个像素的 depth 来确定覆盖了哪些区域，并通过 depth mask 来编码：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/20/GfhTI1pDSQ2m3yt.jpg" width = "45%" height = "45%" alt="图28 - tile depth mask"/>
</div>

因为是一个 tile（线程组）拥有一个 depth mask，我们要为它定义一个 groupshared 变量，用于存储 tile 中每个像素所在的区间。depth mask 每位（bit）都代表一个区间，有像素落入这个区间，则该区间为 1。所以每个线程代表一个像素，通过深度贴图获取该像素的深度，从而计算出该像素所在 depth 区间的位置索引，根据位置索引将 1 进行左移得到该像素的 depth mask，同时每个像素（线程）原子操作按位或就可以获取到整个 tile 的 depth mask 了，代码大致如下：  

    groupshared uint tileDepthMask;

    [numthreads(THREAD_NUM_X, THREAD_NUM_Y, 1)]
    void TiledLightCulling(...)
    {
        // initialize shared memory
        if (groupIndex == 0)
        {
            #if _TILE_CULLING_SPLIT_DEPTH
            tileDepthMask = 0;
            #endif
        }
        GroupMemoryBarrierWithGroupSync();

        // get the minimum and maximum depth of the tile
        ...
        GroupMemoryBarrierWithGroupSync();

        // 2.5D Culling: Get the tile depth mask
        float tileDepthMin = asfloat(tileMinDepthInt);
        float tileDepthMax = asfloat(tileMaxDepthInt);

        #if _TILE_CULLING_SPLIT_DEPTH
        float invDepthRange = 32.0 / (tileDepthMax - tileDepthMin + 0.00001);

        if (inScreen && isNotSkybox)
        {
            uint depthSlot = floor((linearDepth - tileDepthMin) * invDepthRange);
            InterlockedOr(tileDepthMask, 1 << depthSlot);
        }
        
        GroupMemoryBarrierWithGroupSync();
        #endif

        ...
    }

②光源也同样可以根据光源范围来编码成 depth mask，灯光的 depth mask 和 tile 的 depth mask 按位与，即可获得相交测试结果：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/20/7ELgapVer5lcF8P.jpg" width = "45%" height = "45%" alt="图29 - light depth mask"/>
</div>

道理和上面是一样的，用光源的 min/max depth 得到光源的 depth mask，代码如下：  

    uint GetLightBitMask(float tileDepthMin, float lightDepthMin, float lightDepthMax, float invDepthRange)
    {
        uint depthSlotMin = floor((lightDepthMin - tileDepthMin) * invDepthRange);
        uint depthSlotMax = floor((lightDepthMax - tileDepthMin) * invDepthRange);
        depthSlotMin = clamp(depthSlotMin, 0, 31);
        depthSlotMax = clamp(depthSlotMax, 0, 31);

        uint lightMask = 0;
        for (uint i = depthSlotMin; i <= depthSlotMax; i++)
        {
            lightMask |= (1 << i);
        }
    }

上面代码（即提出者最原始的代码）使用了一个 for 循环去计算光源的 depth mask，但我在网上看到了一个更好的办法不用使用 for 循环，代码如下：  

    uint lightMask = 0xFFFFFFFF;
    lightMask >>= 31 - (depthSlotMax - depthSlotMin);
    lightMask <<= depthSlotMin;
    return lightMask;

假设一个 light 的 depth mask 从第 11 个 bit 开始，第 21 个 bit 结束，它的 depth mask 为 0000000000111111111110000000000。而 `uint lightMask = 0xFFFFFFFF` 为 1111111111111111111111111111111，首先右移 31 - (21 - 11)，得到 0000000000000000000011111111111。然后左移 11 位，得到 0000000000111111111110000000000 。

得到光源的 depth mask 后，直接和 tile depth mask 按位与，就可以得到测试结果：  

    uint lightMask = GetLightBitMask(tileDepthMin, lightDepthMin, lightDepthMax, invDepthRange);
    bool intersect2_5D = lightMask & tileDepthMask;

最终效果简单演示如下，图中有一个立方体和一堵墙，中间有两盏灯，都影响不到立方体和墙。关闭 2.5D culling 的情况下，立方体的边缘因为深度范围过大，导致 AABB 检测不准确。开启 2.5D culling 后则可以正确剔除这两盏灯光：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/20/BhvwSTVrqa7Uzux.png" width = "100%" height = "100%" alt="图30 - 中图：关闭 2.5D culling；右图：开启 2.5D culling"/>
</div>

但是上面还是只解决了 point light 的问题，因为我们计算灯光深度时，直接将灯光当作了球体计算 min/max depth，对于 spot light 来说，这个深度范围还是太广了。

## Spot Light Min/Max Depth
这里大致参考了 Unity URP 的 ZBinning Cluster Light Culling 里对 Spot Light 的 Min/Max Depth 的计算。

> Unity 的 ZBinning 参考的是 Call of Duty Infinite Warfare 的 ZBinning 算法：https://advances.realtimerendering.com/s2017/2017_Sig_Improved_Culling_final.pdf 。这个 ZBinning 算法虽然是 Cluster Based Light Culling 里使用的算法，但是大致逻辑和 2.5D Culling 几乎可以说是别无二致。所以 2.5D Culling 的实现和之前的 spot light 剔除可以说为之后实现 Cluster Based Light Culling 也打下了一定的基础。

