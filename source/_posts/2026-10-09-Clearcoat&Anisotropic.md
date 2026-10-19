---
title: Clear coat & Anisotropic 材质
date: 2026-10-09 18:59:14
categories: 
  - [游戏开发, 图形]
  - [游戏开发, Unity]
tags:
  - 图形
  - 游戏开发
  - Unity
  - PBR
  - Shader
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/09/04/1Kto/Clearcoat-Anisotropic.gif
mathjax: true
description: XXXXXXXXXXXXXXXXXXXXXXXXXX
---

> 之前在 [Custom Better PBR in Unity](https://ybniaobu.github.io/2024/10/22/2024-10-22-BetterPBR1/) 中实现了 BRDF 的标准/基础材质模型，主要用于模拟**各向同性 Isotropic** 的**电介质 Dielectric**（非金属 non-metallic）或**导体 Conductor**（金属 metallic）材质表面。本篇文章在此基础上继续补充**清漆/透明涂层 Clear coat** 和**各向异性 Anisotropic** 材质。
>
> 本文主要参考了 Filament 的 [Physically Based Rendering](https://google.github.io/filament/Filament.md.html)，以及 Khronos 的 glTF 2.0 材质扩展：[KHR_materials_clearcoat](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_clearcoat) 与 [KHR_materials_anisotropy](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_anisotropy)。

# Clear coat

标准材质模型适合描述由单一层构成的各向同性表面，但**多层材质 Multi-layer Material** 其实相当常见，尤其是在标准层之上再覆盖一层薄薄半透明层的材质，在现实中的例子有诸如车漆、易拉罐、漆面木材、亚克力等。这种清漆层可以作为标准材质模型的一种扩展来模拟，即在原模型上再叠加一个高光波瓣，同样使用 Cook-Torrance 微表面 BRDF，即：

$$ f_r(l, v) = \cfrac {D(h)F(l, h)G(l, v)} {4(v \cdot n)(l \cdot n)} $$

由于清漆层通常固定为低粗糙度的各向同性电介质，除了选用标准材质模型的 DFG 项外，我们还可以选用一个计算代价更低的 V 项来替换 Smith-GGX 可见性项，但不会明显牺牲视觉质量，即 Kelemen 提出的一个遮蔽阴影函数，公式和代码如下：  

$$ V(l,h) = \cfrac {1}{4(l \cdot h)^2} $$

    float V_Kelemen(float LoH)
    {
        return 0.25 / (LoH * LoH);
    }

> 这个近似可见性项的提出主要是为了快速计算以及便于进行重要性采样，它出自 Csaba Kelemen 与 László Szirmay-Kalos 在 Eurographics 2001 的文章 A Microfacet Based Coupled Specular-Matte BRDF Model with Importance Sampling 中提出的模型。同时，这个近似只有当表面趋近完美镜面，此时微表面法线 $\,h\,$ 与宏观法线 $\,n\,$ 高度接近，即 $\,l \cdot h \approx n \cdot l \approx n \cdot v\,$ 时，代替 Smith-GGX 可见性项的误差才小。反过来，粗糙度一旦升高，$\,h\,$ 与 $\,n\,$ 偏离越大，该近似就会带来可见误差并破坏能量守恒。由于清漆层通常是低粗糙度的，故可以套用此近似项。

总的来说，clear coat BRDF 是一个 Cook-Torrance 镜面反射微表面 BRDF，其中法线分布函数选用 GGX、菲涅尔项选用 Schlick 近似式、可见性项选用 Kelemen 近似。当然可见性项选用 Smith-GGX 完全也是可以的，只是要多付出一点计算量以换取更物理正确的效果，在清漆层这种低粗糙度的场景下，两者的视觉结果几乎没有差别，如果实现上希望清漆层与基础材质共用同一套 DFG 项，直接复用 Smith-GGX 也确实更省事。

## 材质参数

折射率和 IOR

## 直接光部分

## IBL 部分

# Anisotropic
