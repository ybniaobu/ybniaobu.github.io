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

> 一个 SubShader 可理解为 Shader 中的一个渲染方案。即针对不同的渲染情况，需要编写不同的子着色器。  
> 
> In a shader program, a **"pass"** refers to a single rendering operation that is performed on a set of geometry or pixels. It can include multiple shader stages, such as vertex, geometry, and fragment shaders. 

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

**SubShader 中定义了一系列 Pass 以及可选的状态 \[RenderSetup\] 和标签 \[Tags\] 设置。每个 Pass 定义了一次完整的渲染流程**，但如果 Pass 的数目过多，往往会造成渲染性能的下降。 因此，我们应尽量使用最小数目的 Pass。状态和标签同样可以在 Pass 声明。不同的是，SubShader 中的一些标签设置是特定的。也就是说，这些标签设置和 Pass 中使用的标签是不一样的。而对于状态设置来说，其使用的语法是相同的。但是，如果我们在 SubShader 进行了这些设置，那么将会用于所有的 Pass。

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
Pass 语义块包含的语义如下：  

``` 
Pass {
	  [Name]
	  [Tags]
	  [RenderSetup]
	  // Other Code
}
```

我们可以在 Pass 中定义该 Pass 的名称，例如：`Name "MyPassName"`。通过这个名称，我们可以使用 ShaderLab 的 UsePass 命令来直接使用其他 Unity Shader 中的 Pass，例如：`UsePass "MyShader/MYPASSNAME"`。由于 Unity 内部会把所以 Pass 的名称转换成大写字母表示，因此，在使用 UsePass 命令时必须使用大写形式的名字。

可以对 Pass 设置渲染状态， SubShader 的状态设置同样适用于 Pass，除此之外，还可以使用固定管线的着色器命令，见后面。Pass 同样可以设置标签，但它的标签不同于 SubShader 的标签。这些标签也是用于告诉渲染引擎我们希望怎么样来渲染该物体。

| 标签类型 | 说明 | 例子 |
| :---- | :---- | :---- |
| LightMode | 定义该 Pass 在 Unity 的渲染流水线中的角色 | Tags { "LightMode" = "ForwardBase" } |
| RequireOptions | 用于指定当满足某些条件时才渲染该 Pass, 它的值是一个由空格分隔的字符串。目前， Unity 支待的选项有：SoftVegetation 。在后面的版本中，可能会增加更多的选项 | Tags { "RequireOptions" = "SoftVegetation" } |

Unity Shader 还支持一些特殊的 Pass：  
UsePass：可以使用该命令来复用其他 Unity Shader 中的 Pass；  
GrabPass：该 Pass 负责抓取屏幕并将结果储存在一张纹理中，以用于后续的 Pass 处理。

### Fallback
Fallback 指令用于告诉 Unity：如果上面所有 SubShader 不能在该显卡上运行，那么就使用这个最低级的 Shader 吧。它的语义如下：  

```
Fallback "name"
// 或者
Fallback off
```

事实上 Fallback 还会影响阴影的投射。在渲染阴影纹理时，Unity 会在每个 Unity Shader 中寻找一个阴影投射的 Pass。通常情况下，我们不需要自己专门实现一个 Pass，Fallback 使用的内置 Shader中包含一个通用的 Pass。因此，为每个 UnityShader 正确设置 Fallback 是非常重要的。

### 其他语义
除了上述语义，还有一些不常用的语义：比如，使用 CustomEditor 语义来扩展材质面板的编辑界面；使用 Category 语义来对 Unity Shader 中的命令进行分组。


## Unity Shader 的形式
Unity Shader 最重要的任务还是指定各种着色器所需的代码。这些着色器代码可以写在 SubShader 语义块里（表面着色器的做法），也可以写在 Pass 语义块里（顶点/片元着色器和固定函数着色器的做法）。

### 表面着色器 Surface Shader
表面着色器是 Unity 自己创造的着色器类型，代码量很少，但是 Unity 在背后做了很多工作，渲染代价比较大。本质上与顶点/片元着色器相同，Unity 内部会将该着色器转换为对应顶点/片元着色器，表面着色器是对顶点/片元着色器更高一层的抽象，Unity 为我们处理了很多光照细节。一个非常简单的表面着色器示例代码如下：

```
Shader "Custom/Simple Surface Shader" {
    SubShader {
        Tags { "RenderType" = "Opaque" }
        CGPROGRAM
        #progma surface surf Lambert
        struct Input{
            float color : COLOR;
        };
        void surf (Input IN, inout SurfaceOutput o) {
            o.Albedo = 1;
        }
        ENDCG
    }
    Fallback "Diffuse"
}
```

表面着色器被定义在 SubShader 语义块中的 CGPROGRAM 和 ENDCG 之间，表面着色器不需要关心使用多少个 Pass、每个 Pass 如何渲染等问题，Unity 会在背后做好这些事情，把表面着色器转换成一个包含多 Pass 的顶点/片元着色器。我们只需要告诉表面着色器使用哪些纹理填充颜色，使用哪个法线纹理去填充法线，使用 Lambert 光照模型等。

**CGPROGRAM 和 ENDCG 之间的代码使用的是 CG/HLSL 编写的，需要将 CG/HLSL 语言嵌入 ShaderLab 语言中。**但是这里的 CG/HLSL 是 Unity 封装后的，它的语法和标准的 CG/HLSL 几乎一致，但是有细微差别，有些函数和用法 Unity 没有提供支持。

### 顶点/片元着色器 Vertex/Fragment Shader
在 Unity 中，我们可以使用 Cg/HLSL 语言来编写顶点/片元着色器。它们更加复杂，但灵活性也更高。一个非常简单的顶点/片元着色器示例代码如下：  

```
Shader "Custom/Simple Surface Shader" {
    SubShader {
        Pass { 
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            float4 vert(float4 v : POSITION) : SV_POSITION {
                return mul (UNITY_MATRIX_MVP, v);
            }

            fixed4 frag() : SV_Target {
                return fixed4(1.0, 0.0, 0.0, 1.0);
            }

            ENDCG
        }
    }
}
```

和表面着色器类似，顶点/片元着色器的代码也需要定义在 CGPROFRAM 和 ENDCG 之间。但不同的是，顶点/片元着色器是写在 Pass 语义块中，而非 SubShader 内的。我们需要自己定义每个 Pass 需要使用的 Shader 代码。

### 固定函数着色器 Fixed Function Shader
对于一些较旧的设备，不支持可编程管线着色器。因此，我们需要使用固定函数着色器来完成渲染。这些着色器往往只可以完成一些非常简单的效果。一个非常简单的固定函数着色器示例代码如下：  

```
Shader "Tutorial/Basic" {
    Properties {
        _Color ("Main Color", Color) = (1, 1, 1, 1)
    }
    SubShader {
        Pass {
            Material {
                Diffuse [_Color]
            }
            Lighting On
        }
    }
}
```

固定函数着色器的代码被定义在 Pass 语义块内，相当于 Pass 的一些渲染设置。对于固定函数着色器，需要完全使用 ShaderLab 语法，而非 Cg/HLSL 。

由于绝大多数 GPU 都支持可编程的渲染管线，这种固定渲染管线的编程方式已经逐渐被抛弃了。实际上，在现在的 Unity 中，所有固定函数着色器都会在背后被 Unity 编译成对应的顶点/片元着色器，因此，真正意义上的固定函数着色器已经不存在了。

### 选择哪种着色器
①除非你有非常明确的需求必须要使用固定函数着色器，例如需要在非常旧的设备上运行你的游戏，否则请使用可编程管线的着色器，即表面着色器或顶点/片元着色器；  
②如果你想和各种光源打交道，你可能更喜欢使用表面着色器，但需要小心它在移动平台的性能表现；  
③如果你需要使用的光照数目非常少，例如只有一个平行光，那么使用顶点/片元着色器是一个更好的选择；  
④如果你有很多自定义的渲染效果，那么请选择顶点/片元着色器。

## 其他补充
### Unity Shader 和真正的 Shader 的区别
①在 Shader 中，我们仅可以编写特定类型的 Shader，例如顶点着色器、片元着色器等。而在 Unity Shader 中，我们可以在同一个文件里同时包含需要的顶点着色器和片元着色器代码；  
②在 Shader 中，我们无法设置一些渲染设置，例如是否开启混合、深度测试等，这些是开发者在另外的代码中自行设置的。而在 Unity Shader 中，我们通过一行特定的指令就可以完成这些设置；  
③在 Shader 中，我们需要编写冗长的代码来设置着色器的输入和输出，要小心地处理这些输入输出的位置对应关系等。而在 Unity Shader 中，我们只需要在特定语句块中声明一些属性，就可以依靠材质来方便地改变这些属性。而且对于模型自带的数据（如顶点位置、纹理坐标、法线等），Unity Shader 也提供了直接访问的方法，不需要开发者自行编码来传给着色器。

由于 Unity Shader 的高度封装性，可以编写的 Shade 类型和语法被限制了，对应一些特定类型的 Shader 如曲面细分着色器和几何着色器等相关功能就支持得差一点。作为开发者，我们绝大部分时候只需要和 Unity Shader 打交道，而不需要关心渲染引擎底层得实现细节。


# 第三章 Shader 数学基础
## 笛卡儿坐标系
我们平时使用的是**笛卡儿坐标系 Cartesian Coordinate System**。在三维笛卡儿坐标系中，需要定义三个坐标轴和一个原点。这三个坐标轴也就是该坐标系的**基矢量 basis vector**。若这三个坐标轴是相互垂直的，且长度为1，这样的基矢量被称为**标准正交基 orthonormal basis**，但这不是必须的。若这三个坐标轴仅仅相互垂直的，这样的基矢量被称为**正交基 orthogonal basis**。

三维笛卡儿坐标系并不都是等价的，具有**旋向性 handedness**，即分为**左手坐标系 left-handed coordinate space** 和**右手坐标系 right-handed coordinate space**。除了坐标轴朝向不同之外，左手坐标系和右手坐标系对于正向旋转的定义也不同，即**左手法则 left-handed rule**和**右手法则 right-handed rule**。

在 Unity 中使用的坐标系就是左手坐标系。但在观察空间使用右手坐标系，观察空间是以摄像头为原点的坐标系，摄像头的前方是 z 轴的负方向，z 轴坐标的减少意味着景深的增加。

## 点和矢量
**矢量或向量 vector** 与**标量 scalar** 区分。矢量指 n 维空间中一种包含了**模 magnitude** 和方向的有向线段。

矢量的运算此处不再摘录，见 [Unity基础 - 三维数学基础](https://ybniaobu.github.io/2023/07/09/2023-07-09-Unity%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86%E6%9D%82%E8%AE%B0/#%E4%B8%89%E7%BB%B4%E6%95%B0%E5%AD%A6%E5%9F%BA%E7%A1%80)。

## 矩阵
矩阵的运算此处也不再摘录，见 [线性代数基础](https://ybniaobu.github.io/2023/07/12/2023-07-12-%E7%BA%BF%E6%80%A7%E4%BB%A3%E6%95%B0%E5%9F%BA%E7%A1%80/)。

在 Shader 的计算中，经常使用 4X4 的矩阵运算。

### 特殊的矩阵
***1. 方块矩阵 square matrix***  
方块矩阵，简称**方阵**，即行列数相等的矩阵。矩阵的某些运算和性质只有方阵才有。

如果一个矩阵除了对角元素外的所有元素都为 0，那么该矩阵称为**对角矩阵 diagonal matrix**。

***2. 单位矩阵 identity matrix***  
使用 I 来表示，MI = IM = M。

***3. 转置矩阵 transposed matrix***  
写作 $M^T$ ，公式为 $M_{ij}^T = M_{ji}$。转置矩阵有如下性质：  
①$(M^T)^T=M$  
②$(AB)^T=B^TA^T$  

***4. 逆矩阵 inverse matrix***  
给定一个方阵 M，$MM^{-1}=M^{-1}M=I$。但并非所有方阵都有对应的逆矩阵，若一个矩阵的**行列式 determinant** 不为 0，那么它就是可逆的。若一个矩阵有对应的逆矩阵，就可以说这个矩阵是**可逆的 invertible**或者说是**非奇异的 nonsingular**。若一个矩阵没有对应的逆矩阵，它就是**不可逆的 noninvertible** 或者说是**奇异的 singular**。

逆矩阵有如下性质：  
①$(M^{-1})^{-1}=M$  
②$I^{-1}=I$  
③$(M^{T})^{-1}=(M^{-1})^{T}$  
④$(AB)^{-1}=B^{-1}A^{-1}$  

在写 Shader 的过程中，矩阵运算可以调用第三方库（如 C++ 数学库 Eigen）。在 Unity 中也有内置的工具。

***5. 正交矩阵 orthogonal matrix***  
如果一个方阵 M 和它的转置矩阵的乘积是单位矩阵的话，则这个矩阵是**正交的 orthogonal**。即 $MM^{T}=M^{T}M=I$ ，$M^{T}=M^{-1}$。

上述公式比较重要，因为逆矩阵的求解运算量很大，而转置矩阵只需要将矩阵翻转即可。

正交矩阵的特点：根据正交矩阵的定义，可以写成以下公式（其中 c 为向量）：  

$$ M^TM = \begin{bmatrix} - & c_1 & - \\ - & c_2 & - \\ - & c_3 & - \end{bmatrix} \begin{bmatrix} | & | & | \\ c_1 & c_2 & c_3 \\ | & | & | \end{bmatrix} = \begin{bmatrix} c_1 \cdot c_1 & c_1 \cdot c_2 & c_1 \cdot c_3 \\ c_2 \cdot c_1 & c_2 \cdot c_2 & c_2 \cdot c_3 \\ c_3 \cdot c_1 & c_3 \cdot c_2 & c_3 \cdot c_3 \end{bmatrix} = \begin{bmatrix} 1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix} = I $$  

可以得到9个等式：$c_1 \cdot c_1 = 1$，$c_1 \cdot c_2 = 0$，$c_1 \cdot c_3 = 0$，$c_2 \cdot c_1 = 0$，$c_2 \cdot c_2 = 1$，$c_2 \cdot c_3 = 0$，$c_3 \cdot c_1 = 0$，$c_3 \cdot c_2 = 0$，$c_3 \cdot c_3 = 1$。

根据以上等式，可以得出以下结论：  
①矩阵的每一行，即 $c_1$，$c_2$，$c_3$ 都是单位矢量，因为只有这样它们与自己的点积才能是1；  
②矩阵的每一行，即 $c_1$，$c_2$，$c_3$ 之间相互垂直，因为只有这样他们之间的点积才是0；  
③上述两条结论对于矩阵的每一列都适用，所以正交矩阵的转置矩阵也是正交矩阵。  
④**一个正交矩阵的行和列分别构成了一组标准正交基**。若使用一组正交基来构建一个矩阵，这个矩阵可能不是一个正交矩阵，因为基矢量的长度可能不为 1。

### 行向量还是列向量
行向量还是列向量与矩阵相乘会有一些差异：假设一个向量 v = (x, y, z)，和另一个矩阵 M：

$$ M = \begin{bmatrix} m_{11} & m_{12} & m_{13} \\ m_{21} & m_{22} & m_{23} \\ m_{31} & m_{32} & m_{33} \end{bmatrix} $$

①行向量乘法，向量在矩阵左边：  

$$ vM = \begin{bmatrix} xm_{11}+ym_{21}+zm_{31} & xm_{12}+ym_{22}+zm_{32} & xm_{13}+ym_{23}+zm_{33} \end{bmatrix} $$  

②列向量乘法，矩阵在向量左边：  

$$ Mv = \begin{bmatrix} xm_{11}+ym_{12}+zm_{13} \\ xm_{21}+ym_{22}+zm_{23} \\ xm_{31}+ym_{32}+zm_{33} \end{bmatrix} $$  

比较一下可以知道，列向量和行向量得到的结果不一样。这意味着在和矩阵相乘时，选择行向量还是列向量是非常重要的，会影响矩阵乘法的结果。DirectX 使用的是行向量，openGL 使用的也是列向量。

在 Unity 中使用的是列向量，之后的内容，如无特殊情况，都使用列矩阵。即 $ CBAv = (C(B(Av))) $。该公式等价于下面的行向量运算：$ vA^TB^TC^T = (((vA^T)B^T)C^T) $。若矩阵是对称矩阵，那么列向量和行向量不会对结果产生影响，因为对称矩阵的转置是其本身。

## 矩阵变换
### 变换
变换指把一些数据，如点、方向甚至是颜色等，通过某种方式进行转换的过程。在计算机图形学中，变换举足轻重。

**线性变换 linear transform** 指的是可以保留矢量加和标量乘的变换。**缩放 scale**、**旋转 rotation**、**错切 shear**、**镜像 reflection**、**正交投影 orthographic projection** 都是线性变换。

但是平移变换就不是线性变换。因此我们不能用一个 3 × 3 的矩阵来表示平移变换。由于平移变换是非常常见的一个变换，这样就有了**仿射变换 affine transform**，仿射变换就是合并线性变换和平移变换的变换类型。仿射变换可以使用一个 4 × 4 的矩阵来表示，即需要将矢量扩展到四维空间中，这就是**齐次坐标空间 homogeneous space**。

下表给出了图形学中常见变换矩阵的名称和它们的特性：  

| 变换名称 | 是线性变换吗 | 是仿射变换吗 | 是可逆矩阵吗 | 是正交矩阵吗 |
| :---- | :----: | :----: | :----: | :----: |
| 平移矩阵 | N | Y | Y | N |
| 绕坐标轴旋转的旋转矩阵 | Y | Y | Y | Y |
| 绕任意轴旋转的旋转矩阵 | Y | Y | Y | Y |
| 按坐标轴缩放的缩放矩阵 | Y | Y | Y | N |
| 错切矩阵 | Y | Y | Y | N |
| 镜像矩阵 | Y | Y | Y | Y |
| 正交投影矩阵 | Y | Y | N | N |
| 透视投影矩阵 | N | N | N | N |


### 齐次坐标
由于 3 × 3 的矩阵不能表示平移操作，就把其扩展到 4 × 4 的矩阵。为此需要把三维矢量转换为四维矢量，即齐次坐标 homogeneous coordinate（书中所说的齐次坐标泛指四维齐次坐标）。齐次坐标可以理解为为了方便计算而使用的一个表达方式。

将三维坐标转换为齐次坐标：对于一个点，x，y，z 坐标不变，将 w 坐标设为 1，即 $[x , y , z , 1]$；对于一个方向矢量，x，y，z 坐标不变，将 w 坐标设为 0，即 $[x , y , z , 0]$。这样就会可以，当使用矩阵对点变换时，平移、旋转、缩放都可以施加于该点；若用于一个向量，平移效果会被忽略。

### 分解基础变换矩阵
纯平移、纯缩放、纯旋转的变换矩阵叫做基础变换矩阵。可以把一个基础变换矩阵分解为 4 个组成部分：

$$ \begin{bmatrix} M_{3 \times 3} & t_{3 \times 1} \\ 0_{1 \times 3} & 1 \end{bmatrix} $$

左上角的矩阵 $ M_{3\times3} $ 用于表示旋转和缩放，$t_{3\times1}$ 用于表示平移，$0_{1\times3}$ 表示零矩阵，右下角元素为标量1。

### 平移矩阵
使用矩阵乘法来表示对一个 *点* 的平移变换：

$$ \begin{bmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix} = \begin{bmatrix} x + t_x \\ y + t_y \\ z + t_z \\ 1 \end{bmatrix} $$

点的x、y、z分量分别增加一个位置偏移，可以看作点 $(x, y, z)$ 在坐标空间被平移了 $(t_x,t_y,t_z)$ 个单位。

若使用矩阵乘法来表示对一个 *向量* 的平移变换：  

$$ \begin{bmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ z \\ 0 \end{bmatrix} = \begin{bmatrix} x \\ y \\ z \\ 0 \end{bmatrix} $$

可以发现，平移变换不会对向量产生任何影响。

平移矩阵的逆矩阵就是反向平移得到的矩阵（可以看出，平移矩阵不是一个正交矩阵）：  

$$ \begin{bmatrix} 1 & 0 & 0 & -t_x \\ 0 & 1 & 0 & -t_y \\ 0 & 0 & 1 & -t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

### 缩放矩阵
使用矩阵乘法来表示对一个 *点* 的缩放变换：  

$$ \begin{bmatrix} k_x & 0 & 0 & 0 \\ 0 & k_y & 0 & 0 \\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ z \\ 1 \end{bmatrix} = \begin{bmatrix} k_xx \\ k_yy \\ k_zz \\ 1 \end{bmatrix} $$

使用矩阵乘法来表示对一个 *向量* 的缩放变换： 

$$ \begin{bmatrix} k_x & 0 & 0 & 0 \\ 0 & k_y & 0 & 0 \\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} x \\ y \\ z \\ 0 \end{bmatrix} = \begin{bmatrix} k_xx \\ k_yy \\ k_zz \\ 0 \end{bmatrix} $$

如果缩放系数 $k_x = k_y = k_z$ ，把这样的缩放叫做**统一缩放 uniform scale**，否则称为**非统一缩放 ununiform scale**。统一缩放是扩大整个模型，非统一缩放会拉伸或挤压模型。统一缩放不会改变角度和比例信息，而非统一缩放会改变与模型相关角度和比例。例如对法线进行变换时，如果存在非统一缩放，直接使用用于变换顶点的变换矩阵，会产生错误结果。

缩放矩阵的逆矩阵是使用原缩放系数的倒数来对点或向量进行缩放（缩放矩阵一般不是正交矩阵）：  

$$ \begin{bmatrix} \frac {1}{k_x} & 0 & 0 & 0 \\ 0 & \frac {1}{k_y} & 0 & 0 \\ 0 & 0 & \frac {1}{k_z} & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

上面的矩阵只适用于沿坐标轴方向进行缩放。若需要在任意方向上进行缩放，就需要使用复合变换。其中一个方法的主要思想就是，先将缩放轴变换成标准坐标轴，然后进行沿坐标轴缩放，再使用逆变换得到原来的缩放轴朝向。

### 旋转矩阵
①绕 x 轴旋转 θ 度：  

$$ R_x(θ) = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & \cos θ & - \sin θ & 0 \\ 0 & \sin θ & \cos θ & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

②绕 y 轴旋转 θ 度： 

$$ R_y(θ) = \begin{bmatrix} \cos θ & 0 & \sin θ & 0 \\ 0 & 1 & 0 & 0 \\ - \sin θ & 0 & \cos θ & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

③绕 z 轴旋转 θ 度： 

$$ R_z(θ) = \begin{bmatrix} \cos θ & - \sin θ & 0 & 0 \\ \sin θ & \cos θ & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

旋转矩阵的逆矩阵是旋转相反角度得到的变换矩阵。旋转矩阵是正交矩阵，而且多个旋转矩阵串联同样是正交的。

### 复合变换
将平移，旋转，缩放组合起来，形成一个复杂的变换过程（假设 y 轴旋转）：

$$ P_{new} = M_{translation}M_{rotation}M_{scale}P_{old} = \begin{bmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} \cos θ & 0 & \sin θ & 0 \\ 0 & 1 & 0 & 0 \\ - \sin θ & 0 & \cos θ & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} k_x & 0 & 0 & 0 \\ 0 & k_y & 0 & 0 \\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

$$ = \begin{bmatrix} k_x\cos θ & 0 & k_z \sin θ & t_x \\ 0 & k_y & 0 & t_y \\ - k_x \sin θ & 0 & k_z \cos θ & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

绝大多数情况下，我们约定变换的顺序即为**先进行缩放，再进行旋转，最后进行平移**。这样约定有原因的，比如初始位置为原点，先按 (0, 0, 5) 平移，再按坐标轴放大 2 倍，变为了 (0, 0, 10)，这不是我们想要的，应该先在原点缩放，再进行平移。

除了注意不同类型的变换顺序之外，也要注意**旋转的变换顺序**。不同的旋转顺序会导致不同结果，所以当给定旋转角度 $θ_x, θ_y, θ_z$ 时，需要定义一个旋转顺序。在 Unity 中，旋转顺序按照先绕 z 轴旋转，再绕 x 轴旋转，最后绕 y 轴旋转。

旋转顺序的定义可以有2种方式：  
①绕坐标系 E 下的 z 轴旋转 $θ_z$，绕坐标系 E 下的 x 轴旋转 $θ_x$，绕坐标系 E 下的 y 轴旋转 $θ_y$，即：进行一次旋转时不一起旋转当前坐标系。（全局坐标旋转）  
②绕坐标系 E 下的 z 轴旋转 $θ_z$，绕坐标系 E 的 z 轴旋转后的新坐标系 E' 下的 x 轴旋转 $θ_x$，绕坐标系 E' 下绕 x 轴旋转后的新坐标系 E'' 下的 y 轴旋转 $θ_y$，即：在旋转时，把坐标系一起转动。（局部坐标旋转）

方式 1 按 zxy 顺序旋转，和方式 2 按 yxz 旋转的效果是完全一样的，而 Unity 中按 zxy 顺序旋转指的是在方式 1 情况下的旋转顺序，相当于对应方式 2 的 yxz 顺序，即组合旋转变换矩阵为 $M_{rotate_y}M_{rotate_x}M_{rotate_z}$

> 上述说明符合：**基于全局坐标系的旋转变换左乘旋转矩阵，基于自身坐标系的旋转变换右乘旋转矩阵**。


## 坐标空间
在渲染管线中有提到坐标空间的变换，可以回去查阅。顶点着色器的基本功能就是把模型的顶点坐标从模型空间转换到齐次裁剪坐标空间中。

### 坐标空间的变换
坐标空间会形成层次结构，即每个坐标空间都是另一个坐标空间的子空间，每个空间都有一个父坐标空间。对坐标空间的变换本质上就是在父子空间之间变换。

假设，有父坐标空间 P 以及一个子坐标空间 C 。一般会有两种需求：一种需求是把子坐标空间下表示的点或向量 $A_c$ 转换到父坐标空间下的 $A_p$ 。另一个需求，即把父坐标空间下表示的点或向量 $B_p$ 转换到子坐标空间下的 $B_c$ 。公式表示如下：  

$$ A_p = M_{c \rightarrow p} A_c $$
$$ B_c = M_{p \rightarrow c} B_p $$

其中，$M_{p \rightarrow c}$ 是 $M_{c \rightarrow p}$ 的逆矩阵（反向变换）。只需要解出两者之一，另一个矩阵可以通过求解逆矩阵得到。

接下来说明如何求解从子坐标空间到父坐标空间的变换矩阵 $M_{c \rightarrow p}$ ：假设已知子坐标空间 C 的 3 个坐标轴在父坐标空间 P 下的表示为向量 $x_c$、$y_c$、$z_c$，以及子坐标原点位置 $O_c$。当给定一个子坐标空间中的一点 $A_c = (a, b, c)$，求其在父坐标空间下的位置 $A_p$ ，可以得到：  

$$ \begin{aligned} A_p &= O_c + ax_c + by_c + cz_c \\ &= (x_{O_c}, y_{O_c}, z_{O_c}) + a(x_{x_c}, y_{x_c}, z_{x_c}) + b(x_{y_c}, y_{y_c}, z_{y_c}) + c(x_{z_c}, y_{z_c}, z_{z_c}) \\ &= (x_{O_c}, y_{O_c}, z_{O_c}) + \begin{bmatrix} x_{x_c} & x_{y_c} & x_{z_c} \\ y_{x_c} & y_{y_c} & y_{z_c} \\ z_{x_c} & z_{y_c} & z_{z_c} \end{bmatrix} \begin{bmatrix} a \\ b \\ c \end{bmatrix} \\ &= (x_{O_c}, y_{O_c}, z_{O_c}) + \begin{bmatrix} | & | & | \\ x_c & y_c & z_c \\ | & | & | \end{bmatrix} \begin{bmatrix} a \\ b \\ c \end{bmatrix} \end{aligned} $$

再把上述公式的加法（平移变换）用齐次坐标表示：  

$$ \begin{aligned} A_p &= (x_{O_c}, y_{O_c}, z_{O_c}, 1) + \begin{bmatrix} | & | & | & 0 \\ x_c & y_c & z_c & 0 \\ | & | & | & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} a \\ b \\ c \\ 1 \end{bmatrix} \\ &= \begin{bmatrix} 1 & 0 & 0 & x_{O_c} \\ 0 & 1 & 0 & y_{O_c} \\ 0 & 0 & 1 & z_{O_c} \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} | & | & | & 0 \\ x_c & y_c & z_c & 0 \\ | & | & | & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} a \\ b \\ c \\ 1 \end{bmatrix} \\ &= \begin{bmatrix} | & | & | & x_{O_c} \\ x_c & y_c & z_c & y_{O_c} \\ | & | & | & z_{O_c} \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} a \\ b \\ c \\ 1 \end{bmatrix} \\ &= \begin{bmatrix} | & | & | & | \\ x_c & y_c & z_c & O_c \\ | & | & | & | \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} a \\ b \\ c \\ 1 \end{bmatrix} \end{aligned} $$

由此得到变换矩阵 $M_{c \rightarrow p}$ ，可以看出该变换矩阵实际上就是将坐标空间 C 在坐标空间 P 的三个坐标轴向量依次放入矩阵的前三列，再把原点坐标放入第四列即可（三个坐标轴向量不一定是单位向量）。

在 Shader 中，会经常看到截取变换矩阵的前三列来对法线方向、光照方向进行空间变换，因为方向没有位置，因此坐标空间的原点变换可以忽略。

如果 $M_{c \rightarrow p}$ 是一个正交矩阵（即三轴为标准正交基），那么 $M_{p \rightarrow c}$ 只需要求 $M_{c \rightarrow p}$ 的转置就可以得到，计算转置比求逆简单：

$$ M_{p \rightarrow c} = \begin{bmatrix} | & | & | \\ x_p & y_p & z_p \\ | & | & |  \end{bmatrix} = M_{c \rightarrow p}^{-1} = M_{c \rightarrow p}^{T} = \begin{bmatrix} - & x_c & - \\ - & y_c & - \\ - & z_c & - \end{bmatrix} $$

上述公式可以得出，若 $M_{A \rightarrow B}$ 是一个正交矩阵，那么可以提取它的第一列来得到坐标空间 A 的 x 轴在坐标空间 B 下的表示，还可以提取它的第一行来得到坐标空间 B 的 x 轴在坐标空间 A 下的表示。反过来，无论我们知道 B 空间的 x 轴、y 轴和 z 轴在坐标空间 A 下的表示，还是 A 空间在 B 空间下的表示，都可以得到坐标 A 到 B 的变换矩阵，只是一个按列排放，一个按行排放。

> 若要得到 A 到 B 的变换矩阵，知道 A 在 B 的表示，按列排放；知道 B 在 A 的表示，按行排放。可以快速验证变换矩阵按行还是列摆放是否正确，即使用 $M_{A \rightarrow B}$ 来变换 $x_B$，其结果应该是 $(1, 0, 0)$。

### 顶点的坐标空间变换过程
在渲染管线中，一个顶点需要经过多个坐标空间变换才能最终画在屏幕上。一个顶点最开始是在模型空间中定义的，最后会变换到屏幕空间中，得到真正的屏幕像素坐标。

### 模型空间 model space
模型空间也称为**对象空间 object space** 或者**局部空间 local space**。当模型移动或旋转时，模型空间也会跟随移动或旋转。在 Unity 中使用左手坐标系来定义模型空间。模型空间的原点和坐标轴通常由美术人员在建模软件里确定好，导入 Unity 后，在顶点着色器中访问到的顶点的坐标都是相对于模型空间的原点（通常为模型重心）定义的。

### 世界空间 world space
世界空间即 Unity 的全局坐标系，在 Unity 中，世界空间同样使用左手坐标系，其 x 轴、y 轴、z 轴是固定不变的。在 Unity 中，可以通过调整 Transform 组件中的 Position 属性来改变模型的位置，这里的位置指相对于该模型父节点的原点定义的，即其父节点的模型空间中的位置。若该 Transform 没有任何父节点，那么这个位置就是在世界坐标系中的位置。Rotation 和 Scale 也是同理。

顶点变换的第一步，就是将顶点坐标从模型空间变换到世界空间，这个变换通常叫做**模型变换 model transform**。

假设 Unity 中一个模型的一个顶点的模型坐标为 (0, 2, 4)。该模型没有父节点，其 Transform 组件上的信息为 Position：(5, 0, 25)；Rotation：(0, 150, 0)；Scale：(2, 2, 2)。注意变换的顺序，先缩放，再旋转，最后平移。据此构建变换矩阵：

$$ \begin{aligned} M_{model} &= \begin{bmatrix} 1 & 0 & 0 & t_x \\ 0 & 1 & 0 & t_y \\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} \cos θ & 0 & \sin θ & 0 \\ 0 & 1 & 0 & 0 \\ - \sin θ & 0 & \cos θ & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} k_x & 0 & 0 & 0 \\ 0 & k_y & 0 & 0 \\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \\ &= \begin{bmatrix} 1 & 0 & 0 & 5 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 25 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} -0.866 & 0 & 0.5 & 0 \\ 0 & 1 & 0 & 0 \\ -0.5 & 0 & -0.866 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} 2 & 0 & 0 & 0 \\ 0 & 2 & 0 & 0 \\ 0 & 0 & 2 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix} \\ &= \begin{bmatrix} -1.732 & 0 & 1 & 5 \\ 0 & 2 & 0 & 0 \\ -1 & 0 & -1.732 & 25 \\ 0 & 0 & 0 & 1 \end{bmatrix} \end{aligned} $$

接下来对顶点进行模型变换：  

$$ P_{world} = M_{model}P_{model} = \begin{bmatrix} -1.732 & 0 & 1 & 5 \\ 0 & 2 & 0 & 0 \\ -1 & 0 & -1.732 & 25 \\ 0 & 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} 0 \\ 2 \\ 4 \\ 1 \end{bmatrix} = \begin{bmatrix} 9 \\ 4 \\ 18.072 \\ 1 \end{bmatrix} $$

故该顶点的世界坐标为 (9, 4, 18.072)。

### 观察空间 view space
观察空间也称为**摄像机空间 camera space**