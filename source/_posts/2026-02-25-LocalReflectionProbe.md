---
title: Local Reflection Probe
date: 2026-02-25 18:50:29
categories: 
  - [图形学]
  - [unity, pipeline]
tags:
  - 图形学
  - 游戏开发
  - unity
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/02/25/S9sk/LocalReflectionProbe.gif
mathjax: true
description: 本文章主要内容有XXXXXXXXXXXXXXXXXXXXXXX。
---

> 之前在跟随 Catlike Coding 的教程时（即 [Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)文章中），只实现了 Reflection Probe 最简单的功能，并且当时的理解非常肤浅，十分小瞧了该技术，实际上有很多 trick 能让反射探针的反射效果更好。后来在自定义管线实现 HBIL 和接入 APV 时，搜索了一些全局光照管线或间接光照管线的一些文章或 PPT，逐渐意识到了自己对 Reflection Probe 理解的不到位。刚好又碰巧发现了这篇博客文章：[Image-based Lighting approaches and parallax-corrected cubemap](https://seblagarde.wordpress.com/2012/09/29/image-based-lighting-approaches-and-parallax-corrected-cubemap/)（这篇博客文章是对其作者在 SIGGRAPH 2012 的[同名演讲](https://seblagarde.wordpress.com/wp-content/uploads/2012/08/parallax_corrected_cubemap-siggraph2012.pdf)的更新），我觉得这篇文章对反射探针相关技术的总结非常详细，然后国内互联网上跟反射探针相关的文章非常稀少，所以我觉得我有必要将上述博客文章的<u>重要内容</u>翻译下来，以作备忘，便于在自定义管线中添加反射探针更多的功能。同时，在翻译的过程中，我也会添加一些自己的理解或其他补充内容。  

# IBL for Specular Lighting
Cubemap 可以被分为两种：  
**① Infinite Cubemap**：提供无限距离的环境光照，没有位置的概念，可以理解为全局的立方体贴图。在 Unity 中可以通过 `ReflectionProbe.defaultTexture` 获取到，也就是 Generate Lighting 后在场景文件夹里多出来的那个 Cubemap，之前在 [Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)中实现的 Indirect Specular Lighting，若场景中没有其他反射探针，物体默认采样到的就是这个全局 Cubemap。因为没有位置的概念，全局 Cubemap 通常使用反射方向 R 采样；  
**② Local Cubemap**：这类立方体贴图具有位置属性，代表有限距离内的环境光照，也就是 Unity 的 Reflection Probe 组件。从理论上来说，Local Cubemap 生成的光照，只有在生成的位置采样才拥有正确的效果，在其他位置上采样具有视差问题（使用视角反射方向 R 采样时）。这类立方体贴图通常使用在室内，需要使用 tricks 来弥补视差问题，也就是**视差校正 Parallax Correction**。在 Unity URP 中就可以通过开启 Box Projection 来激活视差校正。

常见的 Local Cubemap 应用策略有如下几种（文章因为比较老，文章中提到的策略都比较古早了，但了解历史沿革也有利于我们进一步理解该技术。现在常见的策略主要是 Forward+ 或 Deferred+ 所带来的 **Per-Pixel Reflection Probes**，我下面也会提及）：  

## Object Based Cubemap
每个物体都链接着一个或多个 Reflection Probe，使用离物体最近的一个或几个反射探针作为该物体的间接反射光来源。半条命2（Half Life 2）使用的就是这个策略，静态物体在离线时会被链接上几个最近的反射探针，而动态物体则在运行时做动态查询。[Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)中实现的其实就是 Object Based Cubemap 或者可以称为 **Per-Object Reflection Probes**，我们可以在场景物体的 Mesh Renderer Component 中的 Probes 相关属性看到该物体链接着的一个或两个反射探针，只要在 UnityInput.hlsl 中声明 `unity_SpecCube0` 和 `unity_SpecCube1` 分别对应物体链接着的反射探针，就可以在 Shader 中做混合。

这套方案的缺点非常明显，就是临近的几个物体可能会采样到不同的反射探针，然后造成反射光照的接缝或者不自然的问题。同时，墙壁地板等较大物体，可能会同时被多个不同的环境所影响，但是也只能采样到一个最近的反射探针。还有就是，动态物体会切换当前影响到它的反射探针，从而导致反射光照突变的现象，虽然该现象可以被 **Cubemap Blending** 减轻。

## Zone/Camera Based Cubemap
作者文章中的 Zone Based Cubemap 和 Global and Local Cubemap No Overlapping 我觉得都属于该范畴。①Zone Based Cubemap 就是场景的每个区域 Zone 都分配一个 Infinite Cubemap，当相机进入一个区域，所有物体的反射光照都采样该区域的反射探针，从一个区域到另外一个区域的突变现象还是通过 Cubemap Blending 解决；②Global and Local Cubemap No Overlapping 则是所有的物体都默认使用同一个 Global Infinite Cubemap，同时又会摆放一些不重叠的 Local Cubemap，当相机进入 Local Cubemap，所有的物体就不再采样 Global Cubemap 而是采样 Local Cubemap。当相机从室内转移到室外，就可以通过 Sky Visibility 对室内的 Cubemap 和室外的 Cubemap 即 Global Cubemap 做混合；

作者在 SIGGRAPH 2012 的演讲中则提出了 **Point of Interest (POI) Based Cubemap**，本质也是 Zone/Camera Based Cubemap，作者的方法是提前将基于 Point of Interest 的多个最近的 Cubemap 提前使用一个 Pass 做混合得到一个 Cubemap，然后在着色时使用。只不过 POI 可以是基于 Camera，也可以是游戏主角，也可以是场景中的任意角色。虽然作者的方法比较老了，但是 POI 的思想还是可以借鉴的。

## Draw Cubemap Bounding Volume
作者文章中的 Global and Local Cubemap Overlapping 本质上就会绘制 Cubemap 体积，渲染方法类似于 Light Volume 或 Shadow Volume 技术，使用两个 pass 、模板测试和深度测试进行绘制，只能在延迟管线中使用，因为需要使用 G-Buffer 中的信息进行采样和间接反射光照的计算，最终通过 weighted additive blending 应用多个 Cubemap 进行混合。可以看出 Global and Local Cubemap Overlapping 允许 Local Cubemap 之间的相互重叠以及混合，并且可以让墙壁地板等较大物体的不同位置采样到不同的反射探针。

## Pixel Based Cubemap

> 这部分内容作者的文章中并没有，然后我在网上也没有搜索到专门讲 Per-Pixel Reflection Probes 的文章，所以下面的内容基本上都是我自己的理解。

现在基于反射探针的间接反射光照渲染更多是**逐像素 Per-Pixel** 实现的，就是对每个屏幕空间的像素寻找到最近的一个或数个 Cubemap 进行采样以及混合。但是如果遍历**视锥体剔除 Frustum Cull** 后的所有 Cubemap 相对来说可能会比较费时，当然能确保视锥体中始终只有 2-3 个反射探针，这么做也没啥毛病。为了减少遍历的性能消耗，方案就是对 Cubemap 进行剔除，跟 Light Culling 是一样的，所以在 Forward+ 或 Deferred+ 管线当中，Cubemap Culling 可以在 Tile-based Culling 或 Cluster-based Culling 的 Pass 中顺便完成。剔除完就可以在着色阶段基于每个像素对影响到它的反射探针进行混合。

还有个问题就是我们需要同时采样多个 Cubemap，最好的方法肯定是使用 Bindless Texture API，但是 Unity 目前还没有支持该 API。Unity 的方案是将 Cubemap 都渲染到一个 Texture Atlas 当中，然后在间接光计算时采样该 Atlas，当然如果可以确保屏幕中反射探针数量有限，也可以直接申明并绑定多个 TextureCube 进行采样。**Cubemap Blending** 的相关代码我这里就不详细展开了，不同的需求完全可以有不同的方案，Unity 的方案简而言之就是基于反射探针的 Box Size 以及 Priority（Importance）属性来选择规定数目的反射探针（比如最多混合 4 个反射探针），使用 Blend Distance 计算 Blend Weight 并混合。可以去参考 Unity URP 的代码，代码在 ShaderLibrary -> GlobalIllumination.hlsl 里。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/07/0Sij/LocalReflectionProbe01_CubemapBl.png" width = "40%" height = "40%" alt="图1 - Cubemap Blending 效果：球 3 和 球 1 都只受一个反射探针影响，分别是室内和室外的一个。球 2 同时受到室内和室外的反射探针的影响，并在反射探针的 Blend Distance 范围内。可以看出，若物体从室内向室外移动，反射光照的变化会相对自然很多。"/>
</div>

# Parallax Correction
正如之前所说，Cubemap 本质上是一个 Infinite Box，当作为 Local Cubemap 使用时需要解决视差问题。**视差校正 Parallax Correction** 技术就是将 Local Cubemap 视作一个几何形状，称之为**反射代理体 Reflection Proxies**，这个代理体可以是 Sphere 或者 Box，然后在 Shader 中求视角反射方向 R 与代理体的交点，将交点的方向作为 Cubemap 的采样方向，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/07/iFy5/LocalReflectionProbe02_ParallaxC.png" width = "60%" height = "60%" alt="图2 - 图片来自于 Real-Time Rendering 第 11 章 Global Illumination。图中黑色圆圈代表要渲染的物体，点 P 就是着色点，蓝色的圈代表 Cubemap。常规用法下，我们会使用视角反射方向 r 去采样 Cubemap，如左图。而右图中，我们希望 Cubemap 来代表黑色的房间，可以使用红色方框来作为反射代理体，只要从 p 点出发，沿着 r 跟代理体求交，就可以计算出新的方向 r'，由此就可以获取到相对更加精确的反射信息。"/>
</div>

但是上图中代理体的假设，会在黑色房间下方的角出现误差，因为代理体不能完全匹配其代表的几何体。理论上代理体越匹配其代表的环境几何体，所得到的反射光照越精确。但是使用复杂的 Mesh 作为反射代理体，求交实在太费时了，所以一般会使用 Box 这种简单的几何体，求交比较容易，牺牲精确度的同时提高渲染效率。

## AABB 代理体
因为绝大多数情况下，我们的场景房间都是世界空间轴对齐的，所以可以假设反射代理体都是简单的 AABB 体积，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/07/a4kS/LocalReflectionProbe03_Intersect.png" width = "35%" height = "35%" alt="图3 - Intersection between vector R and the AABB volume"/>
</div>

注意 AABB 中心不一定是 Cubemap 的中心，可以改变 Cubemap 捕获环境的位置 【书签】

求交的代码大致如下：  

    float3 DirectionWS = PositionWS - CameraWS;
    float3 ReflDirectionWS = reflect(DirectionWS, NormalWS);

    // Following is the parallax-correction code
    // Find the ray intersection with box plane
    float3 FirstPlaneIntersect = (BoxMax - PositionWS) / ReflDirectionWS;
    float3 SecondPlaneIntersect = (BoxMin - PositionWS) / ReflDirectionWS;
    // Get the furthest of these intersections along the ray
    // (Ok because x/0 give +inf and -x/0 give –inf )
    float3 FurthestPlane = max(FirstPlaneIntersect, SecondPlaneIntersect);
    // Find the closest far intersection
    float Distance = min(min(FurthestPlane.x, FurthestPlane.y), FurthestPlane.z);

    // Get the intersection position
    float3 IntersectPositionWS = PositionWS + ReflDirectionWS * Distance;
    // Get corrected reflection
    ReflDirectionWS = IntersectPositionWS - CubemapPositionWS;
    // End parallax-correction code

    return texCUBE(envMap, ReflDirectionWS);

这部分代码类似于基于**平板法 Slab Method** 的 Ray 与 AABB 求交（关于平板法的详细介绍可以查看这篇文章：[Fast, Branchless Ray/Bounding Box Intersections](https://tavianator.com/2011/ray_box.html) ）。这个方法的思想就是将 AABB 视作三对独立的平行平面

## OBB 代理体

看一下 URP 反射探针旋转的代码

# Cubemap Normalization
这部分内容在作者文章的 Reusing Available Cubemap 部分

# Distance Based Roughness


# 其他补充
简单补充一下与 Screen-Space Reflections 的结合，以及 Planar Reflection 、 SSPR、Ray-traced Reflection