---
title: Local Reflection Probe
date: 2026-02-25 18:50:29
categories: 
  - [游戏开发, 图形]
  - [游戏开发, Unity]
tags:
  - 图形
  - 游戏开发
  - Unity
  - Pipeline
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/02/25/S9sk/LocalReflectionProbe.gif
mathjax: true
description: 本文章主要内容有基于反射探针的间接反射光照常见渲染策略，以及如何利用 Cubemap Blending, Parallax Correction, Cubemap Normalization 以及 Distance Based Roughness 来提高反射探针达成的渲染效果。
---

> 之前在跟随 Catlike Coding 的教程时（即 [Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/#Reflections)文章中），只实现了 Reflection Probe 最简单的功能，并且当时的理解非常肤浅，十分小瞧了该技术，实际上有很多 trick 能让反射探针的反射效果更好。后来在自定义管线实现 HBIL 和接入 APV 时，搜索了一些全局光照管线或间接光照管线的一些文章或 PPT，逐渐意识到了自己对 Reflection Probe 理解的不到位。刚好又碰巧发现了这篇博客文章：[Image-based Lighting approaches and parallax-corrected cubemap](https://seblagarde.wordpress.com/2012/09/29/image-based-lighting-approaches-and-parallax-corrected-cubemap/)（这篇博客文章是对其作者在 SIGGRAPH 2012 的[同名演讲](https://seblagarde.wordpress.com/wp-content/uploads/2012/08/parallax_corrected_cubemap-siggraph2012.pdf)的更新），我觉得这篇文章对部分反射探针相关技术的描述非常到位，然后国内互联网上跟反射探针相关的文章非常稀少，所以我觉得我有必要将上述博客文章的<u>重要内容</u>翻译下来，以作备忘，便于在自定义管线中添加反射探针更多的功能。同时，在翻译的过程中，我也会添加一些自己的理解或其他补充内容，毕竟这篇文章有点年份了。  

# IBL for Specular Lighting
Cubemap 可以被分为两种：  
**① Infinite Cubemap**：提供无限距离的环境光照，没有位置的概念，可以理解为全局的立方体贴图。在 Unity 中可以通过 `ReflectionProbe.defaultTexture` 获取到，也就是 Generate Lighting 后在场景文件夹里多出来的那个 Cubemap，之前在 [Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)中实现的 Indirect Specular Lighting，若场景中没有其他反射探针，物体默认采样到的就是这个全局 Cubemap。因为没有位置的概念，全局 Cubemap 通常使用反射方向 R 采样；  
**② Local Cubemap**：这类立方体贴图具有位置属性，代表有限距离内的环境光照，也就是 Unity 的 Reflection Probe 组件。从理论上来说，Local Cubemap 生成的光照，只有在生成的位置采样才拥有正确的效果，在其他位置上采样具有视差问题（使用视角反射方向 R 采样时）。这类立方体贴图通常使用在室内，需要使用 tricks 来弥补视差问题，也就是**视差校正 Parallax Correction**。在 Unity URP 中就可以通过开启 Box Projection 来激活视差校正。

常见的 Local Cubemap 应用策略有如下几种（作者的文章因为比较老，提到的策略都比较古早了，但了解历史沿革也有利于我们进一步理解该技术。现在常见的策略主要是 Forward+ 或 Deferred+ 所带来的 **Per-Pixel Reflection Probes**，我下面也会提及）：  

## Object Based Cubemap
每个物体都链接着一个或多个 Reflection Probe，使用离物体最近的一个或几个反射探针作为该物体的间接反射光来源。半条命2（Half Life 2）使用的就是这个策略，静态物体在离线时会被链接上几个最近的反射探针，而动态物体则在运行时做动态查询。[Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)中实现的其实就是 Object Based Cubemap 或者称为 **Per-Object Reflection Probes**，只不过当时没有实现反射探针的混合。我们可以在场景物体的 Mesh Renderer Component 中的 Probes 相关属性看到该物体链接着的数个反射探针，只要在 UnityInput.hlsl 中声明 `unity_SpecCube0` 和 `unity_SpecCube1` 分别对应物体链接着的反射探针，就可以在 Shader 中做混合了。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/08/6neW/LocalReflectionProbe01_PreObject.png" width = "40%" height = "40%" alt="图1 - Per-Object Reflection Probes（URP、HDRP 里看不到这个 UI）"/>
</div>

这套方案的缺点非常明显，比如临近的几个物体可能会采样到不同的反射探针，然后造成反射光照的接缝或者不自然的问题。墙壁地板等较大物体，可能会同时被多个不同的环境所影响，但是也只能采样到一个最近的反射探针。还有就是，动态物体会切换当前影响到它的反射探针，从而导致反射光照突变的现象，虽然该现象可以被 **Cubemap Blending** 减轻。另外，如果两个物体采样到了不同的反射探针，它们的合批也会被打断，影响渲染效率。

> 在 SRP 中可以通过 SupportedRenderingFeatures.rendererProbes 关闭 Mesh Renderer Component 中 Probes 的 UI 显示。如果不采用 Per-Object Reflection Probes 的方案，最好在 Cull 时通过 `cullingParameters.cullingOptions |= CullingOptions.DisablePerObjectCulling;` 来关闭 per-object culling for Lights and Reflection Probes，以防止浪费 CPU 资源。

## Zone/Camera Based Cubemap
作者文章中的 Zone Based Cubemap 和 Global and Local Cubemap No Overlapping 我觉得都属于该范畴。①Zone Based Cubemap 就是场景的每个区域 Zone 都分配一个 Infinite Cubemap，当相机进入一个区域，所有物体的反射光照都采样该区域的反射探针，从一个区域到另外一个区域的突变现象还是通过 Cubemap Blending 解决；②Global and Local Cubemap No Overlapping 则是所有的物体都默认使用同一个 Global Infinite Cubemap，同时又会摆放一些不重叠的 Local Cubemap，当相机进入 Local Cubemap，所有的物体就不再采样 Global Cubemap 而是采样 Local Cubemap。当相机从室内转移到室外，就可以通过 Sky Visibility 对室内的 Cubemap 和室外的 Cubemap 即 Global Cubemap 做混合；

作者在 SIGGRAPH 2012 的演讲中则提出了 **Point of Interest (POI) Based Cubemap**，本质也是 Zone/Camera Based Cubemap，作者的方法是提前将基于 Point of Interest 的多个最近的 Cubemap 提前使用一个 Pass 做混合得到一个 Cubemap，然后在着色时使用。只不过 POI 可以是基于 Camera，也可以是游戏主角，也可以是场景中的任意角色。虽然作者的方法比较老了，但是 POI 的思想还是可以借鉴的。

## Draw Cubemap Bounding Volume
作者文章中的 Global and Local Cubemap Overlapping 本质上就会绘制 Cubemap 体积，渲染方法类似于 Light Volume 或 Shadow Volume 技术，使用两个 pass 、模板测试和深度测试进行绘制，只能在延迟管线中使用，因为需要使用 G-Buffer 中的信息进行采样和间接反射光照的计算，最终通过 weighted additive blending 应用多个 Cubemap 进行混合。可以看出 Global and Local Cubemap Overlapping 允许 Local Cubemap 之间的相互重叠以及混合，并且可以让墙壁地板等较大物体的不同位置采样到不同的反射探针。

## Pixel Based Cubemap

> 这部分内容作者的文章中并没有，然后我在网上也没有搜索到专门讲 Per-Pixel Reflection Probes 的文章，所以下面的内容基本上都是我自己的理解。

现在基于反射探针的间接反射光照渲染更多是**逐像素 Per-Pixel** 实现的，就是对每个屏幕空间的像素寻找到最近的一个或数个 Cubemap 进行采样以及混合，这样就可以解决 Object Based Cubemap 的渲染缺陷，达到更精确的间接反射光照渲染效果。但是若遍历**视锥体剔除 Frustum Cull** 后所有的 Cubemap 相对来说可能会比较费时，当然能确保视锥体中始终只有 2-3 个反射探针，这么做也没啥毛病。为了减少遍历的性能消耗，方案就是对 Cubemap 进行剔除，跟 Light Culling 是一样的，所以在 Forward+ 或 Deferred+ 管线当中，Cubemap Culling 可以在 Tile-based Culling 或 Cluster-based Culling 的 Pass 中顺便完成。剔除完就可以在着色阶段基于每个像素对影响到它的反射探针进行混合。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/08/yv6Z/LocalReflectionProbe02_Reflectio.png" width = "70%" height = "70%" alt="图2 - Unity HDRP 的 Reflection Probe Culling（Tiled），可以看到两个 Reflection Probe 分别影响到了哪些 tile"/>
</div>

还有个问题就是我们需要同时采样多个 Cubemap，最好的方法肯定是使用 **Bindless Texture** API，但是 Unity 目前还没有支持该 API。Unity 的方案是将 Cubemap 使用八面体映射都渲染到一个 Texture Atlas 当中，然后在间接光计算时采样该 Atlas，当然如果可以确保屏幕中反射探针数量有限，也可以直接申明并绑定多个 TextureCube 进行采样。

# Cubemap Blending
正如之前所说，**Cubemap Blending** 是为了防止动态物体在反射探针之间移动时，造成的反射效果突然变化所引起的不自然感，故可以对影响到动态物体的多个 Cubemap 进行混合。至于如何混合，不同的需求完全可以有不同的方案，可以根据反射探针的距离、大小或者自定义的优先级 Priority 等因素来选择规定数目的反射探针（比如最多混合 4 个反射探针）进行混合，混合的系数可以由设定的混合距离、自定义的系数等决定。

在 Unity URP 的 Reflection Probe 中，我们可以看到 Importance、Blend Distance 和 Box Size 相关属性。跟上面说的类似，URP 就是根据 Box Size、Importance 属性来选择最多 2 个反射探针的，先根据 Importance 选择，若 Importance 一样则选择 Box Size 更小的，然后使用采样点距离 Box 边缘的距离和 Blend Distance 属性计算 Blend Weight 并混合，具体详见官方文档 [Reflection Probes in URP](https://docs.unity3d.com/6000.5/Documentation/Manual/urp/lighting/reflection-probes-introduction.html) 。具体代码我就不详细阐述了，可以参考 URP 或 HDRP，URP 混合的代码在其 package 的 ShaderLibrary -> GlobalIllumination.hlsl 里。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/08/Yq2k/LocalReflectionProbe03_CubemapBl.png" width = "40%" height = "40%" alt="图3 - Cubemap Blending 效果（After Parallax Correction）：球 3 和 球 1 都只受一个反射探针影响，分别是室内和室外的一个。球 2 同时受到室内和室外的反射探针的影响，并在反射探针的 Blend Distance 范围内，故呈现出了混合后的效果。可以看出，若物体从室内向室外移动，反射光照的变化会相对自然很多。"/>
</div>


# Parallax Correction
正如之前所说，Cubemap 本质上是一个 Infinite Box，若使用视角反射方向 R 作为采样方向，只有某个物体刚好和反射探针重合时，渲染出来的结果才是正确的，物体离 reflection probe 越远，产生的偏差会越大。所以当作为 Local Cubemap 使用时需要解决视差问题，**视差校正 Parallax Correction** 技术就是将 Local Cubemap 视作一个几何形状，称之为**反射代理体 Reflection Proxies**，这个代理体可以是 Sphere 或者 Box，我们不再使用视角反射方向 R 作为采样方向，而是在 Shader 中求出视角反射方向 R 与代理体的交点，将交点的方向作为 Cubemap 的采样方向，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/08/5Pfa/LocalReflectionProbe04_ParallaxC.png" width = "60%" height = "60%" alt="图4 - 图片来自于 Real-Time Rendering 第 11 章 Global Illumination。图中黑色圆圈代表要渲染的物体，点 P 就是着色点，蓝色的圈代表 Cubemap。常规用法下，我们会使用视角反射方向 r 去采样 Cubemap，如左图。而右图中，我们希望 Cubemap 来代表黑色的房间，可以使用红色方框来作为反射代理体，只要从 p 点出发，沿着 r 跟代理体求交，就可以计算出新的方向 r'，由此就可以获取到相对更加精确的反射信息。"/>
</div>

但是上图中代理体的假设，会在黑色房间下方的角出现误差，因为代理体不能完全匹配其代表的几何体。理论上代理体越匹配其代表的环境几何体，所得到的反射光照越精确。但是使用复杂的 Mesh 作为反射代理体，求交实在太费时了，所以一般会使用 Box 这种简单的几何体，求交比较容易，牺牲精确度的同时提高渲染效率。下面我们来讲解一下如何跟 Box 求交：  

## AABB 代理体
因为绝大多数情况下，我们的场景房间都是世界空间轴对齐的，所以可以假设反射代理体都是简单的 AABB 体积，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/08/fk6N/LocalReflectionProbe05_Intersect.png" width = "35%" height = "35%" alt="图5 - Intersection between vector R and the AABB volume"/>
</div>

图中的 Cubemap 是在点 C 生成的，在 Cubemap 周围生成了一个 Box 形状的反射代理体，即图中的黑色矩形。<u>一定要注意反射代理体 AABB 的中心不一定是 Cubemap 生成的位置</u>。在 Unity 中也是可以改变 Cubemap 捕获环境的位置的，通过 Box Offset 属性来改变。求交的逻辑就是根据着色点和视角反射方向 R 找到和 AABB 的交点 P，然后使用 C 到 P 的方向 R' 采样 Cubemap，作者文章中展示的代码如下：  

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

这部分代码类似于基于**平板法 Slab Method** 的 Ray 与 AABB 求交（关于平板法的详细介绍可以查看这篇文章：[Fast, Branchless Ray/Bounding Box Intersections](https://tavianator.com/2011/ray_box.html) ）。这个方法的思想就是将 AABB 视作三对独立的平行平面，比如 x 方向的右平面就是 BoxMax.x，代码中的 `FirstPlaneIntersect` 和 `SecondPlaneIntersect` 就是在计算 6 个平面对应下述射线公式的参数 t：  

$$ x(t) = OriginPos.x + t \cdot Dir.x $$

若 x(t) 为 BoxMax.x，则 t 等于：  

$$ t = \cfrac {BoxMax.x - OriginPos.x} {dir.x} $$

这个 t 可以理解为射线原点跟每个平面的交点的距离与方向各分量的倍数。`max(FirstPlaneIntersect, SecondPlaneIntersect);` 就是选择与射线同方向的交点的 3 个 t，其中最小的 t 就是与 AABB 的交点，其它的 t 一定是在 AABB 外面相交的。得到 t 就可以利用它算出交点的准确坐标，计算方式和上面公式一样。<u>最后注意新的反射方向 R' 是 Cubemap 生成点 C 到交点 P 的方向</u>，生成点 C 即代码中的 `CubemapPositionWS`。

## OBB 代理体
但是有时候场景房间不一定是世界空间轴对齐的，比如室内的斜坡，抑或是楼梯等等。此时就需要将反射代理体旋转一定的角度，而 AABB 旋转后就是 OBB。作者文章中 OBB 求交的代码如下：  

    float3 DirectionWS = normalize(PositionWS - CameraWS);
    float3 ReflDirectionWS = reflect(DirectionWS, NormalWS);

    // Intersection with OBB convertto unit box space
    // Transform in local unit parallax cube space (scaled and rotated)
    float3 RayLS = MulMatrix( float(3x3)WorldToLocal, ReflDirectionWS);
    float3 PositionLS = MulMatrix( WorldToLocal, PositionWS);

    float3 Unitary = float3(1.0f, 1.0f, 1.0f);
    float3 FirstPlaneIntersect  = (Unitary - PositionLS) / RayLS;
    float3 SecondPlaneIntersect = (-Unitary - PositionLS) / RayLS;
    float3 FurthestPlane = max(FirstPlaneIntersect, SecondPlaneIntersect);
    float Distance = min(FurthestPlane.x, min(FurthestPlane.y, FurthestPlane.z));

    // Use Distance in WS directly to recover intersection
    float3 IntersectPositionWS = PositionWS + ReflDirectionWS * Distance;
    float3 ReflDirectionWS = IntersectPositionWS - CubemapPositionWS;

    return texCUBE(envMap, ReflDirectionWS);

但是作者的代码实际上有点小问题，整体思路是没问题的，就是将 OBB 转换为局部空间的单位 Box，此时 OBB 就退化成了 AABB，便于求交。将着色点和视角反射方向 R 都转换到 AABB 的局部空间内，然后计算出交点的 t，而这个 t 可以直接用于求出世界空间的交点，之所以可以这么做，是因为 t 本质上是倍数，视角反射方向 R 在局部空间被缩放了导致倍数不会发生变化。

然而，这样计算出来的 IntersectPositionWS 并不是反射代理体旋转后与视角反射方向 R 的交点，而是反射代理体旋转前与视角反射方向 R 的交点。我更改后的代码大致如下：  

    // float3 IntersectPositionWS = PositionWS + ReflDirectionWS * Distance;
    // float3 ReflDirectionWS = IntersectPositionWS - CubemapPositionWS;

    float3 IntersectPositionLS = positionLS + rayLS * Distance;
    float3 IntersectPositionWS = IntersectPositionLS * BoxExtent + BoxCenter;
    float3 RealReflDirectionWS = normalize(IntersectPositionWS - CubemapPositionWS);

代码看起来很奇怪，但是我实现后测试下来是没问题的。首先，IntersectPositionLS 很简单就是求转换到局部空间后的视角反射方向 R 在局部空间的单位 Box 内的交点。若此时将 IntersectPositionLS 左乘 LocalToWorld 矩阵，得到的还是反射代理体旋转前与视角反射方向 R 的交点，因为 WorldToLocal 与 LocalToWorld 抵消了。所以我们不能考虑矩阵的旋转信息，只考虑缩放和移动，即 TRS 矩阵当中的 TS。因为缩放是不会影响移动的，所以 `IntersectPositionWS = IntersectPositionLS * BoxExtent + BoxCenter`，BoxExtent 就是缩放的三个分量，BoxCenter 就是移动的距离。

额外提一嘴，计算 WorldToLocal 矩阵时，Scale 的信息并不是反射探针本身的 Scale，Position 的信息也不一定是反射探针本身的 position！！！Scale 应该是 `BoxSize * 0.5f`（即 BoxExtent），之所以乘 0.5，是因为代码中使用的局部空间的单位 Box 为 `float3(1.0f, 1.0f, 1.0f)`，本质上边长为 2，所以需要将反射代理体边长 BoxSize 除以 2，即 BoxExtent 的值。Position 应该是 BoxCenter，有时候反射探针的 position 是 CubemapPositionWS，别搞错了，这取决于引擎如何定义反射探针的 position，可以定义为捕获 Cubemap 的位置（Unity URP 的做法），也可以定义为 BoxCenter（Unity HDRP 的做法）。

Unity 对于 Reflection Probe 旋转的处理跟作者文章中的方法不同，直接向 GPU 传递了 Reflection Probe 旋转的四元数，将着色点和视角反射方向先旋转回 AABB，计算求交后，将新的采样方向再旋转到 OBB。具体代码我就不摘抄了，还是在 URP package 的 ShaderLibrary -> GlobalIllumination.hlsl 里，四元数的上传在 RunTime -> ReflectionProbeManager.cs 里。


# Cubemap Normalization
这部分内容在作者文章的 Reusing Available Cubemap 部分，**Cubemap Normalization** 技术可以让 Cubemap 更加匹配物体所处的光照环境（间接光环境），主要是为了处理物体反射间接光照的遮蔽问题，本质上属于 **Specular Occlusion (SO)** 的范畴，比如一个室内中心有一个反射探针，那沙发底下的地板会错误采样到反射探针的较亮区域，导致沙发底下的地板具有较亮的反射光，显得非常不真实。

实现 Specular Occlusion 的另外一种方式，之前在[屏幕空间环境光遮蔽（一）SSAO](https://ybniaobu.github.io/2025/08/01/2025-08-01-SSAO1/#Specular-Occlusion) 和 [Custom Better PBR in Unity](https://ybniaobu.github.io/2024/10/22/2024-10-22-BetterPBR1/#Specular-occlusion) 都有提到过。Frostbite 是通过利用视线方向、Ambient Occlusion 和 Roughness 拟合了一个 Specular Occlusion，对于完全粗糙表面，SO = AO，对于光滑表面，越接近 Grazing Angle，AO 的影响就越大。亦或者采用 GTSO 的方式利用 GTAO 顺便计算出来的 Bent Normal 和观察方向的夹角估计出 SO。但毕竟上述方法都是屏幕空间方法，提供的遮蔽效果范围有限，无法解决一些比较大的遮蔽问题，特别是离开屏幕的物体导致的遮蔽。

而 **Cubemap Normalization** 可以解决屏幕空间遮蔽不能解决的问题，可以将其定义为将环境贴图除以某个值归一化后，再乘上 diffuse 环境光，以此来匹配当前的环境光照。diffuse 环境光可以来自 SH（Light Probe）、Light Map 亦或是实时计算出来的漫反射全局光照。Cubemap Normalization 具体方法大致有如下：  

①Multiply By Ambient Lighting Ratio：在烘焙时，计算出 Cubemap 的 SH Irradiance，然后在运行的时候，用反射方向采样，得到 Cubemap SH。着色点计算反射间接光照时，将采样 Cubemap 得到的反射光照和 SH （或者 SH 的 Luma）相除得到 Normalize 后的 Cubemap，再乘上当前着色点的漫反射 Irradiance（或者 Irradiance 的 Luma），由此来修改反射光。当然，在实践中，完全可以在烘焙时，就将 Cubemap 除以 SH，这样在运行时直接乘上漫反射 GI 就行。这个方法可以在这几篇 PPT 都有说明：[Rendering of Call of Duty: Infinite Warfare](https://www.activision.com/cdn/research/2017_DD_Rendering_of_COD_IW.pdf)，[Getting More Physical in Call of Duty: Black Ops II](https://blog.selfshadow.com/publications/s2013-shading-course/lazarov/s2013_pbs_black_ops_2_slides_v2.pdf) 以及 [The Indirect Lighting Pipeline of God of War](https://media.gdcvault.com/gdc2019/presentations/Hobson_Josh_The_Indirect_Lighting.pdf) 。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/09/Iv0t/LocalReflectionProbe06_CubemapNo.png" width = "60%" height = "60%" alt="图6 - Cubemap Normalization"/>
</div>

②Color Correct The Cubemap：比如给 Cubemap 去饱和 Desaturation，将 Cubemap 转变为灰色 luminance，然后使用 Diffuse 环境光的颜色重新上色，其实这个方法跟上面很类似，但好处就是可以只使用一个通道来存储 Cubemap。

> 这后面的所有内容都是额外补充的，不在开头提到的博客文章里。 

# Distance Based Roughness
镜面反射 BRDF lobe 的**覆盖面积 Footprint** 是受到距离的影响的。给定一个着色点，如果着色点距离镜面反射所呈现的物体越近，镜面反射越清晰，反之越模糊，如下图所示：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/09/nQ4b/LocalReflectionProbe07_DistanceB.png" width = "50%" height = "50%" alt="图7 - Example of distance-based roughness with a textured wall and a glossy metallic plane. With distance, reflections get blurrier as the BRDF footprint gets wider."/>
</div>

上图来源于 [Moving Frostbite to Physically Based Rendering 3.0](https://media.contentapi.ea.com/content/dam/eacom/frostbite/files/course-notes-moving-frostbite-to-pbr-v32.pdf) ，Frostbite 的做法就是使用着色点与 Cubemap 代理体反射交点的距离来大致估计 BRDF footprint，通过改变 roughness 来近似匹配 footprint，称之为 **Distance Based Roughness**，这点其实《Real-Time Rendering, Fourth Edition》的 11.6 章节中也有提到。Frostbite 还给出了代码，如下：  

    float ComputeDistanceBaseRoughness(float distInteresectionToShadedPoint, float distInteresectionToProbeCenter, float linearRoughness)
    {
        // To avoid artifacts we clamp to the original linearRoughness
        // which introduces an acceptable bias and allows conservation
        // of mirror reflection behavior for a smooth surface .
        float newLinearRoughness = clamp(distInteresectionToShadedPoint / distInteresectionToProbeCenter * linearRoughness, 0, linearRoughness);
        return lerp(newLinearRoughness, linearRoughness, linearRoughness);
    }

Frostbite 还提到了这么做的另外一个好处，就是 Screen-Space Reflection (SSR) 需要 fallback 到反射探针，考虑 Distance Based Roughness 可以提高 SSR 和反射探针过渡衔接的渲染效果。


# 其他补充
## Relightable Cubemap
有时候当场景环境发生动态变化时，特别是拥有动态光源时，我们会希望 Reflection Probe 能够跟着周围环境的变化而实时变化。但是因为 Cubemap 有六个面，每帧实时更新的渲染压力太大了，除了渲染六个面，我们可能还需要实时预过滤 Cubemap 来生成 mipmap。一种优化方案是每帧只更新 Cubemap 的一个面来减少开销，Unity 反射探针的 Realtime 模式就有这个功能。还有一种优化方案就是提前烘焙好 Cubemap 的 GBuffer 存储起来，在需要更新 Cubemap 时，加载 GBuffer 和当前的光照信息进行计算，这个方法我就不详细讲了，具体方案可以查看这几篇文章：[Graphics Study: Red Dead Redemption 2](https://imgeself.github.io/posts/2020-06-19-graphics-study-rdr2/)、[Real-Time Samurai Cinema](https://advances.realtimerendering.com/s2021/jpatry_advances2021.pdf)、[Rendering of Call of Duty: Infinite Warfare](https://www.activision.com/cdn/research/2017_DD_Rendering_of_COD_IW.pdf)。快速实时 GGX 预过滤 Cubemap 可以查看这篇文章：[Fast Filtering of Reflection Probes](https://www.ppsloan.org/publications/ggx_filtering.pdf) 。

## Reflection Probe 放置建议
Cubemap 可以被分为 Global Cubemap、Local Cubemap，那么我们放置反射探针（Local Cubemap）时可以参照同样的思路，将反射探针也分为不同的层级，层级越大反射探针的范围越大，层级越小反射探针越放置在需要细节的地方，如下图：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/03/09/7jhU/LocalReflectionProbe08_CubemapPl.png" width = "30%" height = "30%" alt="图8 - Reflection Probe Placement"/>
</div>

## 其他反射技术
虽然 Reflection Probe 能够提供可以接受的反射光照信息，但是若需要更加精确的反射效果，就需要下述开销更大的技术，这些技术有（这里就简单介绍一下）：  
**①屏幕空间反射 Screen Space Reflection (SSR)**：其原理就是在屏幕空间沿着视角反射方向进行光线步进，利用 Depth 进行求交采样颜色贴图作为反射颜色。详细思路可以参考这几篇文章：[Efficient GPU Screen-Space Ray Tracing](https://jcgt.org/published/0003/04/04/paper.pdf)、[Stochastic Screen-Space Reflections](https://advances.realtimerendering.com/s2015/index.html) 。  
**②平面反射 Planar Reflection**：该技术适用于平面的物体，比如水面、镜面、玻璃等等，就是将摄像机沿着镜面对称，将场景重新渲染一遍，是个极高性能开销的技术。  
**③屏幕空间平面反射 Screen Space Plane Reflection (SSPR)**：SSR 的简化限制版，也只适用于平面的物体，但性能非常优秀，适用于手机也想要平面反射效果，算法详见 [Optimized pixel-projected reflections for planar reflectors](https://advances.realtimerendering.com/s2017/PixelProjectedReflectionsAC_v_1.92.pdf) 。  
当然最准确最完美的方法一定是 **Ray-Traced Reflection**。