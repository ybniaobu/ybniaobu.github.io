---
title: PBR 流程 DCC 软件功能备忘
date: 2025-12-05 16:36:28
tags:
  - 3D建模
  - DCC
categories:
  - [DCC]
top_img: /images/black.jpg
cover: https://s2.loli.net/2022/11/29/uXNZDocJpQx5yrh.jpg
description: （长期不间断更新）本文章主要记录常用于 PBR 流程 3D 建模的 DCC 软件（诸如 blender, Zbrush, RizomUV, SP, Toolbag 等）的功能，作为备忘用途，便于日后查阅。
---

> 最好是基于**目的需求**进行记录，以方便理解记忆。

# PBR 流程
* **低模包裹高模原则**，但是低模和高模相互之间穿插也是可以的。若高模包裹低模会出现渲染伪影或细节丢失。  
* 当转折面的角度小于或等于 90° 时，需要进行 uv 拆边，并将转折边设置为硬边；否则，不进行拆分并设置为软边。**软边 uv 断不断开可视情况而定，但硬边 uv 必须明确断开**。

# Blender
## 建模习惯
* **布线原理**：以四边面为基础，通过**三极点 + 五极点**的组合扩展出任意形状，必要时可增加**缓冲带**。具体做法就是，将模型形状分为三边面四边面五边面的组合，三边面中一个三极点，五边面中一个五极点：  

<div align="center">  
<img src="https://s2.loli.net/2025/12/09/aGt4ZR6Srb2H1NC.png" width = "60%" height = "60%" alt="图1 - 三极点和五极点"/>
</div>

* **减面（减线）原理**：
    - 方法 1：利用循环边原理减线，即所谓的五转三、四转二、三转一，以及二转一，他们的原理是一样的：  

    <div align="center">  
    <img src="https://s2.loli.net/2025/12/09/H9G82dwaWtFymTu.png" width = "60%" height = "60%" alt="图2 - 循环减线"/>
    </div>

    - 方法 2：**钻石分面**，其实就是图 1 中最右边的示例，因为最中间的四边面像钻石而得名。它也是四转二，只不过和循环边原理不同，本质就是使用三极点 + 五极点的组合：  

    <div align="center">  
    <img src="https://s2.loli.net/2025/12/09/loqsugmBkY73AFE.png" width = "15%" height = "15%" alt="图3 - 钻石分面"/>
    </div>

* 在倒角之前使用**折痕边**，这样子可以在表面细分修改器下提前看到倒角的效果，就是要注意最后倒角时，将折痕边权重都清零。倒角的最好方式是使用**倒角修改器 + 倒角权重**，可以避免倒角工具的破坏性建模。

## 特殊需求
* 模型中间**开圆洞**，编辑模式下，选择需要开洞的矩形区域的顶点（建议先内插一次）：
    - 方法 1：可以使用 Blender 自带插件 LoopTools，右键 -> LoopTools -> Circle 圆形化该矩形；  
    - 方法 2：或者使用自带的 To Sphere 工具，快捷键：<kbd>ctrl</kbd> + <kbd>shift</kbd> + <kbd>S</kbd>。  

* **均匀分布**线上的顶点：选择需要均匀分布的顶点，右键 -> LoopTools -> Space。




# Zbrush
* **循环对称**：我们知道按 <kbd>X</kbd>、<kbd>Y</kbd>、<kbd>Z</kbd> 可以让笔刷沿 x、y、z 轴对称，但有时我们想围绕着圆环雕刻数个花纹（比如每 60 度一个花纹），此时可以开启循环对称，点击循环 Transform ->激活对称 Activate Symmetry -> R，还可以选择循环的个数（即径向个数 Radial Count）。

* **还原模型指定部分**至初始状态：需要在模型处于初始状态时（未雕刻时），点击工具 Tool -> 变换目标 Morph Target-> 存储变换目标 StoreMT，然后雕刻时若发现哪里雕刻得不好，但是又不想撤销，就可以使用笔刷中的 **Morph 笔刷**还原模型。  

* **ZRemesher 布线辅助**：有时 ZRemesher 重拓扑的布线不是我们想要的，我们可以使用 **ZRemesherGuides 笔刷** 来在模型表面画线，用于辅助布线，画完线再点击 ZRemesher 重拓扑，此时就会按照自己画的线进行布线。



# RizomUV

# SP

# Toolbag

