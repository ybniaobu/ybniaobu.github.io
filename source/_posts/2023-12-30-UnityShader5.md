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

> 本读书笔记为高级篇的最后一章和扩展篇，主要内容为高级篇最后一章渲染优化；扩展篇的表面着色器以及 PBS。
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
Unity 内置了一些工具，用来查看和渲染相关的统计数据，包括渲染统计窗口 Rendering Statistics Window、性能分析器 Profiler，以及帧调试器 Frame Debugger。

### 渲染统计窗口
就是 Game 窗口的 Stats 按钮。图片不放出了，主要包括 3 个方面的信息：音频 Audio、图像 Graphics 和网络 Network。而图像相关的渲染统计结果包含了很多重要的渲染数据，如下表：  

| 信息名称 | 描述 |
| :---- | :---- |
| 每帧的时间和 FPS | Frame per Second，后面的括号中显示的是当前帧的耗时 |
| CPU：main / render thread | main 是主线程的耗时，render thread 是渲染线程的耗时 |
| Batches | 当前帧需要进行的批处理数目 |
| Saved by batching | 合并的批处理数目，这个数字表明了批处理为我们节省了多少 draw call |
| Tris / Vertes | 当前帧绘制的三角面片和顶点的数目 |
| Screen | 屏幕的大小，及它占用的内存大小 |
| SetPass calls | 渲染使用的 Pass 的数目，每个 Pass 都需要 Unity 的 runtime 来绑定一个新的 Shader。数目越大，越容易产生 CPU 的性能瓶颈 |
| Shadow casters | 当前帧投射阴影的物体数目 |
| Visible skinned meshes | 渲染的蒙皮网格数目 |
| Animation / Animator components playing | 当前帧播放的 Animation / Animator 数目 |


### 性能分析器
Window -> Analysis -> Profiler 打开性能分析器 Profiler 。这里图片也不放了。性能分析器中的渲染模块 Rendering 提供了更多关于渲染的统计信息，比如，draw call 数目、动态批处理/静态批处理的数目、渲染纹理的数目和内存占用等。

使用 Unity 性能分析器可以通过三种主要方式记录数据：  
①在目标平台上的播放器中对应用程序进行性能分析；  
②在 Unity 编辑器中以运行模式对应用程序进行性能分析；  
③对 Unity 编辑器进行性能分析。

获得有关应用程序性能的最佳方法是在打算发布它的终端平台上对它进行性能分析。详见官方文档：https://docs.unity3d.com/2022.3/Documentation/Manual/profiler-profiling-applications.html

### 帧调试器
Window -> Analysis -> Frame Debugger 打开帧调试器面板。在该窗口中可以看到每一个 draw call 的工作和结果。之前讲过，不再重复摘抄了。需要详细信息见官方文档：https://docs.unity3d.com/2022.3/Documentation/Manual/frame-debugger-window.html

### 其他性能分析工具
①对 Android 平台来说：  
&emsp;&emsp; - 高通的 Adreno 分析工具可以对不同的测试机进行详细的性能分析。  
&emsp;&emsp; - 英伟达的 NVPerfHUD 工具可以帮助我们得到几乎所有需要的性能分析数据，如：每个 Draw Call 的 GPU 时间、每个 Shader 花费的 cycle 数目等。  
②对 iOS 平台来说：  
&emsp;&emsp; - PowerVRAM 的 PVRUniSCo Shader 分析器可以给出大致的性能评估。
&emsp;&emsp; - XCode 的 OpenGL ES Driver Instruments 可以给出一些宏观上的性能信息，如：设备利用率、渲染器利用率等。

相比 Android，对 iOS 的性能分析更加困难（工具较少）。且 PowerVR 芯片采用了基于瓦片的延迟渲染器，想得到每个 Draw Call 花费的 GPU 时间几乎不可能，所以一些宏观上的统计数据更有参考价值。

## 减少 draw call 数目
最常见的优化技术大概就是**批处理 batching**了。批处理的实现原理就是为了减少每一帧需要的 draw call 的数量。为了把一个对象渲染到屏幕上，CPU 需要检查哪些光源影响了该物体，绑定 shader 并设置它的参数，再把渲染命令发送给 GPU。当场景中包含了大量对象时，这些操作会非常耗时。批处理就是在每次调用 draw call 时尽可能的处理多个物体。

什么样的物体可以进行一起批处理：即使用相同材质的物体。因为它们之间的不同仅仅在于顶点数据的差别，可以把这些顶点数据合并在一起发送给 GPU，就可以完成一次批处理。

Unity 支持两种批处理方式：  
①**动态批处理**：有点在于一切都是Unity自动完成，无需自己做任何操作，物体可移动。缺点是限制太多，一不小心就破坏这种机制，导致无法动态批处理一些使用了相同材质的物体；  
②**静态批处理**：自由度高，限制少。但是占用更多内存，而且经过静态批处理后的物体不能再移动（即使脚本中尝试改变物体的位置也是无效的）。

### 动态批处理
如果场景中有一些模型共享了同一材质并满足了一些条件，Unity 就自动把它们进行批处理，从而只需要花费一个 draw call 就可以渲染所有模型。动态批处理的基本原理就是每一帧把可以进行批处理的模型网格进行合并，再把合并后模型数据传递给 GPU，然后使用同一个材质对其渲染。除了实现方便，动态批处理的另一个好处是，经过批处理的物体仍然可以移动，这是因为由于在处理每帧时 Unity 都会重新合并一次网格。

模型网格批处理合并的主要的条件限制（随着 Unity 版本变化，条件可能会改变）：  
①网格的顶点属性规模要小于 900。例如，如果 Shader 中需要使用顶点位置、法线、纹理坐标这 3 个顶点属性，想要让模型被动态批处理，他们的顶点数目不能超过 300；  
②一般来说，所有对象都需要使用同一个缩放尺度。一个例外情况是，如果所有物体都使用了不同的非统一缩放，那么也是可以被动态批处理的。但在 Unity 5 中，这种对模型缩放的限制不存在了；  
③使用光照纹理 lightmap 的物体需要额外的渲染参数，例如，在光照纹理上的索引、偏移量和缩放信息等。为了让这些物体可以被动态批处理，需要保证它们指向光照纹理的同一个位置；  
④多 Pass 的 shader 会中断批处理。前向渲染中，有时需要额外的 Pass 来为模型添加更多的光照效果，这样会导致模型无法被完全动态批处理。Unity 只会批处理第一个 Pass，不能为额外的逐像素光源进行批处理。

### 静态批处理
相对于动态批处理来说，静态批处理适合于任何大小的几何模型。其实现原理是，只在运行开始阶段，把需要进行静态批处理的模型合并到一个新的网格结构中，这意味着这些模型不可以在运行时刻被移动。静态批处理的另一个缺点在于，它往往需要占用更多的内存来存储合并后的几何结构。因为，若在静态批处理前一些物体共享了相同的网格，那么在内存中每一个物体都会对应一个该网格的复制品，即一个网格会变成多个网格再发给 GPU。如果这一类使用同一网格的对象很多，那么将会成为一个性能瓶颈。

静态批处理的实现，只需要把物体面板右上角的 **Static** 勾选上即可，实际上只需要在其右边的下拉菜单里勾选 Batching Static 即可。

在内部实现上，Unity 首先把这些静态物体变换到世界空间下，为它们构建成一个更大的顶点和索引缓存。对于使用同一材质的物体，Unity 只需调用一个 Draw Call 就可以绘制全部物体；对于使用不同材质的物体，静态批处理同样可以提升渲染性能，尽管仍然需要调用多个 Draw Call，但静态批处理可以减少这些 Draw Call 之间的状态切换，而这些切换往往是费时的操作。我们可以从 Unity 的分析器中观察在应用静态批处理前后 **VBO total**（**Vertex Buffer Object**，顶点缓冲对象）的变化。VBO 的数目变大，即它需要更多的内存存储合并后的几何结构，即如果一些物体共享了相同的网格，那么在内存中每一个物体都会对应一个该网格的复制品。

若场景中包含了除了平行光以外的其他光源，并且在 shader 中定义了额外的 pass 来处理它们，这些 pass 将不会被批处理，而平行光的 base pass 仍然可以被静态批处理。

### 共享材质
无论是动态批处理还是静态批处理，都要求模型之间共享同一个材质。但不同模型之间总需要有不同的渲染属性，如：不同的纹理、颜色等，这时需要一些策略尽可能地合并材质。

如果两个材质之间只有使用的纹理不同，可以这些纹理合并到一张更大的纹理中，这张更大的纹理被称为一张**图集 atlas**。一旦使用了同一张纹理，就可以使用同一个材质，再使用不同的采样坐标对纹理采样即可。

但除了纹理不同外，不同的物体在材质上还有一些微小的参数变化。但是只要调整了参数，就会影响到所有使用了这个材质的对象。一个常用的解决方案就是使用网格的顶点数据（最常见的就是顶点颜色数据）来存储这些参数。前面说过，经过批处理后的物体会被处理成更大的 VBO 发送给 GPU，VBO 中的数据可以作为输入传递给顶点着色器，因此可以对 VBO 中的数据进行控制，从而达到不同的目的。

如果需要在脚本中访问共享材质，可以使用 `Renderer.sharedMaterial` 来修改共享材质。

### 批处理的注意事项
①尽可能选择静态批处理，要时刻小心对内存的消耗，记住经过静态批处理的物体不可以再被移动；  
②如果无法进行静态批处理而需要动态批处理时，小心各种限制条件；  
③游戏中的小道具，如可以拾取的金币，可以使用动态批处理；  
④对于包含动画的物体，无法全部使用静态批处理，而如果不动的部分可以把这部分标示为 static；  
⑤批处理需要把多个模型变换到 *世界空间* 下合并他们，因此，如果 shader 存在一些在模型空间下的坐标的运算，那么往往会得到错误的结果，可以在这部分的 shader 中使用 **DisableBatching** 标签来强制使用该 shader 的材质不会被批处理；  
⑥对于使用 *半透明材质* 的物体通常需要使用严格的从后往前的绘制顺序来保证透明混合的正确性。对于这些物体，Unity 会首先保证它们的绘制顺序，再尝试对它们进行批处理。即当绘制顺序无法满足时，批处理无法被成功应用。


## 减少需要处理的顶点数目
顶点数目可能会造成 GPU 的性能瓶颈。有 3 个常用的顶点优化策略。

### 优化几何体
优化网格来尽可能减少模型中三角面片的数目或顶点数目。Unity 的渲染统计窗口中可以查看渲染当前帧需要的三角面片数目和顶点数目。需要注意的是：Unity 中显示的顶点数目往往要多于建模软件里显示的顶点数。主要有几个原因：  
①三维软件是以人类的角度理解顶点，而 Unity 是站在 GPU 的角度计算顶点数的；  
②在 GPU 看来，有时候一个顶点需要拆分成多个顶点，其原因主要有两个：一是为了**分离纹理坐标 uv splits**，另一个是为了**产生平滑的边界 smoothing splits**。建模时一个顶点的纹理坐标有多个，例如：一个立方体，它的 6 个面之间虽然使用了一些相同的顶点，但在不同面上，同一个顶点的纹理坐标可能并不相同。而对 GPU 来说，顶点的每个属性和顶点之间必须是一对一的关系，所以必须将这个顶点拆分成多个具有不同纹理坐标的顶点。而平滑边界也是类似，顶点可能会对应多个法线信息或切线信息，这通常是因为我们要决定这个边是**硬边 hard edge** 还是**平滑边 smooth edge**。

使用可以移除不必要的硬边以及纹理衔接，避免边界平滑和纹理分离，来尽可能减少顶点数目。

### 模型的 LOD 技术
**LOD，Level of Detail** 技术的原理是，当一个物体离摄像机很远时，模型上的很多细节是无法被察觉到的，因此 LOD 允许当对象逐渐远离摄像机时，减少模型上的面片数量，从而提高性能。  

在 Unity 中，可以使用 LOD Group 组件来为一个物体构建一个 LOD。当需要为一个对象准备多个包含不同细节程度的模型，然后把它们赋给 LOD Group 组件中的不同等级，Unity 会自动判断当前位置上使用哪个等级的模型。

### 遮挡剔除技术
**遮挡剔除 Occlusion Culling** 可以用来消除那些被其他物体遮挡住的物体，这意味着看不见的顶点不会被 GPU 渲染，从而提升性能。

注意，区分遮挡剔除和摄像机的视锥体剔除 Frustum Culling。视锥体剔除只会剔除掉那些不在相机的视野范围内的对象，但不会判断视野中是否有物体被其他物体遮挡。而遮挡剔除会使用一个虚拟的相机来遍历场景，从而构建一个潜在可见的对象集合的层级结构。运行时，每个相机将会使用这个数据来识别哪些物体可见，哪些物体被挡住。

遮挡剔除不仅可以减少处理的顶点数目，还可以减少 overdraw，提高游戏性能。


## 减少需要处理的片元数目
过多的片元是造成 GPU 性能瓶颈的另一原因，其优化的重点在于减少 **Overdraw**，Overdraw 指的是同一个像素被绘制了多次。

Unity 提供了查看 overdraw 的视图，在 Scene 视图的 2D 按钮前一个 Shading Mode 的球体按钮下拉菜单里可以选择 Overdraw。实际上，这里的视图只是提供了查看物体互相遮挡的层数，并不是真正绘制屏幕的最终 overdraw。也就是说，它显示的是，没有使用任何深度测试和其他优化策略时的 overdraw。这种视图是通过把所有对象都渲染成一个透明的轮廓，通过查看透明颜色的累计程度来判断物体之间的遮挡。

### 控制绘制顺序
为了最大限度地避免 overdraw，一个重要的优化策略就是控制绘制顺序。由于深度测试的存在，从而如果保证了所有物体都是从前往后进行绘制，从很大程度上减少 overdraw。因为后面绘制的物体无法通过深度测试，从而无法被绘制出来。  

在 Unity 中，渲染队列数目小于 2500（如 “Background”、“Geometry”、“AlphaTest”）的对象都被认为是不透明 opaque 的物体，这些物体总体上是从前到后绘制的。而使用其他队列（如 “Transparent”、“Overlay” 等）的物体，则是从后到前，意味着开发者尽可能把物体的队列设置为不透明物体的渲染队列，而经可能避免使用半透明队列。

而且，我们还可以充分利用渲染队列，比如：  
①在第一人称射击游戏，游戏的主角的 shader 往往比较复杂。但是通常会挡住屏幕的很大一部分区域，因此可以将主角的渲染队列调小，先渲染；  
②对于一些敌方角色，由于它们通常在掩体的后面，所以可以将它们的渲染队列调大，这样先渲染不透明掩体，掩体后面的敌人就不用再被渲染；  
③对于天空盒来说，它几乎覆盖所有的像素，而且它永远会出现在所有物体的后面。将它的队列设置为 “Geometry+1”，就可以保证其最后渲染，减少 Overdraw。

### 时刻警惕透明物体
半透明物体因为没有开启深度写入，需要从后往前进行渲染，这意味着一定会产生 overdraw 现象。

例如，对于 GUI 对象来说，如果大多被设置成了半透明，并且占据比例太多，那么会造成大量 overdraw。因此，场景中若包含了大面积的半透明物体，或者很多层相互覆盖的半透明对象，或者是透明的粒子效果，在移动设备上也会造成大量的 overdraw。

可以尽量减少窗口中的 GUI 所占的面积，如果没办法，可以把 GUI 的绘制和三维场景的绘制交给不同的摄像机，而其中负责三维场景的摄像机的视角范围经历不要与 GUI 的相互重叠。

在移动平台上，透明度测试也会影响游戏性能。透明度测试会使用 discard 或 clip 操作，导致一些硬件的优化策略失效。比如：PowerVR 使用的基于瓦片的延迟渲染技术，为了减少 Overdraw 它会调用片元着色器前就判定哪些瓦片被真正渲染。但由于透明度测试在片元着色器中使用了 discard 函数改变了片元是否会被渲染的结果，因此 GPU 无法使用上述的优化策略，只有执行了所有的片元着色器后，GPU 才知道哪些片元会被真正渲染到屏幕上，这样原先减少的 Overdraw 的优化就都失效了。

### 减少实时光照和阴影
如果场景中包含了过多的点光源，并且使用了多个 Pass 的 Shader，那么很可能会造成性能下降。对于逐像素的点光源，使用了逐像素的 Shader 不仅会提高 draw call，同时会增加 overdraw。这是因为，对于逐像素的光源来说，被这些光源照亮的物体需要被再渲染一次，而且也会中断批处理。

模拟光源的方法有：  
①使用烘焙技术，把光照提前烘焙到一张**光照纹理 lightmap** 中，在运行时对纹理进行采样即可。  
②使用 **God Ray**，即**丁达尔效应**，不是真的光源，而是通过透明纹理模拟得到的。  

开发者还可以把复杂的光照计算存储到一张**查找纹理 lookup texture**，即**查找表 lookup table, LUT**，中。然后只需要使用光源方向、视角方向、法线方向等参数，对 LUT 采样得到光照结果即可。对于主要角色使用更大分辨率的 LUT，一些 NPC 就使用较小的 LUT，这样可以优化性能。

实时阴影同样是非常消耗性能的效果，不仅是 CPU 需要提交更多的 Draw Call，GPU 也要做贡多的处理。所以尽量减少实时阴影，可以使用烘焙把静态物体的阴影信息存储到光照纹理当中，而只对动态物体使用适当的实时阴影。


## 节省带宽
大量使用未经压缩的纹理以及过大的分辨率都会造成由于带宽而引发的性能瓶颈。  

### 减少纹理大小
需要注意的是，所有纹理的长宽比最好是正方形，而且长宽值最好是 2 的整数幂。很多优化策略只有在这种时候才可以发挥最大效用。在 Unity 5 中，即使导入的纹理长宽值并不是 2 的整数幂，Unity 也会自动把长宽转换到离他最近的 2 的整数幂。

 除此之外，还应该尽可能使用**多级渐变纹理技术 mipmapping** 和**纹理压缩**。  
 ①纹理的属性面板里，勾选 **Generate Mip Maps**，Unity 就会为同一张纹理创建出很多大小不同的小纹理。而在游戏运行时可以根据物体的远近来动态选择使用哪一个纹理。  
 ②而纹理压缩，不同的 GPU 架构有它自己的纹理压缩格式，例如 PowerVRAM 的 PVRTC 格式、Tegra 的 DXT 格式、Adreno 的 ATC 格式。所幸的是，Unity 会根据不同的设备选择不同的压缩格式，只需设置为自动压缩即可。但是对一些有一定画质要求的纹理，比如 GUI，可以选择不压缩。

 ### 利用分辨率缩放
 过高的分辨率也是造成性能下降的原因之一，尤其对于很多低端手机，除了分辨率高其他硬件条件不尽如人意，也就是游戏性能的两大瓶颈：过大的屏幕分辨率、糟糕的 GPU。

对于特定设备，将其屏幕分辨率设低，再放大到屏幕的尺寸，虽然降低游戏效果，但是可以带来性能上的提升。Unity 中设置分辨率可以调用`Screen.SetResolution`。


## 减少计算复杂度
计算复杂度同样会影响游戏的渲染性能，可通过两方面的技术来减少计算复杂度：Shader 的 LOD 技术、代码方面的优化。

 ### Shader 的 LOD 技术
 跟之前提到的**模型的 LOD 技术**类似，**Shader 的 LOD 技术**可以控制使用的 Shader 等级。它的原理是，只有 Shader 的 LOD 值小于某个设定的值，这个 Shader 才会被使用，而使用了那些超过设定值的 Shader 的物体将不会被渲染。

 通常会在 SubShader 中使用类似下面的语句来指明该 Shader 的 LOD 值：  

     SubShader {
        Tags { "RenderTYpe" = "Opaque"}
        LOD 200

在 Unity Shader 的导入面板上看到该 Shader 使用的 LOD 值。默认情况下，LOD 等级是无限大的，即任何被当前显卡支持的 Shader 都可以被使用。有时需要去掉一些使用了复杂计算的 Shader 渲染，这时可以使用 `Shader.maximumLOD` 或 `Shader.globalMaximumLOD` 来设置允许的最大 LOD 值。

Unity 内置的 Shader 使用了不同的 LOD 值，例如：Diffuse 的 LOD 值为 200，Bumped Specular 的 LOD 值为 400。

### 代码方面的优化
游戏需要计算的对象、顶点和像素排序是：对象数 < 顶点数 < 像素数。因此需要尽可能需要把计算放在对象或逐顶点上，例如在实现高斯模糊或边缘检测，把采样坐标的计算放在顶点着色器中，该做法好于原片着色器中。

尽可能使用低精度的浮点值进行运算：  
①最高精度的 float/highp 适用于存储顶点坐标等变量，但它计算速度最慢，所以应尽量避免在片元着色器中使用这种精度的计算；  
②half/mediump 适用于一些标量、纹理坐标等变量，其计算速度约为 float 的两倍；  
③fixed/lowp 适用于大多数颜色变量和归一化后的方向矢量，对于一些精度要求不高的计算，尽量使用这种精度，其计算速度约为 float 的 4 倍。但要避免频繁的 swizzle 操作（如：color.xwxw），同时避免不同精度间的转换。

对于绝大多数 GPU 来说，在使用**插值寄存器**把数据从顶点着色器传递给下一个阶段时，我们应该使用尽量少的插值变量。例如，如果需要对两个纹理坐标进行插值，通常需要会把它们打包在同一个 float4 类型的变量中，两个纹理坐标分别对应了 xy 分量和 zw 分量。然而对于 PowerVR 平台来说，这种插值变量是非常廉价的，直接把不同的纹理坐标存储在不同的插值变量中，性能会更好。尤其是，如果在 PowerVR 上使用类似 `tex2D(_MainTex, uv.zw)` 语句进行采样，GPU 就无法进行一些纹理的预读取，因为它会认为这些纹理坐标是需要依赖其他数据的。

尽可能不使用全屏的屏幕后处理效果。如果必须使用，尽量使用 fixed/lowp 进行低精度运算（纹理坐标除外，可使用 half/mediump）；高精度运算可使用查找表（LUT）或转移到顶点着色器进行处理；尽量把多个特效合并到一个 Shader 中，例如：颜色校正和添加噪声等屏幕特效在 Bloom 特效的最后一个 Pass 中进行合成。  

其他代码优化规则：  
①尽量不要使用分支语句和循环语句；  
②避免使用 sin、tan、pow、log 等较为复杂的数学运算，用查找表代替；  
③尽量不要使用 discard 操作，这会影响硬件的某些优化。

### 根据硬件条件进行缩放
保证游戏基本的配置可以在所有的平台上运行良好；对于一些具有更高表现能力的设备，可以开启一些更“养眼”的效果，如：使用更高分辨率、开启屏幕后处理特效、开启粒子效果等。


## 扩展阅读
Unity 官方手册给出了很多优化的建议，建议详细阅读。


# 第十六章 Unity 的表面着色器
2009年（Unity 2.x），Unity 的渲染工程师 Aras 连续发表3篇名为《Shaders must die》的博客。博客中，Aras 认为：把渲染流程分为顶点和像素的抽象层面是不易理解的，这种在顶点/几何/片元着色器上的操作是对硬件友好的一种方式，他提出应该划分为：表面着色器、光照模型和光照着色器这样的层面。  
①表面着色器定义模型表面的反射率、法线和高光等；  
②光照模型选择使用兰伯特还是 Blinn-Phong 等模型；  
③光照着色器负责计算光照衰减、阴影等。

这样绝大多数时候开发者只需要和表面着色器打交道，如：混合纹理、颜色等；而光照模型是可以提前定义好的，只需要选择几种预定义的光照模型即可；光照着色器一旦由系统实现后，更不会轻易改动。这样大大减轻 Shader 编写者的工作量。2010年的 Unity 3 中，Surface Shader 被加入到 Unity 中。

## 表面着色器的一个例子
准备工作如下：  
①新建名为 Scene_17_1 的场景，并去掉天空盒，在场景中新建一个胶囊体；  
②新建名为 BumpedSpecularMat 的材质，并赋给场景中的胶囊体；  
③新建名为 Chapter17-BumpedDiffuse 的 Unity Shader，并赋给上一步创建的材质。

这里使用表面着色器来实现了一个使用了法线纹理的漫反射效果，下面代码参考的是 Unity 内置的 DefaultResourcesExtra/Bumped Diffuse 的代码：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 17/Bumped Diffuse" {
    Properties {
        _Color ("Main Color", Color) = (1.0, 1.0, 1.0, 1.0)
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _BumpMap ("Normalmap", 2D) = "bump" {}
    }

    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 300

        CGPROGRAM

        #pragma surface surf Lambert
        
        sampler2D _MainTex;
        sampler2D _BumpMap;
        fixed4 _Color;

        struct Input {
            float2 uv_MainTex;
            float2 uv_BumpMap;
        };

        void surf(Input IN, inout SurfaceOutput o) {
            fixed4 tex = tex2D(_MainTex, IN.uv_MainTex);
            o.Albedo = tex.rgb * _Color.rgb;
            o.Alpha = tex.a * _Color.a;
            o.Normal =  UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));
        }

        ENDCG
    }
    Fallback "Legacy Shaders/Diffuse"
}
```

在材质面板上拖拽一张漫反射纹理、一张法线纹理，分别在对应路径为：Assets/Textures/Chapter17/Mud_Diffuse.tif 和 Assets/Textures/Chapter17/Mud_Normal.tif。我们还可以向场景中添加一些点光源和聚光灯，我们不需要对代码做任何改动，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/03/wWrkYsa3RSv6XDl.jpg" width = "60%" height = "60%" alt="图83- 表面着色器的例子。左图：在一个平行光下的效果。右图：添加了一个点光源（蓝色）和一个聚光灯（紫色）后的效果。"/>
</div>

从上面例子来看，相比顶点/片元着色器，表面着色器的代码很少。而且，可以轻松地实现常见的光照模型，不需要和任何光照变量打交道，Unity 帮我们处理好了每个光源的光照结果。

和顶点/片元着色器需要包含到一个特定的 Pass 块中不同，**表面着色器的 Cg 代码是直接而且必需写在 SubShader 块中，Unity 会在背后生成多个 Pass**。

## 编译指令
一个表面着色器中最重要的部分是**两个结构体**以及它的**编译指令**，两个结构体是表面着色器中不同函数之间信息传递的桥梁，而编译指令是开发者和 Unity 沟通的重要手段。

编译指令最重要的作用是指明该表面着色器使用的**表面函数**和**光照函数**，并设置一些可选参数。表面着色器的 Cg 块中第一句往往就是编译指令，编译指令的一般格式如下：  

    #pragma surface surfaceFunction lightModel [optionalparams]

其中，**#pragma surface** 用于指明该编译指令是用于定义表面着色器的，在它后面需要指明使用的表面函数 surfaceFunction 和光照模型 lightModel，同时还可以使用一些可选参数来控制表面着色器的一些行为。

### 表面函数
一个对象的表面属性定义了它的反射率、光滑度、透明度等值，而表面函数用于定义这些表面属性。surfaceFunction 通常就是名为 surf 的函数（函数名可以是任意的），其函数格式是固定的：  

    void surf (Input IN, inout SurfaceOutput o)
    void surf (Input IN, inout SurfaceOutputStandard o)
    void surf (Input IN, inout SurfaceOutputStandardSpecular o)

其中，后两个是基于物理的渲染。**SurfaceOutput**、**SurfaceOutputStandard** 和 **SurfaceOutputStandardSpecular** 都是 Unity 内置的结构体，需要配合不同的光照模型使用，后面会介绍。

在表面函数中，会使用输入结构体 **Input IN** 来设置各种表面属性，并把这些属性存储在结构体  SurfaceOutput、SurfaceOutputStandard 和 SurfaceOutputStandardSpecular 中，再传递给光照函数计算光照结果。

可以在 Unity 手册的表面着色器例子中找到更多示例：https://docs.unity3d.com/Manual/SL-SurfaceShaderExamples.html

### 光照函数
光照函数会使用表面函数中设置的各种表面属性，来应用某些光照模型，进而模拟物体表面的光照效果。Unity 内置了基于物理的光照模型函数：**Standard**、**StandardSpecular**（在 UnityPBSLighting.cginc 文件中被定义），以及简单的非基于物理的光照模型函数：**Lambert**、**BlinnPhong**（在 Lighting.cginc 文件中被定义）。

当然，也可以定义自己的光照函数。例如，可以使用下面的函数来定义用于前向渲染中的光照函数：  

    // 用于不依赖视角的光照模型，例如：漫反射
    half4 Lighting<Name> (SurfaceOutput s, half3 lightDir, half atten);
    // 使用依赖视角的光照模型，例如：高光反射
    half4 Lighting<Name> (SurfaceOutput s, half3 lightDir, half3 viewDir, half atten);

### 其他可选参数
**可选参数 optionalparams** 包含了很多有用的指令类型，例如：开启/设置透明混合/透明度测试，指明自定义的顶点和颜色修改函数，控制生成的代码等。下面选取了一些比较重要和常用的参数进行更深入地说明，可以在 Unity 官方手册查看更多：  
①自定义的修改函数：除了表面函数和光照模型外，表面着色器还可以支持其他两种自定义的函数：**顶点修改函数 vertex:VertexFunction**和**最后的颜色修改函数 finalcolor:ColorFunction**。顶点修改函数允许开发者自定义一些顶点属性，例如，把顶点颜色传递给表面函数，或是修改顶点位置，实现某些顶点动画等。最后的颜色修改函数则可以在颜色绘制到屏幕前，最后一次修改颜色值，例如实现自定义的雾效等。  
②阴影：可以通过一些指令来控制和阴影相关的代码。例如，**addshadow** 参数会为表面着色器生成一个阴影投射的 pass。通常情况下，Unity 可以直接在 FallBack 中找到通用的光照模式为 ShadowCaster 的 pass，从而将物体正确地渲染到深度和阴影纹理中。但是对于一些进行了顶点动画、透明度测试地物体，就需要特殊处理，正如第 10 章最后说的一样；**fullforwardshadows** 参数则可以在前向渲染路径中支持所有光源类型的阴影。默认情况下，Unity 只支持最重要的平行光的阴影效果，如果需要让点光源或聚光灯在前向渲染中也有阴影，就可以添加这个参数；如果不想对这个 Shader 的物体进行任何阴影计算，就可以使用 **noshadow** 参数来禁用阴影。  
③透明度混合和透明度测试：可以通过 **alpha** 和 **alphatest** 指令来控制透明度混合和透明度测试。例如，**alphatest:VariableName** 指令会使用名为 VariableName 的变量来剔除不满足条件的片元。此时，还需要使用前面提到的 **addshadow** 参数来生成正确的阴影投射的 Pass。  
④光照：**noambient** 参数会告诉 Unity 不要应用任何环境光照或光照探针 light probe；**novertexlights** 参数告诉 Unity 不要应用任何逐顶点光照；**noforwardadd** 会去掉所有前向渲染中的额外的 Pass，也就是说，这个 Shader 只会支持一个逐像素的平行光，而其他的光源会按照逐顶点或 SH 的方法来计算光照影响；还有一些用于控制光照烘焙、雾效模拟的参数，如 **nolightmap**、**nofog** 等。
⑤控制代码的生成：默认情况下，Unity 会为一个表面着色器生成相应的前向渲染路径、延迟渲染路径使用的 Pass，这会导致生成的 Shader 文件比较大。如果确定该表面着色器只会在某些渲染路径中使用，可使用 **exclude_path:deferred**、**exclude_path:forward** 和 **exclude_path:prepass** 来告诉 Unity 不需要为某些渲染路径生成代码。


## 两个结构体
表面着色器支持最多自定义 4 个关键的函数：表面函数（设置表面属性，如反射率、法线等）、光照函数（定义光照模型）、顶点修改函数（修改、传递顶点属性）、颜色修改函数（修改最后的颜色）。那么这些函数之间的信息是如何传递的？这就是两个结构体的工作。

一个表面着色器需要使用两个结构体：表面函数的输入结构体 **Input**，以及存储了表面属性的结构体 **SurfaceOutput**、**SurfaceOutputStandard** 或 **SurfaceOutputStandardSpecular** 。

### 数据来源：Input 结构体
**Input** 结构体包含了许多表面属性的数据来源，因此，它会作为表面函数的输入结构体（如果自定义了顶点修改函数，它还会是顶点修改函数的输出结构体）

Input 支持很多内置的变量名，比如本章开头的表面着色器代码中的 Input 结构体包含了主纹理和法线纹理的采样坐标 **uv_MainTex** 和 **uv_BumpMap**。这些采样坐标必须以 “uv” 为前缀（也可以用 “uv2” 为前缀，表明使用次纹理坐标集合），后面紧跟纹理名称。

Input 结构体中内置的其他变量：  

| 变量 | 描述 |
| :---- | :---- |
| float3 viewDir | 包含了视角方向，可用于计算边缘光照等 |
| 使用 COLOR 语义定义的 float4 变量 | 包含了插值后的逐顶点颜色 |
| float4 screenPos | 包含了屏幕空间的坐标，可以用于反射或屏幕特效 |
| float3 worldPos | 包含了世界空间下的位置 |
| float3 worldRefl | 包含了世界空间下的反射方向，前提是没有修改表面法线 o.normal |
| float3 worldRefl; <br> INTERNAL_DATA | 如果修改了表面法线 o.normal，需要使用该变量告诉 Unity 要基于修改后的法线计算世界空间下的反射方向。在表面函数中，我们需要使用 WorldReflectionVector(IN, o.normal) 来得到世界空间下的反射方向 |
| float3 worldNormal | 包含了世界空间的法线方向。前提是没有修改表面法线 o.normal |
| float3 worldNormal; <br> INTERNAL_DATA | 如果修改了表面法线 o.normal，需要使用该变量告诉 Unity 要基于修改后的法线计算世界空间下的法线方向。在表面函数中，我们需要使用 WorldNormalVector(IN, o.normal) 来得到世界空间下的法线方向 |

这些内置变量 Unity 会自动帮我们计算好，我们只需要在表面函数中直接使用它们即可。但有个例外：自定义了顶点修改函数，并需要向表面函数中传递一些自定义的数据，如：自定义雾效，需要在顶点修改函数中根据顶点在视角空间下的位置信息计算雾效混合系数，这样我们就可以在 Input 结构体中定义一个名为 half fog 的变量，把计算结果存储在该变量后进行输出。

### 表面属性：SurfaceOutput 结构体
有了 Input 结构体来提供所需要的数据后，就可以据此计算各种表面属性。**SurfaceOutput**、**SurfaceOutputStandard**、**SurfaceOutputStandardSpecular** 就是存储这些表面属性的结构体，它会作为表面函数的输出，随后会作为光照函数的输入来进行各种光照计算。

相比于 Input 结构体的自由性，这个结构体里面的变量是提前就声明好的，不可以增加也不会减少（如果没有对某些变量赋值，就会使用默认值）。

**SurfaceOutput** 的声明可以在 Lighting.cginc 文件中找到：  

    struct SurfaceOutput {
        fixed3 Albedo; //对光源的反射率，通常由纹理采样和颜色属性的乘积计算得到
        fixed3 Normal; //表面法线方向
        fixed3 Emission; //自发光，Unity 通常会在片元着色器最后输出前（并在最后的顶点函数被调用前，如果定义了的话），使用类似 c.rgb += o.Emission 语句进行简单的颜色叠加。
        half Specular; //高光反射中的指数部分的系数，影响高光反射的计算，比如：BlinnPhong 光照函数。使用如下语句计算高光反射的强度：float spec = pow(nh, s.Specular * 128.0) * s.Gloss;
        fixed Gloss; //高光反射中的强度系数，与 Specular 类似，计算公式见上面的代码，一般在包含高光反射的光照模型中使用。
        fixed Alpha; //透明通道。如果开启了透明度，会使用该值进行颜色混合。
    };

**SurfaceOutputStandard** 和 **SurfaceOutputStandardSpecular** 的声明可以在 UnityPBSLighting.cginc 中找到：  

    struct SurfaceOutputStandard {
        fixed3 Albedo; // base (diffuse or specular) color
        fixed3 Normal; // tangent space normal, if written
        half3 Emission;
        half Metallic; // 0：non-metal，1：metal
        half Smoothness; // 0：rough，1：smooth
        half Occlusion; // occlusion (default 1)
        fixed Alpha; // alpha for transparencies
    };

    struct SurfaceOutputStandardSpecular {
        fixed3 Albedo; // diffuse color
        fixed3 Specular; // specular color
        fixed3 Normal; // tangent space normal, if written
        half3 Emission;
        half Smoothness; // 0：rough，1：smooth
        half Occlusion; // occlusion (default 1)
        fixed Alpha; // alpha for transparencies
    };

在一个表面着色器内，要根据我们选择使用的光照模型选择上述之一：  
①如果使用了非基于物理的光照模型，比如 **Lambert** 和 **BlinnPhong**，通常使用 SurfaceOutput 结构体；  
②如果使用了基于物理的光照模型，比如 **Standard** 或 **StandardSpecular**，分别使用 **SurfaceOutputStandard** 或 **SurfaceOutputStandardSpecular** 结构体。其中，SurfaceOutputStandard 结构体用于默认的金属工作流程（Metallic Workflow），对应了 Standard 光照函数；而 SurfaceOutputStandardSpecular 结构体用于高光工作流程（Specular Workflow），对应了 StandardSpecular 光照函数。


## Unity 背后做了什么
Unity 在背后会根据表面着色器生成一个包含了很多 Pass 的顶点/片元着色器。这些 Pass 有些是为了针对不同的渲染路径。例如：  
①默认情况下，Unity 会为前向渲染路径生成 LightMode 为 **ForwardBase** 和 **ForwardAdd** 的 Pass；  
②为 Unity 5 之前的延迟渲染路径生成 LightMode 为 **PrePassBase** 和 **PrePassFinal** 的 Pass；  
③为 Unity 5 之后的延迟渲染路径生成 LightMode 为 **Deferred** 的 Pass；  

还有一些 Pass 是用于产生额外的信息。例如，为了给光照映射和动态全局光照提取表面信息，Unity 会生成一个 LightMode 为 **Meta** 的 Pass；有些表面着色器由于修改了顶点位置，因此，可以利用 addshadow 编译指令为它生成相应的 LightMode 为 ShadowCaster 的阴影投射 Pass。

这些 Pass 的生成都是基于表面着色器中的编译指令和自定义函数，可在每个编译完成的表面着色器的面板上，点击 “Show generated code” 按钮，查看 Unity 为这个表面着色器生成的所有顶点/片元着色器。

通过查看这些代码，以 Unity 生成的 LightMode 为 ForwardBase 的 Pass 为例，它的渲染计算流水线如下图所示：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/04/WXdUP4Obtuhpgzf.jpg" width = "100%" height = "100%" alt="图84- 表面着色器的渲染计算流水线。黄色：可以自定义的函数。灰色：Unity 自动生成的计算步骤"/>
</div>

Unity 对该 Pass 的自动生成过程大致如下：  
①复制代码  
直接将表面着色器中的 CGPROGRAM 和 ENDCG 之间的代码复制过来，这些代码包括 Input 结构体、表面函数、光照函数（如果有自定义）等变量和函数的定义，它们会在之后的处理过程中被当做正常的结构体和函数进行调用。  
②生成 v2f_surf 结构体  
Unity 会分析上述代码，据此生成顶点着色器的输出 v2f_surf 结构体，用于顶点着色器和片元着色器之间的数据传递。Unity 会分析我们在自定义函数所使用的变量，例如，纹理坐标、视角方向、反射方向等。如果需要，它就会在 v2f_surf 中生成相应的变量。而且，即便有时我们在 Input 中定义了某些变量（如某些纹理坐标），但是 Unity 在分析后续代码时发现我们并没有使用这些变量，那么这些变量实际上是不会在 v2f_surf 中生成的。也就是说，Unity 做了一些优化。v2f_surf 还包含一些其他需要的变量，如：阴影纹理坐标、光照纹理坐标、逐顶点光照等。  
③生成顶点着色器  
&emsp;&emsp; - 如果自定义了**顶点修改函数**，Unity 会首先调用它修改修改顶点数据，或填充自定义的 Input 结构体中的变量；然后，Unity 会分析顶点修改函数中修改的数据，在需要时通过 Input 结构体将修改结果存储到 v2f_surf 相应的变量中。  
&emsp;&emsp; - 计算 v2f_surf 中其他生成的变量值，包括：顶点位置、纹理坐标、法线方向、逐顶点光照、光照纹理采样坐标等，可通过编译指令来控制某些变量是否需要计算。  
&emsp;&emsp; - 最后将 v2f_surf 传递给接下来的片元着色器。  
④生成片元着色器  
&emsp;&emsp; - 使用 v2f_surf 中对应变量填充 Input 结构体，如：纹理坐标、视角方向等。  
&emsp;&emsp; - 调用自定义的**表面函数**填充 SurfaceOutput 结构体。  
&emsp;&emsp; - 调用**光照函数**得到初始颜色值。如果使用内置的 Lambert 或 BlinnPhong 光照函数，还会计算动态全局光照，并添加到光照模型的计算中。  
&emsp;&emsp; - 进行其他的颜色叠加。如：若没有使用光照烘焙，还会添加逐顶点光照的影响。  
&emsp;&emsp; - 最后，如果自定义了**最后的颜色修改函数**，Unity 会调用它进行最后的颜色修改。  


## 表面着色器实例分析
下面代码的效果是对模型进行膨胀，在顶点修改函数中沿着顶点法线方向扩张顶点位置。为了分析表面着色器中 4 个可自定义函数的原理，在下面都使用了自定义的实现：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 17/Normal Extrusion" {
	Properties {
		_ColorTint ("Color Tint", Color) = (1,1,1,1)
		_MainTex ("Base (RGB)", 2D) = "white" {}
		_BumpMap ("Normalmap", 2D) = "bump" {}
		_Amount ("Extrusion Amount", Range(-0.5, 0.5)) = 0.1
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 300
		
		CGPROGRAM
		
		// surf - which surface function.
		// CustomLambert - which lighting model to use.
		// vertex:myvert - use custom vertex modification function.
		// finalcolor:mycolor - use custom final color modification function.
		// addshadow - generate a shadow caster pass. Because we modify the vertex position, the shder needs special shadows handling.
		// exclude_path:deferred/exclude_path:prepas - do not generate passes for deferred/legacy deferred rendering path.
		// nometa - do not generate a “meta” pass (that’s used by lightmapping & dynamic global illumination to extract surface information).
		#pragma surface surf CustomLambert vertex:myvert finalcolor:mycolor addshadow exclude_path:deferred exclude_path:prepass nometa
		#pragma target 3.0
		
		fixed4 _ColorTint;
		sampler2D _MainTex;
		sampler2D _BumpMap;
		half _Amount;
		
		struct Input {
			float2 uv_MainTex;
			float2 uv_BumpMap;
		};
		
		void myvert (inout appdata_full v) {
			v.vertex.xyz += v.normal * _Amount;
		}
		
		void surf (Input IN, inout SurfaceOutput o) {
			fixed4 tex = tex2D(_MainTex, IN.uv_MainTex);
			o.Albedo = tex.rgb;
			o.Alpha = tex.a;
			o.Normal = UnpackNormal(tex2D(_BumpMap, IN.uv_BumpMap));
		}
		
		half4 LightingCustomLambert (SurfaceOutput s, half3 lightDir, half atten) {
			half NdotL = dot(s.Normal, lightDir);
			half4 c;
			c.rgb = s.Albedo * _LightColor0.rgb * (NdotL * atten);
			c.a = s.Alpha;
			return c;
		}
		
		void mycolor (Input IN, SurfaceOutput o, inout fixed4 color) {
			color *= _ColorTint;
		}
		
		ENDCG
	}
	FallBack "Legacy Shaders/Diffuse"
}
```

①在顶点修改函数中，使用顶点法线对顶点位置进行膨胀；  
②表面函数使用主纹理设置了表面属性中的反射率，并使用法线纹理设置了表面法线方向；  
③光照函数实现了简单的兰伯特漫反射光照模型；  
④最后的颜色修改函数，简单地使用颜色参数对输出颜色进行调整。

注意，除了上面四个函数外，我们在 \#pragma surface 的编译指令一行中还指定了一些额外的参数：  
①由于修改了顶点位置，为了产生正确的阴影效果，不能直接依赖 FallBack 中找到的阴影投射 Pass，使用 addshadow 参数告诉 Unity 生成一个对应当前表面着色器的阴影投射 Pass。  
②默认情况下，Unity 会为所有支持的渲染路径生成相应的 Pass，为了缩小自动生成的代码量，我们使用 **exclude_path:deferred** 和 **exclude_path:prepass** 来告诉 Unity 不要为延迟渲染路径生成相应的 Pass。  
③最后，使用 **nometa** 参数取消对提取元数据的 Pass 的生成。

--- 

点击 “Show generated code” 按钮，可看到 Unity 生成的顶点/片元着色器。在这个将近 600 行的代码里，Unity 一共为该表面着色器生成了 3 个 Pass，它们的 LightMode 分别是 ForwardBase、ForwardAdd 和 ShadowCaster，分别对应了前向渲染路径中的处理逐像素平行光的 Pass、处理其他逐像素光的 Pass、处理阴影投射的 Pass。还可以看到生成的代码中有大量的 \#ifdef 和 \#if 语句，这些语句可以帮我们判断一些渲染条件，如：是否使用动态光照纹理、是否使用逐顶点光照、是否使用屏幕空间的阴影等。

下面分析 Unity 生成的 ForwardBase Pass：  
①Unity 首先指明了一些编译指令：  

    // ---- forward rendering base pass:
    Pass {
	    Name "FORWARD"
	    Tags { "LightMode" = "ForwardBase" }

        CGPROGRAM
        // compile directives
        #pragma vertex vert_surf
        #pragma fragment frag_surf
        #pragma target 3.0
        #pragma multi_compile_instancing
        #pragma multi_compile_fwdbase
        #include "HLSLSupport.cginc"
        #define UNITY_INSTANCED_LOD_FADE
        #define UNITY_INSTANCED_SH
        #define UNITY_INSTANCED_LIGHTMAPSTS
        #define UNITY_INSTANCED_RENDERER_BOUNDS
        #include "UnityShaderVariables.cginc"
        #include "UnityShaderUtilities.cginc"

②之后出现的是一些自动生成的注释：  

    // Surface shader code generated based on:
    // vertex modifier: 'myvert'
    // writes to per-pixel normal: YES
    // writes to emission: no
    // writes to occlusion: no
    // needs world space reflection vector: no
    // needs world space normal vector: no
    // needs screen space position: no
    // needs world space position: no
    // needs view direction: no
    // needs world space view direction: no
    // needs world space position for lighting: no
    // needs world space view direction for lighting: no
    // needs world space view direction for lightmaps: no
    // needs vertex color: no
    // needs VFACE: no
    // needs SV_IsFrontFace: no
    // passes tangent-to-world matrix to pixel shader: YES
    // reads from normal: no
    // 2 texcoords actually used
    //   float2 _MainTex
    //   float2 _BumpMap

尽管这些对渲染结果没有影响，但我们可以从这些注释中理解到 Unity 的分析过程和它的分析结果。

③随后，Unity 定义了一些宏来辅助计算：  

    #define INTERNAL_DATA half3 internalSurfaceTtoW0; half3 internalSurfaceTtoW1; half3 internalSurfaceTtoW2;
    #define WorldReflectionVector(data,normal) reflect (data.worldRefl, half3(dot(data.internalSurfaceTtoW0,normal), dot(data.internalSurfaceTtoW1,normal), dot(data.internalSurfaceTtoW2,normal)))
    #define WorldNormalVector(data,normal) fixed3(dot(data.internalSurfaceTtoW0,normal), dot(data.internalSurfaceTtoW1,normal), dot(data.internalSurfaceTtoW2,normal))

在本例中这些宏没有被用到。这些宏是为了在修改了表面法线的情况下，辅助计算得到世界空间下的反射方向和法线方向，与之对应的是 Input 结构体中的一些变量。  

④接着，Unity 把我们在表面着色器中编写的 Cg 代码复制过来，作为 Pass 的一部分，以便后续调用。

⑤然后，Unity 定义了顶点着色器到片元着色器的插值结构体 v2f_surf。在定义前，Unity 使用 \#ifdef 语句来判断是否使用了光照纹理，并为不同的情况生成不同的结构体。主要区别是，如果没有使用光照纹理，就需要定义一个存储逐顶点和 SH 光照的变量。

    // vertex-to-fragment interpolation data
    // no lightmaps:
    #ifndef LIGHTMAP_ON
    // half-precision fragment shader registers:
    #ifdef UNITY_HALF_PRECISION_FRAGMENT_SHADER_REGISTERS
    struct v2f_surf {
        UNITY_POSITION(pos);
        float4 pack0 : TEXCOORD0; // _MainTex _BumpMap
        float4 tSpace0 : TEXCOORD1;
        float4 tSpace1 : TEXCOORD2;
        float4 tSpace2 : TEXCOORD3;
        fixed3 vlight : TEXCOORD4; // ambient/SH/vertexlights
        UNITY_LIGHTING_COORDS(5,6)
        #if SHADER_TARGET >= 30
        float4 lmap : TEXCOORD7;
        #endif
        UNITY_VERTEX_INPUT_INSTANCE_ID
        UNITY_VERTEX_OUTPUT_STEREO
    };
    #endif
    // with lightmaps:
    #ifdef LIGHTMAP_ON
    // half-precision fragment shader registers:
    #ifdef UNITY_HALF_PRECISION_FRAGMENT_SHADER_REGISTERS
    struct v2f_surf {
        UNITY_POSITION(pos);
        float4 pack0 : TEXCOORD0; // _MainTex _BumpMap
        float4 tSpace0 : TEXCOORD1;
        float4 tSpace1 : TEXCOORD2;
        float4 tSpace2 : TEXCOORD3;
        float4 lmap : TEXCOORD4;
        UNITY_LIGHTING_COORDS(5,6)
        UNITY_VERTEX_INPUT_INSTANCE_ID
        UNITY_VERTEX_OUTPUT_STEREO
    };
    #endif

上面很多变量，我们之前都碰到过，只是这里的名称不同。如：pack0 存储的是主纹理和法线纹理的采样坐标；tSpace0、tSpace1 和 tSpace2 存储了从切线空间到世界空间的变换矩阵。一个比较陌生的变量是 vlight，Unity 会把逐顶点和 SH 光照的结果存储到该变量里，并在片元着色器中和原光照结构进行叠加（如果需要的话）。  

⑥随后，Unity 定义了真正的顶点着色器。顶点着色器首先会调用自定义的顶点修改函数来修改一些顶点属性：  

    // vertex shader
    v2f_surf vert_surf (appdata_full v) {
        v2f_surf o;
        ...
        UNITY_INITIALIZE_OUTPUT(v2f_surf,o);
        ...
        myvert (v);
        ...

在我们的实现中，只对顶点坐标进行了修改，而不需要向 Input 结构体中添加并存储新的变量。也可以使用另一个版本的函数声明来把顶点修改函数中的某些计算结果存储到 Input 结构体当中：  

    void vert(inout appdata_full v, out Input o);

之后的代码是用于计算 v2f_surf 中的各个变量的值。例如，计算经过 MVP 矩阵变换后的顶点坐标；使用 TRANSFORM_TEX 内置宏计算两个纹理的采样坐标，并存储在 o.pack0 的 xy 和 zw 分量中：计算从切线空间到世界空间的变换矩阵，并把矩阵的每一行分别存储在 o.tSpace0、o.tSpace1 和 o.tSpace2 变量中；判断是否使用了光照映射和动态光照映射，并在需要时将两种光照纹理的采样计算结果存储在 o.lmap 的 xy 和 zw 分量中；判断是否使用了光照映射，如果没有则计算该顶点的 SH 光照（一种快速计算光照的方法），把结果存储到 o.vlight 中；判断是否开启了逐顶点光照，如果是则计算最重要的 4 个逐顶点光照的光照结果，把结果叠加到 o.vlight 中。这部分代码不摘抄了。最后，计算阴影坐标并传递给片元着色器。  

⑦在 Pass 的最后，Unity 定义了真正的片元着色器。Unity 首先利用插值后的结构体 v2f_surf 来初始化 Input 结构体中的变量。

    // fragment shader
    fixed4 frag_surf (v2f_surf IN) : SV_Target {
        ...
        // prepare and unpack data
        Input surfIN;
        ...
        UNITY_INITIALIZE_OUTPUT(Input,surfIN);
        surfIN.uv_MainTex = IN.pack0.xy;
        surfIN.uv_BumpMap = IN.pack0.zw;

随后，Unity 声明一个 SurfaceOutput 结构体的变量，并对其中的表面属性进行初始化，再调用表面函数：

    #ifdef UNITY_COMPILER_HLSL
    SurfaceOutput o = (SurfaceOutput)0;
    #else
    SurfaceOutput o;
    #endif
    o.Albedo = 0.0;
    o.Emission = 0.0;
    o.Specular = 0.0;
    o.Alpha = 0.0;
    o.Gloss = 0.0;
    fixed3 normalWorldVertex = fixed3(0,0,1);
    o.Normal = fixed3(0,0,1);

    // call surface function
    surf (surfIN, o);

上述代码中，Unity 还使用 #ifdef 语句判断当前编译语言是否是 HLSL，如果是则使用更严格的声明方式来声明 SurfaceOutput 结构体（DirectX 平台有更严格的语义要求）。当对各个表面属性进行初始化后，Unity 调用了表面函数 surf 来填充这些表面属性。  

之后，Unity 进行真正的光照计算，首先计算得到光照衰减和世界空间下的法线方向：  

    // compute lighting & shadowing factor
    UNITY_LIGHT_ATTENUATION(atten, IN, worldPos)
    fixed4 c = 0;
    float3 worldN;
    worldN.x = dot(_unity_tbn_0, o.Normal);
    worldN.y = dot(_unity_tbn_1, o.Normal);
    worldN.z = dot(_unity_tbn_2, o.Normal);
    worldN = normalize(worldN);
    o.Normal = worldN;

其中，变量 c 用于存储最终的输出颜色，初始化为 0。随后 Unity 判断是否关闭光照映射，如果关闭了，则把逐顶点的光照结果叠加到输出颜色中：  

    #ifndef LIGHTMAP_ON
    c.rgb += o.Albedo * IN.vlight;
    #endif // !LIGHTMAP_ON

而如果需要使用光照映射，Unity 就会使用之前计算的光照纹理采样坐标，对光照纹理进行采样并解码，得到光照纹理中的光照结果。这部分代码不摘抄。

如果没有使用光照映射，则需要使用自定义的光照模型计算光照结果：  

    // realtime lighting: call lighting function
    #ifndef LIGHTMAP_ON
    c += LightingCustomLambert (o, lightDir, atten);
    #else
    c.a = o.Alpha;
    #endif

而如果使用了光照映射，Unity 会根据之前由光照纹理得到的结果得到颜色值，并叠加到输出颜色 c 中；如果还开启了动态光照映射，Unity 还会计算对动态光照纹理的采样结果，同样把结果叠加到输出颜色 c 中。这部分代码不摘抄。

最后，Unity 调用自定义的颜色修改函数，对输出颜色 c 进行最后的修改：  

    mycolor (surfIN, o, c);
    UNITY_OPAQUE_ALPHA(c.a);
    return c;

在上面的代码中，Unity 还使用了内置宏 UNITY_OPAQUE_ALPHA（在 UnityCG.cginc 中被定义）来重置片元的透明通道。在默认情况下，所有不透明类型的表面着色器和透明通道都会被重置为 1.0，不管我们是否在光照函数中改变了它。如果要保留它的透明通道，可以在表面着色器的编译指令中添加 keepalpha 参数。

--- 

至此，ForwadBase Pass 分析结束，而 ForwardAdd Pass 和前者类似，只是代码更加简单，Unity 去掉了对逐顶点光照和各种判断是否使用了光照映射的代码，因为这些额外的 Pass 不需要考虑这些。

最后一个重要的 Pass 是 ShadowCaster Pass，相比之前的两个 Pass 更加短小简单，它通过调用自定义的顶点修改函数来保证计算阴影时使用的是和之前一致的顶点坐标。自定义的阴影投射 Pass 同样使用了 V2F_SHADOW_CASTER、TRANSFER_SHADOW_CASTER_NORMALOFFSET 和 SHADOW_CASTER_FRAGMENT 来计算阴影投射。

## Surface Shader 的缺点
表面着色器虽然带来了很大的便利，但是只是 Unity 在顶点/片元着色器上面提供的一种封装，任何在表面着色器中能完成的事情，都可以在顶点/片元着色器中重现，但是这句话反过来不成立。

表面着色器虽然可以快速实现各种光照效果，但是失去了对各种优化和各种特效实现的控制。因此，使用表面着色器往往会对性能产生一定的影响。而内置的 Shader，例如 Diffuse、Bumped Specular 等都是使用表面着色器编写的。尽管 Unity 提供了移动平台的相应版本，例如 Mobile/Diffuse 和 Mobile/Bumped Specular 等，但这些版本的 Shader 只是去掉了额外的逐像素 Pass、不计算全局光照和其他一些光照计算上的优化。但是想要进行更深层的优化，表面着色器就不能满足需求了。

除了性能问题外，表面着色器还无法完成一些自定义的渲染效果。

建议：  
①如果需要和各种光源打交道，尤其是 Unity 中的全局光照，使用表面着色器会更便利，但需要注意性能；  
②如果处理光源数目较少，例如只有一个平行光，使用顶点/片元着色器是更好的选择；  
③如果有很多自定义的渲染效果，建议选择顶点/片元着色器。


# 第十七章 基于物理的渲染
之前学习的 Lambert 光照模型、Phong 光照模式和 Blinn-Phong 光照模型都是经验模型。Unity 5 正式将**基于物理的渲染技术 Physically Based Shading, PBS** 引入到引擎渲染中。Unity 5 引入了一个名叫 **Standard Shader** 的可在不同材质之间通用的着色器，该着色器就是基于物理的光照模型。

## PBS 的理论和数学基础
本节主要参考了 Naty Hoffman 在 SIGGRAPH 2013 上的名为 Background：Physics and Math of Shading 的演讲。

### 光是什么
在物理学中，光是一种电磁波。材质和光线相交会发生两种物理现象：**吸收 absorption** 和**散射 scattering**（其实还有自发光现象）。  
①光线被吸收是由于光被转化成了其他能量，但吸收并不会改变光的传播方向；  
②散射则不会改变光的能量，但会改变它的传播方向。

光传播过程中，影响光的一个重要特性：材质的**折射率 refractive index**。在均匀介质中，光是沿直线传播的。但光传播过程中，如果折射率突变，会发生光的散射现象。

---

在现实中，光和物体的交互是非常复杂的，大多数情况下并不存在一种可分析的解决方法，为了在渲染中对光照建模，往往只考虑一种特殊情况：两个介质的边界是无限大且**光学平滑 optically flat** 的。尽管实际物体表面并不是无限延伸的，也不是绝对光滑的，但相比光的波长，它们的大小可以被近似认为是无限大以及光学平滑的。在这样的前提下，光在不同介质的边界会被分割成两个方向：反射方向和折射方向。而有多少百分比的光会被反射则是由**菲涅耳等式 Fresnel equations** 来描述的。

<div  align="center">  
<img src="https://s2.loli.net/2024/01/05/ibW7h28n4GSzvUH.jpg" width = "40%" height = "40%" alt="图85- 在理想的边界处，折射率的突变会把光线分成两个方向"/>
</div>

虽然相比光的波长，物体表面可以被认为是光学平坦。但物体表面是有很多肉眼不可见的凹凸不平的平面，物体表面和光照发生的各种行为更像是一系列微小的光学平滑平面和光交互的结果，每个小平面会把光分割成不同的方向。

这种建立在**微表面**的模型更容易解释物体的粗糙平滑程度，如下图所示。一个光滑物体的表面，尽管它的表面仍然由很多凹凸不平的微表面构成，但这些微表面的法线方向变化角度小，因此由这些表面反射的光线方向变化也比较小，这使得物体的高光反射更加清晰。而粗糙表面则相反，由此得到的高光反射效果更模糊。

<div  align="center">  
<img src="https://s2.loli.net/2024/01/05/c68FQLRyZspxPzB.jpg" width = "70%" height = "70%" alt="图86- 左图：光滑表面的微平面的法线变化较小，反射光线的方向变化也更小。右图：粗糙表面的微平面的法线变化较大，反射光线的方向变化也更大"/>
</div>

上面说的是微表面的反射，下面讨论被微表面折射的光。这些光被折射到物体的内部，一部分被介质吸收，另一部分又被散射到外部。金属材质具有很高的吸收系数，因此，所有被折射的光往往会被立刻吸收，被金属内部的自由电子转化为其他形式的能量（即金属几乎没有次表面散射）。而非金属材质则会同时表现出吸收和散射两种现象，而被散射到外部的光又被称为**次表面散射光 subsurface-scattered light，SSS**。如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/05/A59bZB4tIVlgRmL.jpg" width = "35%" height = "35%" alt="图87- 微表面对光的折射。这些被折射的光中一部分被吸收，一部分又被散射到外部"/>
</div>

从渲染的层级上考虑光与表面的交互行为。那么，由微表面反射的光可以被认为是该点上一些方向变化不大的反射光，如下图黄线部分所示。而折射光线（蓝线）需要更多考虑，那些次表面散射光会从不同的入射点位置从物体内部再次射出。如果像素要大于这些散射距离的话，意味着这次次表面散射产生的距离可以被忽略，如下面右图所示。如果像素要小于这些散射距离，我们就不可以忽略了，要实现更真实的次表面散射效果，需要使用特殊的渲染模型，即次表面散射渲染技术。

<div  align="center">  
<img src="https://s2.loli.net/2024/01/05/IT54o1vLn3gPZ7E.jpg" width = "70%" height = "70%" alt="图88- 次表面散射。左图：次表面散射的光线会从不同于入射点的位置射出。如果这些距离值小于需要被着色的像素大小，那么渲染就可以完全在局部完成（右图）。否则，就需要使用次表面散射渲染技术"/>
</div>


### 渲染方程
下面的内容建立在不考虑次表面散射的距离，而完全使用局部着色渲染的前提下。那么如何使用数学表达式来表示上面的光照模型。

首先，用**辐射率 radiance** 来量化光。辐射率是单位面积、单位方向上光源的辐射通量，通常用 $\,L\,$ 表示，被认为是对单一光线的亮度和颜色评估。在渲染中，我们通常会基于表面的入射光线的入射辐射率 $\,L_i\,$ 来计算出出射辐射率 $\,L_o\,$，这个过程也往往被称为是**着色 shading** 过程。

James Kajiya 在他 1986 年那篇著名的 **The Rendering Equation** 论文中给出了**渲染方程**的表述形式，由此在理论上完美地给出了基于物理渲染的方向。这个公式第一次从数学建模的角度描述了渲染到底是在解决什么问题，这个在图形学中大名鼎鼎的渲染方程公式如下：  

$$ L_o(v) = L_e(v) + \int_{\Omega} f(w_i, v) L_i(w_i)(n \cdot w_i)dw_i $$

上述公式的理解：给定观察视角 $\,v\,$ ，该方向上的出射辐射率 $\,L_o(v)\,$ 等于该点向观察方向发出的自发光辐射率 $\,L_e(v)\,$ 加上所有有效的入射光 $\,L_i(w_i)\,$ 到达观察点的辐射率积分和。下图给出了渲染等式中各个部分的通俗解释：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/05/McVnlTzp3gq7OwJ.jpg" width = "90%" height = "90%" alt="图89- 渲染方程参数的通俗解释"/>
</div>

渲染方程是计算机图形学的核心公式，当去掉其中的自发光项 $\,L_e(v)\,$ 后，剩余的部分就是著名的**反射等式 Reflectance Equation**。可以这样理解反射等式：我们要计算表面上某点的出射辐射率，已知到该点的观察方向，该点的出射辐射率是由很多不同方向的入射辐射率叠加后的结果。$\,f(w_i, v)\,$，即 **BRDF** 表达部分（详见后面），表示了不同方向的入射光在该观察方向上的权重分布，我们把这些不同方向的光辐射率（$\,L_i(w_i)\,$部分）乘以观察方向上所占的权重（$\,f(w_i, v)\,$ 部分），再乘以它们在该表面的投影结果（$\,(n \cdot w_i)\,$ 部分），最后把这些值加起来（即做积分）就是最后的出射辐射率。

在实时渲染中，自发光项通常就是直接加上某个自发光值。除此之外，积分累加部分在实时渲染中也基本无法实现，因此积分部分通常会被若干精确光源的叠加所代替，而不需要计算所有入射光线在半球面上的积分。

### 精确光源
在真实的物理世界中，所有的光源都是有面积概念的，即所谓的面光源。由于面光源的光照计算通常要耗费大量的时间，因此在实时渲染中，我们通常会使用**精确光源 punctual light sources** 来近似模拟这些面光源。图形学中常见的精确光源类型有点光源、平行光和聚光灯等，这些精确光源被认为是大小为无限小且方向确定的，尽管这并不符合真实的物理定义，但它们在大多数情况下都能得到令人满意的渲染效果。

我们使用 $\,l_c\,$ 来表示它的方向，使用 $\,c_{light}\,$ 表示它的颜色。使用精确光源，我们就可以简化上面的反射等式。即对于一个精确光源，我们可以使用下面的等式来计算它在某个观察方向 $\,v\,$ 上的出射辐射率：  

$$ L_o(v) = \pi f(l_c, v) c_{light}(n \cdot l_c) $$

上面的式子使用一个特定的方向的 $\,f(l_c,v)\,$ 值来代替积分操作，这大大简化了计算。如果场景中包含了多个精确光源，我们可以把它们分别代入上面的式子进行计算，然后把它们的结果相加即可。也就是说，反射等式可以简化成下面的形式：

$$ L_o(v) = \sum_{i=0}^n L_o^i(v) = \sum_{i=0}^n \pi f(l_c^i, v) c_{light}(n \cdot l_c^i) $$

$\,f(l_c,v)\,$ 项实际上描述了当给定某个入射方向的入射光后，有多少百分比的光照被反射到了观察方向上，即**双向反射分布函数 BRDF**。

### 双向反射分布函数 BRDF
要想得到出射辐射率 $\,L_o\,$，可以使用 **BRDF Bidirectional Reflectance Distribution Function，双向反射分布函数**来定量分析。大多数情况下，BRDF 用 $\,f(l, v)\,$ 来表示，其中 $\,l\,$ 为入射方向，$\,v\,$ 为观察方向（即双向）。绕着表面法线旋转入射方向或观察方向不影响 BRDF 的结果的情况被称为是**各项同性 isotropic** 的 BRDF；与之对应的则是**各项异性 anisotropic** 的 BRDF。

BRDF 有两种理解方式：  
①给定入射角度后，BRDF 可以给出所有出射方向上的反射和散射光线的相对分布情况；  
②给定观察方向（即出射方向）后，BRDF 可以给出从所有入射方向到该出射方向的光线分布。  

一个更直观的理解是，当一束光线沿入射方向 $\,l\,$ 到达表面某点时，$\,f(l, v)\,$ 表示了有多少部分的能量被反射到观察方向 $\,v\,$ 上。

BRDF 是反射等式中的重要组成部分，BRDF 决定了着色过程是否是基于物理的。这可以由 BRDF 是否满足两个特性来判断：它是否满足**交换律 reciprocity** 和**能量守恒 energy conservation**。

①交换律要求当交换 $\,l\,$ 和 $\,v\,$ 的值后，BRDF 的值不变，即：  

$$ f(l, v) = f(v, l) $$

②能量守恒则要求表面反射的能量不能超过入射的光能，即：  

$$ \forall l, \int_\Omega f(l, v)(n \cdot l)dw_o \le 1 $$

> $\,\forall\,$ 是全称量化符号，读作任意

基于这些理论，BRDF 可以用于描述两种不同的物理现象：表面反射和次表面散射。针对每种现象，BRDF 通常会包含一个单独的部分来描述它们，用于描述表面反射的部分被称为**高光反射项 specular term**，以及用于描述次表面散射的**漫反射项 diffuse term**。

--- 

至于如何得到不同材质的 BRDF，一种完全真实的方法是使用精确的光学仪器在真实的物理世界中对这些材质进行测量，通过不断改变光照入射方向和观察方向并对当前材质的反射光照进行采样，我们就可以得到它的 BRDF 图像切片。

一些机构和组织向公众公开了他们测量出来的 BRDF 数据库，以便供研究人员进行分析和研究，例如 MERL BRDF 数据库（ http://www.merl.com/brdf/ ）以及 MIT CSAIL 数据库（ http://people.csail.mit.edu/addy/research/brdf/ ），这些真实材质的 BRDF 数据反映了它们在不同光照和观察角度下的反射情况。

在学术界，学者们也基于复杂的物理和光学理论分析归纳出了一些通用的 BRDF 数学模型，来对 BRDF 分析模型中的高光反射项和漫反射项进行数学建模。这些由研究人员得到的 BRDF 模型也可以被称为是分析型 BRDF 模型，这些 BRDF 模型通常包含了大量的物理和光学参数。  

通过对真实材质的 BRDF 图像和现有的分析型 BRDF 模型进行对比，研究人员发现，其实很多材质是无法被现有的任何一种分析型 BRDF 模型所良好的描述出来。因此，许多最新的研究都选择使用基于数据驱动的方法，来开发出一些新的分析型 BRDF 模型，从而使其可以和真实材质的 BRDF 图像尽可能地接近。但遗憾的是，由于真实世界的光照和材质都非常复杂，因此很难有一种可以满足所有真实材质特性的 BRDF 模型。Disney 向公众开源了一个名为 **BRDF Explorer**（ http://github.com/wdas/brdf ）的软件，来让用户可以直观地对比各种分析型 BRDF 模型与真实测量得到的 BRDF 值之间的差异。

可以认为，这些测量得到的真实 BRDF 数据库是研究人员在开发新的基于物理的 BRDF 光照模型的重要依据，除了满足交换律和能量守恒两个条件外，一个分析型 BRDF 模型应当与测量得到的 BRDF 数据在尽可能多得材质范围内（或这些特定的材质）、具有尽可能得相似的表现才可能被广泛应用。Disney 通过对大量现有的分析型 BRDF 模型进行对比，并结合对真实材质的 BRDF 数据的观察，开发出了适用于 Disney 动画渲染流程的 BRDF 模型，也被称为 **Disney BRDF**。在下面的内容中，我们会介绍**漫反射项**和**高光反射项**的一些常见的 BRDF 数学模型，同时给出 Disney BRDF 模型中的相应表示。

### 漫反射项
之前所学习的 Lambert 模型就是最简单、也是应用最广泛的漫反射 BRDF。准确的 **Lambertian BRDF** 的表示为：  

$$ f_{Lambert}(l,v) = \cfrac {c_{diff}} {\pi} $$

其中，$\,c_{diff}\,$ 表示漫反射光线所占的比例，它也通常被称为是漫反射颜色 diffuse color。和之前讲的 Lambert 光照模型不太一样的是，上面的式子实际上是一个定值，我们常见到的余弦因子部分，$\,(n \cdot l)\,$ 实际是反射等式的一部分，而不是 BRDF 的部分。上面的式子之所以要除以 $\, \pi \,$，是因为我们假设漫反射在所有方向上的强度都是相同的，而 BRDF 要求在半球内的积分值为 1。因此，给定入射方向 $\,l\,$ 的光源在表面某点的出射漫反射辐射率为：

$$ L_o(v) = \pi f_{Lambert}(l, v) c_{light}(n \cdot l) = c_{diff} \times c_{light}(n \cdot l) $$

可以看出这个结果和之前一直使用的漫反射项基本一样。尽管 Lambert 模型简单且易于实现，但它具有完美均匀的散射，但真实世界中很少有材质符合。不过，一些游戏引擎和实时渲染器出于性能的考虑会使用 Lambert 模型作为它们 PBS 模型中的漫反射项，例如虚幻引擎 4（Unreal Engine 4）。

---

通过对真实材质的 BRDF 数据进行分析，研究人员发现许多材质在掠射角度表现出了明显的高光反射峰值，而且还与表面的粗糙度有着强烈的联系。粗糙表面在掠射角容易形成一条亮边，而相反地光滑表面则容易在掠射角形成一条阴影边。这些都是 Lambert 模型所无法描述的。下图显示了这样的例子，注意图中在掠射角的光照效果。

<div  align="center">  
<img src="https://s2.loli.net/2024/01/05/z5k1rVCIngwKNG4.jpg" width = "80%" height = "80%" alt="图90- 从左到右：粗糙材质和光滑材质的真实漫反射结果，以及 Lambert 漫反射结果"/>
</div>

因此，许多基于物理的渲染选择使用更加复杂的漫反射项来模拟更加真实次表面散射的结果。例如，在 **Disney BRDF** 中，它的漫反射项为：  

$$ f_{diff} (l, v) = \cfrac {baseColor} {\pi} (1 + (F_{D90} - 1)(1 - n \cdot l)^5) (1 + (F_{D90} - 1)(1 - n \cdot v)^5) $$

$$ 其中，F_{D90} = 0.5 + 2roughness(h \cdot l)^2 $$

其中，baseColor 是表面颜色，通常由纹理采样得到，roughness 是表面的粗糙度。上面的漫反射项既考虑了在掠射角漫反射项的能量变化，还考虑了表面的粗糙度对漫反射的影响。Disney 使用了 Schlick 菲涅耳近似等式来模拟在掠射角的反射变化，同时使用表面粗糙度来进一步修改它，这使得光滑材质可以在掠射角具有更为明显的阴影边，而又使得粗糙材质在掠射角具有亮边。而上面的式子也正是 Unity 5 内部使用的漫反射项。

### 高光反射项
在现实生活中，几乎所有的物体都或多或少有高光反射现象。John Hable 在他的文章中就强调了 **Everything is Shiny**。在基于物理的渲染中，BDRF 中的**高光反射项**大多数都是建立在**微面元理论 microfacet theory** 的假设上的。微面元理论认为，物体表面实际是由许多人眼看不到的微面元组成的，虽然物体表面并不是光学平滑的，但这些微面元可以被认为是光学平滑的，也就是说它们具有完美的高光反射。当光和这些微面元相交时，光线会被分割成两个方向，即反射方向和折射方向。这里我们只需要考虑被反射的光线，而折射光线已经在之前的漫反射项中考虑过了。当然，微面元理论也仅仅是真实世界的散射的一种近似理论，它也有自身的缺陷，仍然有一些材质是无法使用微面元理论来描述的。

假设表面法线为 $\,n\,$，这些微面元的法线 $\,m\,$ 并不都等于 $\,n\,$，因此，不同的微面元会把同一入射方向的光线反射到不同的方向上。而当我们计算 BRDF 时，入射方向 $\,l\,$ 和观察方向 $\,v\,$ 都会被给定，这意味着只有一部分微面元反射的光线才会进入到我们的眼睛中，这部分微面元会恰好把光线反射到方向 $\,v\,$ 上，即它们的法线 $\,m\,$ 等于 $\,l\,$ 和 $\,v\,$ 的一半，也就是我们一直看到的半角度矢量 $\,h\,$（**halfangle vector**，也被称为 **half vector**），如下图的 (a) 所示：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/06/PUrFwhC4LVujkMv.jpg" width = "90%" height = "90%" alt="图91- （a） 那些 m = h 的微面元会恰好把入射光从 l 反射到 v 上，只有这部分微面元才可以添加到 BRDF 的计算中。（b）一部分满足（a）的微面元会在 I 方向上被其他微面元遮挡住，它们不会接受到光照，因此会形成阴影。（c）还有一部分满足（a）的微面元会在反射方向 v 上被其他微面元挡住，因此，这部分反射光也不会被看到"/>
</div>

然而，这些 $\,m = h\,$ 的微面元反射也并不会全部添加到 BRDF 的计算中。这是因为，它们其中一部分会在入射方向 $\,l\,$ 上被其他微面元挡住 **shadowing**，如上图（b）所示，或是在它们的反射方向 $\,v\,$ 上被其他微面元挡住了 **masking**，如上图（c）所示。微面元理论认为，所有这些被遮挡住的微面元不会添加到高光反射项的计算中（实际上它们中的一些由于多次反射仍然会被我们看到，但这不在微面元理论的考虑范围内）。

基于微面元理论的这些假设，BRDF 的高光反射项可以用下面的通用形式来表示：  

$$ f_{spec}(l,v) = \frac {F(l,h)G(l,v,h)D(h)} {4(n \cdot l)(n \cdot v)} $$

这就是著名的 Torrance-Sparrow 微面元模型。上述公式的理解：  
①$\,D(h)\,$ 是微面元的**法线分布函数 normal distribution function，NDF**，它用于计算有多少比例的微面元的法线满足 $\,m=h\,$，只有这部分微面元才会把光线从 $\,l\,$ 方向反射到 $\,v\,$ 上。  
②$\,G(l, v, h)\,$ 是**阴影-遮掩函数 shadowing-masking function**，它用于计算那些满足 $\,m=h\,$ 的微面元中有多少会由于遮挡而不会被人眼看到，因此它给出了活跃的微面元 active microfacets 所占的浓度，只有活跃的微面元才会成功地把光线反射到观察方向上。  
③$\,F(l, h)\,$ 则是这些活跃微面元的**菲涅尔反射 Fresnel reflectance 函数**，它可以告诉我们每个活跃的微面元会把多少入射光线反射到观察方向上，即表示了反射光线占入射光线的比率。事实上，现实生活中几乎所有的物体都会表现出菲涅耳现象。  
④最后，分母 $\,4(n ⋅ l)(n⋅ v)\,$ 是用于校正从微面元的局部空间到整体宏观表面数量差异的校正因子。

这些不同的部分又可以衍生出很多不同的 BRDF 模型。首先是菲涅耳反射函数部分 $\,F(l, h)\,$。

#### 菲涅耳反射函数
菲涅耳反射函数计算了光学表面反射光线所占的部分，它表明了当光照方向和观察方向夹角逐渐增大时高光反射强度增大的现象。完整的菲涅耳等式非常复杂，包含了诸如复杂的折射率等与材质相关的参数。为了给美术人员提供更加直观且方便调节的参数，大多数 PBS 实现选择使用 **Schlick 菲涅耳近似等式**来得到近似的菲涅尔反射效果：  

$$ F_{Schlick} (l,h) = c_{spec} + (1 - c_{spec})(1 - (l \cdot h))^5 $$

其中，$\,c_{spec}\,$ 是材质的高光反射颜色。通过对真实世界材质的观察，人们发现金属材质的高光反射颜色值往往比较大，而非金属材质的反射颜色值则往往较小。

#### 法线发布函数
法线分布函数 $\,D(h)\,$ 表示了对于当前表面来说有多少比例的微面元的法线满足 $\,m=h\,$，这意味着只有这些微面元才会把光线从 $\,l\,$ 方向反射到 $\,v\,$ 上。对于大多数表面来说，微面元的法线朝向并不是均匀分布的，大部分的微面元会具有和表面法线 $\,n\,$ 相同的面法线。法线分布函数的值必须是非负的标量值，它决定了高光区域的大小、亮度和形状，因此是高光反射项中非常重要的一项。一个直观的感受是，当表面的粗糙度下降时，应该有更多的微面元的面法线满足 $\,m=n\,$，因此法线分布函数应该考虑到**表面粗糙度**的影响。  

我们之前学习的 Blinn-Phong 模型就是一种非常简单的模型。Blinn 在他的论文中改进了 Phong 模型并提出了 Blinn-Phong 模型，使它更贴合微面元 BRDF 模型的理论。**Blinn-Phong 模型**使用的法线分布函数 $\,D(h)\,$ 为：  

$$ D_{blinn} (h) = \cfrac {gloss + 2} {2 \pi} (n \cdot h)^{gloss} $$

其中，gloss 是与表面粗糙度相关的参数，它的值可以是任意非负数。上面的式子和我们之前所见的 Blinn-Phong 模型有所不同，这是因为我们在里面加入了归一化因子，这是因为法线分布函数必须满足一个条件，即所有微面元的投影面积必须等于该区域宏观表面的投影面积。因此，上述公式也被称为是**归一化的 Phong 法线分布函数**。

但实际上，Blinn-Phong 模型并不能真实地反映很多真实世界中物体的微面元法线方向分布，它其实完全是一种经验型模型，因此，很多更加复杂的分布函数被提了出来，例如 **GGX**、**Beckmann** 等。**Beckmann 分布**来源于高斯粗糙分布的一种假设，而且在表现上和 Phong 分布非常类似，但它的计算却要复杂很多。 **GGX 分布**（也被称为 **Trowbridge-Reitz 法线分布函数**）是一种更加新的法线分布函数，它的公式如下：  

$$ D_{GGX}(h) = \cfrac {\alpha ^ 2} {\pi ((\alpha ^ 2 - 1)(n \cdot h)^2 + 1)^2} $$

其中，参数 α 是与表面粗糙度相关的参数。与 Blinn-Phong 的法线分布相比，GGX 分布具有更明亮、更狭窄且拖尾更长的高光区域，它的结果更接近于一些测量得到的真实材质的 BRDF 分布。  

在 Disney BRDF 中，Disney 认为对于很多材质来说，GGX 表现出来的高光拖尾仍然不够长。他们选择使用一种更加广义的法线分布模型，即 **Generalized-Trowbridge-Reitz, GTR 分布**。GTR 分布于 GGX 分布很类似，但它的分母部分的指数不是 2，而是一个可调参数。Disney 使用两个不同指数的 GTR 分布作为两个高光反射片，其中第一个反射片用于表示基本材质层，第二个反射片用于表示基本材质表面的清漆层。除此之外，他们还发现令 $\,α = roughness^2\,$ 可以在材质粗糙度上得到更加线性的变化。否则，直接使用 roughness 作为参数的话会导致在光滑材质和粗糙材质之间插值出来的材质总是偏粗糙的。

#### 阴影-遮挡函数
阴影-遮挡函数 $\,G(l, v, h)\,$ 也被称为**几何函数 geometry function**，它表明了具有给定面法线 $\,m\,$ 的微面元在沿着入射方向 $\,l\,$ 和观察方向 $\,v\,$ 上不会被其他微面元挡住的概率。

在微面元理论的 BRDF 中，$\,m\,$ 可以使用半向量 $\,h\,$ 来代替，因为只有这部分微面元才会把光线从 $\,l\,$ 方向反射到 $\,v\,$ 上。由于 $\,G(l, v, h)\,$ 表示的是一个概率值，因此它的值是一个范围在 0 到 1 之间的标量。学术界发表了许多对于 $\,G(l, v, h)\,$ 的分析模型，这些公式大多建立在一些简化的表面模型基础下。许多已发表的微面元 BRDF 模型习惯把 $\,G(l, v, h)\,$ 和高光反射项的分母 $\,(n ⋅ l)(n ⋅ v)\,$ 部分结合起来，即把 $\,G(l, v, h)\,$ 除以 $\,(n ⋅ l)(n ⋅ v)\,$ 的部分合在一起讨论，这是因为这两个部分都和微面元的可见性有关，因此 Naty Hoffman 在他的演讲中称这个合项为**可见性项 visibility term**。

一些 BRDF 模型选择完全省略可见性项，即把该项的值设为 1。这意味着，这些 BRDF 中的 $\,G(l, v, h)\,$ 表达式等同于：  

$$ G_{implicit} (l,v,h) = (n \cdot l_c) (n \cdot v) $$

上述的 $\,G_{implicit}\,$ 实现不需要任何计算量（因为可以直接和高光反射项的分母进行抵消），并且在一些程度上可以反映正确的变化趋势。例如，当从掠射角进行观察或光线从掠射角射入时，该项会趋近于 0，这是符合我们的认知的，因为在掠射角时微面元被其他微面元遮挡的概率会非常大。然而，这种 $\,G_{implicit}\,$ 的实现忽略了材质粗糙度的影响，缺乏一定的物理真实性，因为我们希望粗糙的表面具有更高阴影和遮挡概率。

> 这里掠射角应该指入射角或出射角接近于90度时，这样余弦值为 0。

---

通常，阴影-遮挡函数 $\,G(l, v, h)\,$ 依赖于法线分布函数 $\,D(h)\,$，因为它需要结合 $\,D(h)\,$ 来保持 BRDF 能量守恒的规定。最早的阴影-遮挡函数之一是 **Cook-Torrance 阴影遮挡函数**：  

$$ G_{ct}(l,v,h) = min(1, \cfrac {2(n \cdot h)(n \cdot v)} {(v \cdot h)}, \cfrac {2(n \cdot h)(n \cdot l)} {(v \cdot h)}) $$

Cook-Torrance 阴影遮挡函数在电影行业被应用了很长时间，但它实际上是基于一个非真实的微几何模型，而且同样不受材质粗糙度的影响。后来，Kelemen 等人提出了一个对于 CookTorrance 阴影遮挡函数非常快速且有效的近似实现：  

$$ \cfrac {G_{ct}(l,v,h)} {(n \cdot l)(n \cdot v)} \approx \cfrac {1} {(l \cdot h)^2} $$

---

目前在图形学中广受推崇的是 **Smith 阴影-遮掩函数**。Smith 函数比 Cook-Torrance 函数更加精确，而且考虑进了表面粗糙度和法线分布的影响。原始的 Smith 函数是为 Beckmann 法线分布函数所涉及的，而 Walter 等人随后将其通用化，使其可以匹配任何法线分布函数，并给出了针对 Beckmann 和 GGX 法线分布函数的更加高效的**近似 Smith 模型**。在 Disney 的 BRDF 模型中，它的阴影-遮掩函数 $\,G(l, v, h)\,$ 就使用了 Walter 等人提出的为 GGX 设计的 Smith 模型：  

$$ G(l,v,h) = \cfrac {2} {1 + \sqrt{1 + \alpha_g^2 \tan \theta_v^2}} $$

$$ 其中，\alpha_g = （0.5 + \frac {roughness} {2})^2 $$

上述公式中的 $\,θ_v\,$ 表示观察方向 $\,v\,$ 和表面法线 $\,n\,$ 之间的夹角。根据艺术家的反馈以及对测量得到的 BRDF 图像的观察，Disney 在上述式子中重新映射了 $\,α_g\,$ 和 $\,roughness\,$ 之间的关系，由此得到了一个在视觉上让艺术家更加满意的效果。这种 Smith 阴影-遮掩函数被广泛应用在电影行业中，但可以看出，相比于 Kelemen 改进后的 Cook-Torrance 阴影遮挡函数，Smith 函数的计算量要明显高很多。

--- 

至此，我们已经介绍完了基于物理的 BRDF 模型的基础理论，并给出了一些常见的 BRDF 模型，例如 Phong、Beckmann、GGX 模型以及 Disney BRDF 模型的实现等。尽管存在很多基于物理的 BRDF 模型，但在真实的电影或游戏制作中，我们希望在直观性和物理可信度之间找到一个平衡点，使得实现的 BRDF 既可以让美术人员直观地调节各个参数，而又有一定的物理可信度。Disney 的 BRDF 模型就是一个很好的例子，它使用尽可能少的若干直观参数来代替那些晦涩难懂的物理变量，而且这些参数的范围被精妙地设计为 0 到 1，为此 Disney 会在必要时重新映射 BRDF 中的变量范围，例如在阴影-遮掩函数中他们重新映射了粗糙度变量的范围。

### PBS 中的光照
要想得到画面出色的渲染效果，仅仅应用以上这些公式是远远不够的，我们还需要为这些 PBS 材质搭配以出色的光照。

在上面的内容中我们已经介绍了精确光源。随着新的技术不断被提出，**实时面光源**也不再是一个奢侈的梦想。在 SIGGRAPH 2016 上，Eric Heitz 和 Stephen Hill 在他们名为 **Real-Time Area Lighting: a Journey from Research to Production** 的演讲中就分享了如何实现实时面光源的渲染，这也是 Unity 的 Adam Demo 中使用的技术，读者可以在 Unity Labs 的相关文章中找到相关内容和源码实现。除了上述两种光源外，**基于图像的光照 image-based lighting, IBL** 同样是非常重要的光照来源。

基于图像的光照通常指的是把场景中远处的光照存储在类似环境贴图的图像中。这些环境贴图可以表示光滑物体表面反射的环境光，从而允许我们可以快速得到拥有很高细节的真实光照效果。在 Unity 中，这种光照通常是由**反射探针 Reflection Probes** 机制来实现的，我们可以在 Shader 中获取当前物体所在的反射探针并在需要时对它们的采样结果进行混合。当然，我们也可以实现一套自己的 IBL 机制，Sébastien Lagarde 在他的博文里详细介绍了 IBL 的一些实现方法以及如何得到视差正确的局部环境贴图的方法，非常值得一看。  

### Unity 中的 PBS 实现
需要注意的是，随着 Unity 的不断更新，其选择使用的 BRDF 模型也可能会发生变化，例如在 Unity 5.3 之前，Unity 的 PBS 中的法线分布函数 $\,D(h)\,$ 采用的是我们之前提到的归一化的 Phong 法线分布函数，而在 Unity 5.3 及其之后的版本（截止到本书完成时为 Unity 5.6）中，法线分布函数改为采用 GGX 分布。因此，这里旨在让我们了解完整的 PBS 数学模型，如果希望了解当前所用 Unity 版本的 PBS 所使用的 BRDF 模型，强烈建议去翻看内置的 Shader 文件。  

在之前的内容中，我们提到了 Unity 5 的 PBS 实际上是受 Disney BRDF 的启发。这种 BRDF 最大的好处之一就是很直观，只需要提供一个万能的 Shader 就可以让美术人员通过调整少量参数来渲染绝大部分常见的材质。我们可以在 Unity 内置的 UnityStandardBRDF.cginc 文件中找到它的实现。

总体来说，Unity 5 一共实现了两种 PBS 模型。一种是基于 GGX 模型的，另一种则是基于归一化的 Blinn-Phong 模型的，这两种模型使用了不同的公式来计算高光反射项中的法线分布函数 $\,D(h)\,$ 和阴影-遮掩函数 $\,G(l, v, h)\,$。Unity 5.3 以前的版本默认会使用基于归一化后的 Blinn-Phong 模型来实现基于物理的渲染，但在 Unity 5.3 及后续版本中，默认将使用 GGX 模型，这和很多其他主流引擎的选择一致。

①**漫反射项：**  
在这两种实现中，Unity 使用的 BRDF 中的漫反射项都与 Disney BRDF 中的漫反射项相同，即：  


$$ f_{diff} (l, v) = \cfrac {baseColor} {\pi} (1 + (F_{D90} - 1)(1 - n \cdot l)^5) (1 + (F_{D90} - 1)(1 - n \cdot v)^5) $$

$$ 其中，F_{D90} = 0.5 + 2roughness(h \cdot l)^2 $$

baseColor 一般由纹理采样和颜色参数共同决定。  

②**菲涅耳反射函数：**  
高光反射项中的菲涅耳反射函数 $\,F(l, h)\,$ 也与 Disney BRDF 中的一致，即使用的 Schlick 菲涅耳近似等式：  

$$ F_{Schlick} (l,h) = c_{spec} + (1 - c_{spec})(1 - (l \cdot h))^5 $$

其中，$\,c_{spec}\,$ 一般由纹理采样或高光颜色所决定。

③**法线分布函数：**  
Unity 5 两种 PBS 模型的主要区别在于它们所选择的法线分布函数及其对应的阴影-遮掩函数的不同。基于 GGX 模型的 PBS 的法线分布函数为 GGX 分布，基于归一化 Blinn-Phong 模型的 PBS 的法线分布函数则为归一化后的 Phong 分布。它们的公式分别如下：  

$$ D_{GGX}(h) = \cfrac {\alpha ^ 2} {\pi ((\alpha ^ 2 - 1)(n \cdot h)^2 + 1)^2}, \alpha = {roughness}^2 $$

$$ D_{blinn} (h) = \cfrac {\alpha + 2} {2 \pi} (n \cdot h)^{\alpha}, \alpha = \cfrac {2} {roughness^4} - 2 $$

需要注意的是，不同 Unity 版本在上述实现上可能会略有不同，比如 roughness 的指数部分会有所不同。

④**Unity 5.3 之前的阴影-遮掩函数：**  
阴影-遮掩函数的选择更加复杂一些。在 Unity 5.3 之前的版本中，基于 GGX 模型和归一化 Blinn-Phong 模型的 PBS 的阴影-遮掩函数分别是为 GGX 和 Beckmann 设计的 Smith-Schlick 模型。它们的公式分别如下：  

$$ \cfrac {G_{GGX}(l,v,h)} {(n \cdot l)(n \cdot v)} = \cfrac {1} {((n \cdot l)(1 - k) + k)((n \cdot v)(1 - k) + k)}, k = \frac {roughness^2} {2} $$

$$ \cfrac {G_{Beckmann}(l,v,h)} {(n \cdot l)(n \cdot v)} = \cfrac {1} {((n \cdot l)(1 - k) + k)((n \cdot v)(1 - k) + k)}, k = roughness^2 \sqrt {\cfrac {2} {\pi}} $$

⑤**Unity 5.3 以后 GGX 的阴影-遮掩函数：**  
尽管很多文献都曾推荐使用上述的 Smith-Schlick 阴影-遮掩函数，然而，Naty Hoffman 和 Eric Heitz 都指出，这种 Smith-Schlick 阴影-遮掩函数是作者 Schlick 对一个错误版本的 Smith 模型的近似公式，这意味着这些 Smith-Schlick 阴影-遮掩函数并不是基于物理的，因为它不能保证微面元投影区域面积的守恒定律。在 Unity 5.3 及其后续版本中，Unity 为基于 GGX 的 PBS 模型改用了 Smith-Joint 阴影-遮掩函数。Smith-Joint 阴影-遮掩函数的公式如下：  

$$ \begin{align*} \cfrac {G_{SmithJoint}(l,v,h)} {(n \cdot l)(n \cdot v)} & = \cfrac {1} {(n \cdot l)(n \cdot v)(1 + \Lambda(w_o) + \Lambda(w_i))} \\ &= \cfrac {1} {(n \cdot l)(n \cdot v)(1 + 0.5(-1 + \sqrt {1 + \alpha_g^2 \tan \theta_v^2 }) + 0.5(-1 + \sqrt {1 + \alpha_g^2 \tan \theta_l^2 }))} \\ &= \cfrac {2} {(n \cdot l) \sqrt {\alpha_g^2 + (n \cdot v)^2(1 - \alpha_g^2)} + (n \cdot v) \sqrt {\alpha_g^2 + (n \cdot l)^2(1 - \alpha_g^2)}} \\ & \approx \cfrac {2} {(n \cdot l)((n \cdot v)(1 - \alpha_g) + \alpha_g)  + (n \cdot v)((n \cdot l)(1 - \alpha_g) + \alpha_g)} \end{align*} $$

$$ 其中，\alpha_g = roughness^2 $$

在上述的式子中，$\,\Lambda(w_o)\,$ 和 $\,\Lambda(w_i)\,$ 分别评估出射方向和入射方向上的阴影和遮掩，基于这种分开计算的 $\,\Lambda(w_o)\,$ 和 $\,\Lambda(w_i)\,$ 的 Smith 模型，Eric Heitz 针对 $\,\Lambda(w_o)\,$ 和 $\,\Lambda(w_i)\,$ 的不同组合方式列举了四种形式的阴影-遮掩函数。上述的公式显示了其中一种被称为 **Height-Correlated Masking and Shadowing** 的组合方式，也是 Eric 建议在实践中使用的一种方式。由于原始的 Smith-Joint 阴影遮掩函数涉及两个开根号操作，处于性能方面的考虑，Unity 在实现上选择使用上述仅包含乘法的近似公式来简化计算。尽管在数学上这个近似公式并不正确，但从效果上来看是足够接受的。

--- 

如果读者想要深入了解基于物理的渲染的数学原理和应用的话，可以参见本章的扩展阅读部分。需要再次强调的是，由于 Unity 版本的不同，内置 PBS 的实现也可能会发生变化。除此之外，在学术界和工业界仍然不断有新的或改良后的 BRDF 模型的出现，读者也可以根据项目需要选择与 Unity 实现不同的 BRDF 模型。尤其是如果需要在移动端应用基于物理的渲染，除了效果外性能是我们最应当关心的问题之一，此时我们可能需要针对移动平台对采用的 BRDF 模型进行一些修改，读者可以在本章的扩展阅读部分中找到更多的资料。


## PBS 实践
本节中，将在 Unity Shader 中实现之前提到的 BRDF 模型。读者可以发现，把 PBS 应用到自己的材质中并不是一件非常困难的事情。我们回顾使用了精确光源简化后的渲染方程：  

$$ L_o(v) = L_e(v) + \sum_{i=0}^{n} L_o^i(v) = L_e(v) + \sum_{i=0}^{n} \pi f(l_c^i,v) c_{light} (n \cdot l_c^i) $$

其中，$\,L_e(v)\,$ 是自发光部分，$\,f(l_c^i, v)\,$ 是最为关键的 BRDF 模型部分。BRDF 的高光反射项则可以用下面的通用形式来表示：  

$$ f_{spec}(l,v) = \cfrac {F(l,h)G(l,v,h)D(h)} {4(n \cdot l)(n \cdot v)} $$

在本例中，我们会使用 Disney BRDF 中的漫反射项、Schlick 菲涅耳近似等式、基于 GGX 模型的法线分布函数和 Smith-Joint 阴影-遮掩函数作为 BRDF 光照模型的实现。

### 实践
准备工作如下：  
①在 Unity 中新建一个场景，名为 Scene_18_2。我们使用本书资源中的天空盒材质 EveningSkyboxHDR，在 Window -> Lighting -> Skybox 中代替场景默认的天空盒。  
②新建两个材质，分别名为 CustomPBSCubeMat 和 CustomPBSSphereMat。  
③新建一个 Unity Shader，名为 Chapter18-CustomPBR。把它赋给第 2 步中创建的材质。  
④在场景中放置一个球体和立方体，并把第 2 步中的两个材质分别赋给两个物体。

Chapter18-CustomPBR 的 Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book v2/Chapter 18/Custom PBR" {
    Properties {
        _Color ("Color", Color) = (1, 1, 1, 1)
        _MainTex ("Albedo", 2D) = "white" {} //漫反射材质纹理
        _Glossiness ("Smoothness", Range(0.0, 1.0)) = 0.5
        _SpecularColor ("Specular", Color) = (0.2, 0.2, 0.2)
        _SpecGlossMap ("Specular (RGB) Smoothness (A)", 2D) = "white" {} //_SpecGlossMap 的 RGB 通道值用于控制材质的高光反射颜色；_SpecGlossMap 的 A 通道值和 _Glossiness 用于共同控制材质的粗糙度
        _BumpScale ("Bump Scale", Float) = 1.0
        _BumpMap ("Normal Map", 2D) = "bump" {}
        _EmissionColor ("Color", Color) = (0, 0, 0)
        _EmissionMap ("Emission", 2D) = "white" {}
    }

    SubShader {
        Tags { "RenderType"="Opaque" }
        LOD 300

        Pass {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag
            
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            #include "UnityCG.cginc"
            #include "HLSLSupport.cginc"

            //通过使用#pragma target 3.0 来指明使用 Shader Target 3.0，这是因为基于物理渲染涉及了较多的公式，因此需要较多的数学指令来进行计算，这可能会超过 Shader Target 2.0 对指令数目的规定，因此我们选择使用更高的 Shader Target 3.0。
            #pragma target 3.0

            #pragma multi_compile_fwdbase
            #pragma multi_compile_fog

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed _Glossiness;
            fixed4 _SpecularColor;
            sampler2D _SpecGlossMap;
            float4 _SpecGlossMap_ST;
            float _BumpScale;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            fixed4 _EmissionColor;
            sampler2D _EmissionMap;
            float4 _EmissionMap_ST;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                float4 texcoord : TEXCOORD0;
            };
            struct v2f {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
                float4 TtoW0 : TEXCOORD1;
                float4 TtoW1 : TEXCOORD2;
                float4 TtoW2 : TEXCOORD3;
                SHADOW_COORDS(4) // Defined in AutoLight.cginc
                UNITY_FOG_COORDS(5) // Defined in UnityCG.cginc
            };

            //选择使用 Disney BRDF 中的漫反射项实现，CustomDisneyDiffuseTerm 函数的实现如下。UNITY_INV_PI 是在 UnityCG.cginc 文件中定义的宏变量，即圆周率 π 的倒数。在上面的实现中，我们还使用了 Cg 关键词 inline 来修饰函数声明，inline 的作用是用于告诉编译器应该尽可能使用内联调用的方式来调用该函数，减少函数调用的开销。  
            inline half3 CustomDisneyDiffuseTerm(half NdotV, half NdotL, half LdotH, half roughness, half3 baseColor) {
                half fd90 = 0.5 + 2 * LdotH * LdotH * roughness;
                // Two schlick fresnel term
                half lightScatter = (1 + (fd90 - 1) * pow(1 - NdotL, 5));
                half viewScatter = (1 + (fd90 - 1) * pow(1 - NdotV, 5));

                return baseColor * UNITY_INV_PI * lightScatter * viewScatter;
            }

            //可见性项 V，它计算的是阴影-遮掩函数除以高光反射项的分母部分后的结果。CustomSmithJointGGXVisibilityTerm 函数的实现如下：  
            inline half CustomSmithJointGGXVisibilityTerm(half NdotL, half NdotV, half roughness) {
                // Original formulation:
                // lambda_v = (-1 + sqrt(a2 * (1 - NdotL2) / NdotL2 + 1)) * 0.5f;
                // lambda_l = (-1 + sqrt(a2 * (1 - NdotV2) / NdotV2 + 1)) * 0.5f;
                // G = 1 / (1 + lambda_v + lambda_l);

                // Approximation of the above formulation
                half a = roughness * roughness;
                half lambdaV = NdotL * (NdotV * (1 - a) + a);
                half lambdaL = NdotV * (NdotL * (1 - a) + a);

                return 0.5f / (lambdaV + lambdaL + 1e-5f);
            }

            //法线分布项 D，CustomGGXTerm 函数的实现如下：  
            inline half CustomGGXTerm(half NdotH, half roughness) {
                half a = roughness * roughness;
                half a2 = a * a;
                half d = (a2 - 1.0f) * NdotH * NdotH + 1.0f;
                return UNITY_INV_PI * a2 / (d * d + 1e-7f);
            }

            //菲涅耳反射项 F，CustomFresnelTerm 函数的实现如下：  
            inline half3 CustomFresnelTerm(half3 c, half cosA) {
                half t = pow(1 - cosA, 5);
                return c + (1 - c) * t;
            }

            //菲涅耳插值 CustomFresnelLerp 的函数实现如下：  
            inline half3 CustomFresnelLerp(half3 c0, half3 c1, half cosA) {
                half t = pow(1 - cosA, 5);
                return lerp (c0, c1, t);
            }
            
            v2f vert(a2v v) {
                v2f o;
                UNITY_INITIALIZE_OUTPUT(v2f, o); // Defined in HLSLSupport.cginc

                o.pos = UnityObjectToClipPos(v.vertex); // Defined in UnityCG.cginc
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex); // Defined in UnityCG.cginc

                float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
                fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w;

                o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);

                //We need this for shadow receving
                TRANSFER_SHADOW(o); // Defined in AutoLight.cginc
                //We need this for fog rendering
                UNITY_TRANSFER_FOG(o, o.pos); // Defined in UnityCG.cginc
                return o;
            }

            half4 frag(v2f i) : SV_Target {
                /////首先需要为后续计算准备好所有的输入数据，这些输入大多来源于材质面板中的各个属性，例如漫反射颜色 diffColor 和高光反射颜色 specColor、粗糙度 roughness、世界空间下的法线方向、光源方向、观察方向、反射方向等。我们还使用内置宏 UNITY_LIGHT_ATTENUATION 计算了阴影和光照衰减值 atten。除此之外，我们还计算了一个变量 oneMinusReflectivity，这个变量并不是我们之前提到的 BRDF 中需要的变量，它主要是为了计算掠射角的反射颜色，从而得到效果更好的菲涅耳反射效果。
                half4 specGloss = tex2D(_SpecGlossMap, i.uv);
                specGloss.a *= _Glossiness;
                half3 specColor = specGloss.rgb * _SpecularColor.rgb;
                half roughness = 1 - specGloss.a;

                half oneMinusReflectivity = 1 - max(max(specColor.r, specColor.g), specColor.b);

                half3 diffColor = _Color.rgb * tex2D(_MainTex, i.uv).rgb * oneMinusReflectivity;

                half3 normalTangent = UnpackNormal(tex2D(_BumpMap, i.uv));
                normalTangent.xy *= _BumpScale;
                normalTangent.z = sqrt(1.0 - saturate(dot(normalTangent.xy, normalTangent.xy)));
                half3 normalWorld = normalize(half3(dot(i.TtoW0.xyz, normalTangent), dot(i.TtoW1.xyz, normalTangent), dot(i.TtoW2.xyz, normalTangent)));
                
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                half3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos)); // Defined in UnityCG.cginc
                half3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos)); // Defined in UnityCG.cginc

                half3 reflDir = reflect(-viewDir, normalWorld);

                UNITY_LIGHT_ATTENUATION(atten, i, worldPos); // Defined in AutoLight.cginc

                /////接下来开始计算最重要的 BRDF 光照模型。在此之前，我们先准备好各个角度的余弦值，即之前公式中的各个点乘项。通过使用 Cg 的 saturate 函数，我们把这些点乘值的范围截取到了[0, 1]之间，来避免背光面的光照
                half3 halfDir = normalize(lightDir + viewDir);
                half nv = saturate(dot(normalWorld, viewDir));
                half nl = saturate(dot(normalWorld, lightDir));
                half nh = saturate(dot(normalWorld, halfDir));
                half lv = saturate(dot(lightDir, viewDir));
                half lh = saturate(dot(lightDir, halfDir));

                //计算 BRDF 中的漫反射项：  
                half3 diffuseTerm = CustomDisneyDiffuseTerm(nv, nl, lh, roughness, diffColor);

                //计算高光反射项：
                half V = CustomSmithJointGGXVisibilityTerm(nl, nv, roughness);
                half D = CustomGGXTerm(nh, roughness * roughness);
                half3 F = CustomFresnelTerm(specColor, lh);
                half3 specularTerm = F * V * D;

                //计算自发光项：
                half3 emisstionTerm = tex2D(_EmissionMap, i.uv).rgb * _EmissionColor.rgb;

                //为了得到更加真实的光照，还需要计算基于图像的光照部分（IBL），详见补充：  
                half perceptualRoughness = roughness * (1.7 - 0.7 * roughness);
                half mip = perceptualRoughness * 6;
                half4 envMap = UNITY_SAMPLE_TEXCUBE_LOD(unity_SpecCube0, reflDir, mip); // Defined in HLSLSupport.cginc
                half3 decodeEnvMap = DecodeHDR(envMap, unity_SpecCube0_HDR);  //Decode the 4 channels HDR data to RGB format. Otherwise the indirectLight will be too bright because the cube map contains high dynamic range colors, which allows the values greater than 1.
                
                half grazingTerm = saturate((1 - roughness) + (1 - oneMinusReflectivity));
                half surfaceReduction = 1.0 / (roughness * roughness + 1.0);
                half3 indirectSpecular = surfaceReduction * decodeEnvMap.rgb * CustomFresnelLerp(specColor, grazingTerm, nv);

                //最后按照渲染方程把所有项加起来：
                half3 col = emisstionTerm + UNITY_PI * (diffuseTerm + specularTerm) * _LightColor0.rgb * nl * atten + indirectSpecular;

                UNITY_APPLY_FOG(i.fogCoord, c.rgb); // Defined in UnityCG.cginc

                return half4(col, 1);
                }
            ENDCG
        }
    }
    Fallback Off
}
```

关于 IBL 部分的补充：IBL 部分的主要思想是使用材质粗糙度对环境贴图进行 LOD，Level Of Detail 采样，这是因为粗糙度越大的材质，反射的环境光照应该越模糊，而这可以通过对环境贴图不同级数的多级渐远纹理 mipmaps 进行采样来模拟得到。级数越高，在多级渐远纹理中对应的纹理就越小，图像也就越模糊。

为了计算需要采样的多级渐远纹理的级数，我们将材质粗糙度乘以某个常数（在上述实现中该常数为 6），这个常数表明了整个粗糙度范围内多级渐远纹理的总级数。需要注意的是，这种由粗糙度计算级数的方法并不是唯一的，读者可以在 UnityImageBasedLighting.cginc 文件的 perceptualRoughnessToMipmapLevel 函数中找到相关实现。然后，我们使用该级数和反射方向来对环境贴图进行采样。其中，unity_SpecCube0 包含了该物体周围当前活跃的反射探针 Reflection Probe 中所包含的环境贴图。尽管我们没有在场景中手动放置任何反射探针，但 Unity 会根据 Window -> Lighting -> Skybox 中的设置，在场景中生成一个默认的反射探针。由于在本节的准备工作中我们在 Window -> Lighting -> Skybox 中设置了自定义的天空盒，因此此时 unity_SpecCube0 中包含的就是这个自定义天空盒的环境贴图。如果我们在场景中放置了其他反射探针，Unity 则会根据相关设置和物体所在的位置自动把距离该物体最近的一个或几个反射探针数据传递给 Shader。尽管在之前的内容中，我们是使用 samplerCUBE 来声明一个立方体贴图并使用 texCUBE 来采样它，但是 Unity 内置反射探针的立方体贴图则是以一种特殊的方式声明的，这主要是为了在某些平台下可以节省 sampler slots。读者可以在 UnityShaderVariables.cginc 文件中找到 unity_SpecCube0 的声明，Unity 主要是通过 HLSLSupport.cginc 文件中定义的内置宏 UNITY_DECLARE_TEXCUBE 来实现的。 由于这样的特殊性， 在采样 unity_SpecCube0 时我们也应该使用内置宏如 UNITY_SAMPLE_TEXCUBE（在 HLSLSupport.cginc 文件中被定义）来采样。由于在这里我们还需要对指定级数的多级渐远纹理采样，因此我们使用内置宏 UNITY_SAMPLE_TEXCUBE_LOD（在 HLSLSupport.cginc 文件中被定义）来实现。至此，我们得到了采样后的环境光照颜色 envMap。

对环境光采样后变量 envMap 为 HDR 值 RGBM，直接使用会导致材质过曝，因此添加了 DecodeHDR 函数将 HDR 转换为 RGB。

然后，为了给 IBL 添加更加真实的菲涅耳反射，我们对高光反射颜色 specColor 和掠射颜色 grazingTerm 进行菲涅耳插值。掠射颜色 grazingTerm 是由材质粗糙度和之前计算得到的 oneMinusReflectivity 共同决定的。使用掠射角度进行菲涅耳插值的好处是，我们可以在掠射角得到更加真实的菲涅耳反射效果，同时还考虑了材质粗糙度的影响。除此之外，我们还使用了由粗糙度计算得到的 surfaceReduction 参数进一步对 IBL 的进行修正。

CustomFresnelLerp 的函数的实现和之前实现的 CustomFresnelTerm 函数很类似，不同的是这里使用参数 t 来混合两个颜色。尽管 grazingTerm 被声明为单一维数的 half 变量，在传递给 CustomFresnelLerp 时它会自动被转换成 half3 类型的变量，这在 Cg 中被称为是 **"Smearing" Of Scalars To Vectors**。

---

若场景中存在多个光源，需要实现 Forward Add Pass。Forward Add Pass 的实现与 Forward Base Pass 基本一致，其中不同的是，Forward Add Pass 不需要计算雾效、自发光和 IBL 的部分，因为这些只需要在 Forward Base Pass 计算一遍即可。其他实现在此不再赘述。

保存后返回场景，再调整相关参数，Color 为黑色，Smoothness 为 0.75，Specular 为白色，不添加任何贴图的效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/07/duosjItMTSVGU9P.jpg" width = "80%" height = "80%" alt="图92- 使用自定义的基于物理渲染的材质"/>
</div>

需要注意的是，我们还需要保证 Edit -> Project Settings -> Player -> Color Space 中的选项是 Linear，即线性空间，只有这样才能保证我们的计算是在线性空间下进行的，且输出的为线性颜色。与线性空间相关的是**伽马校正**，详见后面。

在上面的内容中，我们依靠自定义的函数实现了一个基于 GGX BRDF 模型的 Shader。实际上，Unity 已经帮我们实现了很多 BRDF 模型中的函数，并为我们提供了现成的基于物理着色的 Shader，也就是 **Standard Shader**。


## Unity 5 的 Standard Shader
在 Unity 5 中新创建一个模型或是新创建一个材质时，其默认使用的着色器都是一个名为 Standard 的着色器。Unity 支持两种流行的基于物理的工作流程：**金属工作流 metallic workflow** 和**高光反射工作流 specular workflow**。其中，金属工作流是默认的工作流程，对应的 Shader 为 Standard Shader。而如果想要使用高光反射工作流，就需要在材质的 Shader 下拉框中选择 Standard（Specular setup）。在内部实现上，这两种工作流实际上最终都会使用同一套 BRDF 模型，不同的是 BRDF 模型中各个输入参数的来源不同而已。

### 它们是如何实现的
Standard 和 Standard（Specular setup）的 Shader 源代码可以在 Unity 内置的 builtin_shaders-XXXXXX/DefaultResourcesExtra 文件夹中找到，这些 Shader 依赖于 builtin_shaders-XXXXXX/CGIncludes 文件夹中定义的一些头文件。这些相关的头文件的名称大多类似于 UnityStandardXXX.cginc，其中定义了和 PBS 相关的各个函数、结构体和宏等。下表列出了这些头文件的名称以及它们的主要用处：  

| 文件 | 描述 |
| :---- | :---- |
| UnityStandardInput.cginc | 声明了 Standard 和 Standard (Specular setup) Shader 使用的所有材质参数（如 _Color、_MainTex、_EmissionMap），定义了顶点着色器的输入结构体 VertexInput，还定义了相关辅助函数用于从这些输入中计算得到相关的材质变量，例如 Albedo 函数可以从 _MainTex 和 _Color 参数中计算得到漫反射颜色，Occlusion 函数可以从 _OcclusionMap 中计算得到遮挡值 |
| UnityLightingCommon.cginc | 定义了和 PBS 光照相关的各个结构体，例如 UnityLight、UnityIndirect、UnityGI 和 UnityGIInput 等 |
| UnityStandardCore.cginc | 定义了 Standard 和 Standard (Specular setup) Shader 使用的顶点/片元着色器（ 如 vertForwardBase 和 fragForwardBase ）、相关的结构体 （ 如 VertexOutputForwardBase 和 FragmentCommonData ） 和辅助函数（如 MetallicSetup、SpecularSetup、MainLight、PerPixelWorldNormal、FragmentGI）等 |
| UnityStandardCoreForwardSimple.cginc | 定义了简化版的顶点/片元着色器（ 如 vertForwardBaseSimple 和 fragForwardBaseSimple）、相关的结构体（如 VertexOutputBaseSimple）和辅助函数（如 FragmentSetupSimple、MainLightSimple）等。在默认情况下，Unity 会将 UNITY_STANDARD_SIMPLE（在 UnityStandardConfig.cginc 文件中被定义）设为 0，即不使用这些简化后的实现 |
| UnityPBSLighting.cginc | 定义了表面着色器使用的标准光照函数和相关的结构体等，如 LightingStandardSpecular 函数和 SurfaceOutputStandardSpecular 结构体，这些定义主要是为 Unity 表面着色器（Surface Shader）服务的。除此之外，还定义 PBS 函数的调用宏 UNITY_BRDF_PBS，Unity 会根据当前平台设置等为 UNITY_BRDF_PBS 设置不同性能的函数入口，如 BRDF1_Unity_PBS、BRDF2_Unity_PBS 和 BRDF3_Unity_PBS 等函数，而这些函数是在 UnityStandardBRDF.cginc 文件中被定义的 |
| UnityStandardBRDF.cginc | 实现了 Unity 中基于物理的渲染技术，定义了 BRDF1_Unity_PBS、BRDF2_Unity_PBS 和 BRDF3_Unity_PBS 等函数，来实现不同平台下的 BRDF。这个文件包含了关键的 BRDF 模型的实现部分，包括漫反射项和高光反射项的函数实现（如 DisneyDiffuse、SmithJointGGXVisibilityTerm）和相关的辅助函数（如 常用的数学函数 Pow4 和 Pow5 、 PerceptualRoughnessToSpecPower） |
| UnityGlobalIllumination.cginc | 定义了计算全局光照的 UnityGlobalIllumination 函数，该函数在 FragmentGI 函数（在 UnityStandardCore.cginc 中被定义）中被调用，它会从光照贴图、光照探针、反射探针等输入中读取数据，计算全局光照中的间接漫反射颜色和高光反射颜色，并存储 UnityGI 结构体中的 UnityIndirect 结构体变量中（两个结构体均在 UnityLightingCommon.cginc 中被定义） |
| UnityImageBasedLighting.cginc | 定义了和基于图像的光照相关的结构体和函数，这些结构体和函数会在计算全局光照时被使用，例如 Unity_GlossyEnvironment 函数会采样反射探针中的数据，它可以用于计算间接的高光反射 |
| UnityStandardUtils.cginc | Standard Shader 使用的一些辅助函数，将来可能会移到 UnityCG.cginc 文件中 |
| UnityStandardConfig.cginc |  Standard Shader 的相关配置，例如默认情况下使用 GGX 模型来实现 BRDF（将 UNITY_BRDF_GGX 设为 1） |
| UnityStandardMeta.cginc | 定义了 Standard Shader 中 “LightMode” 为 “Meta” 的 Pass（用于提取光照纹理和全局光照的相关信息）使用的顶点/片元着色器，以及它们使用的输入/输出结构体 |
| UnityStandardShadow.cginc | 定义了 Standard Shader 中 “LightMode” 为 “ShadowCaster” 的 Pass（用于投射阴影）使用的顶点/片元着色器，以及它们使用的输入/输出结构体 |

Standard.shader 和 StandardSpecular.shader 的代码分析如下：  
①总体来讲，这两个 Shader 的代码基本相同—它们都定义了两个 SubShader，第一个 SubShader 使用的计算更加复杂，Unity 为其定义了前向渲染路径和延迟渲染路径使用的 Pass，以及用于投射阴影和提取元数据的 Pass；第二个 SubShader 也定义了 4 个 Pass，其中两个 Pass 用于前向渲染路径，一个 Pass 用于投射阴影，另一个 Pass 用于提取元数据，该 SubShader 和第一个 SubShader 的主要区别在于取消了一些计算，例如不计算视差贴图、不计算软阴影等。  
②它们最大的不同之处在于，它们在设置 BRDF 的输入时使用了不同的函数来设置各个参数 — 基于金属工作流的 Standard Shader 使用 MetallicSetup 函数来设置各个参数，基于高光反射工作流的 Standard（Specular setup）Shader 使用 SpecularSetup 函数来设置。MetallicSetup 和 SpecularSetup 函数均在 UnityStandardCore.cginc 文件中被定义。

下图出了 Standard Shader 中用于前向渲染路径的典型实现，这是由对内置文件的分析所得：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/08/v3fX5dsCAmJiOHw.jpg" width = "100%" height = "100%" alt="图93- Standard Shader 中前向渲染路径使用的 Pass（简化版本的 PBS 使用了 VertexOutputBaseSimple 等结构体来代替相应的结构体）"/>
</div>

### 如何使用 Standard Shader
金属材质和非金属材质的漫反射和高光反射的一些特点：  
①金属材质  
&emsp;&emsp; - 几乎没有漫反射，因为所有被吸收的光都会被自由电子立刻转化为其他形式的能量；  
&emsp;&emsp; - 有非常强烈的高光反射；  
&emsp;&emsp; - 高光反射通常是有颜色的，例如金子的反光颜色为黄色。  
②非金属材质  
&emsp;&emsp; - 大多数角度高光反射的强度比较弱，但在掠射角时高光反射强度反而会增强，即菲涅耳现象；  
&emsp;&emsp; - 高光反射的颜色比较单一；  
&emsp;&emsp; - 漫反射的颜色多种多样。

在 Unity 官方提供的示例项目 **Shader Calibration Scene** 中，Unity 提供了两个非常有参考价值的校准表格，如下图所示，它们分别对应了金属工作流和高光反射工作流使用的参考属性值，来方便我们针对不同类型的材质来调整参数。

<table><tr>
<td><img src='https://s2.loli.net/2024/01/08/IrVKbMOlyYTFfoh.png' width="400" alt="图94- 金属工作流使用的校准表格"></td>
<td><img src='https://s2.loli.net/2024/01/08/E14Qf7imDrz3xkw.png' width="400" alt="图95- 高光反射工作流使用的校准表格"></td>
</tr></table>

需要注意的是，为了保证数学上准确，光照计算应该在线性空间执行。在使用标准纹理时，需要在 Edit -> Project Settings -> Player -> Color Space 中选择 Linear 线性空间，这是因为基于物理的渲染需要使用线性空间来进行相关计算。

在金属工作流中，材质面板中的 **Albedo** 定义了物体的整体颜色，它通常就是我们视觉上认为的物体颜色。从亮度来看，非金属材质的亮度范围通常在 50～243，而金属材质的亮度一般在 186~255 之间。材质面板中的下一个属性是 **Metallic**，它定义了该物体表面看起来是否更像金属或非金属。同样，我们也可以使用一张纹理来采样得到表面的 Metallic 值，此时该纹理中的 R 通道值将对应了 Metallic 值。最后一个重要的材质属性是 **Smoothness**，它是上一个属性 Metallic 的附属值，定义了从视觉上来看该表面的光滑程度。如果我们在设置 Metallic 属性时使用的是一张纹理，那么这张纹理的 A 通道就对应了表面的 Smoothness 值（此时纹理的 GB 通道则被忽略）。

高光反射工作流，使用了不同含义的 **Albedo** 属性，并使用 **Specular** 代替了上述的 **Metallic** 属性。在高光反射工作流中，材质的 **Albedo** 属性定义了表面的漫反射强度。对于非金属材质，它的值通常仍然是视觉上认为的物体颜色，但对于金属材质，Albedo 的值通常非常接近黑色（还记得吗，金属材质几乎不存在次表面散射的现象）。高光反射工作流的 **Specular** 属性则定义了表面的高光反射强度。非金属材质通常使用一个灰度值范围在 0～55 的深灰色来作为 Specular 值，表明非金属材质的高光反射较弱。金属材质则通常会使用视觉上认为的该金属的颜色作为它的 Specular 值。**Specular** 属性同样也有一个子属性 **Smoothness**，它定义了从视觉上来看该表面的光滑程度。和上面的金属工作流类似，如果使用了一张纹理来为 Specular 属性赋值，那么纹理的 RGB 通道对应了 Specular 属性值，A 通道对应了 Smoothness 属性值。

上述材质属性都属于材质面板中的 **Main Maps** 部分，除了上述提到的属性外，Main Maps 还包含了其他材质属性，例如，切线空间下的法线纹理、遮挡纹理、自发光纹理等。Main Maps 部分的下面还有一个 **Secondary Maps** 的属性部分，这个部分的属性是用来定义额外的细节信息，这些细节通常会直接绘制在 Main Maps 的上面，来为材质提供更多的微表面或细节表现。

除了上述属性，我们还可以为 Standard Shader 选择它使用的渲染模式，即材质面板上的 **Render Mode** 选项。Standard Shader 支持 4 种渲染模式，分别是 Opaque、Cutout、Fade 和 Transparent。其中，**Opaque** 用于渲染最常见的不透明物体，这也是默认的渲染模式。对于像玻璃这样的材质，我们可以选择 **Transparent** 模式，在这个渲染模式下，Albedo 属性的 A 通道用于控制材质的透明度。而在 **Cutout** 渲染模式下，Albedo 属性中纹理的 A 通道会成为一个遮掩纹理，而它的子属性 Alpha Cutoff 将是透明度测试时使用的阈值。**Fade** 模式和 Transparent 模式是类似的，不同的是，在 Transparent 模式下，当材质的透明值不断降低时，它的反射仍然能被保留，而在 Fade 模式下，该材质的所有渲染效果都会逐渐从屏幕上淡出。

需要注意的是，尽管 Standard Shader 的材质面板有许多可供调节的属性，但我们不用担心由于没有使用一些属性而会对性能有所影响。Unity 在背后已经进行了高度优化，在我们生成可执行程序时，Unity 会检查哪些属性没有被使用到，同时也会针对目标平台进行相应的优化。这些逻辑是通过使用一个 C# 脚本文件来自定义材质面板的行为来实现的，读者可以在 builtin_shaders-XXXXX/ Editor/ StandardShaderGUI.cs 中找到这个文件。这个脚本的核心思想是通过判断用户是否设置了某一材质属性来决定是否开启相应的 shader feature，例如，如果用户没有设置法线纹理，那么该脚本就会关闭名为 _NORMALMAP 的 shader feature，从而在 Shader 逻辑中跳过相关代码。

想要让整个场景的渲染结果令人满意，尤其包含了复杂光照的场景，仅仅有这些使用了 PBS 的材质是不够的，我们需要使用 Unity 提供的其他一些重要的技术，例如 HDR 格式的 Skybox、全局光照、反射探针、光照探针、HDR 和屏幕后处理等。更多内容读者阅读本章的扩展阅读部分。


## 一个更加复杂的例子
场景对应了本书资源中的 Scene_18_3。本场景使用的元素大多来源于 Unity 官方的示例项目 **Viking Village**。别忘了先把色彩空间改为线性空间。下面不再放出图片，了解原理即可。

### 设置光照环境
首先需要为场景设置光照环境，在默认情况下，Unity 5 中一个新创建的场景会包含一个默认的 Skybox。在本例中，我们使用一个自定义的 Skybox 来代替默认值，使用的是 SunsetSkyboxHDR。

本例中的 Skybox 使用了一个 **HDR** 格式的 Cubemap，后面会详细讲解 HDR，这里，只需要知道，使用 HDR 格式的 Skybox 可以让场景中物体的反射更加真实，有利于我们得到更加可信的光照效果。

我们还可以设置场景使用的**环境光照**，这些环境光照可以对场景中所有的物体表面产生影响。在 Lighting -> Environment 的设置面板中，我们可以选择环境光照的来源（Ambient Source 选项，目前版本叫 **Environment Lighting Source** 属性），是来自于场景使用的 Skybox，还是使用渐变值，亦或是某个固定的颜色。我们还可以设置环境光照的强度（Ambient Intensity 参数，目前版本叫 **Environment Lighting Intensity Multiplier** 属性），如果想要场景中的所有物体不接受任何环境光照，可以把该值设为 0。在使用了 Standard Shader 的前提下，如果我们关闭场景中所有的光源，并把环境光照的强度设为 0，场景中的物体仍然可以接受一些光照。图片这里不放出了，总之还有物体接受到的**反射**光。

默认的反射源（Reflection Source 选项，目前版本叫 **Environment Reflections Source**）是场景使用的 Skybox。可以把反射源设置为自定义，即 Custom。在渲染实现上，即便场景中没有任何光源，Unity 在内部仍然会调用 ForwardBase Pass（假设使用的是前向渲染路径的话），并使用反射的光照信息来填充光源信息，再进行基于物理的渲染计算。读者可以通过帧调试器 Frame Debugger 来查看渲染过程。需要注意的是，这里设置的反射源是默认的反射源，如果我们在场景中添加了其他反射探针 Reflection Probes，物体可能会使用其他反射源。当默认反射源是 Skybox 时，Unity 会由场景使用的 Skybox 生成一个 Cubemap，我们可以通过 **Resolution** 选项来控制它每个面的分辨率。

--- 

除了 Standard Shader 外，Unity 还引入了一个重要的流水线 - **实时全局光照 Global Illumination，GI** 流水线。使用 GI，场景中的物体不仅可以受直接光照的影响，还可以接受间接光照的影响。直接光照指的是那些直接把光照射到物体表面的光源，在本书之前的章节中，我们使用的都是直接光照来渲染场景中的物体。但在现实生活中，物体还会受到间接光照的影响。例如，想象一个红色墙壁旁边放置了一个球体，尽管墙壁本身不发光，但球体靠近墙的一面仍会有少许的红色，这是由于红色墙壁把一些间接光照投射到了球体上。在 Unity 中，间接光照指的就是那些被场景中其他物体反弹的光，这些间接光照会受反弹光的表面的颜色影响（例如之前例子中的红色的墙壁），这些表面会在反弹光线时把自身表面的颜色添加到反射光的计算中。在 Unity 5 中，我们可以使用这些直接光照和间接光照来创建更加真实的视觉效果。

---

首先设置场景使用的**直接光照** - 一个平行光。为了让渲染效果更加真实可信，我们需要保证平行光的方向和 Skybox 中的太阳或其他光源的位置一致，使得物体产生的光照信息可以与 Skybox 互相吻合。有时，我们可能会使用一张**耀斑纹理 Flare Texture** 来模拟太阳等光源，此时我们同样需要确保平行光的方向与耀斑纹理的位置一致。与之类似的还有平行光的颜色，我们应该尽量让平行光的颜色和场景环境相匹配。若场景的光照环境为日落时分，平行光的颜色可为浅黄色；若场景的光照环境更接近傍晚，此时平行光的颜色为淡蓝色。

在平行光面板的 **Mode** 选项中，选择 **Realtime** 模式，这意味着，场景中受平行光影响的所有物体都会进行实时的光照计算，当光源或场景中其他物体的位置、旋转角度等发生变化时，场景中的光照结果也会随之变化。然而，实时光照往往需要较大的性能消耗，对于移动平台这样资源比较短缺的平台，我们可以选择 **Baked** 模式，此时，Unity 会把该光源的光照效果烘焙到一张**光照纹理 lightmap** 中，这样我们就不用实时为物体计算复杂的光照，而只需要通过纹理采样来得到光照结果。选择烘焙模式的缺点在于，如果场景中的物体发生了移动，但是它的阴影等光照效果并不会发生变化。**Mix** 模式则允许我们混合使用实时模式和烘焙模式，它会把场景中的静态物体（即那些被标识为 **Static** 的物体）的光照烘焙到光照纹理中，但仍然会对动态物体产生实时光照。

Unity 5 引入了实时间接光照的功能，在这个系统下，场景中的直接光照会在场景中各个物体之间来回反射，产生**间接光照**。当一条光线从光源被发射出来后，它会与场景中的一些物体相交，第一个和光线相交的物体受到的光照即为直接光照。当得到直接光照在该物体上的光照结果后，该物体还会继续反射该光线，从而对其他物体产生间接光照。此后与该光线相交的物体，就会受到间接光照的影响，同时它们也会继续反射。当经过多次反射后，该光线最后完全消失。这些间接光照的强度是由 GI 系统计算得到的默认亮度值。调整 Lighting -> Environment -> Environment Reflections **Bounces** 参数可以调节这些间接光照的强度。当我们把它设为 1 时，意味着一条光线仅会和一个物体相交，不再被继续反射，也就是说，场景中的物体只会受到直接光照的影响。需要注意的是，间接光照还有可能来自一些自发光的物体。

### 放置反射探针
在实时渲染中，我们经常会使用 Cubemap 来模拟物体的反射效果。然而，如果我们永远使用同一个 Cubemap，当移动物体周围的场景发生较大变化时，就很容易出现“穿帮镜头”，因为物体的环境反射并没有随着环境变化而发生变化。

一种解决办法是可以在脚本中控制何时生成从当前位置观察到的 Cubemap，而 Unity 5 为我们提供了一种更加方便的途径，即使用**反射探针 Reflection Probes**。反射探针的工作原理和光照探针 Light Probes 类似，它允许我们在场景中的特定位置上对整个场景的环境反射进行采样，并把采样结果存储在每个探针上。当游戏中包含反射效果的物体从这些探针附近经过时，Unity 会把从这些邻近探针存储的反射结果传递给物体使用的反射纹理。如果物体周围存在多个反射探针，Unity 还会在这些反射结果之间进行插值，来得到平滑渐变的反射效果。实际上，Unity 会在场景中放置一个默认的反射探针，这个反射探针存储了对场景使用的 Skybox 的反射结果，来作为场景的环境光照。如果我们需要让场景中的物体包含额外的反射效果，就需要放置更多的反射探针。

反射探针同样有 3 种类型：  
①**Baked**，这种类型的反射探针是通过提前烘焙来得到该位置使用的 Cubemap 的，在游戏运行时反射探针中存储的 Cubemap 并不会发生变化。需要注意的是，这种类型的反射探针在烘焙时同样只会处理那些静态物体（即那些被标识为 Reflection Probe Static 的物体）；  
②**Realtime**，这种类型则会实时更新当前的 Cubemap，并且会受到动态物体的影响；当然，这种类型的反射探针需要花费更多的处理时间，因此，在使用时应当非常小心它们的性能。幸运的是，Unity 允许我们从脚本中通过触发来精确控制反射探针的更新；  
③**Custom**，这种类型的探针既可以让我们从编辑器中烘焙它，也可以让我们使用一个自定义的 Cubemap 来作为反射映射，但自定义的 Cubemap 不会被实时更新。  

在放置反射探针时，选取的位置并不是任意的。通常来说，反射探针应该被放置在那些具有明显反射现象的物体的旁边，或是一些墙角等容易发生遮挡的物体周围。当我们放置好探针后，我们还需要为它们定义每个探针的影响区域，当反射物体进入到这个区域后，反射探针就会对物体的反射产生影响。通常情况下，反射探针的影响区域之间往往会有所重叠。此时，Unity 会计算反射物体的包围盒与这些重叠区域的交叉部分，并据此来选择使用的反射映射。如果当前的目标平台使用的是 SM 3.0 及以上的话，Unity 还可以允许我们在这些互相重叠的反射探针之间进行混合，来实现平缓的反射过渡效果。

---

使用 Unity 内置的反射探针的另一个好处是，我们可以模拟**互相反射 interreflections**。例如，假设场景中有两面互相面对面的镜子，在理想情况下，它们不仅会反射自己对面的那面镜子，也会反射那面镜子里反射的图像。只要反射光线没有被完全吸收，反射就会一直进行下去。要实现这种效果，就需要追踪光线的反射轨迹，这是传统的反射方法所无法实现的。Unity 5 引入的 GI 系统让这种效果变成了可能，我们可以在每个相互反射的物体的位置处放置一个反射探针，并把每个物体的 Mesh Renderer 组件中的 Reflection Probes 设置为 Simple，这样保证它们只会使用离它们最近的一个反射探针。默认情况下，反射探针只会捕捉一次反射，也就是说，左边物体使用的反射探针只会捕捉到由右边物体第一次反射过来的光线。但在理想情况下，反射过来的光线会继续被左边的物体反射，并对右边的物体造成影响。Unity 允许我们控制物体之间这样来回反射的次数，这可以通过改变 Reflection Bounces 参数来实现。

使用反射探针往往会需要更多的计算时间。这些探针实际上也是通过在它的位置上放置一个摄像机，来渲染得到一个 Cubemap。如果我们把反弹次数设置的很大，或是使用实时渲染，那么这些探针很可能会造成性能瓶颈。

### 线性空间
在使用基于物理的渲染方法渲染整个场景时，我们应该使用**线性空间 Linear Space** 来得到最好的渲染效果。使用线性空间可以得到更加真实的效果。但它的缺点在于，需要一些硬件支持来实现线性计算，但一些移动平台对它的支持并不好。这种情况下，我们往往只能退而求其次，选择伽马空间进行渲染和计算。下面会解释**伽马校正 Gamma Correction** 的相关内容。实际上，当我们在默认的伽马空间下进行渲染计算时，由于使用了非线性的输入数据，导致很多计算都是在非线性空间下进行的，这意味着我们得到的结果并不符合真实的物理期望。除此之外，由于输出时没有考虑显示器的显示伽马的影响，会导致渲染出来的画面整体偏暗，总是和真实世界不像。


## 答疑解惑
### 什么是全局光照
**全局光照**，指的就是模拟光线是如何在场景中传播的，它不仅会考虑那些直接光照的结果，还会计算光线被不同的物体表面反射而产生的间接光照。

通常来讲，这些间接光照的计算是非常耗时间的，通常不会用在实时渲染中。一个传统的方法是使用光线追踪，来追踪场景中每一条重要的光线的传播路径。使用光线追踪能得到非常出色的画面效果，因此，被大量应用在电影制作中。但是，这种方法往往需要大量时间才能得到一帧，并不能满足实时的要求。

Unity 采用了 **Enlighten 解决方案**来让全局光照能够在各种平台上有不错的性能表现。事实上，Enlighten 也已经被集成在虚幻引擎 Unreal Engine 中，它已经在很多 3A 大作中展现了自身强大的渲染能力。总体来讲，Unity 使用了实时+预计算的方法来模拟场景中的光照。其中，实时光照用于计算那些直接光源对场景的影响，当物体移动时，光照也会随之发生变化。但正如我们之前所说，实时光照无法模拟光线被多次反射的效果。为了得到更加真实的渲染效果，Unity 又引入了**预计算光照**的方法，使得全局光照甚至在一些高端的移动设备上也可以达到实时的要求。

预计算光照包含了我们常见的光照烘焙，也就是指我们把光源对场景中静态物体的光照效果提前烘焙到一张光照纹理中，然后把这张光照纹理直接贴在这些物体的表面，来得到光照效果。这些光照纹理不仅存储了直接光照的结果，还包含了那些由物体反射得到的间接光照。但是，这些光照纹理无法在游戏运行时不断更新，也就是说，它们是静态的。不过这种方法的确为移动平台的复杂光照模拟提供了一个有效途径。

由于静态的光照烘焙无法在光照条件改变时更新物体的光照效果，因此，Unity 使用了**预计算实时全局光照 Precomputed Realtime GI** 为我们提供了一个解决途径，来动态地为场景实时更新复杂的光照结果。使用这种技术我们可以让场景中的物体包含丰富的全局光照效果，例如多次反射等，并且这些计算都是实时的，可以随着光源和物体的移动而发生变化。这是使用之前的实时光照或烘焙光照所无法实现的。

实现的原理：一旦物体和光源的位置被固定了，这些物体对光线的反弹路径以及漫反射光照（我们假设漫反射光照在各个方向的分布是相同的）也是固定的，也就是说是和摄像机无关的。因此，我们可以使用预计算方法来把这些物体之间的关系提前计算出来，而在实时运行时，只要光源的位置（光源的颜色是可以实时变化的）不变，即便改变了光源颜色和强度、物体材质属性（指的是漫反射和自发光相关的属性），这些信息就一直有效，不需要实时更新。

在预计算阶段，Enlighten 会在由所有静态物体组成的场景上，进行简化的“光线追踪”过程。在这个过程中 Enlighten 会自动把场景分割成很多个子系统，它并不是为了得到精确的光照效果，而是为了得到场景中物体之间的关系。需要注意的是，这些预计算都是在静态物体上进行的，因此，为了利用上述的预计算方法，我们至少需要把场景中的一个物体标识为 Static（至少需要把 Lightmap Static 勾选上）。一个例外是物体的高光反射，这是和摄像机的位置相关的，Unity 的解决方案是使用反射探针，正如我们之前看到的那样。对于动态移动的物体来说，我们可以使用光照探针来模拟它的光照环境。因此，在实时运行时，Unity 会利用预计算得到的信息来计算光照信息，并把它们存储在额外的光照纹理、光照探针或 Cubemap 中，再和物体材质进行必要的光照计算，得到最后的渲染效果。

Unity 全新的全局光照解决方案可以大大提高一些基于 PC /游戏机等平台的大型游戏的画面质量，但如果要在移动平台上使用仍需要非常小心它的性能。

### 什么是伽马校正
Unity 默认的空间是伽马空间，要把伽马空间转换到线性空间，就需要进行**伽马校正 Gamma Correction**。

伽马校正中的伽马一词来源伽马曲线。通常，伽马曲线的表达式如下：  

$$ L_{out} = L_{in}^{\gamma} $$

最开始的时候，人们使用伽马曲线来对拍摄的图像进行**伽马编码 gamma encoding**。事情的起因可以从在真实环境中拍摄一张图片说起。摄像机的原理可以简化为，把进入到镜头内的光线亮度编码成图像（例如一张 JEPG）中的像素。如果我们只用 8 位空间来存储像素的每个通道的话，这意味着 0～1 区间可以对应 256 种不同的亮度值。但是，后来人们发现，人眼有一个有趣的特性，就是对光的灵敏度在不同亮度上是不一样的。在正常的光照条件下，人眼对较暗区域的变化更加敏感。即亮度上的线性变化对人眼感知来说是非均匀的。

如果使用 8 位空间来存储每个通道的话，我们仍然把 0.5 亮度编码成值为 0.5 的像素，那么暗部和亮部区域我们都使用了 128 种颜色来表示，但实际上，对亮部区域使用这么多颜色是种存储浪费。一种更好的方法是，我们应该把把更多的空间来存储更多的暗部区域，这样存储空间就可以被充分利用起来了。摄影设备如果使用了 8 位空间来存储照片的话，会使用大约为 0.45 的编码伽马来对输入的亮度进行编码，得到一张编码后的图像。因此，图像中 0.5 像素值对应的亮度其实并不是 0.5，而大约为 0.22。这是因为：  

$$ 0.5 \approx 0.22^{0.45} $$

对拍摄图像使用的伽马编码使得我们可以充分利用图像的存储空间。但当把图片放到显示器里显示时，我们应该对图像再进行一次解码操作，使得屏幕输出的亮度和捕捉到的亮度是符合线性的。

这时，人们发现了一个奇妙的巧合，CRT 显示器本身几乎已经自动做了这个解码操作！在早期，**CRT Cathode Ray Tube，阴极射线管**几乎是唯一的显示设备。这类设备的显示机制是，使用一个电压轰击它屏幕上的一种图层，这个图层就可以发亮，我们就可以看到图像了。但 CRT 显示器有一个特性，它的输入电压和显示出来的亮度关系不是线性的，也就是说，如果我们把输入电压调高两倍，屏幕亮度并没有提高两倍。我们把显示器的这个伽马曲线称为**显示伽马 diplay gamma**。非常巧合的是，CRT 的显示伽马值大约就是编码伽马的倒数。CRT 显示器的这种特性，正好补偿了图像捕捉设备的伽马曲线。虽然现在 CRT 设备很少见了，并且后来出现的显示设备有着不同的伽马响应曲线，但是，人们仍在硬件上做了调整来提供兼容性。下图展示了编码伽马和显示伽马在图像捕捉和显示时的作用：  

<div  align="center">  
<img src="https://s2.loli.net/2024/01/09/xI3SbLC8jWRMGUm.jpg" width = "80%" height = "80%" alt="图96- 编码伽马和显示伽马"/>
</div>

随后，微软联合爱普生、惠普提供了 **sRGB 颜色空间标准**，推荐显示器的显示伽马值为 2.2，并配合 0.45 的编码伽马就可以保证最后伽马曲线之间可以相互抵消（因为 2.2 × 0.45 ≈ 1）。绝大多数的摄像机、PC 和打印机都使用了上述的 sRGB 标准。

事实上，由于游戏界长期以来都忽视了伽马校正的问题，造成了我们渲染出来的游戏总是暗沉沉的，总是和真实世界不像。由于编码伽马和显示伽马的存在，我们一不小心就可能在非线性空间下进行计算，或是使得输出的图像是非线性的。

对于输出来说，如果我们直接输出渲染结果而不进行任何处理，在经过显示器的显示伽马处理后，会导致图像整体偏暗，出现失真的状况。使用了线性空间，Unity 会在把像素写入颜色缓冲前进行一次伽马校正，来抵消屏幕的显示伽马的作用，此时得到屏幕亮度才是真正跟像素值成正比的。伽马的存在还会对颜色**混合**造成影响。

---

实际上，渲染中非线性输入最有可能的来源就是纹理。为了充分利用存储空间，大多数图像文件都进行了提前的校正，即已经使用了一个编码伽马对像素值编码。但这意味着它们是非线性的，如果我们在 Shader 中直接使用纹理采样值就会造成在非线性空间的计算，使得结果和真实世界的结果不一致。我们在使用多级渐远纹理 mipmaps 时也需要注意。如果纹理存储在非线性空间中，那么在计算多级渐远纹理时就会在非线性空间里计算。由于多级渐远纹理的计算是种线性计算 - 即采样的过程，需要对某个方形区域内的像素取平均值，这样就会得到错误的结果。正确的做法是，我们要把非线性的纹理转换到线性空间后再计算多级渐远纹理。

如上所说，伽马的存在使得我们很容易得到非线性空间下的渲染结果。在游戏渲染中，我们应该保证所有的输入都被转换到了线性空间下，并在线性空间下进行各种光照计算，最后在输出前通过一个编码伽马进行伽马校正后再输出到颜色缓冲中。Unity 的颜色空间设置就可以满足我们的需求。

当我们选择**伽马空间**时，实际上就是“放任模式”，不会对 Shader 的输入进行任何处理，即使输入可能是非线性的；也不会对输出像素进行任何处理，这意味着输出的像素会经过显示器的显示伽马转换后得到非预期的亮度，通常表现为整个场景会比较昏暗。

当选择**线性空间**时，Unity 会把输入纹理设置为 sRGB 模式，在这种模式下，硬件在对纹理进行采样时会自动将其转换到线性空间中；并且，GPU 会在 Shader 写入颜色缓冲前自动进行伽马校正或是保持线性在后面进行伽马校正，这取决于当前的渲染配置：  
①如果我们开启了 HDR 的话，渲染就会使用一个浮点精度的缓冲。这些缓冲有足够的精度不需要我们进行任何伽马校正，此时所有的混合和屏幕后处理都是在线性空间下进行的。当渲染完成要写入显示设备的后备缓冲区 back buffer 时，再进行一次最后的伽马校正。  
②如果我们没有使用 HDR，那么 Unity 就会把缓冲设置成 sRGB 格式，这种格式的缓冲就像一个普通的纹理一样，在写入缓冲前需要进行伽马校正，在读取缓冲时需要再进行一次解码操作。如果此时开启了混合（像我们之前的那样），在每次混合时，硬件会首先把之前颜色缓冲中存储的颜色值转换回线性空间中，然后再与当前的颜色进行混合，完成后再进行伽马校正，最后把校正后的混合结果写入颜色缓冲中。这里需要注意，透明通道是不会参与伽马校正的。

然而，Unity 的线性空间并不是所有平台都支持的，例如，移动平台就无法使用线性空间。此时，我们就需要自己在 Shader 中进行伽马校正。对非线性输入纹理的校正代码通常如下：  

    float3 diffuseCol = pow(tex2D( diffTex, texCoord ), 2.2 );

在最后输出前，对输出像素值的校正代码通常如下面这样：  

    fragColor.rgb = pow(fragColor.rgb, 1.0/2.2);
    return fragColor;

但是，手工对输出像素进行伽马校正会在使用混合时出现问题。这是因为，校正会导致写入颜色缓冲内的颜色是非线性的，这样混合就发生在非线性空间中。一种解决方法是，在中间计算时不要对输出颜色值进行伽马校正，但在最后需要进行一个屏幕后处理操作来对最后的输出进行伽马校正，也就是说我们需要保证伽马校正发生在渲染的最后一步中，但这可能会造成一定的性能损耗。

若我们有足够多的颜色空间可以利用，不需要为了充分利用存储空间进行伽马编码的工作了。这就是我们下面要讲的 HDR。

### 什么是 HDR
**HDR** 是 **High Dynamic Range** 的缩写，即**高动态范围**，与之相对的是**低动态范围 Low Dynamic Range，LDR**。

通俗来讲，动态范围指的就是最高的和最低的亮度值之间的比值。在真实世界中，一个场景中最亮和最暗区域的范围可以非常大，例如，太阳发出的光可能要比场景中某个影子上的点的亮度要高出几万倍，这些范围远远超过图像或显示器能够显示的范围。

通常在显示设备使用的颜色缓冲中每个通道的精度为 8 位，意味着我们只能用这 256 种不同的亮度来表示真实世界中所有的亮度，因此，在这个过程中一定会存在一定的精度损失。早期的拍摄设备利用人眼的特点，使用了伽马曲线来对捕捉到的图像进行编码，尽可能充分地利用这些有限的存储空间。

HDR 使用远远高于 8 位的精度（如 32 位）来记录亮度信息，使得我们可以表示超过 0～1 内的亮度值，从而可以更加精确地反映真实的光照环境。尽管我们最后还是需要把信息转换到显示设备使用的 LDR 内，但中间的计算却可以让我们得到更加真实可信的效果。Nvidia 曾总结过使用 HDR 进行渲染的动机：让亮的物体可以真的非常亮，暗的物体可以真的非常暗，同时又可以看到两者之间的细节。

使用 HDR 来存储的图像被称为**高动态范围图像 HDRI**。使用了一张 HDRI 图像来作为场景的 Skybox 可以更加真实地反映物体周围的环境，从而得到更加真实的反射效果。不仅如此，HDR 对与光照叠加也有非常重要的作用。如果我们的场景中有很多光源或是光源强度很大，那么一个物体在经过多次光照渲染叠加后最终得到的光照亮度很可能会超过 1。如果没有使用 HDR，这些超过 1 的部分全部会截取到 1，使得场景丢失了很多亮部区域的细节。但如果开启了 HDR，我们就可以保留这些超过范围的光照结果，尽管最后我们仍然需要把它们转换到 LDR 进行显示，但我们可以使用**色调映射 tonemapping** 技术来控制这个转换的过程，从而允许我们最大限度地保留需要的亮度细节。

HDR 的使用可以允许我们在屏幕后处理中拥有更多的控制权，比如 Bloom 效果。Bloom 效果需要检测屏幕中亮度大于某个阈值的像素，把它们提取出来后进行模糊，再叠加到原图像中。但是，如果不使用 HDR 的话，我们只能使用小于 1 的阈值来提取需要的像素，会造成某些偏白的不希望的区域也会出现泛光的效果。如果我们使用 HDR，我们只需要使用超过 1 的阈值来只提取那些非常亮的区域即可。

HDR 也有自身的缺点，首先由于使用了浮点缓冲来存储高精度图像，不仅需要更大的显存空间，渲染速度会变慢，除此之外，一些硬件并不支持 HDR。而且一旦使用了 HDR，我们无法再利用硬件的抗锯齿功能。事实上，在 Unity 中如果我们同时打开了硬件的抗锯齿（在 Edit -> Project Settings -> Quality -> Anti Aliasing 中打开）和摄像机的 HDR，Unity 会发出警告来提示我们由于开启了抗锯齿，因此，无法使用 HDR 缓冲。尽管如此，我们可以使用基于屏幕后处理的抗锯齿操作来弥补这一点。

在 Unity 中使用 HDR 也非常简单，我们可以在 Camera 组件面板中打开 HDR 选项即可。此时，场景就会被渲染到一个 HDR 的图像缓冲中，这个缓冲的精度范围可以远远超过 0～1。最后，我们可以再使用一个色调映射的屏幕后处理脚本来把 HDR 图像转换到 LDR 图像进行显示。详见 Unity 官方手册中的高动态范围渲染一节。

## 扩展阅读
近年来，Unity 在 Unite 和 SIGGRAPH 等大会上也分享不少关于 PBS 的技术资料，可以在这些地方找到非常丰富的资料。