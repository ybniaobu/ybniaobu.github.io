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
Tile-based 的光源分类使用的是一个基于 tile 的二维空间范围，以及几何物体的深度边界；而 Cluster-based 则将视锥体划分为一组三维单元格，称为**聚类 cluster**，聚类的划分是在整个视锥体上进行的，独立于场景中的几何形状，如下图所示：  

<div align="center">  
<img src="https://s2.loli.net/2025/06/09/eJoPiS7p2bhuWKr.png" width = "60%" height = "60%" alt="图5 - Cluster-based Light Culling"/>
</div>

光源会基于 cluster 来进行分类，并形成对应的光源列表，由于 cluster 并不依赖场景几何的 z-depth，故无论是透明的还是不透明的物体，都可以使用该物体的位置来检索相关的光源列表。本篇文章就不详细介绍该技术了，以后实现的时候再行阐述。


# Compute Shader 同步基础
## groupshared
## 原子操作
## Barrier
## Wave Operation

# Tile-Based Light Culling
## 球形相交测试
### Sphere-Frustum Test
### Cone Test
### Spherical-sliced Cone Test
## Spot Light 相交测试