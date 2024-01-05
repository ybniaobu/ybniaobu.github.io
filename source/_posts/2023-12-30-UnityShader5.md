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
在现实生活中，几乎所有的物体都或多或少有高光反射现象。John Hable 在他的文章中就强调了 **Everything is Shiny**。在基于物理的渲染中，BDRF 中的**高光反射项**大多数都是建立在**微面元理论 microfacet theory** 的假设上的。微面元理论认为，物体表面实际是由许多人眼看不到的微面元组成的，虽然物体表面并不是光学平滑的，但这些微面元可以被认为是光学平滑的，也就是说它们具有完美的高光反射。

