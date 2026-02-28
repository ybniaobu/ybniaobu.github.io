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
description: 本文章主要内容有XXXXXXXXXXXXXXXXXXXXXXX。
---

> 之前在跟随 Catlike Coding 的教程时（即 [Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)文章中），只实现了 Reflection Probe 最简单的功能，并且当时的理解非常肤浅，十分小瞧了该技术，实际上有很多 trick 能让反射探针的反射效果更好。后来在自定义管线实现 HBIL 和接入 APV 时，搜索了一些全局光照管线或间接光照管线的一些文章或 PPT，逐渐意识到了自己对 Reflection Probe 理解的不到位。刚好又碰巧发现了这篇博客文章：[Image-based Lighting approaches and parallax-corrected cubemap](https://seblagarde.wordpress.com/2012/09/29/image-based-lighting-approaches-and-parallax-corrected-cubemap/)（这篇博客文章是对其作者在 SIGGRAPH 2012 的[同名演讲](https://seblagarde.wordpress.com/wp-content/uploads/2012/08/parallax_corrected_cubemap-siggraph2012.pdf)的更新），我觉得这篇文章对反射探针相关技术的总结非常详细，然后国内互联网上跟反射探针相关的文章非常稀少，所以我觉得我有必要将上述博客文章的<u>重要内容</u>翻译下来，以作备忘，便于在自定义管线中添加反射探针更多的功能。同时，在翻译的过程中，我也会添加一些自己的理解或其他补充内容。

# IBL for specular lighting
Cubemap 可以被分为两种：  
**① Infinite Cubemap**：提供无限距离的环境光照，没有位置的概念，可以理解为全局的立方体贴图。在 Unity 中可以通过 `ReflectionProbe.defaultTexture` 获取到，也就是 Generate Lighting 后在场景文件夹里多出来的那个 Cubemap，之前在 [Unity Custom SRP 基础（三）](https://ybniaobu.github.io/2025/01/07/2025-01-07-CustomSRP3/)中实现的 Indirect Specular Lighting，若场景中没有其他反射探针，物体默认采样到的就是这个全局 Cubemap。因为没有位置的概念，全局 Cubemap 通常使用反射方向 R 采样；  
**② Local Cubemap**：这类立方体贴图具有位置属性，代表有限距离内的环境光照，也就是 Unity 的 Reflection Probe 组件。从理论上来说，Local Cubemap 生成的光照，只有在生成的位置采样才拥有正确的效果，在其他位置上采样具有视差问题（使用反射方向 R 采样时）。这类立方体贴图通常使用在室内，需要使用 tricks 来弥补视差问题，也就是**视差校正 Parallax Correction**。在 Unity URP 中可以通过开启 Box Projection 来激活视差校正。

And as we often need to blend multiple cubemap,  I will define different cubemap blending method :
– Sampling K cubemaps in the main shader and do a weighted sum. Expensive.
– Blending cubemap on the CPU and use the resulted cubemap in the shader. Expensive depends on the resolution and required double buffering resources to avoid GPU stall.
– Blending cubemap on the GPU and use the resulted cubemap in the shader. Fast.
– Only with a deferred or light-prepass engine: Apply K cubemaps by weighted additive blending. Each cubemap bounding volume is rendered to the screen and normal+roughness from G-Buffer is used to sample the cubemap.

## Object Based Cubemap

