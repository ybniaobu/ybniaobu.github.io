---
title: 《Unity Shader入门精要》读书笔记（五）
date: 2023-12-30 16:05:42
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
top_img: /images/black.jpg
cover: https://s2.loli.net/2023/12/30/hc2s7BS45l1wUdQ.gif
mathjax: true
---

> 本读书笔记为高级篇的最后一章和扩展篇，主要内容为XXXXXXXXXXXXXXXXXXXXXXXXX。
> 读书笔记是对知识的记录与总结，但是对比较熟悉的内容不会再行描述。

# 第十五章 Unity 中的渲染优化技术
在本章中，将会阐述一些 Unity 常见的优化技术。这些优化技术都是和渲染有关的，例如：使用批处理、LOD 技术（Level of Detail）等。

## 移动平台的特点
和 PC 平台相比，移动平台上的 GPU 架构有很大的不同。由于处理资源等条件的限制，移动设备上的 GPU 架构专注于尽可能使用更小的带宽和功能，也由此带来许多和 PC 不同的现象。

例如，为了尽可能移除一些隐藏的表面，减少 overdraw（同一像素绘制多次），PowerVR 芯片（用于 iOS 设备和一些 Android 设备）使用**基于瓦片的延迟渲染 Tiled-based Deferred Rendering, TBDR** 架构，把所有渲染图像装入一个个瓦片中，由硬件找到可见的片元，并只对它们执行片元着色器；另一些基于瓦片的 GPU 架构，如：Adreno（高通的芯片）和 Mali（ARM 的芯片），则会使用 Early-Z 或相似的技术进行低精度的深度检测，剔除不需要渲染的片元。还有一些 GPU，如 Tegra（英伟达的芯片），则使用了传统的架构设计，因此在这些设备上，overdraw 更可能造成性能的瓶颈。

由于这些芯片架构造成的不同，一些游戏往往需要针对不同的芯片发布不同的版本，以便对每个芯片进行更有针对性的优化。

## 影响性能的因素
游戏主要使用两种计算资源：GPU 和 CPU。CPU 主要负责保证帧率，GPU 主要负责分辨率相关。把造成游戏性能瓶颈的主要原因分成以下几方面：  
①CPU：  
&emsp;&emsp; - 过多的 drawcall，每次调用 Draw Call，CPU 往往需要改变很多渲染状态的设置，这些操作非常耗时，若大部分时间花费在提交 Draw Call 的准备工作上，会导致性能下降；  
&emsp;&emsp; - 复杂的脚本或者模拟（物理、布料、蒙皮、粒子等）。  
②GPU  
&emsp;&emsp; - 过多的顶点或过多的逐顶点计算；  
&emsp;&emsp; - 过多的片元（可能是由于分辨率造成，也有可能是 overdraw）或过多的逐片元计算。  
③带宽  
&emsp;&emsp; - 使用了尺寸很大且未压缩的纹理；  
&emsp;&emsp; - 分辨率过高的帧缓存。

--- 

在了解上面的基本内容后，本章后续涉及的优化技术有：  
①CPU 优化：  
&emsp;&emsp; - 使用批处理技术减少 Draw Call 数目；  
②GPU 优化：  
减少需要处理的顶点数目：  
&emsp;&emsp; - 优化几何体；  
&emsp;&emsp; - 使用模型的 LOD（Level of Detail）技术；  
&emsp;&emsp; - 使用遮挡剔除 Occlusion Culling 技术；  
减少需要处理的片元数目：  
&emsp;&emsp; - 控制绘制顺序；  
&emsp;&emsp; - 警惕透明物体；  
&emsp;&emsp; - 减少实时光照；  
减少计算复杂度：  
&emsp;&emsp; - 使用 Shader 的 LOD（Level of Detail）技术；  
&emsp;&emsp; - 代码方面的优化；  
③节省内存带宽：  
&emsp;&emsp; - 减少纹理大小；  
&emsp;&emsp; - 利用分辨率缩放。

## Unity 中的渲染分析工具
