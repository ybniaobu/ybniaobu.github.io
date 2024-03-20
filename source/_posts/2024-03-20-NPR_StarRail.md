---
title: 基于星穹铁道的卡通渲染
date: 2024-03-20 13:25:53
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
  - NPR
top_img: /images/black.jpg
cover: https://s2.loli.net/2024/03/20/w9by4joTmI1McSC.jpg
mathjax: true
description: 本文以米哈游的崩坏星穹铁道的卡通人物渲染来切入 NPR (Non-Photorealistic Rendering) ，从而对卡通渲染有个初步的了解，并加深对渲染的理解，为做出自己的风格化渲染打下基础。
---

> 本文主要借鉴了 b 站 up 主给我柠檬养乐多你会跟我玩吗的视频，bv 号：BV1CN411C7qx。

# 模型处理
## Blender 模型预处理
米哈游的模型都公布在模之屋 (https://www.aplaybox.com/) ，由于公布的模型都是 MMD 软件的 pmx 格式，我们需要使用 Blender 软件的插件 MMD_Tools (https://github.com/UuuNyaa/blender_mmd_tools) 转换为 FBX 格式，方便导入到 Unity。笔者使用的 blender 版本为 3.6.8 LTS，当前 MMD_Tools 插件只支持到 3.6 LTS。

具体的模型处理操作这里不进行图文说明了，依据本文开头说的视频中描述，步骤大致如下：  
①导入模型后，先把 joints 和 rigidbodies 删除；  
②进入编辑模式，将除了脸以外的使用同一张贴图的材质进行合并，脸不合并是因为我们只需要对脸进行描边，而不需要对眼睛、眉毛、牙齿舌头嘴巴进行描边。（视频里最后剩下了以下材质：上衣、下衣、头发、牙舌口、目、眉毛、眼睛1、脸、表情）；  
③把带内侧二字的材质删除，该材质使用的模型顶点区域是和外侧共用的，而且视频中使用的布洛妮娅的模型的裙内侧贴图是纯蓝色，可以直接在 Shader 中渲染背面时将基础颜色设置为蓝色（我觉得对于内侧贴图有图画的可以保留，不重复描边即可<font color=Red>【待实验】</font>）；  
④为了在 Unity 中减少渲染的纹理消耗，将形态键分离（模型中自带一些表情相关**形态键 Shape Keys**，Unity 中叫**混合形状 BlendShapes**，也有叫**变形目标 Morph Target** 的），选择跟脸相关材质对应的模型顶点（视频中选择了脸、眉毛、牙舌口、目、眼睛1、表情），并进行分离，也就是脸对应一个 mesh，身体对应一个 mesh，然后删除身体的 mesh 里的所有的形态键，并删除脸 mesh 和身体 mesh 中不属于该 mesh 的多余的材质；

## 导出 FBX
随后导出 FBX 格式，在 Blender 的导出设置中 **Apply Scalings** 选择 **FBX Units Scale** 可以解决导入 Unity 时缩放为 100 的问题。**Forward** 选择导出物体在 Blender 中的我们想要的面朝方向，**Up** 选择导出物体在 Blender 中我们想要的向上的方向，并在 Unity FBX 导入界面勾选 **Bake Axis Conversion**，这样可以解决旋转出错的问题。比如 Blender 的猴头，默认是面朝 Blender 的 -Y 轴，向上方向为 Z 轴，故导出时 Forward 选择 -Y Forward，Up 选择 Z Up，勾选 Bake Axis Conversion 后，清除物体的方向，就可以发现猴头在 Unity 面朝 Z 方向，上方为 Y 方向，符合 Unity 默认方向。

# 贴图
## 贴图预处理
> 有些贴图模之屋的文件中并不包含，比如光照贴图、渐变纹理以及面部的 SDF 贴图（**Signed Distance Field** 建议去额外了解该项技术），需要自己去获取。我贴图是在 https://github.com/umaichanuwu/GenshinLinks 中下载的。

在 Unity 导入贴图后，首先对所有贴图关闭压缩，即 Compression 选择 None；所有的光照贴图 LightMap 关闭 **sRGB（Color Texture）**，原因见下面。【书签】

## 伽马校正
在 Unity 中，若选择了**线性空间**渲染，对于 sRGB 方式存储的贴图（比如**存储颜色的贴图**），需要勾选 sRGB（Color Texture），这样这些经过 Gamma 0.45 处理的贴图在 Shader 的计算输入时（采样时）会自动进行一个 Gamma 2.2 处理变换到线性空间，这样 Shader 在线性空间中计算会更加准确，Unity URP 在渲染的最后会调用一个 Pass 再进行一次**伽马校正**（**伽马编码**），即 Gamma 0.45，这样显示器输出 Gamma 2.2 平衡后颜色才会正确。但是对于**法线贴图**、**光照贴图**等这些直接按线性方式存储的贴图，需要取消勾选 sRGB（Color Texture），这样 Unity 就不会自动**去除伽马校正 Remove Gamma Correction**，即 Gamma 2.2 处理，保证在 Shader 的计算中是线性的，同样 Unity 会自动进行伽马校正，从而正确地在显示器显示。如下图：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/20/B8XVCEP1TpvLNjc.png" width = "70%" height = "70%" alt="图1 - 线性空间"/>
</div>

除了纹理，在线性空间模式下，ShaderLab 的颜色属性也会被认为是 sRGB 颜色，会自动进行 Remove Gamma Correction。但是对于材质的 Float 或 Vector 属性，不会自动进行色彩空间转换，因此对于一些跟色彩相关的 Float 或 Vector 属性，比如金属度，需要把它们指定到 sRGB 空间，此时可以添加 `[Gamma]` 特性。另外要注意，Alpha 通道是不受 Gamma 校正的影响的。

## 贴图作用分析
