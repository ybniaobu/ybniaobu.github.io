---
title: 《Unity Shader入门精要》读书笔记（一）
date: 2023-09-15 12:03:32
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
top_img: /images/black.jpg
cover: https://s2.loli.net/2023/09/19/cDvdURBPhjwkOsY.gif
mathjax: true
---
  
> 本读书笔记主要内容为XXXXXXXXXXXXXX。
> 读书笔记是对知识的记录与总结，但是对比较熟悉的内容不会再行描述。

# 第一章 渲染管线 Rendering Pipeline
## 什么是渲染管线
**渲染管线**也称为**渲染流水线**，是将三维场景模型转换到屏幕像素空间输出的过程。渲染流程可分为三个阶段：**应用阶段**、**几何阶段**、**光栅化阶段**：  
**①应用阶段**：这是一个由开发者完全控制的阶段，在这一阶段将进行数据准备，并通过 CPU 向 GPU 输送数据，例如顶点数据、摄像机位置、视锥体数据、场景模型数据、光源等等；此外，为了提高渲染性能，还会对这些数据进行处理，比如**剔除 culling** 不可见物体；最后还要设置每个模型的渲染状态，这些渲染状态包括但不限于所使用的材质、纹理、shader等。这一阶段最重要的输出是渲染所需的几何信息，即**渲染图元 rendering primitives**，通俗来讲渲染图元可以是点、线、面等；  
**②几何阶段**：几何阶段运行在 GPU 中，几何阶段和每个渲染图元打交道。几何阶段最重要的任务是将顶点坐标变换到屏幕空间中，再交给光栅器进行处理。通过对输入的渲染图元进行处理后，将输出屏幕空间的二维顶点坐标、每个顶点对应的深度值、着色等相关信息，传递给下个阶段；  
**③光栅化阶段**：光栅化阶段运行在 GPU 中，其主要任务是决定每个渲染图元中哪些像素应该被绘制在屏幕上，它需要对上一阶段得到的逐顶点数据进行插值，然后进行逐像素处理。

## CPU 和 GPU 之间的通信（应用阶段）
渲染管线的起点是 CPU ，应用阶段可分为以下三个阶段：  
**①把数据加载到显存**：所有渲染所需的数据都需要从硬盘 Hard Disk Drive, HDD 中加载到系统内存 Random Access Memory, RAM 中。然后网格和纹理等数据又被加载到显卡上的存储空间，即显存 Video Random Access Memory, VRAM 中。大多数显卡没有直接访问 RAM 的能力，将数据加载到显存中使 GPU 能更快的访问这些数据。当把数据加载到显存后，内存中的数据便可以释放了，但对于一些还需要使用的数据则需要继续保留在内存中，如 CPU 需要网格数据进行碰撞检测；  
**②设置渲染状态**：渲染状态的一个通俗解释就是，定义了场景中的网格是怎样被渲染的。例如，使用哪个顶点着色器 Vertex Shader /片段着色器 Fragment Shader、光源属性、材质等。
**③调用 Draw Call**：当所有的数据准备好后，CPU 就需要调用一个渲染指令告诉 GPU，按照上述设置进行渲染，这个渲染命令就是 **Draw Call**。Draw Call 命令仅仅会指向一个需要被渲染的图元列表，而不包含任何材质信息，因为这些信息已经在上一个阶段中完成。给定 Draw Call 后 GPU 就会根据渲染状态和所有输入的顶点数据来进行计算，并输出到显示设备中，所执行的操作便是下述 GPU 渲染管线的内容。  

## GPU 渲染管线（几何阶段和光栅化阶段）

<div  align="center">  
<img src="https://s2.loli.net/2023/09/15/BkGnZ46bTAVrR1J.png" width = "60%" height = "60%" alt="图1- GPU 渲染管线。颜色表示了不同阶段的可配置性或可编程性：绿色表示该流水线阶段是完全可编程控制的，黄色表示该流水线阶段可以配置但不是可编程的，蓝色表示该流水线阶段是由 GPU 固定实现的，开发者没有任何控制权。实线表示该 shader 必须由开发者编程实现，虚线表示该 Shader 是可选的。"/>
</div>

### 顶点着色器 Vertex Shader
顶点着色器的处理单位是顶点，即对于输入的每个顶点都会调用一次顶点着色器。顶点着色器本身无法得到顶点与顶点之间的关系，例如无法得到两个顶点是否属于同一个三角网格，因为这样的独立性，可以优化处理每一个顶点。  

顶点着色器的两个任务是**坐标变换**和**逐顶点光照**：  
**①坐标变换，即将顶点坐标从模型空间转换到齐次裁剪空间。**当顶点坐标被变换到齐次裁剪空间后，通常再由硬件做透视除法，最终得到**归一化的设备坐标 Normalized Device Coordinates, NDC**。OpenGL（Unity 使用的 NDC）的 z 分量范围在\[-1, 1\]之间，而在 DirectX 中，NDC 的 z 分量范围是\[0, 1\]。

> 内容扩展：**MVP 矩阵**是**模型 Model**、**观察 View**、**投影 Projection** 三个矩阵的合称。这三个矩阵代表了物体顶点坐标从局部空间转换到裁剪空间，最后以屏幕坐标的形式结束。模型矩阵表示顶点坐标从物体自身局部空间 Local Space 转换到世界空间 World Space；观察矩阵表示从世界空间到观察空间 View Space ；投影矩阵表示从观察空间到裁剪空间 Clip Space。需要注意的是：投影矩阵并不代表这一步矩阵乘法过程中包含了投影，而是在下一步通过透视除法将 xyz 分量除以 w 才会发生投影。

**②逐顶点光照 per-vertex lighting**，也被称为**高洛德着色 Gouraud Shading**。在逐顶点光照中，会在每个顶点上计算光照，然后会在渲染图元内部进行线性插值，最后输出成像素颜色。

但顶点光照效果通常不尽人意，因此通常在片元着色器中执行逐片元光照计算。顶点着色器可以有不同的输出方式。最常见的输出路径是经光栅化后交给片元着色器进行处理。而在现代的 Shader Model 中，还可以把数据发送给曲面细分着色器或几何着色器。

### 曲面细分着色器 Tessellation Shader
曲面细分着色器是一个可选的阶段。曲面细分是利用镶嵌化处理技术对三角形进行细分，以此来增加物体表面的三角面数量。如果为这些细分的顶点再准备一些位置信息，有助于展现一个细节更加丰富的模型，这也是**贴图置换 Displacement Mapping** 的基本思路。

### 几何着色器 Geometry Shader
几何着色器也是一个可选的阶段。顶点着色器以顶点数据作为输入，而几何着色器则以完整的图元 Primitive 作为输入数据。与顶点着色器不能销毁或创建顶点不同，几何着色器的主要亮点就是可以创建或销毁几何图元，此功能让 GPU 可以实现一些有趣的效果。例如，根据输入图元类型扩展为一个或更多其他类型的图元，或者不输出任何图元。需要注意的是，几何着色器的输出图元不一定和输入图元相同。几何着色器的一个拿手好戏就是将一个点扩展为一个四边形(即两个三角形)。

### 裁剪 Clipping
裁剪操作就是将相机看不到的物体、顶点剔除，使其不被下一阶段处理。只有当图元完全位于视锥体内时，才会将它送到下一阶段，对于部分位于视锥体内的图元，外部的顶点将被剔除掉。由于已经知道在 NDC 下的顶点位置，即顶点位置在一个立方体内，因此裁剪就变得简单：只需要将图元裁剪到单位立方体内，只有在单位立方体的图元才需要被继续处理。因此，完全在单位立方体外部的图元被舍弃，完全在单位立方体内部的图元将被保留。和单位立方体相交的图元会被裁剪，新的顶点会被生成，原来在外部的顶点会被舍弃。裁剪这一步骤是硬件的固定操作，因此是不可编程的，但是可以自定义一个裁剪操作来配置。

### 屏幕映射 Screen Mapping
主要是**视口变换 Viewport Transformation**。这一步输入的坐标仍是三维坐标（范围在单位立方体内），屏幕映射的任务就是将每个图元的 x、y 值变换到**屏幕坐标系 Screen Coordinates**，屏幕坐标系是一个二维坐标系。由于输入坐标范围在\[-1, 1\]，因此这是一个缩放到屏幕分辨率大小的过程。对于输入的坐标 z 值不做任何处理，实际上屏幕坐标系和 z 坐标一起构成**窗口坐标系 Window Coordinates**，这些值会被一起传递到光栅化阶段。

OpenGL 和 DirectX 的屏幕坐标系存在差异：OpenGL 把屏幕的左下角当成最小的窗口坐标值，而 DirectX 则定义了屏幕的左上角为最小的窗口坐标值。

### 三角形设置 Triangle Setup
由这一步进入光栅化阶段。上个阶段的输出信息包括屏幕坐标系下的顶点位置以及深度值（z 坐标）、法线方向、视角方向等。光栅化阶段的两个重要目标是**计算每个图元覆盖了那些像素**，以及**为这些像素计算颜色**。

光栅化第的第一个流水线阶段是**三角形设置**，这个阶段会计算光栅化一个三角形网格所需的信息。上一阶段输出的都是三角网格的顶点，但如果要得到整个三角形网格对像素的覆盖情况，就必须计算每条边上的像素坐标。为了能计算边界像素的坐标信息，就需要得到三角形边界的表示方式。这样一个计算三角网格表示数据的过程就叫做三角形设置。它的输出是为下一阶段做准备。

### 三角形遍历 Triangle Traversal
该阶段会检查像素是否被三角网格覆盖，被覆盖则生成一个**片元 fragment**，这个阶段也被称为**扫描变换 Scan Conversion**。

<div  align="center">  
<img src="https://s2.loli.net/2023/09/18/njUJblGaDHO9is4.png" width = "60%" height = "60%" alt="图2- 三角形遍历的过程。根据几何阶段输出的顶点信息，最终得到该三角网格覆盖的像素位置。对应像素会生成一个片元，而片元中的状态是对三个顶点的信息进行插值得到的。例如，图中三个顶点的深度进行插值得到其重心位置对应的片元的深度值为-10.0。"/>
</div>

需要注意的是，一个片元并不是真正意义上的像素，而是很多状态的集合，这些状态用于计算每个像素的最终颜色，包括屏幕坐标、深度、法线、纹理坐标等，最后需要一系列测试才会成为像素。

### 片元着色器 Fragment Shader
**片元着色器**是一个非常重要的可编程着色器阶段，在 DirectX 中，片元着色器被称为**像素着色器 Pixel Shader**。

前面的光栅化阶段实际上并不会影响屏幕上每个像素的颜色值，而是会产生一系列的数据信息，用来表述一个三角网格是怎样覆盖每个像素的，而片元就负责存储这样一系列信息，真正会对像素产生影响的是下一个流水线阶段：**逐片元操作 Per-Fragment Operations**。

片元着色器的输入是上一个阶段对顶点信息进行插值的结果，是根据从顶点着色器输出的数据插值得到的。而它的输出是像素颜色值。但是至此，屏幕具体的像素都没有任何变化，这些都是预备的数据，直到逐片元操作。

这一阶段可以完成很多重要的渲染技术，其中最重要的技术有纹理采样。为了在片元着色器中进行纹理采样，通常会在顶点着色器阶段输出每个顶点对应的纹理坐标，然后经过光栅化阶段对三角网格的三个顶点对应的纹理坐标进行插值后，就可以得到其覆盖的片元的纹理坐标了。

<div  align="center">  
<img src="https://s2.loli.net/2023/09/18/VLsdUI7utyM5boC.png" width = "60%" height = "60%" alt="图3- 根据上一步插值后的片元信息，片元着色器计算该片元的输出颜色。"/>
</div>

### 逐片元操作 Per-Fragment Operations
逐片元操作是渲染管线的最后一个阶段。**逐片元操作**是 OpenGL 的说法，DirectX 称为**输出合并阶段 Output-Merger**。

这一阶段有几个主要任务：  
①决定每个片元的可见性，这涉及到很多测试功能，例如模板测试、深度测试；  
②如果一个片元通过了所有的测试，就需要把这些片元的颜色值和已经存储在颜色缓冲区中的颜色值进行混合。

> 逐片元操作阶段是高度可配置性的，测试顺序并不是唯一的，不同的图形接口的实现细节也不太一样。这里给出两个最基本的测试：模板测试和深度测试的实现过程。

**模板测试 Stencil Test** 与之相关的是模板缓冲 Stencil Buffer。模板测试通常用来限制渲染的区域，渲染阴影，轮廓渲染等。如果开启了模板测试，GPU 会首先读取（使用读取掩码）模板缓冲区中该片元位置的模板值，然后将该值和读取（使用读取掩码）到的参考值进行比较，这个比较函数可以是由开发者指定的，例如，小于时舍弃该片元，或者大于等于时舍弃该片元。模板测试是高度可配置的，无论一个片元有没有经过模板测试，都可以根据模板测试和下面的深度测试结果来修改模板缓冲区。

> Stencil 的英英释义：device that has a sheet perforated with printing through which ink or paint can pass to create a printed pattern.

**深度测试 Depth Test** 同样是可以高度配置的。如果开启了深度测试，GPU 会把该片元的深度值和已经存在于深度缓冲区中的深度值进行比较，这个比较函数也是可以由开发者设置，通常这个值是小于等于的关系，即若这个片元的深度值大于等于当前深度缓冲区中的值，就舍弃它。因为我们总想显示出离相机最近的物体(不包括透明/半透明)，而那些被遮挡的就不需要出现在屏幕。如果一个片元通过了测试，那么开发者还可以指定是否要用这个片元的深度值覆盖所有的深度值。

**混合 Blend** 如果一个片元通过了所有测试，就需要解决是否需要混合操作。渲染的过程是一个物体接着一个物体画到屏幕前。每一个像素的颜色信息被存储在一个名为颜色缓冲的地方。因此当我们执行这次渲染时，颜色缓冲中往往已经有了上次渲染的颜色结果。所以是使用这次渲染得到的颜色完全覆盖掉之前的结果，还是进行其他处理？对于不透明的物体，开发者要关闭混合操作，这样片元着色器计算出的颜色值就会直接覆盖掉颜色缓冲区中的像素值。但对于半透明物体，就需要使用混合操作。

> 不透明的物体当然是看到最近的，那么透明的呢？需要进行片元着色器中的颜色和颜色缓冲器的颜色混合。举个例子，一个玻璃放在眼前，我既可以看到玻璃，又可以看到玻璃之后的，实际看到的是这两个颜色的混合。

开发者可以选择开启/关闭混合功能。如果没有开启混合，就会直接使用片元的颜色覆盖掉颜色缓冲区中的颜色。如果开启了混合，GPU 会取出片元着色器得到的颜色（**源颜色**）和颜色缓冲区存在的颜色（**目标颜色**），之后按照设定的函数进行混合，这个混合函数通常和透明度通道息息相关，例如可以根据透明通道的值进行相加、相减、相乘等。

> 上面给出的测试顺序并不是唯一的。逻辑上来说测试是在片元着色器之后进行，但想充分提高 GPU 性能，会尽可能的在执行片元着色器之前就执行测试操作，从而避免将不需要渲染的片元流入到后续的运算中。Unity 中深度测试在片元着色器前，这称为 Early-Z 技术。但将测试提前可能会与片元着色器中的一些操作冲突，如透明度测试，后面章节会提到。

渲染中的图元必定是不能显示的，就像舞台更改背景，背景准备完毕，红布一拉，舞台展现在众人眼前。GPU 会使用**双重缓冲 Double Buffering** 的策略。对场景的渲染在幕后发生，即在**后置缓冲 Back Buffer** 中。**前置缓冲 Front Buffer** 中的内容就是之前显示在屏幕上的图像。GPU 不断交换两者内容，保证看到的都是连续的画面。

## 一些专业术语
### 图形 API
OpenGL 和 DirectX 这些图像应用编程接口，架起了上层应用程序和底层 GPU 的沟通桥梁。一个应用程序向这些接口发送渲染命令，即 Draw Call，而这些接口会依次向显卡驱动发生渲染命令，显卡驱动把 OpenGL 或者 DirectX 的函数调用翻译成 GPU 能听懂的语言，同时把数据转换为 GPU 所支持的格式。

### 着色语言 Shading Language
在可编程管线出现前，要编写着色器，需要汇编语言。后来出现了相对更“高级”的着色语言，这些语言会被编译为汇编语言，也被称为中间语言 Intermediate Language, IL。这些中间语言再交给显卡驱动来翻译成真正的机器语言，即 GPU 可以理解的语言。

常见的着色语言有 DirectX 的 **HLSL**，High Level Shading Language；OpenGL 的 **GLSL**，OpenGL Shading Language；NVIDIA 的 **Cg**，C for Graphic。  
① GLSL 的优点在于它的跨平台性，它可以在 Windows、Linux、Mac 甚至移动平台等多种平台上工作，但这种跨平台性是由于 OpenGL 没有提供着色器编译器，而是由显卡驱动来完成着色器的编译工作。即 GLSL 是依赖硬件，而非操作系统层级的。但这也意味着 GLSL 的编译结果将取决于硬件供应商，这可能造成编译结果不一致的情况。比如 NVIDIA、ATI 对 GLSL 的实现不尽相同。  
② HLSL 由微软控制着着色器的编译，就算使用了不同的硬件，同一个着色器的编译结果也是一样的。但支持 HSLS 的平台相对较少，几乎完全是微软自己的产品，如Windows、Xbox 360等。这是因为在其他平台上没有可以编译 HSLS 的编译器。  
③ Cg 则是真正意义上的跨平台。它会根据平台的不同，编译成相应的中间语言。Cg 语言的跨平台性很大原因取决于与微软的合作哦，这也导致 Cg 语言的语法与 HLSL 非常像，Cg 语言可以无缝移植成 HLSL 代码。但缺点是可能无法完全发挥出 OpenGL 的最新特性。

### Draw Call
Draw Call 就是 CPU 调用图像编程接口，如 OpenGL 中的 glDrawElements 命令或者 DirectX 中的 DrawIndexedPrimitive 命令，以命令 GPU 进行渲染操作。

Draw Call 中造成性能问题的是 CPU，而非 GPU。

① CPU 和 GPU 并行工作  
若 CPU 需要等待 GPU 完成上一个渲染任务才能再次发送渲染命令，会造成效率低下。为了并行工作，使用了**命令缓冲区 Command Buffer**：命令缓冲区包含一个命令队列，CPU 通过图像编程接口向命令缓冲区中添加命令，而 GPU 从中读取命令并执行，从而使它们相互独立工作。命令缓冲区的命令有很多种类，而 Draw Call 就是其中的一种。

②为什么 Draw Call 多了会影响帧率？  
CPU 每次调用 Draw Call 都要经过一系列工作，多次调用 Draw Call 会进行许多重复性操作。如果 Draw Call 太多，CPU 会把大量时间花费在提交 Draw Call 上，造成 CPU 的过载。

③如何减少 Draw Call  
方法很多，其一就是**批处理 Batching** 方法：将许多小的 Draw Call 合并成一个大的 Draw Call。由于要在 CPU 的内存里合并网格，合并的过程很耗时，所以批处理技术跟适用于静态物体。

在游戏开发中，为减少 Draw Call 的开销，尽量避免使用大量很小的网格，若不可避免考虑合并它们；避免使用过多材质，考虑空用材质。

### 固定渲染管线
**固定函数的流水线 Fixed-Function Pipeline，简称固定管线**。通常指在较旧的 CPU 上实现的渲染流水线。这种流水线只给开发者提供一些配置操作，但开发者没有完全控制权。随着时代发展，可编程渲染管线应运而生。


# 第二章 Unity Shader
在没有 Unity 这类编辑器的情况下，若想对模型设置渲染状态，需要大量复杂初始化的代码。Unity 提供了能够让开发者管理着色器代码以及渲染设置（比如开启关闭混合、深度测试、设置渲染顺序等）的地方，也就是 Unity Shader。

## Material 和 Unity Shader

> 材质和着色器老是傻傻分不清，材质可以理解为物体的不同物理属性，比如颜色、纹理和反射率等等，而着色器根据这些数据渲染。

在 Unity 中需要配合使用**材质 Material 和 Unity Shader** 来达到需要的效果。首先创建需要的 Unity Shader 和材质，然后把 Unity Shader 赋给材质，并在材质面板上调整属性（如使用的纹理、漫反射系数等）。最后将材质赋给相应模型。Unity Shader 定义了渲染所需代码、属性和指令，而材质允许我们调节这些属性。

① Unity 中的材质  
Unity 中材质需要结合一个 GameObject 的 Mesh 或者 Particle Systems 组件来工作。默认情况下新建的材质使用 Unity 内置的 Standard Shader，是一种基于物理渲染的着色器，见后面章节。

② Unity 中的 Shader
Unity 一共提供了五种 Unity Shader 模板：  
&emsp;&emsp; - Standard Surface Shader：包含了标准光照模型的表面着色器模板；  
&emsp;&emsp; - Unlit Shader：不包含光照（但包含雾效）的基本的顶点/片元着色器；  
&emsp;&emsp; - Image Effect Shader：为我们实现各种屏幕后处理效果提供了一个基础模版；  
&emsp;&emsp; - Compare Shader：产生一种特殊的shader文件，利用GPU的并行性来进行一些与常规渲染流水线无关的计算；  
&emsp;&emsp; - Ray Tracing Shader：支持光追的着色器。

Unity Shader 本质上就是一个文本文件。Unity Shader 有导入设置面板，在 project 里点击任意 Unity Shader 可在 Inspector 面板中看到。

<table><tr>
<td><img src='https://s2.loli.net/2023/09/19/K4aG83wSf5PVhDJ.png' width="300" alt="图4- Unity Shader导入设置面板"></td>
<td><img src='https://s2.loli.net/2023/09/19/WQBKVwqGFHhfpvU.png' width="300" alt="图5- Compile and show code下拉列表"></td>
</tr></table>

可以在 Default Map 中指定该 Unity Shader 使用的默认纹理。在下方的面板中，Unity 会显示出和该 Unity Shader 相关的信息，例如它是否是一个表面着色器 Surface Shader、是否是一个固定函数着色器 Fixed Function Shader，还有些信息和标签设置有关，如是否会投射阴影、使用的渲染队列、LOD 值等。

对于表面着色器，点击 Show generated code 可以打开一个新的文件，该文件里将显示 Unity 在背后为该表面着色器生成的顶点/片元着色器。若该 Unity Shader 是固定函数着色器，在 Fixed function 后面也会出现 Show generated code 按钮。Compile and show code 下拉列表可以让开发者检查 Unity Shader 针对不同图像编程接口最终编译成的 Shader 代码。

## Unity Shader 的基础：ShaderLab
在 Unity 中，所有的 Unity Shader 都是使用 ShaderLab 来编写的。ShaderLab 是 Unity 提供的编写 Unity Shader 的一种说明性语言。使用了嵌套在花括号内部的语义来描述结构，这些结构包含了渲染所需的数据，比如 Properties 语句块中定义了着色器所需的各种属性。一个 Unity Shader 的基础结构如下：

```
Shader "ShaderName" {
    Properties {
      // 属性
    }
    SubShader {
      // 显卡A使用的子着色器
    }
    SubShader {
      // 显卡B使用的子着色器
    }
    Fallback "VertexLit"
}
```

Unity 会根据使用的平台来把这些结构编译成真正的代码和 Shader 文件。

## Unity Shader 的结构
### Unity Shader 的名字
每个 Unity Shader 文件的第一行都需要通过 Shader 语义来指定 Unity Shader 的名字。当为材质选择使用的 Unity Shader 时，名字就会出现在材质的 Inspector 面板的下拉列表里。通过在名字添加斜杠 / ，可以控制 Unity Shader 在材质面版中出现的位置：`Shader "Custom/MyShader"`。那么这个 Unity Shader 在材质面板的位置就是：Shader -> Custom -> MyShader。

### Properties 语义块
Properties 语义块中包含了一系列属性，这些属性会出现在材质面板中。Properties 语义块的定义通常如下：

```
Properties {
    Name ("display name", PropertyType) = DefaultValue
    Name ("display name", PropertyType) = DefaultValue
    // 更多属性
}
```

在 Shader 中需要访问材质属性的名字 Name，这些属性名字由一个下划线开始。display name 则是显示在材质面板上的名字。我们需要为每个属性指定它的类型和默认值。常见属性类型如下表：  

| 属性类型 | 默认值的定义语法 | 示例 |
| :---- | :---- | :---- |
| Int | number | _Int("Int",Int) = 2 |
| Float | number | _Float("Float",Float) = 1.5 |
| Range(min, max) | number | _Range("Range",Range(0.0, 5.0)) = 3.0 |
| Color | (number,number,number,number) | _Color("Color",Color) = (1,1,1,1) |
| Vector | (number,number,number,number) | _Vector("Vector",Vector) = (2,6,3,1) |
| 2D | "defaulttexture" { } | _2D("2D",2D) = "" { } |
| Cube | "defaulttexture" { } | _Cube("Cube",Cube) = "white" { } |
| 3D | "defaulttexture" { } | _3D("3D",3D) = "black" { } |

对于 2D、Cube、3D 这种纹理类型，默认值的定义稍微复杂，默认值是通过一个字符串后跟一个花括号来指定的，其中，字符串要么是空的，要么是内置的纹理名称，如 "white"，“black”，“gray" 或者 "bump”。

### SubShader
每一个 Unity Shader 可以包含多个 SubShader 语义块，但最少要有一个。 当 Unity 需要加载这个 Unity Shader 时，Unity 会扫描所有的 SubShader 语义块， 然后选择第一个能够在目标平台上运行的 SubShader。 如果都不支持的话，Unity 就会使用 Fallback 语义指定的 Unity Shader。Unity 提供这种语义的原因是因为不同显卡的能力不同，高级的显卡能支持更多的指令数，我们希望在差性能显卡使用计算复杂度较低的着色器，在高级显卡上使用计算复杂度较高的着色器。

SubShader 语义块通常如下：  

```
SubShader {
    // 可选的
    [Tags]

    // 可选的
    [RenderSetup]

    Pass {
    }
    // Other Passes
}
```

SubShader 中定义了一系列 Pass 以及可选的状态 \[RenderSetup\] 和标签 \[Tags\] 设置。每个Pass 定义了一次完整的渲染流程，但如果 Pass 的数目过多，往往会造成渲染性能的下降。 因此，我们应尽量使用最小数目的 Pass。状态和标签同样可以在 Pass 声明。不同的是，SubShader 中的一些标签设置是特定的。也就是说，这些标签设置和 Pass 中使用的标签是不一样的。而对于状态设置来说，其使用的语法是相同的。但是，如果我们在 SubShader 进行了这些设置，那么将会用于所有的 Pass。

***状态设置***  
ShaderLab 提供了一系列渲染状态的指令，这些指令可以设置显卡的各种状态，例如混合/开启深度测试等。常见的渲染状态设置选项如下表：

| 状态名称 | 设置指令 | 解释 |
| :---- | :---- | :---- |
| Cull | Cull Back &verbar; Front &verbar; Off | 设置剔除模式：剔除背面/正面/关闭剔除 |
| ZTest | ZTest Less Greator &verbar; LEqual &verbar; GEqual &verbar; Equal &verbar; NotEqual &verbar; Always | 设置深度测试时使用的函数 |
| ZWrite | ZWrite On &verbar; Off	| 开启/关闭深度写入 |
| Blend | Blend SrcFactor DstFactor | 开启并设置混合模式 |

当在 SubShader 块中设置了上述渲染状态时，将会应用到所有的 Pass，如果不想这样，可以在 Pass 语义块中单独进行上面的设置。

***SubShader 的标签***  
标签 Tags 是一个键值对，键和值都是字符串类型，这些键值对用来告诉 Unity 的渲染引擎希望怎样以及何时渲染这个对象。标签结构：`Tags { "TagName1" = "Value1" "TagName2" = "Value2" }`

SubShader 的标签块支持的标签类型如下：

| <font size=2>标签类型</font> | <font size=2>说明</font> | <font size=2>例子</font> |
| :---- | :---- | :---- |
| <font size=2>Queue</font> | <font size=2>控制渲染顺序，指定该物体属于哪个渲染队列，通过这种方式可以保证所有的透明物体可以在所有不透明物体后面被渲染，我们也可以自定义使用的渲染队列来控制物体的渲染顺序</font>	| <font size=2>Tags {  "Queue" = "Transparent" }</font> |
| <font size=2>RenderType</font> | <font size=2>对着色器进行分类，例如这是一个不透明的着色器，或是一个透明的着色器等。可以被用于着色器替换 Shader Replacement 功能</font> | <font size=2>Tags { "RenderType" = "Opaque" }</font> |
| <font size=2>DisableBatching</font> | <font size=2>一些 SubShader 在使用 Unity 的批处理功能时会出现问题，例如使用了模型空间下的坐标进行顶点动画。这时可以通过该标签来直接指明是否对该 SubShader 使用批处理</font> | <font size=2>Tags { "DisableBatching" = "True" }</font> |
| <font size=2>ForceNoShadowCasting</font> | <font size=2>控制使用该 SubShader 的物体是否会投射阴影</font> | <font size=2>Tags { "ForceNoSdowCasting" = "True" }</font> |
| <font size=2>lgnoreProjector</font> | <font size=2>如果该标签值为 “True” 那么使用该 SubShader 的物体将不会受 Projector 的影响。通常用于半透明物体</font> | <font size=2>Tags { "lgnoreProjector" = "True" }</font> |
| <font size=2>CanUseSpriteAtlas</font>	| <font size=2>当该 SubShader 是用于精灵 sprites 时，将该标签设为 “False”</font> | <font size=2>Tags { "CanUseSpriteAtlas" = "False" }</font> |
| <font size=2>PreviewType</font> | <font size=2>指明材质面板将如何预览该材质。默认情况下，材质将显示为一个球形，我们可以通过把该标签的值设为 “Plane” “SkyBox” 来改变预览类型</font> | <font size=2>Tags { "PreviewType" = "Plane" }</font> |

注意：上述标签可以在 SubShader 中声明，而不可以在 Pass 块中声明，Pass 块虽然也可以定义标签，但这些标签是不同于 SubShader 的标签类型。

***Pass 语义块***

