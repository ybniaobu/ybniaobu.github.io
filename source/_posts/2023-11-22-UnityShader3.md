---
title: 《Unity Shader入门精要》读书笔记（三）
date: 2023-11-22 16:32:01
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
top_img: /images/black.jpg
cover: https://s2.loli.net/2023/11/23/L3ts4WnThMlDN9d.gif
mathjax: true
---

> 本读书笔记为中级篇，主要内容为更复杂的光照的前向渲染、延迟渲染、光照衰减和阴影；高级纹理的立方体纹理、渲染纹理和程序纹理；动画的纹理动画和顶点动画。  
> 读书笔记是对知识的记录与总结，但是对比较熟悉的内容不会再行描述。

# 第八章 更复杂的光照
在前面的学习中，场景中仅有一个光源且光源类型是平行光。但是在实际的游戏开发中，往往需要处理数量更多、类型更复杂的光源。更重要的是，想要得到阴影。本章最后会给出包含了完整光照计算的 Unity Shader。

## Unity 的渲染路径
在 Unity 中，**渲染路径 Rendering Path** 决定了光照是如何应用到 Unity Shader 中的。我们需要为每个 Pass 指定它使用的渲染路径，这样才能让它告诉 Unity 光源和处理后的光照信息从哪些数据中去取。

Unity 5.0 之前，有 3 种渲染路径：**前向渲染路径 Forward Rendering Path**、**延迟渲染路径 Deferred Rendering Path** 和**顶点照明渲染路径 Vertex Lit Rendering Path**。Unity 5.0 之后，顶点照明渲染路径已不建议使用（但 Unity 依然兼容）；且新的延迟渲染路径代替了原来的延迟渲染路径（老版本的延迟渲染路径在 Unity 中依然兼容）。

大多数情况下，一个项目只会使用一种渲染路径。可以为整个项目设置渲染时的渲染路径：Edit -> Project Settings -> Graphics -> Tier Settings -> Rendering Path。但有时，希望可以使用多个渲染路径，例如摄像机 A 渲染的物体使用前向渲染路径，摄像机 B 渲染的物体使用延迟渲染路径，可以在每个摄像机组件的 Rendering Path 修改，以覆盖 Project Settings 中的设置。如果当前显卡不支持所选择的渲染路径，Unity 会自动使用更低一级的渲染路径。比如：GPU 不支持延迟渲染，则会使用前向渲染。

完成设置后，就可以在 Unity Shader 的 Pass 中设置标签 LightMode。

    Pass {
        Tags { "LightMode" = "ForwardBase" }

下表给出了 Pass 的 LightMode 标签支持的渲染路径设置选项：  

| 标签名 | 描述 |
| :---- | :---- |
| Always | 不管使用哪种渲染路径，该 Pass 总会被渲染，但不会计算任何光照 |
| ForwardBase | 用于**前向渲染**。该 Pass 会计算环境光、最重要的平行光，逐顶点/SH 光源和 Lightmaps |
| ForwardAdd | 用于**前向渲染**。该 Pass 会计算额外的逐像素光源，每个 Pass 对应一个光源 |
| Deferred | 用于**延迟渲染**。该 Pass 会渲染 G 缓冲(G-buffer) |
| ShadowCaster | 把物体的深度信息渲染到阴影映射纹理(shadowmap)或一张深度纹理中 |
| PrepassBase | 用于**遗留的延迟渲染**。该 Pass 会渲染法线和高光反射的指数部分 |
| PrepassFinal | 	用于**遗留的延迟渲染**。该 Pass 通过合并纹理、光照和自发光来渲染得到最后的颜色 |
| Vertex、VertexLMRGBM 和 VertexLM | 用于**遗留的顶点照明渲染** |

指定渲染路径是我们和 Unity 底层渲染引擎的一次重要的沟通。Unity 会根据 Pass 中 LightMode 标签的设置，对一些内置光照变量进行赋值。如果没有设置标签或设置了不正确的标签，当我们调用光照变量时，会得到不正确的结果。如果没有指定任何渲染路径，在 Unity 5.x 版本中会被当成一个和顶点照明渲染路径等同的 Pass。

### 前向渲染路径
前向渲染路径是传统的渲染方式，也是最常用的一种渲染路径。

#### 前向渲染路径的原理
每进行一次完整的前向渲染，我们需要渲染该对象的图元，并计算两个缓冲区的信息：颜色缓冲区、深度缓冲区。先根据深度缓冲来决定一个片元是否可见，如果可见再更新颜色缓冲区中的颜色值。下面伪代码描述了大致的过程：  

    Pass {
        for (each primitive in this model) {
            for (each fragment covered by this primitive) {
                if (failed in depth test) {
                    // 如果没有通过深度测试，说明该片元不可见
                    discard;
                }
                else {
                    // 如果该片元可见，则进行光照计算
                    float4 color = Shading(materialInfo, pos, normal, lightDir, viewDir);
                    // 更新帧缓冲
                    writeFrameBuffer(fragment, color);
                }
            }
        }
    }

每个逐像素光源都需要进行上述渲染流程。如果一个物体受到多个逐像素光源的影响，则需要执行多个 Pass，每个 Pass 计算一个逐像素光源的光照结果，然后在帧缓冲中把这些光照结果混合，得到最终的颜色值。Pass 执行次数 = N 个物体 * M 个光源。渲染引擎会限制每个物体的逐像素光照数目，以此来限制 Pass 执行的次数。

#### Unity 中的前向渲染
Unity 中，前向渲染路径有 3 种处理光照的方式：**逐顶点处理**、**逐像素处理**、**球谐函数 Spherical Harmonics, SH 处理**。

而决定一个光源使用哪种处理模式取决于它的**类型 Type** 和**渲染模式 Render Mode**（光源的 Light 组件中可以设置这些属性）。光源类型指的是该光源是平行光、点光或者聚光，而光源的渲染模式指的是该光源是否是**重要的 Important**，如果把一个光照模式设置为 Important，意味着我们希望 Unity 认真对待它，把它当作逐像素光源来处理。

在前向渲染中，当我们渲染一个物体时，Unity 会根据场景中各个光源的设置以及这些光源对物体的影响程度（比如距离物体远近、光源强度等）对这些光源进行一个重要度排序。其中，一定数目的光源会按逐像素的方式处理，然后最多有 4 个光源按逐顶点的方式处理，剩下的光源可以按 SH 方式处理，Unity 使用的判断规则如下：  
①场景中最亮的平行光总是按逐像素处理；  
②渲染模式被设置成 **Not Important** 的光源，会按逐顶点或 SH 处理；  
③渲染模式被设置成 **Important** 的光源，会按逐像素处理；  
④如果根据以上规则得到的像素光源数量小于 **Quality Setting** 中的逐像素光源数量 Pixel Light Count，会有更多的光源以逐像素的方式进行渲染。

在 Pass 中进行光照计算时，前向渲染一共有两种：Base Pass 和 Addtional Pass。通常来说，这两种 Pass 进行的标签和渲染设置以及常规光照计算如下图所示：

<div  align="center">  
<img src="https://s2.loli.net/2023/11/23/q8WythHV5gzfed9.jpg" width = "60%" height = "60%" alt="图35- 前向渲染的两种 Pass"/>
</div>

上图的说明：  
①在渲染设置中，除了设置了 Pass 标签外，还使用了 **#pragma multi_compile_fwdbase** 和 **#pragma multi_compile_fwdadd** 这样的编译指令。这些编译指令会保证 Unity 为相应类型的 Pass 生成需要的 Shader 变种，这些变种会处理不同条件下的渲染逻辑，比如是否使用光照贴图、当前处理哪些光源类型、是否开启了阴影等，同时 Unity 也会在背后声明内置变量传递给 Shader 中。总的来说，只有分别为 Base Pass 和 Addtional Pass 使用以上指令才可以在 Pass 中得到一些正确的光照变量，例如光照衰减值等。  
②Base Pass 旁边的注释给出了 Base Pass 中支持的一些光照特性，例如在 Base Pass 中，可以访问光照纹理 lightmap。  
③Base Pass 中的渲染的平行光默认是支持阴影的（如果开启了光源的阴影功能），而 Addtional Pass 中渲染的光源在默认情况下是没有阴影效果的，即便我们在它的 Light 组件中设置了有阴影的 Shadow Type 也无济于事。但是在 Additional Pass 中，使用 #pragma multi_compile_fullshadows 代替 #pragma multi_compile_fwdadd 编译指令，为点光源和聚光灯开启阴影效果，但这需要 Unity 在内部使用更多的 Shader 变种。  
④环境光和自发光也是在 Base Pass 中计算的。这是因为，对于一个物体来说，环境光和自发光我们只希望计算一次即可，而如果在 Addtional Pass 中计算这两种光照，就会造成叠加多次环境光和自发光，不是我们想要的。  
⑤在 Addtional Pass 的渲染设置中，我们还开启和设置了混合模式。我们希望每个 Addtional Pass 可以与上一次的光照结果在帧缓存中进行叠加，从而得到最终的有多个光照的渲染效果。如果我们没有开启和设置混合模式，那么 Addtional Pass 的渲染结果会覆盖掉之前的渲染结果，看起来就好像该物体只受该光源影响。通常情况下，选择的混合模式是 **Blend One One**。  
⑥对于前向渲染来说，一个 Unity Shader 通常会定义一个 Base Pass (也可以定义多次，例如双面渲染的情况)以及一个 Additional Pass。一个 Base Pass 只会执行一次(定义了多个 Base Pass 的情况除外)，而一个 Additional Pass 的渲染结果会根据影响该物体的其他逐像素光源的数目被调用多次，即每个逐像素光源都会调用一次 Addtional Pass。

上述给出的光照计算是通常情况下我们在每种 Pass 中进行的计算。实际上渲染路径的设置用于告诉 Unity 该 Pass 在前向渲染路径中的位置，然后底层的渲染引擎会进行相关计算并填充一些内置变量，至于怎么使用内置变量完全看开发者。

#### 内置的光照变量和函数
对于前向渲染（即 LightMode 为 **ForwardBase** 和 **ForwardAdd**）来说，可以访问到以下变量：

| 名称 | 类型 | 描述 |
| :---- | :---- | :---- |
| _LightColor0 | float4 | 该 Pass 处理的逐像素光源的颜色 |
| _WorldSpaceLightPos0 | float4 | _WorldSpaceLightPos0.xyz 是该 Pass 处理的逐像素光源的位置，如果该光源是平行光源，那么 _WorldSpaceLightPos.w 是 0，其他光源类型 w 的值为 1 |
| _LightMatrix0 | float4x4 | 从世界空间到光源空间的变换矩阵，可以用于采样 cookie 和光强衰减 attenuation 纹理 |
| unity_4LightPosX0, unity_4LightPosY0, unity_4LightPosZ0 | float4 | 仅用于 Base Pass，前 4 个非重要的点光源在世界空间中的位置 |
| unity_4LightAtten0 | float4 | 仅用于 Base Pass，存储了前 4 个非重要的点光源的衰减因子 |
| unity_LightColor | half4[4] | 仅用于 Base Pass，存储了前 4 个非重要的点光源的颜色 |

前向渲染中可以使用的内置函数：

| 函数名 | 描述 |
| :---- | :---- |
| float3 WorldSpaceLightDir(float4 v) | **仅可用于前向渲染中**。输入一个模型空间中的顶点位置，返回世界空间中从该点到光源的光照方向。内部实现使用了 UnityWorldSpaceLightDir 函数，没有被归一化 |
| float3 UnityWorldSpaceLightDir(float4 v) | **仅可用于前向渲染中**。输入一个世界空间中的顶点位置，返回世界空间中从该点到光源的光照方向，没有被归一化 |
| float3 ObjSpaceLightDir(float4 v) | **仅可用于前向渲染中**。输入一个模型空间中的顶点位置，返回模型空间中从该点到光源的光照方向，没有被归一化 |
| float3 Shade4PointLights(…) |**仅可用于前向渲染中**。计算四个点光源的光照，它的参数是已经打包进矢量的光照数据，通常就是上一个表格中的内置变量，前向渲染通常会使用这个函数来计算逐顶点光照 |

上面列出的内置变量和函数只是一部分，后面学习中会用到其他一些变量和函数。

### 顶点照明渲染路径
顶点照明渲染路径是对硬件配置要求最少、运算性能最高，但同时得到效果最差的一种类型。它仅支持逐顶点的光源计算，不支持逐像素，所以无法得到：阴影、法线映射、高精度的高光反射等效果，它是前向渲染的一个子集。

#### Unity 中的顶点照明渲染
顶点照明渲染路径通常在一个 Pass 中就可以完成对物体的渲染，在这个 Pass 中，我们会计算关心的光源对物体的照明，且是按照逐顶点处理的。它是 Unity 中最快速的渲染路径，且具有最广泛的硬件支持（游戏机上不支持这种路径）。Unity 5 以后被作为一个遗留的渲染路径，未来的版本中可能会被移除。

#### 可访问的内置变量和函数
在 Unity 中一个顶点照明的 Pass 中最多可以访问 8 个逐顶点光源。顶点照明渲染路径中可以使用的内置变量：

| 名称 | 类型 | 描述 |
| :---- | :---- | :---- |
| unity_LightColor | half4[8] | 光源颜色 |
| unity_LightPosition | float4[8] | xyz 分量是视角空间中的光源位置。如果光源是平行光，那么 z 分量值为 0，其他光源类型 z 分量值为 1 |
| unity_LightAtten | half4[8] | 光源衰减因子，如果光源是聚光灯，x 分量 cos(spotAngle/2)，y 分量是 1/cos(spotAngle/4)。如果是其他类型的光源，x 分量是 -1，y 分量是 1，z 分量是衰减的平方，w 分量是光源范围开更号的结果 |
| unity_SpotDirection | float4[8] | 如果光源是聚光灯的话，值为视角空间的聚光灯的位置；如果是其他类型的光源，值为(0， 0， 1， 0) |

如果影响该物体的光源数目小于 8，则数组中剩余的光源颜色会被设置成黑色。有些变量我们同样可以在前向渲染路径中使用，如：unity_LightColor，但这些变量的数组的维度和数值在不同渲染路径下的值是不同的。

顶点照明渲染路径中可以使用的内置函数：

| 函数名 | 描述 |
| :---- | :---- |
| float3 ShadeVertexLights(float4 vertex, float3 normal) | 输入模型空间中的顶点位置和法线，计算四个逐顶点光源的光照以及环境光，内部实现实际上调用了 ShadeVertexLightsFull 函数 |
| float3 ShadeVertexLightsFull(float4 vertex, float3 normal, int lightCount, bool spotLight) | 输入模型空间中的顶点位置和法线，计算 lightCount 个光源的光照以及环境光，如果 spotLight 值为 true，那么这些光源会被当成聚光灯来处理，虽然结果更精确，但计算更加耗时，否则按点光源处理 |

### 延迟渲染路径
前向渲染的问题是：当场景中包含大量实时光源，使用前向渲染会造成性能急剧下降。每个光源会导致每个物体都会执行多个 Pass，每执行一个 Pass 都重复地渲染一遍物体会存在很多相同的重复计算。

延迟渲染是一种更古老的渲染方式，除了前向渲染中使用的颜色缓冲和深度缓冲外，延迟渲染还使用了额外的缓冲区。这些缓冲区也被称为 **G 缓冲**（**Geometry Buffer**，**G-buffer**）。G 缓冲区存储了我们关心的表面（通常指离相机最近的表面）的其他信息，如：表面的法线、位置、用于光照计算的材质属性等。

#### 延迟渲染的原理
延迟渲染主要包含了两个 Pass。在第一个 Pass 中，不进行任何光照计算，而是仅仅计算哪些片元是可见的，这主要是通过深度缓冲技术来实现，当发现一个片元是可见的，我们就把它的相关信息存储到 G 缓冲区中。然后，在第二个 Pass 中，利用 G 缓冲区中的各个片元信息，例如表面法线、视角方向、漫反射系数等进行真正的光照计算。  

大致可以用下面的伪代码来描述：  

    Pass 1 {
    	// 第一个 Pass 不进行真正的光照计算，仅仅把光照计算需要的信息存储到 G 缓冲中
    	for (each primitive in this model) {
    		for (each fragment covered by this primitive) {
    			if (failed in depth test) {
    				// 如果没有通过深度测试，说明该片元是不可见的
    				discard;
    			} else {
    				// 如果该片元可见，就把需要的信息存储到G缓冲中
    				writeGbuffer(materialInfo, pos, normal, lightDir, viewDir)
    			}
    		}
    	}
    }
    Pass 2 {
    	// 利用 G 缓冲中的信息进行真正的光照计算
    	for (each pixel in the screen) {
    	 	if (the pixel is valid) {
    	 		// 如果像素是有效的
    	 		// 读取它对应的 G 缓冲中的信息
    	 		readGBuffer(pixel, materialInfo, pos, normal, lightDir, viewDir)
    
    	 		// 根据读取到的信息进行光照计算
    	 		float4 color = Shading(materialInfo, pos, normal, lightDir, viewDir);
    	 		// 更新帧缓冲
    	 		writeFrameBuffer(pixel, color);
    	 	}
    	}
    }

延迟渲染使用的 Pass 数目通常是2个，与场景中包含的光源数目没有关系，和屏幕的大小有关。

#### Unity 中的延迟渲染
目前 Unity 有 2 种延迟渲染路径：一种是遗留的延迟渲染路径，一种是 Unity 5 及以后使用的延迟渲染路径。如果游戏中使用了大量的实时光照，那么可以选择延迟渲染路径，但是这种路径需要一定的硬件支持。

新旧延迟渲染路径之间的差别很小，只是使用了不同的技术来权衡不同的需求，比如：旧版的延迟渲染路径不支持 Unity 5 以后版本的基于物理的 Standard Shader。

延迟渲染适合在光源数目众多、使用前向渲染造成性能瓶颈的情况下使用，每个光源都可以按逐像素处理。但它也有缺点：  
①不支持真正的**抗锯齿 anti-aliasing** 功能；  
②不能处理半透明物体；  
③对显卡有要求，必须支持 MRT（Multiple Render Targets）、Shader Mode 3.0 及以上、深度渲染纹理以及双面的模板缓冲。

当使用延迟渲染，Unity 提供了两个 Pass：  
①第一个 Pass 用于渲染 G 缓冲。该 Pass 中把物体的漫反射颜色、高光反射颜色、平滑度、法线、自发光和深度等信息渲染到屏幕空间的 G 缓冲区中对每个物体来说，这个 Pass 仅执行一次。  
②第二个 Pass 用于计算真正的光照模型。这个 Pass 会使用上一个 Pass 中渲染的数据来计算最终的颜色光照颜色，再存储到帧缓冲中。

默认的 G 缓冲区包含了以下几个**渲染纹理 Render Texture，RT**（不同版本的 Unity 可能会不同）：  
①RT0：ARGB32 格式，RBG 通道存储漫反射颜色，A 通道未被使用；  
②RT1：ARGB32 格式，RGB 通道存储高光反射颜色，A 通道存储高光反射的指数部分；  
③RT2：ARGB2101010 格式，RGB 通道存储法线，A 通道未被使用；  
④RT3：ARGB32 格式（非 HDR）或 ARGBHalf（HDR），用于存储自发光 + lightmap + 反射探针（reflection probes）；
⑤深度缓冲和模板缓冲。

第二个 Pass 计算光照，默认情况下仅可以使用 Unity 内置的 Standard 光照模型。若想使用其他模型，需要替换掉原来的 Internal-DeferredShading.shader 文件，详见信息见 <a href="http://docs.unity3d.com/Manual/RenderTech-DeferredShading.html">docs.unity3d.com/Manual/RenderTech-DeferredShading.html </a>

#### 可访问的内置变量和函数
延迟渲染路径中可以使用的内置变量，这些变量可以在 UnityDeferredLibrary.cginc 文件中找到声明：

| 名称 | 类型 | 描述 |
| :---- | :---- | :---- |
| _LightColor | float4 | 光源颜色 |
| _LightMatrix0 | float4x4 | 从世界空间到光源空间的变换矩阵，可以用于采样 cookie 和光强衰减纹理 |

### 选择哪种渲染路径
Unity 的官方文档（<a href="http://docs.unity3d.com/Manual/RenderingPaths.html">docs.unity3d.com/Manual/RenderingPaths.html</a>）中给出了渲染路径的详细比较，包括它们的特性比较（是否支持逐像素光照、半透明物体、实时阴影等）、性能比较以及平台支持。

总的来说，需要根据渲染平台来选择渲染路径。如果当前显卡不支持当前渲染路径，Unity 会自动使用低一级的渲染路径。

本书中，主要使用 Unity 的前向渲染路径。


## Unity 的光源类型
Unity 一共支持 4 种光源类型：平行光、点光源、聚光灯和面光源。面光源仅在烘焙时才有用，不在我们讨论的范围。由于每种光源的几何定义不同，因此它们对应的光源属性也不同。

### 光源类型有什么影响
光源的不同属性会对 Shader 带来影响。最常使用的光源属性有：光源的位置、方向（到某点的方向）、颜色、强度以及衰减（到某点的衰减，与点到光源的距离有关）。这些属性和光源的几何定义息息相关。

①平行光  
平行光可以照亮的范围是没有限制的，它通常作为太阳这样的角色在场景中出现，它没有位置的概念，将它放到场景中任意位置都不影响光照的效果，它的几何属性只有方向，通过调整 Transform 组件中的 Rotation 属性来改变它的光源方向。平行光也没有衰减的概念，即光照强度不会随着距离而发生改变。

②点光源  
点光源的照亮空间是有限的，由空间中的一个球体定义的，点光源可以表示由一个点发出的、向所有方向延展的光。球体的半径可以由面板中的 Range 属性来调整，也可以在 Scene 视图中直接拖拉点光源的线框来修改它的属性。

点光源是有位置属性的，由光源的 Transform 组件中的 Position 属性定义。对于方向属性，我们需要用点光源的位置减去某点的位置得到它到该点的方向。点光源的颜色和强度可以在 Light 组件中调整，同时点光源是会衰减的，随着物体逐渐远离点光源，它接受的光照强度也会越弱。点光源球心处的光照强度最强，球体边界处的最弱，值为 0。其中间的衰减值可以由一个函数定义。

③聚光灯  
聚光灯由空间中的一个锥形区域定义。锥形区域的半径由面板中的 Range 属性决定，锥形的张开角度由 Spot Angle 属性决定。同样可以在 Scene 视图中直接拖拉聚光灯的线框来改变它的属性。同样，对于方向属性，可以用聚光灯的位置减去某点的位置得到它到该点的方向。聚光灯的衰减随物体的远离逐渐减小，锥形的顶点处光照强度最强，锥形的边界处强度为0，中间的衰减值可由一个函数定义，这个函数相比点光源衰减的计算公式更加复杂，因为需要判断一个点是否在椎体的范围内。

### 在前向渲染中处理不同的光源类型
下面来看如何在 Unity Shader 中访问它们的属性：位置、方向、颜色、强度和衰减。本节使用的是前向渲染。

#### 实践
***1. 准备工作***  
①新建名为 Scene_9_2_2_1 的场景，并去掉天空盒子；  
②新建名为 ForwardRenderingMat 的材质；  
③新建名为 Chapter9-ForwardRendering 的 Unity Shader，并赋给上一步的材质；  
④在场景中创建一个胶囊体，并将第二步创建的材质赋给它；  
⑤为了让胶囊体受到多个光源的影响，在场景中新建一个点光源，并将颜色设为绿色（和平行光区分开）；  
⑥保存场景。

***2. 编写 Shader***  
将之前笔记中写过的 Chapter6-BlinnPhong 代码复制覆盖到 Chapter9-ForwardRendering 文件中，做部分修改。关于光源属性的访问代码已用注释说明，代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 9/Forward Rendering"
{
    Properties
    {
        _Diffuse("Diffuse", Color) = (1, 1, 1, 1)
        _Specular("Specular", Color) = (1, 1, 1, 1)
        _Gloss("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" }

        // Base pass，处理环境光和第一个逐像素光照（平行光）
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            // 执行该指令，让前向渲染路径的光照衰减等变量可以被正确赋值，不可缺少
            #pragma multi_compile_fwdbase

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Diffuse;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

                // 环境光只需要计算在 Base Pass 中计算一次即可，Additional Pass 不会再计算。自发光同理，但本例中，假设无自发光效果。
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

                // 若场景中包含了多个平行光，Unity 会选择最亮的平行光传递给 Base Pass 进行逐像素处理，其他平行光会按照逐顶点或在 Additional Pass 中按逐像素的方式处理。如果场景中没有任何平行光，那么 Base Pass 会当成全黑的光源处理。
                // 对于 Base Pass 来说，处理逐像素光源类型一定是平行光，可以使用 _WorldSpaceLightPos0 得到这个平行光的方向；使用 _LightColor0 得到平行光的颜色和强度（_LightColor0 已经是颜色和强度相乘后的结果）
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * max(0, dot(worldNormal, worldLightDir));

                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

                // 因为平行光没有衰减，直接令衰减值为 1.0
                fixed atten = 1.0;
                return fixed4(ambient + (diffuse + specular) * atten, 1.0);
            }
            ENDCG
        }

        // Additional pass，处理其他逐像素光照。Additional pass 中的顶点、片元着色器代码是根据 Base Pass 中的代码复制修改得到的，这些修改一般包括：去掉 Base Pass 中的环境光、自发光、逐顶点光照、SH 光照（球谐光照）的部分，并添加一些对不同光源类型的支持。
        Pass
        {
            Tags { "LightMode" = "ForwardAdd" }

            // 开启混合模式，因为希望帧缓冲中的颜色值和之前的光照结果进行叠加。如果没有使用 Blend 命令的话，Additional Pass 会直接覆盖掉之前的光照结果。本例中，选择的混合系数是 Blend One One，也可以使用其他 Blend 指令，比如：Blend SrcAlpha One
            Blend One One

            CGPROGRAM

            // 同样执行该指令，保证 Additional Pass 中访问到正确的光照变量
            #pragma multi_compile_fwdadd

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed4 _Diffuse;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);

                //由于 Additional Pass 处理的光源类型可能是平行光、点光源或聚光灯。通过使用 #ifdef 指令判断是否定义了 USING_DIRECTIONAL_LIGHT。若当前前向渲染 Pass 处理的光源类型是平行光，那么 Unity 的底层就会定义 USING_DIRECTIONAL_LIGHT。
                #ifdef USING_DIRECTIONAL_LIGHT
                    // 若判断是平行光，方向可以直接通过 _WorldSpaceLightPos0.xyz 得到
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                #else
                    // 若判断是点光源或聚光灯，_WorldSpaceLightPos0 表示世界空间下的光源位置，需要减去世界空间下的顶点位置才能得到光源方向
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz - i.worldPos.xyz);
                #endif

                // 使用 _LightColor0 得到光源（可能是平行光、点光源或聚光灯）的颜色和强度
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * max(0, dot(worldNormal, worldLightDir));

                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

                // 处理不同光源的衰减
                #ifdef USING_DIRECTIONAL_LIGHT
                    // 若为平行光，不衰减，衰减值为 1.0
                    fixed atten = 1.0;
                #else
                    // 若为其他光源，可以使用数学公式来计算衰减，但这些计算往往涉及开根号等计算量大的操作。因此 Unity 选择使用一张纹理作为查找表（Lookup Table, LUT），以在片元着色器中得到光源的衰减
                    // 首先将片元的坐标由世界空间转到光源空间，再对衰减纹理进行采样得到衰减值
                    float3 lightCoord = mul(unity_WorldToLight, float4(i.worldPos, 1)).xyz;
                    fixed atten = tex2D(_LightTexture0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;
                #endif

                return fixed4((diffuse + specular) * atten, 1.0);
            }

            ENDCG
        }
    }

    Fallback "Specular"
}
```

得到如下效果：

<div  align="center">  
<img src="https://s2.loli.net/2023/11/27/xM2JtD8bZmwKSdr.png" width = "70%" height = "70%" alt="图36- 使用一个平行光和一个点光源共同照亮物体。"/>
</div>

#### 实验：Base Pass 和 Additional Pass 的调用
为了更直观地理解前向渲染的细节，接下来通过实验学习了解：

***1. 准备工作***  
①新建名为 Scene_9_2_2_2 的场景，并去掉天空盒子；  
②将平行光的颜色调为绿色；  
③在场景中创建一个胶囊体，将上一节的 ForwardRenderingMat 材质赋给该胶囊体；  
④新建 4 个点光源，颜色调整成相同的红色；  
⑤保存场景。

得到如下效果：

<div  align="center">  
<img src="https://s2.loli.net/2023/11/27/aejdl6LvOsbHEtf.png" width = "70%" height = "70%" alt="图37- 使用1个平行光 + 4个点光源照亮一个物体。"/>
</div>

当我们创建一个光源时，它的默认 Render Mode（Light 组件中设置）是 Auto，意味着 Unity 会自动为我们判断哪些光源会逐像素处理，哪些会逐顶点或球谐（SH）的方式处理。

由于我们没有更改 Edit -> Project Settings -> Quality -> Pixel Light Count 中，因此默认情况下，一个物体可以接收出除最亮的平行光外的 4 个逐像素光照。

而除了平行光在 Base Pass 中按逐像素的方式被处理，场景里刚好有 4 个点光源，它们的 Render Mode 为 Auto，因此它们会在 Additional Pass 中以逐像素的方式被处理，每个光源调用一次 Additional Pass。

***2. 使用帧调试器 Frame Debugger 来查看场景的绘制过程***  
Window -> Analysis -> Frame Debugger，点击 Enable 按钮可以看到场景的渲染事件（这里图就不放出来了）

可以看到渲染胶囊体一共有 6 个关键事件（包括 Clear），它们分别为：在第一个，即 Clear 渲染事件中，Unity 首先清除颜色、深度和模板缓冲，为后续渲染做准备；在第二个渲染事件中，Unity 使用第一个 Pass，即 Base Pass，将平行光的光照渲染到帧缓存中；在后面的四个渲染事件中，Unity 使用第二个 Pass，即 Additional Pass，依次将 4 个点光源的光照应用到物体上，得到最后的渲染结果。

Unity 处理这些点光源的顺序是按照它们的重要度排序的。在这个例子中，由于所有点光源的颜色和强度都相同，因此它们的重要度取决于它们距离胶囊体的远近。若光源的强度和颜色互不相同，那么距离就不再是唯一的衡量标准。Unity 官方文档中，并没有给出上述三个影响因素是如何具体影响排序的，我们仅知道排序结果和这三者都有关系。

如果我们将点光源的 Render Mode 设置为 Not Important，因为我们没有在 Shader 的 Base Pass 中计算逐顶点和球谐 SH 光源，因此这 4 个点光源不会对物体产生任何光照效果。若把平行光的 Render Mode 也设置为 Not Important，物体就会仅显示环境光的光照结果。


## Unity 的光照衰减
之前提到过，Unity 使用一张纹理作为查找表来在片元着色器中计算逐像素光照的衰减。这样计算衰减不依赖于数学公式，只需要使用一个参数值去纹理采样即可。但是使用纹理也有弊端：  
①需要预处理得到纹理采样，而且纹理大小会影响衰减的精度；  
②不直观，不方便，一旦把数据存储到查找表中，就无法使用其他数学公式来计算衰减。

由于这种方法在一定程度上可以提升性能，而且得到的效果在大部分情况下都是良好的，因此 Unity 默认使用这种纹理查找的方法来计算逐像素的点光源和聚光灯的衰减。

### 用于光照衰减的纹理
Unity 内部使用一张名为 _LightTexture0 的纹理来计算光照衰减（使用 cookie 的光源的衰减查找纹理是 _LightTextureB0）。

我们通常只关心 _LightTexture0 对角线上的纹理颜色值，这些值表明了在光源空间中不同位置的点的衰减值。（0，0）表示与光源位置重合的点的衰减值，而（1，1）表示在光源空间中距离最远的点的衰减。

为了对 _LightTexture0 衰减纹理采样得到给定点到光源的衰减值，需要得到该点在光源空间的位置，可以通过 _LightMatrix0 矩阵把给定点的坐标由世界空间转到光源空间：`float3 lightCoord = mul(_LightMatrix0, float4(i.worldPosition, 1)).xyz;`。然后，使用这个坐标的模的平方对衰减纹理进行采样，得到衰减值：`fixed atten = tex2D(_LightTexture0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;`。之所以使用光源空间中顶点距离的平方（通过 dot 函数得到）是为了避免开方操作，然后使用宏 UNITY_ATTEN_CHANNEL 来得到衰减纹理中衰减值所在的分量，得到最终的衰减值。

### 使用数学公式计算衰减
下面代码可以计算光源的线性衰减：  

    float distance = length(_WorldSpaceLightPos0.xyz - i.worldPosition.xyz);
    atten = 1.0 / distance; 

Unity 文档中没有给出内置衰减计算的相关说明，我们无法在 Shader 中通过内置变量得到光源的范围、聚光灯的朝向、张开角度等信息，因此使用公式计算衰减的效果有时不尽如人意，尤其在物体离开光源照明范围时，因为不再执行该光照的 Additional Pass，所以光照效果会发生突变。虽然可以利用脚本将光源信息传递给 Shader，但这样灵活性很低，只能期待未来版本中 Unity 可以完善文档并开放更多参数给开发者。


## Unity 的阴影
### 阴影是如何实现的
实时渲染中，我们最常使用一种名为 **Shadow Map** 的技术，它会首先把摄像机的位置放在与光源重合的位置上，那么场景中该光源的阴影区域就是那些摄像机看不到的地方。Unity 就是使用这种技术。

在前向渲染路径中，如果场景中最重要的平行光开启了阴影，Unity 就会为该光源计算它的阴影映射纹理 shadowmap，这张阴影纹理本质上也是一张深度图，它记录了从该光源的位置出发、能看到的场景中距离它最近的表面位置(深度信息)。

为了得到阴影映射纹理中的深度信息，一种方法是：先把摄像机放到光源的位置，然后按照正常的渲染流程，即调用 Base Pass 和 Additional Pass 来更新深度信息。但是这样会造成一定的性能浪费，所以Unity 使用了一个额外的 Pass 专门更新光源的阴影映射纹理，这个 Pass 的 LightMode 标签设置为 **ShadowCaster**。这个 Pass 的渲染目标不是帧缓存，而是阴影映射纹理（或深度纹理）。Unity 会首先把摄像机放置到光源的位置上，然后调用该 Pass，通过对顶点变换后得到光源空间下的位置，并据此来输出深度信息到阴影映射纹理中。因此当开启了光源的阴影效果后，底层渲染会优先在当前渲染物体的 Unity Shader 中找到 LightMode 为 ShadowCaster 的 Pass，如果没有，会继续在 Fallback 指定的 Unity Shader 中找，如果仍然没有找到，该物体就不会向其他物体投射阴影（它仍然可以接收来自其他物体的阴影）。当找到后，Unity 会使用该 Pass 来更新光源的阴影映射纹理。

- - -

①传统的阴影映射纹理实现：  
在正常渲染的 Pass 中把顶点位置变换到光源空间下，以得到光源空间中顶点的位置。然后使用 xy 分量对阴影映射纹理进行采样，得到阴影纹理记录的深度值，如果它小于这个顶点的 z 分量，则说明这个顶点在阴影中。

②Unity 5 中的阴影采样技术：  
Unity 使用了不同于上述传统的阴影采样技术，即**屏幕空间的阴影映射技术 Screenspace Shadow Map**。该技术原本是延迟渲染中产生阴影的方法，但并不是所有的 Unity 平台都是用该技术，因为其需要显卡支持 MRT，而有些移动平台不支持这种特性。使用该技术时，Unity 会首先调用 LightMode 为 ShadowCaster 的 Pass 得到可投射阴影的光源的阴影映射纹理和相机的深度纹理。然后，根据光源的阴影映射纹理和相机的深度纹理来得到屏幕空间的阴影图。

如果相机深度纹理记录的深度大于转换到光源阴影映射纹理中的深度值，则说明这个物体表面虽然是可见的，但是却处于该光源的阴影中。通过这样的方式，生成的阴影图包含了屏幕空间中所有有阴影的区域。

如果我们想要一个物体接受来自其他物体的阴影，首先需要把表面坐标从模型空间转换到屏幕空间中，然后使用该坐标对阴影图进行采样即可。

- - -

总结，一个物体接收来自其他物体的阴影，以及它向其他物体投射阴影是两个过程：  
①如果想要一个物体接收来自其他物体的阴影，就必须在 Shader 中对阴影映射纹理（包括屏幕空间的阴影图）进行采样，把采样结果和最后的光照结果相乘来产生阴影效果。  
②如果想要一个物体向其他物体投射阴影，就必须把该物体加入光源的阴影映射纹理的计算中，从而让其他物体在对阴影映射纹理采样时可以得到该物体的相关信息。 

### 不透明物体的阴影的实践
准备工作：  
①新建名为 Scene_9_4_2 的场景，并去掉天空盒子；  
②新建名为 ShadowMat 的材质，并将上面一节的 ForwardRendering 赋给它；  
③在场景中创建 1 个正方体、2 个平面，并把上一步创建的材质赋给正方体，平面的材质保持默认；  
④保存场景。

本例中，平行光的 Shadow Type 选择了 Soft Shadows。

#### 让物体投射阴影
在 Unity 中，可以选择是否让一个物体投射或接受阴影，通过设置 Mesh Renderer 组件中的 Cast Shadows 和 Receive Shadows 属性来实现。

当 **Cast Shadows** 被设置为 On，Unity 就会把该物体加入光源的阴影映射纹理的计算中。这个过程是通过为该物体执行 LightMode 为 ShadowCaster 的 Pass 来实现的。

若 **Receive Shadows** 没有开启，当我们调用 Unity 的内置宏和变量计算阴影时，这些宏通过判断该物体没有开启接受阴影的功能，就不会再内部计算阴影。

我们把正方形和两个平面的 Cast Shadows 和 Receive Shadows 都设为开启状态，可以得到下图的结果：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/02/Hbt5K4L3Bo9RSOX.png" width = "70%" height = "70%" alt="图38- 开启 Cast Shadows 和 Receive Shadows，从而让正方体可以投射和接收阴影。"/>
</div>

可以看到，尽管没有对正方体使用的 ForwardRendering 进行任何更改，正方体还是可以向下面的平面投射阴影。这是因为我们将内置的 Specular 作为 Fallback，虽然 Specular 本身也没有包含 LightMode 为 ShadowCaster 的 Pass，但是 Specular 的 Fallback 调用了 VertexLit。我们可以在 Unity 内置的着色器中找到它（内置着色器可以在官网上下载 <a href="http://unity3d.com/cn/get-unity/download/archive">http://unity3d.com/cn/get-unity/download/archive </a>，选择下载的下拉菜单里的 Built in shaders，下载得到的压缩包里面打开 DefaultResourcesExtra -> Normal-VertexLit.shader）。该 shader 里面就有 LightMode 为 ShadowCaster 的 Pass。

这个 LightMode 为 ShadowCaster 的 Pass 在这里就不摘抄了，里面使用了一些宏，主要是为了把深度信息输出到一张深度图或者阴影映射纹理中。总之，想要在 Unity 让物体能够向其他物体投射阴影，一定要在它使用的 Unity Shader 中提供一个 LightMode 为 ShadowCaster 的 Pass。

上图还有个特殊现象，就是右侧的平面没有向下面的平面投射阴影，尽管它的 Cast Shadow 已经被开启了。在默认情况下，计算光源的阴影映射纹理会剔除掉物体的背面。我们可以将 Cast Shadow 设置为 **Two Sided** 来允许对物体的所有面都计算阴影信息。

<div  align="center">  
<img src="https://s2.loli.net/2023/12/04/QXquABdU7ztsi9r.png" width = "70%" height = "70%" alt="图39- 把 Cast Shadows 设置为 Two Sided 可以让右侧平面的背光面也产生阴影。"/>
</div>

但是，这种图中正方体无法接收阴影，是因为使用的 Shader 没有对阴影进行任何处理。而下面的平面可以接收阴影是因为内置的 Standard Shader，该 Shader 进行了接受阴影的相关操作。

#### 让物体接收阴影
为了让立方体可以接收阴影，我们新建一个 Unity Shader，命名为 Chapter9-Shadow，复制 Chapter9-ForwardRendering 的代码，并做部分修改：

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 9/Shadow"
{
    Properties
    {
        _Diffuse("Diffuse", Color) = (1, 1, 1, 1)
        _Specular("Specular", Color) = (1, 1, 1, 1)
        _Gloss("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" }

        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma multi_compile_fwdbase

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"
            #include "AutoLight.cginc" // 下面计算阴影需要使用的宏都是在这个文件声明的

            fixed4 _Diffuse;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                SHADOW_COORDS(2) // 这个宏声明一个用于对阴影纹理采样的坐标。这个宏的参数是下一个可用的插值寄存器的索引值，0 和 1 都使用了，所以传入 2
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                TRANSFER_SHADOW(o); // 这个宏计算 v2f 结构中声明的阴影纹理坐标
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * max(0, dot(worldNormal, worldLightDir));

                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

                fixed atten = 1.0;
                fixed shadow = SHADOW_ATTENUATION(i); // 这个宏计算阴影值
                return fixed4(ambient + (diffuse + specular) * atten * shadow, 1.0);
            }
            ENDCG
        }

        Pass
        {
            ... // 和 Chapter9-ForwardRendering 里一样
        }
    }
    Fallback "Specular"
}
```

**SHADOW_COORDS**、**TRANSFER_SHADOW** 和 **SHADOW_ATTENUATION** 是计算阴影的“三剑客”。这些内置宏帮我们计算了光源的阴影，在 AutoLight.cginc 中可以找到它们的声明：  
①**SHADOW_COORDS**：声明一个名为 _ShadowCoord 的阴影纹理坐标变量；  
②**TRANSFER_SHADOW**：实现会根据平台变化。如果平台支持屏幕空间的阴影映射技术（通过判断是否定义了 UNITY_NO_SCREENSPACE_SHADOWS 来得到），会调用内置的 ComputeScreenPos 函数来计算 _ShadowCoord。如果不支持，会把顶点坐标从模型空间变换到光源空间后存储到 _ShadowCoord 中；  
③**SHADOW_ATTENUATION**：使用 _ShadowCoord 对相关纹理进行采样，得到阴影信息。

内置代码也定义了关闭阴影的处理代码，当关闭阴影时，SHADOW_COORDS 和 TRANSFER_SHADOW 实际没有任何作用，而 SHADOW_ATTENUATION 会直接等于数值 1。

需要注意的是，由于这这宏中会使用上下文变量进行相关计算，例如 TRANSFER_SHADOW 会使用 v.vertex 或 a.pos 来计算坐标，因此为了这些宏能够正常工作，我们需要保证 a2f 结构体中顶点坐标变量名必须是 vertex，顶点着色器的输出结构体必须命名为 v， 且 v2f 中的顶点位置变量必须命名为 pos。

<div  align="center">  
<img src="https://s2.loli.net/2023/12/04/zjDXnOVRS9MwZiY.jpg" width = "70%" height = "70%" alt="图40- 正方体可以接收来自右侧平面的阴影。"/>
</div>

### 使用帧调试器查看阴影绘制过程
在 Window -> Analysis -> Frame Debugger 中打开帧调试器。可以看到主要分为 4 个渲染事件：  
①UpdateDepthTexture：更新摄像机的深度纹理；  
②Shadows.RenderShadowMap：渲染得到平行光的阴影映射纹理；  
③RenderForwardOpaque.CollectShadows：根据深度纹理和阴影映射纹理得到屏幕空间的阴影图；  
④RenderForward.RenderLoopJob：绘制渲染结果。

<div  align="center">  
<img src="https://s2.loli.net/2023/12/04/JPxETG6s1kKIYzL.jpg" width = "80%" height = "80%" alt="图41- 使用帧调试器查看阴影绘制过程。"/>
</div>

上述四个渲染事件的具体图片不在这里放出了。帧调试器右侧面板可以看到具体调用了哪个 Shader 的哪个 Pass 绘制了当前纹理或图片。

### 统一管理光照衰减和阴影
前面的例子实现中，我们把光照衰减因子、阴影值和光照结果相乘得到最终的渲染结果。其实，Unity 提供了 **UNITY_LIGHT_ATTENUATION** 这个内置宏来同时计算这两个信息。

我们新建一个材质，命名为 AttenuationAndShadowUseBuildInFunctionsMat；同时新建名为 Chapter9-AttenuationAndShadowUseBuildInFunctions 的 Shader。将上一节的 Chapter9-Shadow 的代码复制进去，将下面代码删除：

    fixed atten = 1.0;
    fixed shadow = SHADOW_ATTENUATION(i);

修改为

    UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);

**UNITY_LIGHT_ATTENUATION** 接收 3 个参数，它会将光照衰减和阴影值相乘后的结果存储到第一个参数中。无需在代码中声明 atten 参数，该宏会帮我们声明这个变量。第二个参数是结构体 v2f ，这个参数会传递给 **SHADOW_ATTENUATION** ，用来计算阴影值。第三个参数是世界空间下的坐标，这个参数会用于计算光源空间下的坐标，再对光照衰减纹理采样来得到光照衰减。

由于使用了 **UNITY_LIGHT_ATTENUATION**，Base Pass 和 Additional Pass 的代码可以统一，我们不需要在 Base Pass 里单独处理阴影，也不需要在 Additional Pass 中判断光源类型来处理关照衰减，一切都可以通过 **UNITY_LIGHT_ATTENUATION** 来完成。将 Additional Pass 中的两个结构体、顶点着色器和片元着色器中的代码改为和 Bass Pass 一模一样即可。如果希望在 Additional Pass 中也添加阴影效果，需要使用 `#pragma multi_compile_fwdadd_fullshadows` 来代替  Additional Pass 中的 `#pragma multi_compile_fwdadd` 指令。这样，Unity 也会为这些额外的逐像素光源计算阴影，并传递给 Shader。

### 透明度物体的阴影
对于大多数不透明的物体，把 Fallback 设为 VertexLit 就可以得到正确的阴影；但是对于透明物体来说，透明物体的实现通常会使用**透明度测试**或**透明度混合**，我们需要小心设置这些物体的 Fallback。

#### 透明度测试
透明度测试的处理比较简单，但是如果我们仍然直接使用 VertexLit、Diffuse、Specular 等作为回调，往往无法得到正确的阴影，因为透明度测试需要在片元着色器中舍弃某些片元，而 VertexLit 中的阴影映射纹理并没有进行这样的操作，所以会导致被舍弃的片元依然投射了阴影。

我们新建一个 Scene_9_4_5_a 的场景，新建 AlphaTestWithShadowMat 的材质和 Chapter9-AlphaTestWithShadow 的 Unity Shader，将 Chapter8-AlphaTestBothSided 复制进去，然后添加阴影的计算：  
①增加 `#include "AutoLight.cginc"` 头文件  
②在 v2f 使用内置宏：`SHADOW_COORDS(3)`  
③在顶点着色器中使用内置宏：`TRANSFER_SHADOW(o);`  
④在片元着色器中使用内置宏：`UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);`  
⑤把 Fallback 改为 VertexLit

效果如下：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/05/ENogdGBhvMmIlZn.jpg" width = "70%" height = "70%" alt="图42- 错误设置了 Fallback 的使用透明度测试的物体投射阴影。"/>
</div>

可以看出镂空的区域出现了不正常的阴影，这就是因为内置的 VertexLit 提供的 ShadowCaster 的 Pass 没有进行任何透明度测试的计算，因此，它会把整个物体的深度信息写入深度图和者阴影映射纹理中。

而当我们把 Fallback 改为 Transparent/Cutout/VertexLit（之前透明度测试时候使用过的），它的 ShadowCaster Pass 计算了透明度测试，会把裁剪后的物体的深度信息写入深度图和者阴影映射纹理中。需要注意，Transparent/Cutout/VertexLit 计算透明度测试使用了名为 _Cutoff 的属性来进行透明度测试，即我们的 Shader 里必须提供名为 _Cutoff 的属性。更改了 Fallback 后的效果如下：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/05/CEeXPQwrJG8I9Lc.jpg" width = "70%" height = "70%" alt="图43- 正确设置了 Fallback 的使用透明度测试的物体投射阴影。"/>
</div>

但是这样还是不对，出现了一些不应该透过光的部分。出现这种情况的原因是，默认情况下把物体渲染到深度图和阴影映射纹理中仅考虑物体的正面。所以需要把正方体的 Mesh Renderer 组件中的 Cast Shadows 属性设置为 Two Sided。效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/05/JjhxkbsoByz5tdZ.jpg" width = "70%" height = "70%" alt="图44- 正确设置了 Cast Shadow 属性的使用透明度测试的物体投射阴影。"/>
</div>

#### 透明度混合
透明度混合添加阴影相比透明度测试更复杂。所有 Unity 内置的透明度混合，比如：Transparent/VertexLit 等，都没有阴影投射的 Pass，这意味着半透明物体不参与深度图和阴影映射纹理的计算，即不会向其他物体投射阴影，同时也不会接受其他物体的阴影。

我们新建一个 Scene_9_4_5_b 的场景，新建 AlphaBlendWithShadowMat 的材质和 Chapter9-AlphaBlendWithShadow 的 Unity Shader，将 Chapter8-AlphaBlend 复制进去，然后添加阴影的计算，并且它的 Fallback 是内置的 Transparent/VertexLit。效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/11/gRJxAd3GZVUrvFH.jpg" width = "70%" height = "70%" alt="图45- 把使用了透明度混合的 Unity Shader 的 Fallback 设置为内置的
 Transparent/VertexLit。半透明物体不会向下方的平面投射阴影，也不会接收来自右侧平面
的阴影，它看起来就像是完全透明一样"/>
</div>

Unity 不处理半透明物体的阴影，是由于透明度混合需要关闭深度写入，会影响阴影的生成。若想为半透明物体产生正确的阴影，需要在每个光源空间下仍然严格按照从后往前的顺序进行渲染，这会让阴影处理变得复杂，从而影响性能。因此，在 Unity 中，所有内置的半透明 Shader 是不会产生任何阴影效果的。

但是可以通过把它们的 Fallback 设置为 VertexLit、Diffuse 这些不透明物体使用的 Unity Shader，这样 Unity 就会在它的 Fallback 找到一个阴影投射的 Pass。


## 本书使用的标准 Unity Shader
将截止到本节所学的所有基础光照计算整合在一起实现的标准光照着色器如下，都包含了对法线纹理、多光源、光照衰减和阴影的相关处理：  
①BumpedDiffuse，使用 Phong 光照模式，链接如下：  
https://github.com/candycat1992/Unity_Shaders_Book/blob/master/Assets/Shaders/Common/BumpedDiffuse.shader  
②BumpedSpecular，使用 Blinn-Phong 光照模型，链接如下：
https://github.com/candycat1992/Unity_Shaders_Book/blob/master/Assets/Shaders/Common/BumpedSpecular.shader


# 第九章 高级纹理
## 立方体纹理
在图形学中，**立方体纹理 Cubemap** 是**环境映射 Environment Mapping** 的一种实现方法。环境映射可以模拟物体周围的环境，而使用了环境映射的物体可以看起来像镀了层金属一样反射出周围的环境。

和之前的纹理不同，立方体纹理一共包含了 6 张图像，这些图像对应了一个立方体的 6 个面。立方体的每个面表示沿着世界空间下的轴向（上下左右前后）观察所得的图像。而对立方体纹理采样，需要提供一个三维的纹理坐标。这个方向矢量从立方体的中心出发，当它向外部延伸时就会和立方体的 6 个纹理之一发生相交，而采样得到的结果就是由该交点计算而来的。

使用立方体纹理的好处在于，它的实现简单快速，而且得到的效果也比较好。而它的缺点为，例如当场景中引入了新的物体、光源或者物体发生移动时，就需要重新生成立方体纹理。除此之外，立方体纹理也仅可以反射环境，但不能反射使用了该立方体纹理的物体本身。因为立方体纹理不能模拟多次反射的结果，例如两个金属球互相反射的情况（Unity 5 引入的全局光照系统允许实现这样的自反射效果，见第 17 章）。因此应该尽量对于凸面体而不是凹面体使用立方体纹理（因为凹面体回反射自身）。

立方体纹理在实时渲染中有很多应用，最常见的是用于天空盒子 Skybox 以及环境映射。

### 天空盒子
**天空盒子 sky box** 是游戏中用于模拟背景的一种方法。它用来模拟天空（尽管仍可以用它模拟室内等背景）。当我们在背景中使用了天空盒子时，整个场景被包围在一个立方体内。这个立方体每个面使用的技术就是立方体纹理映射技术。

在 Unity 中，使用天空盒子，只需要创建一个 Skybox 材质，再把它赋给该场景的相关设置即可：  
①新建名为 SkyboxMat 的材质；  
②将新建的材质选择 Unity 自带的 Shader - Skybox/6Sided，该材质需要 6 张纹理；  
③使用原书的 6 张纹理资源，在 \Assets\Textures\Chapter10\Cubemaps 目录下，并在纹理的 Inspector 面板上将 Wrap Mode 设置为 Clamp，防止衔接处出现不匹配的现象；  
④将 6 张纹理拖到材质面板对应的纹理属性上（注意位置，posz 纹理对应 Front\[+Z\] 属性）

上面的材质中，除了 6 张纹理属性外还有 3 个属性：  
①**Tint Color**，用于控制该材质的整体颜色；  
②**Exposure**，用于控制天空盒子的亮度；  
③**Rotation**，用于调整天空盒子沿 +y 轴方向的旋转角度。

下面为场景添加 Skybox：  
①新建名为 Scene_10_1_1 的场景；  
②在 Unity 菜单 Window -> Rendering -> Environment 中，将 SkyboxMat 材质赋给 SkyBox 选项；  
③确保场景主相机的 Camera 组件中的 Clear Flags 被设置为 Skybox。

效果如下：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/11/qVs5DNiCmgXUMTJ.jpg" width = "70%" height = "70%" alt="图46- 使用了天空盒子的场景"/>
</div>


需要说明的是，在 Window -> Rendering -> Environment 中设置的天空盒子会应用于该场景中的所有摄像机，如果希望某个摄像机使用不同的 Skybox，需要为该摄像机添加 Skybox 的组件来单独为该摄像机设置 Skybox。

在 Unity 中，天空盒子是在所有不透明物体之后渲染的，而其背后使用的网格式一个立方体或一个细分后的球体。

### 创建用于环境映射的立方体纹理
除了天空盒子，立方体纹理最常见的用处是用于环境映射。通过这种办法，可以模拟出金属质感的材质。

在 Unity 5 中，创建用于环境映射的立方体纹理的办法有三种：  
①第一种方法是直接由一些特殊布局的纹理创建，提供一张具有特殊布局的纹理，例如类似立方体展开图的交叉布局、全景布局等。然后把该纹理的 Texture Type 设置为 Cubemap 即可。在基于物理的渲染中，通常使用一张 HDR 图像来生成高质量的 Cubemap；  
②第二种方法是手动创建一个 Cubemap 资源，再把 6 张图赋给它。在 Unity 5 中，官方推荐使用第一种方法创建立方体纹理，因为第一种方法可以对纹理数据进行压缩，而且支持边缘修正、光滑反射（glossy reflection）和 HDR 等功能；  
③由脚本生成。

前两种方法都需要提前准备好立方体纹理的图像，它们得到的立方体纹理常常是背场景中的物体所共用。而理想情况下，我们希望根据物体在场景中位置的不同，生成它们各自不同的立方体纹理。这时，可以通过利用 Unity 提供的 **Camera.RenderToCubemap** 来实现。该函数可以把从任意位置观察到的场景图像存储到 6 张图像中，从而创建出该位置上对应的立方体纹理。代码如下：  

``` C#
using UnityEngine;
using UnityEditor;
using System.Collections;

public class RenderCubemapWizard : ScriptableWizard {
    
    public Transform renderFromPosition;
    public Cubemap cubemap;
    
    void OnWizardUpdate () {
        helpString = "Select transform to render from and cubemap to render into";
        isValid = (renderFromPosition != null) && (cubemap != null);
    }
    
    void OnWizardCreate () {
        // create temporary camera for rendering
        GameObject go = new GameObject( "CubemapCamera");
        go.AddComponent<Camera>(); 
        // place it on the object
        go.transform.position = renderFromPosition.position;
        // render into cubemap        
        go.GetComponent<Camera>().RenderToCubemap(cubemap);
        
        // destroy temporary camera
        DestroyImmediate( go );
    }
    
    [MenuItem("GameObject/Render into Cubemap")]
    static void RenderCubemap () {
        ScriptableWizard.DisplayWizard<RenderCubemapWizard>(
            "Render cubemap", "Render!");
    }
}
```

在上面的代码中，我们在 renderFromPosition（由用户指定）位置处动态创建一个摄像机，并调用 Camera.RenderToCubemap 函数把从当前位置观察到的图像渲染到用户指定的立方体纹理 cubemap 中，完成后再销毁临时摄像机。由于该代码需要添加菜单栏条目，因此需要把它放在 Editor 文件夹中才能正确执行。

准备完上述代码后，创建一个立方体纹理的其他步骤如下：  
①使用上面相同的场景，并创建一个空的 GameObject 对象，我们会使用该 GameObject 的位置信息来渲染立方体纹理；  
②新建一个存储的立方体纹理，在 Project 下右键 Creat -> Legacy -> Cubemap 来创建，同时还需要在其面板上勾选 Readable 选项，让脚本顺利将图像渲染到该立方体纹理中；  
③在 Unity 菜单栏选择 GameObject -> Render into Cubemap，打开在脚本中实现的用于渲染立方体纹理的窗口，并把第一步的 GameObject 和第二步的 Cubemap_0 拖拽给 Render From Position 和 Cubemap 选项；  
④点击窗口中的 Render! 按钮可观察到现象。而 Face size 选项越大，渲染出来的立方体纹理分辨率越大，效果越好，占用的内存更大。如下图所示：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/11/HzXGrBQZp6cFRYf.jpg" width = "40%" height = "40%" alt="图47- 使用脚本渲染立方体纹理"/>
</div>

准备好了需要的立方体纹理后，就可以对物体使用环境映射技术。而环境映射最常见的应用就是反射和折射。

### 反射
使用了反射的物体可以看起来像镀了层金属。模拟反射效果，只需要通过入射光线的方向和表面法线方向来计算反射方向，再利用反射方向对立方体纹理采样即可。

准备工作如下：  
①新建名为 Scene_10_1_3 的场景，并将天空盒替换成第一小节定义的天空盒；  
②向场景中拖拽一个 Teapot 模型，将其位置调整为前面小节创建的 Cubemap_0 时使用的空 GameObject 的位置；  
③新建名为 ReflectionMat 的材质，并赋给 Teapot 模型；  
④新建名为 Chapter10-Reflection 的 Unity Shader，并赋给上一步创建的材质；

```  C C for Graphics
Shader "Unity Shaders Book/Chapter 10/Reflection"
{
    Properties
    {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _ReflectColor ("Reflect Color", Color) = (1, 1, 1, 1) // 反射颜色
        _ReflectAmount ("Refect Amount", Range(0, 1)) = 1 // 反射程度
        _Cubemap ("Reflection Cubemap", Cube) = "_Skybox" {} // 反射的环境映射纹理
    }

    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        Pass { 
            Tags { "LightMode"="ForwardBase" }
            
            CGPROGRAM
            
            #pragma multi_compile_fwdbase
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            
            fixed4 _Color;
            fixed4 _ReflectColor;
            fixed _ReflectAmount;
            samplerCUBE _Cubemap;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float3 worldPos : TEXCOORD0;
                fixed3 worldNormal : TEXCOORD1;
                fixed3 worldViewDir : TEXCOORD2;
                fixed3 worldRefl : TEXCOORD3;
                SHADOW_COORDS(4)
            };
            
            v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
                o.worldRefl = reflect(-o.worldViewDir, o.worldNormal); //计算该顶点的反射方向，通过 reflect 函数实现。同时物体反射到摄像机中的光线方向，可以通过光路可逆的原则来反向求得。即，计算视角方向关于顶点法线的反射方向来求得入射光线的方向。
                
                TRANSFER_SHADOW(o);
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));        
                fixed3 worldViewDir = normalize(i.worldViewDir);        
                
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
                fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(worldNormal, worldLightDir));
                
                fixed3 reflection = texCUBE(_Cubemap, i.worldRefl).rgb * _ReflectColor.rgb; //对立方体纹理采样使用的是 Cg 的 texCUBE 函数。利用反射方向来对立方体纹理采样，而没有对 worldRefl 归一化是因为采样的参数仅仅是作为方向变量传递给 texCUBE，因此没有必要归一化。
                
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
                
                fixed3 color = ambient + lerp(diffuse, reflection, _ReflectAmount) * atten; //用 ReflectAmount 进行混合漫反射颜色和反射颜色，并和环境光照相加后返回
                
                return fixed4(color, 1.0);
            }
            ENDCG
        }
    }
    FallBack "Reflective/VertexLit"
}
```

在上面计算中，选择的是在顶点着色器中计算反射方向，若在片元着色器中计算，效果会跟细腻。但是这种差别很小，出于性能的考虑，选择了顶点着色器中计算反射方向。

在材质面板中把 Cubemap_0 拖拽到 Reflection Cubemap 属性中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/12/rThjxqKUcZPRJiN.jpg" width = "70%" height = "70%" alt="图48- 使用了反射效果的 Teapot 模型"/>
</div>

### 折射
折射，即当光线从一种介质（比如：空气）斜射入另一种介质（比如：玻璃）时，传播方向会发生改变。当给定入射角时，可以使用**斯涅尔定律 Snell's Law** 来计算反射角。

当光从介质 1 沿着和表面法线夹角为 $\,{\theta}_1\,$ 的方向斜射入介质 2 时，可以使用如下公式计算折射光线与法线的夹角 $\,{\theta}_2\,$ ：

$$ {\eta}_1 \sin {\theta}_1 = {\eta}_2 \sin {\theta}_2 $$

其中，$\,{\eta}_1\,$ 和 $\,{\eta}_2\,$ 分别是两个介质的**折射率 index of refraction**。如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/12/LoNTm43c9qPzeDj.jpg" width = "40%" height = "40%" alt="图49- 斯涅尔定律"/>
</div>

通常来讲，当得到折射方向后，我们会直接使用它来对立方体纹理进行采样，这不符合物理规律。对一个透明物体来说，更准确的模拟方法需要计算两次折射，即进入物体一次，出去一次。但是，想要在实时渲染中模拟出第二次折射方向是比较复杂的，而且由于仅仅模拟一次的效果从视觉上也还过得去，所有通常只模拟一次折射。

准备工作如下：  
①新建名为 Scene_10_1_4 的场景，并将天空盒替换成第一小节定义的天空盒；  
②向场景中拖拽一个 Teapot 模型，并调整其位置；  
③新建名为 RefractionMat 的材质，赋给上一步创建的模型；  
④新建名为 Chapter10-Refraction 的 Unity Shader，赋给上一步创建的材质。

```  C C for Graphics
Shader "Unity Shaders Book/Chapter 10/Refraction" {
    Properties {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _RefractColor ("Refraction Color", Color) = (1, 1, 1, 1)
        _RefractAmount ("Refraction Amount", Range(0, 1)) = 1 
        _RefractRatio ("Refraction Ratio", Range(0.1, 1)) = 0.5 //不同介质的折射率透射比
        _Cubemap ("Refraction Cubemap", Cube) = "_Skybox" {}
    }
    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        Pass { 
            Tags { "LightMode"="ForwardBase" }
            
            CGPROGRAM
            
            #pragma multi_compile_fwdbase	
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            
            fixed4 _Color;
            fixed4 _RefractColor;
            float _RefractAmount;
            fixed _RefractRatio;
            samplerCUBE _Cubemap;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float3 worldPos : TEXCOORD0;
                fixed3 worldNormal : TEXCOORD1;
                fixed3 worldViewDir : TEXCOORD2;
                fixed3 worldRefr : TEXCOORD3;
                SHADOW_COORDS(4)
            };
                
            v2f vert(a2v v) {
                v2f o;
                o.pos = mul(UNITY_MATRIX_MVP, v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(_Object2World, v.vertex).xyz;
                o.worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
                
                //使用 Cg 的 refract 函数来计算折射方向。第一个参数为入射光线的方向，必须是归一化后的矢量；第二个参数是归一化后的表面法线；第三个参数是入射光线所在介质的折射率和折射光线所在介质的折射率的比值。它返回折射方向，它的模等于入射光线的模
                o.worldRefr = refract(-normalize(o.worldViewDir), normalize(o.worldNormal), _RefractRatio);
                
                TRANSFER_SHADOW(o);
                return o;
            }
                
            fixed4 frag(v2f i) : SV_Target {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                fixed3 worldViewDir = normalize(i.worldViewDir);
                				
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
                fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(worldNormal, worldLightDir));
                
                //也不需要对 i.worldRefr 进行归一化处理
                fixed3 refraction = texCUBE(_Cubemap, i.worldRefr).rgb * _RefractColor.rgb;
                
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
                
                //混合颜色
                fixed3 color = ambient + lerp(diffuse, refraction, _RefractAmount) * atten;
                
                return fixed4(color, 1.0);
            }
            ENDCG
        }
    } 
    FallBack "Reflective/VertexLit"
}
```

在材质面板中把 Cubemap_0 拖拽到 Reflection Cubemap 属性中，效果如下：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/12/tW4vblJYuIzegXN.jpg" width = "70%" height = "70%" alt="图50- 使用了折射效果的 Teapot 模型"/>
</div>

### 菲涅耳反射
实时渲染中，我们经常会使用**菲涅耳反射 Fresnel reflection** 来根据视角方向控制反射程度。

简单的讲，就是视线垂直于表面时，反射较弱；而当视线非垂直表面时，夹角越小，反射越明显。菲涅尔反射描述了这种光学现象，当光线照射到物体表面时，一部分发生反射，一部分进入物体内部，发生折射或散射。被反射的光和入射光之间存在一定的比率关系，这个比率关系可以通过菲涅尔等式进行计算。

一个常见的例子是，站在湖边，低头看湖面，会发现水是透明的，但是抬头看远处的水面时，只能看到水面反射的效果，几乎看不到水下的情景。事实上，几乎所有物体都包含了菲涅耳效果，这是基于物理的渲染中非常重要的一项高光反射计算因子（详见第 17 章）。

> 可以在 John Hable 的一篇非常有名的文章 Everything Has Fresnel 中看到现实生活中各种物体的菲涅耳反射效果：http://filmicworlds.com/blog/everything-has-fresnel/

真实世界的菲涅耳等式非常复杂，在实时渲染中，会使用一些近似公式来计算。其中一个著名的近似公式就是 **Schlick 菲涅耳近似等式**：

$$ F_{Schlick}(v, n) = F_0 + (1 - F_0)(1 - v \cdot n)^5 $$

$\,F_0\,$ 是一个反射系数，根据材质的不同数值会有所不同。折射率可以用来计算 $\,F_0\,$，假设折射率 $\,{\eta}_1$ 和 $\,{\eta}_2\,$ ，可以得到以下公式：

$$ F_0 = (\cfrac {\eta_1 - \eta_2}{\eta_1 + \eta_2})^2 $$

另一个应用比较广泛的等式是 **Empirical 菲涅耳近似等式**

$$ F_{Empirical}(v, n) = max(0, min(1, bias + scale \times (1 - v \cdot n)^{power})) $$

其中，bias、scale 和 power 是控制项。

***

使用上面的菲涅耳近似等式，我们可以在边界处模拟反射光强和折射光强/漫反射光强之间的变化，来模拟更加真实的反射效果。下面使用 Schlick 菲涅耳近似等式来模拟，准备工作如下：  
①新建名为 Scene_10_1_5 的场景，并将天空盒替换成前面小节自定义的天空盒材质；  
②向场景中拖拽一个 Teapot 模型，并调整其位置；  
③新建名为 FresnelMat 的材质，赋给上一步的模型；  
④新建名为 Chapter10-Fresnel 的 Unity Shader，赋给上一步的材质。

```  C C for Graphics
Shader "Unity Shaders Book/Chapter 10/Fresnel" {
    Properties {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _FresnelScale ("Fresnel Scale", Range(0, 1)) = 0.5
        _Cubemap ("Reflection Cubemap", Cube) = "_Skybox" {}
    }
    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        Pass { 
            Tags { "LightMode"="ForwardBase" }
        
            CGPROGRAM
            
            #pragma multi_compile_fwdbase
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            
            fixed4 _Color;
            fixed _FresnelScale;
            samplerCUBE _Cubemap;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float3 worldPos : TEXCOORD0;
                fixed3 worldNormal : TEXCOORD1;
                fixed3 worldViewDir : TEXCOORD2;
                fixed3 worldRefl : TEXCOORD3;
                SHADOW_COORDS(4)
            };
            
            v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
                o.worldRefl = reflect(-o.worldViewDir, o.worldNormal);
                TRANSFER_SHADOW(o);
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                fixed3 worldViewDir = normalize(i.worldViewDir);
                
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
                
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
                
                fixed3 reflection = texCUBE(_Cubemap, i.worldRefl).rgb;
                
                fixed fresnel = _FresnelScale + (1 - _FresnelScale) * pow(1 - dot(worldViewDir, worldNormal), 5); //菲尼尔近似等式    
                
                fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(worldNormal, worldLightDir));
                
                fixed3 color = ambient + lerp(diffuse, reflection, saturate(fresnel)) * atten; //混合漫反射光照和反射光照。一些实现也会直接把 fresnel 和反射光照相乘后叠加到漫反射光照上，来模拟光照的效果
                
                return fixed4(color, 1.0);
            }
            ENDCG
        }
    } 
    FallBack "Reflective/VertexLit"
}
```

在材质面板中把 Cubemap_0 拖拽到 Cubemap 属性中。将 _FresnelScale 调节到 1 时，物体完全反射 Cubemap 中的图像；当 _FresnelScale 为 0 时，则是一个具有边缘光照效果的漫反射物体。下图是 _FresnelScale 为 0.5 时的效果。

<div  align="center">  
<img src="https://s2.loli.net/2023/12/13/7TYAsJfDdVH4WrU.jpg" width = "70%" height = "70%" alt="图51- 使用了菲涅耳反射的 Teapot 模型"/>
</div>

在第 14 章会使用菲涅耳反射来模拟一个简单的水面效果。


## 渲染纹理
在之前的学习中，一个摄像机的渲染结果会输出到颜色缓冲中，并显示到我们的屏幕上。现代 GPU 允许我们把整个三维场景渲染到一个中间缓冲中，即**渲染目标纹理 Render Target Texture, RTT**，而不是传统的帧缓冲或后备缓冲 back buffer。与之相关的是**多重渲染目标 Multiple Render Target, MRT**，这种技术指的是 GPU 允许我们把场景同时渲染到多个渲染目标纹理中，而不再需要为每个渲染目标纹理单独渲染完整的场景。延迟渲染就是使用多重渲染目标的一个应用。

Unity 为渲染目标纹理定义了一种专门的纹理类型，即**渲染纹理 Render Texture**。在 Unity 中使用渲染纹理通常有两种方式：  
①在 Project 目录下创建一个渲染纹理，然后把某个摄像机的渲染目标设置成该渲染纹理，这样一来该摄像机的渲染结果就会实时更新到渲染纹理中，而不是显示到屏幕上。使用这种方法，我们还可以选择渲染纹理的分辨率、滤波模式等纹理属性；  
②在屏幕后处理时使用 GrabPass 命令或 OnRenderImage 函数来获取当前屏幕图像，Unity 会把这个屏幕图像放到一张和屏幕分辨率等同的渲染纹理中，我们可以在自定义的 Pass 中把它们当成普通的纹理来处理，从而实现各种屏幕特效。

我们将依次学习这两种方法在 Unity 中的实现（OnRenderImage 函数在第 11 章讲）

### 镜子效果
准备工作：  
①新建名为 Scene_10_2_1 的场景，并去掉天空盒子；  
②新建名为 MirrorMat 的材质；  
③新建名为 Chapter10-Mirror 的 Unity Shader，并赋给上一步创建的材质；  
④创建 6 个立方体，调整其位置和大小，搭建出一个围绕摄像机的封闭空间，给它们附上合适的材质，使其看起来像房间的墙；往封闭空间内加入3个点光源，调整位置，使其照亮封闭空间；  
⑤在封闭空间内创建 3 个球体和 2 个正方体，调整其位置和大小，附上合适的材质，使其作为封闭空间内的装饰物；  
⑥创建一个四边形 Quad，调整其大小和位置，使其作为空间内的一面镜子，并附上第二步创建的材质；  
⑦Project 视图下创建一个渲染纹理（右键单击 Create -> Render Texture），并命名为 MirrorTexture；  
⑧为了得到从镜子出发观察到的场景图像，需要创建一个摄像机，调整它的位置、裁剪平面、视角等，使它显示的图像是我们希望的镜子图像。由于这个摄像机不需要直接显示在屏幕上，而是用于渲染到纹理。因此，把第七步中创建的 MirrorTexture 拖拽到该摄像机的 Target Texture 上。

镜子的实现原理很简单，使用一个渲染纹理作为输入属性，并把该渲染纹理在水平方向上翻转后直接显示到物体上即可，代码如下：  

```  C C for Graphics
Shader "Unity Shaders Book/Chapter 10/Mirror" {
    Properties {
        _MainTex ("Main Tex", 2D) = "white" {}
    }
    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        Pass {
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag
            
            sampler2D _MainTex;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 texcoord : TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };
            
            v2f vert(a2v v) {        //计算纹理坐标
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = v.texcoord;

                //反转 x 分量的纹理坐标，镜子的图像都是左右相反
                o.uv.x = 1 - o.uv.x;
                
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                return tex2D(_MainTex, i.uv); //对纹理进行采样和输出
            }
            ENDCG
        }
    } 
    FallBack Off
}
```

将创建的 MirrorTexture 渲染纹理拖拽到材质的 Main Tex 属性中，效果如下。若图像模糊不清，可以使用更高的分辨率或更多的抗锯齿采样等。但是，更高的分辨率会影响带宽和性能。

<div  align="center">  
<img src="https://s2.loli.net/2023/12/13/zsfMUPOyrHaT3Wq.jpg" width = "70%" height = "70%" alt="图52- 镜子效果"/>
</div>

### 玻璃效果
在 Unity 中，我们还可以在 Unity Shader 中使用一种特殊的 Pass 来完成获取屏幕图像的目的，这就是 **GrabPass**。当我们在 Shader 中定义了一个 GrabPass 后，Unity 会把当前屏幕的图像绘制在一张纹理中，以便我们在后续的 Pass 中访问它。我们通常会使用 GrabPass 来实现诸如玻璃等透明材质的模拟，与使用简单的透明混合不同，使用 GrabPass 可以让我们对该物体后面的图像进行更复杂的处理，例如使用法线来模拟折射效果，而不再是简单的和原屏幕颜色进行混合。

需要注意的是，在使用 GrabPass 的时候，需要额外小心物体的渲染队列设置。正如之前所说，GrabPass 通常用于渲染透明物体，尽管代码里不包含混合指令，但是仍然要设置透明队列，即 "Queue"="Transparent"。这样才能保证不透明的物体绘制在屏幕上。

下面会使用 GrabPass 来模拟一个玻璃效果。首先使用了一张法线纹理来修改模型的法线信息，然后通过一个 Cubemap 来模拟玻璃的反射。模拟折射的时候，使用 GrabPass 获取玻璃后面的屏幕图像，并使用切线空间下的法线对屏幕纹理坐标偏移后，再对屏幕图像进行采样来模拟近似的折射效果。

准备工作：  
①新建名为 Scene_10_2_2 的场景，并去掉天空盒子；  
②新建名为 GlassRefractionMat 的材质；  
③新建名为 Chapter10-GlassRefraction 的 Unity Shader，并赋给上一步的材质；  
④在场景中创建 6 个平面，调整其位置和大小，以其作为墙，构成封闭的房间。在房间内放置 1 个立方体和一个球体，其中球体位于立方体内部，为了模拟玻璃对内部物体的折射效果，将第 2 步创建的材质赋给立方体；  
⑤使用之前小节实现的创建 Cubemap 的脚本来创建使用本场景的环境映射纹理：Project 视图目下右键 Create -> Legacy -> Cubemap，创建立方体纹理，命名为 Glass_Cubemap，并在创建后的 Cubemap 的 Inspector 窗口中勾选 Readable；然后通过在菜单栏 GameObject -> Render into Cubemap，在出现的窗口中将场景里的立方体拖拽到 Render From Position 项，将上一步创建的 Cubemap 拖拽到 Cubemap 项。

```  C C for Graphics
Shader "Unity Shaders Book/Chapter 10/Glass Refraction" {
    Properties {
        _MainTex ("Main Tex", 2D) = "white" {} //材质纹理
        _BumpMap ("Normal Map", 2D) = "bump" {} //法线纹理
        _Cubemap ("Environment Cubemap", Cube) = "_Skybox" {} //模拟反射的环境纹理
        _Distortion ("Distortion", Range(0, 100)) = 10 //控制模拟折射时图像的扭曲程度
        _RefractAmount ("Refract Amount", Range(0.0, 1.0)) = 1.0 //控制折射程度，若为 0 则该玻璃只包含反射效果，若为 1 则只包含折射效果
    }
    SubShader {
        Tags { "Queue"="Transparent" "RenderType"="Opaque" }
        //一个 Transparent，一个 Opaque 看似矛盾。是因为把 Queue 设置为 Transparent 是为了确保该物体渲染时，其他不透明物体已经渲染在屏幕上了。设置 RenderType 是为了使用着色器替换时，该物体在需要时被正确渲染，详见第 12 章
        
        GrabPass { "_RefractionTex" }
        //通过关键词 GrabPass 定义了一个抓取屏幕图像的 Pass，在该 Pass 中定义了一个字符串，该字符串内部的名称决定了抓取得到的屏幕图像会被存入到哪个纹理中。实际上可以省略该字符串，但是声明性能更好，原因见后面
        
        Pass {        
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "UnityCG.cginc"
            
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            samplerCUBE _Cubemap;
            float _Distortion;
            fixed _RefractAmount;
            sampler2D _RefractionTex; //对应 GrabPass 指定的纹理名称
            float4 _RefractionTex_TexelSize; //得到该纹理的纹素大小，例如一个大小为 256×512 的纹理，它的纹素大小为(1/256, 1/512)。在对屏幕图像采样坐标进行偏移时使用该变量 
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT; 
                float2 texcoord: TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float4 scrPos : TEXCOORD0;
                float4 uv : TEXCOORD1;
                float4 TtoW0 : TEXCOORD2;  
                float4 TtoW1 : TEXCOORD3;  
                float4 TtoW2 : TEXCOORD4; 
            };
            
            v2f vert (a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.scrPos = ComputeGrabScreenPos(o.pos); //得到对应被抓取的屏幕图像的采样坐标
                
                o.uv.xy = TRANSFORM_TEX(v.texcoord, _MainTex); //_MainTex 的采样坐标
                o.uv.zw = TRANSFORM_TEX(v.texcoord, _BumpMap); //_BumpMap 的采样坐标
                
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;  
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);  
                fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);  
                fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w; 
                
                o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);  
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);  
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);  
                
                return o;
            }
            
            fixed4 frag (v2f i) : SV_Target {        
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(worldPos));
                
                fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv.zw)); //获取切线空间法线方向
                
                //对屏幕图像的采样坐标进行偏移， _Distortion 越大偏移越大，则变形程度越大。选择使用切线空间下的法线方向来偏移，是因为该空间下法线反映顶点局部坐标下的方向。
                float2 offset = bump.xy * _Distortion * _RefractionTex_TexelSize.xy;
                i.scrPos.xy = offset * i.scrPos.z + i.scrPos.xy;
                //对 scrPos 透视除法得到真正的屏幕坐标（原理见数学基础），再使用该坐标对抓取的屏幕图像进行采样，得到模拟的折射颜色
                fixed3 refrCol = tex2D(_RefractionTex, i.scrPos.xy/i.scrPos.w).rgb;
                
                //将法线从切线空间转换到世界空间
                bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));
                fixed3 reflDir = reflect(-worldViewDir, bump); //得到反射方向
                fixed4 texColor = tex2D(_MainTex, i.uv.xy);
                fixed3 reflCol = texCUBE(_Cubemap, reflDir).rgb * texColor.rgb; //用反射方向对 Cubemap 进行采样
                
                fixed3 finalColor = reflCol * (1 - _RefractAmount) + refrCol * _RefractAmount; //混合反射折射颜色
                
                return fixed4(finalColor, 1);
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```

将书中资源中的 Glass_Diffuse.jpg 、Glass_Normal.jpg 文件和创建的 Glass_Cubemap 赋给材质的 Main Tex 、Normal Map 和 Cubemap 属性后，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/15/YGCFPOfcRNzMyo7.jpg" width = "70%" height = "70%" alt="图53- 玻璃效果"/>
</div>

在前面的实现中，我们在 GrabPass 中使用了一个字符串指明了被抓取的屏幕图像会存储在哪个名称的纹理中，实际上，GrabPass 支持两种形式：  
①直接使用 GrabPass {}，然后在后续的 Pass 中直接使用 _GrabTexture 来访问屏幕图像。当场景中有多个物体都使用了这样的形式来抓取屏幕时，这种方法的性能消耗比较大，因为对于每一个使用了它的物体，Unity 都会为它单独进行一次昂贵的屏幕抓取操作。但这种方法可以让每个物体得到不同的屏幕图像，这取决于它们的渲染队列以及渲染它们时当前屏幕缓冲中的颜色；  
②使用 GrabPass {"TextureName"}，我们可以在后续 Pass 中利用 TextureName 来访问屏幕图像。使用这种方法同样可以抓取屏幕，但 Unity 只会在每一帧时为第一个使用名为 TextureName 的纹理的物体执行一次抓取屏幕操作，而这个纹理同样可以在其他 Pass 中被访问。

### 渲染纹理 vs GrabPass
GrabPass 实现简单，只需几行代码。但在效率上，使用渲染纹理允许我们自定义渲染纹理的大小。而使用 GrabPass 获取到的图像分辨率和显示屏幕是一致的，意味着高分辨率屏幕的设备会造成严重的带宽影响。在移动设备上，GrabPass 不会重新渲染场景，但往往需要 CPU 直接读取后备缓冲（back buffer）中的数据，破坏 CPU 和 GPU 之间的并行性，比较耗时，且在一些移动设备上不支持。  

在 Unity 5 中，Unity 引入了**命令缓冲 Command Buffers** 来允许我们扩展 Unity 的渲染流水线。使用命令缓冲我们也可以得到类似于抓屏的效果，它可以在不透明物体渲染后把当前图像复制到一个临时的渲染目标纹理中，然后在那里进行一些额外的操作，例如模糊等。最后把图像传递给需要使用它的物体进行处理和显示。除此之外，命令缓冲允许我们实现很多特殊的效果，详见 Unity 官方手册的图像命令缓冲：https://docs.unity3d.com/Manual/GraphicsCommandBuffers.html

## 程序纹理
**程序纹理 Procedural Texture** 指的是那些由计算机生成图像，通常由特定的算法来创建个性化图案或非常真实的自然元素，例如木头、石子等。使用程序纹理的好处在于我们可以使用各种参数来控制纹理的外观，而这些参数不仅仅是那些颜色属性，甚至可以是完全不同类型的图案属性，使得我们可以得到更加丰富自然的动画和视角效果。

### 在 Unity 中实现简单的程序纹理
下面使用算法来生成波点纹理，准备工作如下：  
①新建名为 Scene_10_3_1 的场景，并去掉天空盒；  
②新建名为 ProceduralTextureMat 的材质；  
③使用第 7 章创建的名为 Chapter7-SingleTexture 的 Unity Shader，并赋给上一步创建的材质；  
④在场景中新建一个立方体，并将第 2 步创建的材质赋给它；  
⑤无需为 ProceduralTextureMat 赋予任何纹理，因为要创建程序纹理。新建名为 ProceduralTextureGeneration 的 C# 脚本，赋给上一步创建的立方体；

``` C#
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

[ExecuteInEditMode]
public class ProceduralTextureGeneration : MonoBehaviour {
    public Material material = null;

    //下面 region 声明程序纹理的各个参数，SetProperty 是一个开源插件，为了在面板中修改属性时，执行 set 函数，见 http://github.com/LMNRY/SetProperty，在 Assets 下任意名字文件夹下解压即可使用
    #region Material properties
    [SerializeField, SetProperty("textureWidth")] //纹理的大小，数值通常是 2 的整数幂
    private int m_textureWidth = 512;
    public int textureWidth {
        get {
            return m_textureWidth;
        }
        set {
            m_textureWidth = value;
            _UpdateMaterial();
        }
    }

    [SerializeField, SetProperty("backgroundColor")] 
    private Color m_backgroundColor = Color.white;
    public Color backgroundColor {
        get {
            return m_backgroundColor;
        }
        set {
            m_backgroundColor = value;
            _UpdateMaterial();
        }
    }

    [SerializeField, SetProperty("circleColor")] //圆点的颜色
    private Color m_circleColor = Color.yellow;
    public Color circleColor {
        get {
            return m_circleColor;
        }
        set {
            m_circleColor = value;
            _UpdateMaterial();
        }
    }

    [SerializeField, SetProperty("blurFactor")] //模糊因子，用于模糊圆形边界
    private float m_blurFactor = 2.0f;
    public float blurFactor {
        get {
            return m_blurFactor;
        }
        set {
            m_blurFactor = value;
            _UpdateMaterial();
        }
    }
    #endregion

    private Texture2D m_generatedTexture = null; //声明 Texture2D 纹理变量

    void Start () { //进行一些检测
        if (material == null) {
            Renderer renderer = gameObject.GetComponent<Renderer>();
            if (renderer == null) {
                Debug.LogWarning("Cannot find a renderer.");
                return;
            }
            material = renderer.sharedMaterial; //得到物体上的材质
        }
        _UpdateMaterial(); //调用 _UpdateMaterial() 来生成程序纹理
    }

    private void _UpdateMaterial() {
        if (material != null) {
            m_generatedTexture = _GenerateProceduralTexture(); //调用 _GenerateProceduralTexture() 生成一张程序纹理
            material.SetTexture("_MainTex", m_generatedTexture); //利用该函数把生成的纹理赋给材质
        }
    }

    private Color _MixColor(Color color0, Color color1, float mixFactor) {
        Color mixColor = Color.white;
        mixColor.r = Mathf.Lerp(color0.r, color1.r, mixFactor);
        mixColor.g = Mathf.Lerp(color0.g, color1.g, mixFactor);
        mixColor.b = Mathf.Lerp(color0.b, color1.b, mixFactor);
        mixColor.a = Mathf.Lerp(color0.a, color1.a, mixFactor);
        return mixColor;
    }

    private Texture2D _GenerateProceduralTexture() {
        Texture2D proceduralTexture = new Texture2D(textureWidth, textureWidth);

        float circleInterval = textureWidth / 4.0f; //定义圆与圆之间的距离
        float radius = textureWidth / 10.0f; //定义圆半径
        float edgeBlur = 1.0f / blurFactor; //定义模糊系数

        for (int w = 0; w < textureWidth; w++) {
            for (int h = 0; h < textureWidth; h++) {
                Color pixel = backgroundColor; //使用背景颜色初始化

                //依次画九个圆
                for (int i = 0; i < 3; i++) {
                    for (int j = 0; j < 3; j++) {
                        //计算当前绘制的圆的圆心位置
                        Vector2 circleCenter = new Vector2(circleInterval * (i + 1), circleInterval * (j + 1));

                        //计算像素与圆的距离
                        float dist = Vector2.Distance(new Vector2(w, h), circleCenter) - radius;

                        //通过距离模糊圆的边界
                        Color color = _MixColor(circleColor, new Color(pixel.r, pixel.g, pixel.b, 0.0f), Mathf.SmoothStep(0f, 1.0f, dist * edgeBlur));

                        //与之前得到的颜色进行混合
                        pixel = _MixColor(pixel, color, color.a);
                    }
                }
                proceduralTexture.SetPixel(w, h, pixel);
            }
        }
        proceduralTexture.Apply(); //调用 Texture2D.Apply 函数来强制把像素值写入纹理中

        return proceduralTexture;
    }
}
```

效果如下图：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/15/YzuqtnJWpxZhgaV.jpg" width = "70%" height = "70%" alt="图54- 脚本生成的程序纹理"/>
</div>


### Unity 的程序材质
在 Unity 中，有一种专门使用程序纹理的材质，叫做**程序材质 Procedural Materials**。这类材质和我们之前使用的那些材质在本质上是一样的，只是使用的是程序纹理，程序材质使用的程序纹理不是在 Unity 中创建的，而是使用一个名为 **Substance Designer** 的软件在 Unity 外部生成的。这些材质都是以 .sbsar 为后缀的，把这些材质导入 Unity 中后（现在的版本需要在 asset store 中安装 Substance 3D for Unity 插件），Unity 就会生成一个**程序纹理资源 Procedural Material Asset**。程序纹理资源可以包含一个或多个程序材质，通过单击它，我们可以在程序纹理面板中看到材质的使用的 Unity Shader 及其属性、生成程序纹理使用的纹理属性、材质预览等信息。


# 第十章 让画面动起来
## Unity Shader 中的内置时间变量
动画效果往往都是把时间添加到一些变量的计算中，而 Unity Shader 提供了一系列关于时间的内置变量：

| 名称 | 类型 | 描述 |
| :---- | :---- | :---- |
| \_Time | float4 | t是自该场景加载开始所经过的时间，4个分量的值分别是（t/20, t, 2t, 3t） |
| \_SinTime | float4 | t 是时间的正弦值，4 个分量分别是 (t/8, t/4, t/2, t) |
| \_CosTime | float4 | t 是时间的余弦值，4 个分量的值分别是 (t/8, t/4, t/2, t) |
| unity_DeltaTime | float4 | dt 是时间增量，4 个分量的值分别是 (dt, 1/dt, smoothDt, 1/smoothDt) |

## 纹理动画
在资源比较局限的移动平台上，往往会使用纹理动画代替复杂的粒子效果等模拟各种动画效果。

### 序列帧动画
最常见的纹理动画之一就是序列帧动画，其原理很简单，依次播放一系列关键帧图像，当播放速度达到一定数值时，其看起来就是一个连续的动画。其优点在于灵活性很强，不需要任何物理计算就可以得到细腻的动画效果。其缺点在于每张关键帧图像都不一样，从而制造一张出色的序列帧纹理所需要的美术工程量很大。

本书资源提供了一张包含关键帧图像的图像（Assets/Textures/Chapter11/boom.png），接下来通过案例来了解序列帧动画的实现：

完成如下准备工作：  
①新建名为 Scene_11_2_1 的场景，并去掉天空盒；  
②新建名为 ImageSequenceAnimationMat 的材质；  
③新建名为 Chapter11-ImageSequenceAnimation 的 Unity Shader，并赋给上一步创建的材质；  
④在场景中新建一个四边形 Quad，调整它的位置，使其正对相机，并将第2步创建的材质赋给它。

序列帧动画的精髓在于需要在每个时刻计算该时刻下应该播放的关键帧的位置，并对该关键帧进行纹理采样，代码如下：  

```  C C for Graphics
Shader "Unity Shaders Book/Chapter 11/Image Sequence Animation" {
    Properties {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)    
        _MainTex ("Image Sequence", 2D) = "white" {} //包含了所有关键帧图像的纹理
        _HorizontalAmount ("Horizontal Amount", Float) = 4 //该图像在水平方向包含的关键帧图像的个数
        _VerticalAmount ("Vertical Amount", Float) = 4 //该图像在竖直方向包含的关键帧图像的个数
        _Speed ("Speed", Range(1, 100)) = 30 //控制序列帧动画的播放速度
    }
    SubShader {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"} //序列帧通常是透明纹理，从而需要设置 Pass 相关状态，以渲染透明效果
        
        Pass {
            Tags { "LightMode"="ForwardBase" }
            
            ZWrite Off //关闭深度写入
            Blend SrcAlpha OneMinusSrcAlpha //开启并设置混合模式
            
            CGPROGRAM
            
            #pragma vertex vert  
            #pragma fragment frag
            
            #include "UnityCG.cginc"
            
            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            float _HorizontalAmount;
            float _VerticalAmount;
            float _Speed;
              
            struct a2v {  
                float4 vertex : POSITION; 
                float2 texcoord : TEXCOORD0;
            };  
            
            struct v2f {  
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };  
            
            v2f vert (a2v v) {  
                v2f o;  
                o.pos = UnityObjectToClipPos(v.vertex);  
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex); 
                return o;
            }  
            
            fixed4 frag (v2f i) : SV_Target {
                //前三列代码计算行列数，以获取关键帧所在的行列索引值
                float time = floor(_Time.y * _Speed); //_Time.y 是该场景加载后经过的时间，把其与速度属性相乘得到模拟的时间，并用 Cg 的 floor 函数对结果值取整来得到整数时间 time
                float row = floor(time / _HorizontalAmount); //time 除以行的个数的商作为当前对应的行索引
                float column = time - row * _HorizontalAmount; //余数为列索引

                //接下来将采样坐标映射到每个关键帧图像的坐标范围内
                //half2 uv = float2(i.uv.x /_HorizontalAmount, i.uv.y / _VerticalAmount); 这行是将 uv 等分，得到每个子图像的纹理坐标范围
                //uv.x += column / _HorizontalAmount;
                //uv.y -= row / _VerticalAmount; 使用减法是因为序列帧播放顺序是自上而下，而 uv 坐标是自下而上的
                half2 uv = i.uv + half2(column, -row);
                uv.x /=  _HorizontalAmount;
                uv.y /= _VerticalAmount;
                
                fixed4 c = tex2D(_MainTex, uv);
                c.rgb *= _Color;
                
                return c;
            }
            ENDCG
        }  
    }
    FallBack "Transparent/VertexLit"
}
```

> 上面原书中片元着色器代码有点小问题，直接从图片的最下面一行的第一张开始播放，但因为图片的 Wrap Mode 是 Repeat 模式，所以不影响整体表现。加一行 `row -= _VerticalAmount - 1` 就可以从第一行开始播放了。

将 Boom.png 勾选 Alpha Is Transparency 属性，任何赋给材质，并将 Horizontal Amount 和 Vertical Amount 设为 8。效果如下：  

<table><tr>
<td><img src='https://s2.loli.net/2023/12/18/DxIHLk9ZGjSA8Mr.png' width="400" alt="图55- 本节使用的序列帧图像"></td>
<td><img src='https://s2.loli.net/2023/12/18/fpnN4ew8GxQo51Z.gif' width="400" alt="图56- 使用序列帧动画来实现爆炸效果"></td>
</tr></table>

### 滚动的背景
很多 2D 游戏都使用了不断滚动的背景来模拟游戏角色在场景中的穿梭，这些背景往往包含了多个层来模拟一种视差效果。而这些背景的实现往往是利用了纹理动画。

完成如下准备工作：  
①新建名为 Scene_11_2_2 的场景，去掉天空盒，同时将相机的投影模式设置为正交投影；  
②新建名为 ScrollingBackgroundMat 的材质；  
③新建名为 Chapter11-ScrollingBackground 的 Unity Shader，并赋给上一步创建的材质；  
④在场景中创建一个四边形 Quad，调整其位置、大小，使它充满相机的视野范围，并将第2步创建的材质赋给它。

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 11/Scrolling Background" {
    Properties {
        _MainTex ("Base Layer (RGB)", 2D) = "white" {} //第一层（较远）的纹理
        _DetailTex ("2nd Layer (RGB)", 2D) = "white" {} //第二层（较近）的纹理
        _ScrollX ("Base layer Scroll Speed", Float) = 1.0 //第一层的水平滚动速度
        _Scroll2X ("2nd layer Scroll Speed", Float) = 1.0 //第二层的滚动速度
        _Multiplier ("Layer Multiplier", Float) = 1 //控制纹理的整体亮度
    }
    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        Pass { 
            Tags { "LightMode"="ForwardBase" }
            
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "UnityCG.cginc"
            
            sampler2D _MainTex;
            sampler2D _DetailTex;
            float4 _MainTex_ST;
            float4 _DetailTex_ST;
            float _ScrollX;
            float _Scroll2X;
            float _Multiplier;
            
            struct a2v {
                float4 vertex : POSITION;        
                float4 texcoord : TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float4 uv : TEXCOORD0;
            };
            
            v2f vert (a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex); 

                //通过利用 TRANSFORM_TEX 来得到初始纹理坐标，再通过 _Time.y 变量在水平空间上对纹理坐标进行偏移，从而达到滚动的效果。Cg 的 frac 函数返回标量或者矢量的小数部分
                o.uv.xy = TRANSFORM_TEX(v.texcoord, _MainTex) + frac(float2(_ScrollX, 0.0) * _Time.y);
                o.uv.zw = TRANSFORM_TEX(v.texcoord, _DetailTex) + frac(float2(_Scroll2X, 0.0) * _Time.y);

                //最后将两张纹理的纹理坐标存储到同一个变量 o.uv 中，以减少占用的插值寄存器空间
                return o;
            }
            
            fixed4 frag (v2f i) : SV_Target {
                fixed4 firstLayer = tex2D(_MainTex, i.uv.xy);
                fixed4 secondLayer = tex2D(_DetailTex, i.uv.zw);
                
                fixed4 c = lerp(firstLayer, secondLayer, secondLayer.a); //使用第二层纹理的透明通道来混合两张纹理，利用 lerp 函数
                c.rgb *= _Multiplier;        //使用 _Multiplier 与输出颜色进行相乘从而调整背景亮度
                return c;
            }
            ENDCG
        }
    }
    FallBack "VertexLit"
}
```

导入两张背景纹理，资源在 Assets/Textures/Chapter11/far_background.png 和 near_background.png，调整滚动速度，效果如下（只截取了 2 秒时间）：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/18/a6g3Sm9wN1z2fpL.gif" width = "70%" height = "70%" alt="图57- 无限滚动的背景"/>
</div>

## 顶点动画
### 流动的河流
使用正弦函数等来模拟水流的波动效果，准备工作如下：  
①新建名为 Scene_11_3_1 的场景，并去掉天空盒子；  
②将场景相机投影类型调整为正交投影；  
③新建 3 个材质，分别命名为 WaterMat、WaterMat1、WaterMat2（需要模拟多层水流效果）；  
④新建名为 Chapter11-Water 的 Unity Shader，并分别赋给上一步创建的 3 个材质；  
⑤在场景中创建 3 个 Water 模型，调整其位置、大小和方向，将第二步创建的材质分别赋给它们（模型详见Assets/Models/Chap11/water_fall.fbx）

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 11/Water" {
    Properties {
        _MainTex ("Main Tex", 2D) = "white" {} //河流纹理
        _Color ("Color Tint", Color) = (1, 1, 1, 1) //用于控制整体颜色
        _Magnitude ("Distortion Magnitude", Float) = 1 //控制水流波动的幅度
        _Frequency ("Distortion Frequency", Float) = 1 //用于控制波动频率
        _InvWaveLength ("Distortion Inverse Wave Length", Float) = 10 //用于控制波长的倒数，其越大，波长越小
        _Speed ("Speed", Float) = 0.5 //河流纹理的移动速度
    }
    
    SubShader {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent" "DisableBatching"="True"} //通过 DisableBatching 标签可以直接指明是否对该 subshader 采用批处理，一些 SubShader 在使用 Unity 的批处理功能时会出现问题，因为批处理会合并所有相关的模型，而这些模型各自的模型空间就会丢失。而在河流模拟中需要在物体的模型空间下对顶点位置进行偏移，从而需要取消对该 shader 进行批处理
        
        Pass {
            Tags { "LightMode"="ForwardBase" }
            
            ZWrite Off
            Blend SrcAlpha OneMinusSrcAlpha
            Cull Off //关闭剔除功能，从而让水流的每个面都能显示
            
            CGPROGRAM  
            #pragma vertex vert 
            #pragma fragment frag
            
            #include "UnityCG.cginc" 
            
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed4 _Color;
            float _Magnitude;
            float _Frequency;
            float _InvWaveLength;
            float _Speed;
            
            struct a2v {
                float4 vertex : POSITION;
                float4 texcoord : TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };
            
            v2f vert(a2v v) { //在顶点着色器中进行相关的顶点动画
                v2f o;
                
                float4 offset;
                //首先计算顶点位移量，因为只希望对顶点的 x 方向进行位移，因此 yzw 的位移量被设置为 0。这里对 x 方向偏移是因为资源里给的模型的 x 轴是上下方向，可以查看模型的 local 坐标确认。
                offset.yzw = float3(0.0, 0.0, 0.0); 
                //利用 _Frequency 属性和内置的 _Time.y 来控制正弦函数的频率。同时为了让不同的位置具有不同的位移，从而需要加上模型空间下的位置分量，并乘以 _InvWaveLength 来控制波长，最后通过乘以_Magnitude属性来控制波动幅度从而得到最终的位移
                offset.x = sin(_Frequency * _Time.y + v.vertex.x * _InvWaveLength + v.vertex.y * _InvWaveLength + v.vertex.z * _InvWaveLength) * _Magnitude;

                //将位移量添加到顶点位置上，并进行正常的顶点变换即可
                o.pos = UnityObjectToClipPos(v.vertex + offset); 
                
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
                o.uv +=  float2(0.0, _Time.y * _Speed); //这个速度是纹理的流动速度
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                fixed4 c = tex2D(_MainTex, i.uv);
                c.rgb *= _Color.rgb;
                return c;
            } 
            ENDCG
        }
    }
    FallBack "Transparent/VertexLit"
}
```

> 补充：注意看模型的 object space coordinate，和世界坐标是不一样的。若 \_InvWaveLength 为 0，水流是个长方形并且整体上下晃动没有波形。\_Magnitude 控制整体上下晃动的幅度，\_Frequency 控制上下晃动频率。\_InvWaveLength 控制的就是波形，水流的方向和模型空间的 z 轴是一致的，所以对波形影响最大的是 v.vertex.z ，顶点 z 轴的差异产生了波形，而 x 轴也有影响，这个水带的上下波形是有相位差的，这个是由 x 分量产生的影响。

将水体纹理分别赋到3个材质面板的 Main Tex 属性上（纹理详见 Assets/Textures/Chap11/water.psd），并分别配置其他参数，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/19/2pBWSz49kwyIDZE.gif" width = "70%" height = "70%" alt="图58- 使用顶点动画来模拟2D的河流"/>
</div>

### 广告牌
另一种常见的顶点动画就是**广告牌技术 Billboarding**。广告牌技术会根据视角方向来旋转一个被纹理着色的多边形，使得多边形看起来好像总是面对着摄像机。广告牌技术被用于很多应用，比如渲染烟雾、云朵、闪光效果等。

广告牌技术的本质就是构建一个**旋转矩阵**，我们知道一个变换矩阵需要 3 个基向量。广告牌技术使用的基向量通常就是**表面法线 normal**、**指向上方向 up** 以及**指向右方向 right** 。除此之外，还需要指定一个**锚点 anchor location**，这个锚点在旋转过程中是不变的，以此来确定多边形在空间中的位置。

广告牌技术的难点在于如何根据需求来构建 3 个相互正交的基向量。计算过程：  
①通过初始计算得到目标的表面法线（就是视角方向）、指向上的方向（两者一般不垂直）；  
②根据需求确定这两个方向哪一个是固定的：如果模拟草丛，我们希望指向上的方向是固定的，永远是（0, 1,  0）。如果模拟粒子效果，我们希望表面法线是固定的，永远指向视角方向；  
③假设法线方向是固定的，首先根据表面法线、指向上的方向的叉积，得到指向右的方向（通过叉积操作）：  

$$ right = up \times normal $$

④对其归一化后，由法线方向、指向右的方向，计算出正交的指向上的方向：  

$$ up' = normal \times right $$

这样就可以得到用于旋转的 3 个正交基了，如下图所示：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/19/W36kirHYV9SPwg8.jpg" width = "70%" height = "70%" alt="图59- 法线固定（总是指向视角方向）时，计算广告牌技术中的三个正交基的过程"/>
</div>

准备工作如下：  
①新建名为 Scene_11_3_2 的场景，并去掉天空盒子；  
②新建名为 BillboardMat 的材质；  
③新建名为 Chapter11-Billboard 的 Unity Shader，并赋给上一步创建的材质；  
④在场景中创建多个四边形 Quad，调整其位置和大小，并将第二步创建的材质赋给它们，这些四边形就是用于广告牌技术的广告牌。

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 11/Billboard" {
    Properties {
        _MainTex ("Main Tex", 2D) = "white" {} //广告牌显示的透明纹理
        _Color ("Color Tint", Color) = (1, 1, 1, 1) //用于控制显示整体颜
        _VerticalBillboarding ("Vertical Restraints", Range(0, 1)) = 1 //调整固定法线还是固定指向上的方向，即约束垂直方向的程度 
    }
    SubShader {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent" "DisableBatching"="True"} //关闭批处理，因为本例子中需要处理模型的各顶点
        
        Pass { 
            Tags { "LightMode"="ForwardBase" }
            
            ZWrite Off
            Blend SrcAlpha OneMinusSrcAlpha
            Cull Off //关闭剔除功能，从而让广告牌的每个面都能显示出来
        
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Lighting.cginc"
            
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed4 _Color;
            fixed _VerticalBillboarding;
            
            struct a2v {
                float4 vertex : POSITION;
                float4 texcoord : TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };
            
            v2f vert (a2v v) {
                v2f o;
                
                float3 center = float3(0, 0, 0); //选择模型空间的原点作为广告牌的锚点
                float3 viewer = mul(unity_WorldToObject,float4(_WorldSpaceCameraPos, 1)); //通过内置变量获取模型空间下的视角位置
                
                float3 normalDir = viewer - center; //根据观察位置和锚点计算原法线方向（视角方向）
                
                //根据 _VerticalBillboarding 控制垂直方向上的约束度。若 _VerticalBillboarding 为 1，意味着法线方向固定为视角方向；当 _VerticalBillboarding 为 0，法线的 y 轴为 0，则意味着向上方向固定为（0，1，0），因为这样法线和向上方向垂直，而 x、z 根据原法线视角方向变化，得到未归一化的新法线方向
                normalDir.y = normalDir.y * _VerticalBillboarding;
                normalDir = normalize(normalDir); //归一化来得到单位矢量
                
                //为防止法线方向和向上方向平行（如果平行，那么叉积结果为 0），即如果法线已经向上，那么向上方向为朝前方向。
                float3 upDir = abs(normalDir.y) > 0.999 ? float3(0, 0, 1) : float3(0, 1, 0);
                //根据法线方向和原向上方向得到向右方向，并进行归一化
                float3 rightDir = normalize(cross(upDir, normalDir));
                //根据法线方向和向右方向得到最后的新向上方向
                upDir = normalize(cross(normalDir, rightDir));

                //根据顶点位置对于锚点的偏移量以及 3 个基向量得到新的顶点位置
                float3 centerOffs = v.vertex.xyz - center;
                float3 localPos = center + rightDir * centerOffs.x + upDir * centerOffs.y + normalDir * centerOffs.z; 
              
                o.pos = UnityObjectToClipPos(float4(localPos, 1));
                o.uv = TRANSFORM_TEX(v.texcoord,_MainTex);        
                return o;
            }
            
            fixed4 frag (v2f i) : SV_Target {
                fixed4 c = tex2D (_MainTex, i.uv);
                c.rgb *= _Color.rgb;
                return c;
            }
            ENDCG
        }
    } 
    FallBack "Transparent/VertexLit"
```

> 上述 rightDir 和 upDir 是叉乘顺序可能有问题，因为在左手坐标系叉乘结果是左手法则，上述叉乘的顺序会导致基向量从左手坐标系变为右手坐标系，这会导致整个模型镜像变换。若把 Cull Off 改为 Cull back 可以看到模型看不到了。

需要说明的是，上面的例子使用的是 Unity 自带的四边形 Quad 作为广告牌，不能使用平面 Plane。这是因为上述代码的计算是建立在竖直摆放的多边形的基础上的。将星星纹理赋给材质面板的 Main Tex 属性，详见 Assets/Textures/Chap11/star.png，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/19/QxHBOsTvqCmaeNg.gif" width = "70%" height = "70%" alt="图60- 该图显示了当 Vertical Restraints 属性为1，即固定法线方向为观察视角时所得到的效果"/>
</div>

### 注意事项
顶点动画虽然灵活有效，但是有一些注意事项：  
①若在模型空间下进行顶点动画计算，那么批处理往往会破坏这种动画效果。然而，强制取消批处理，会带来一定的性能下降，增加 Draw Call。所以尽量避免使用模型空间下的一些绝对位置和方向来进行计算；  
②给包含顶点动画的物体添加阴影，无法通过调用内置 ShadowCaster Pass 实现，因为这个 Pass 里没有进行相关的顶点动画，得到的阴影是顶点变化前的。此时需要自定义 ShadowCaster Pass，在顶点着色器中重新计算一遍顶点的位置。在前面的实现中，因为 Fallback 设置为 Transparent/VertexLit，而 Transparent/VertexLit 没有定义  ShadowCaster Pass，因此不会产生阴影。

ShadowCaster Pass 的相关代码如下：  

``` C C for Graphics
Pass {
    Tags { "LightMode" = "ShadowCaster" }

    CGPROGRAM

    #pragma vertex vert
    #pragma fragment frag

    #pragma multi_compile_shadowcaster

    #include "UnityCG.cginc"

    float _Magnitude;
    float _Frequency;
    float _InvWaveLength;
    float _Speed;

    struct v2f { 
        V2F_SHADOW_CASTER;
    };

    v2f vert(appdata_base v) {
        v2f o;

        float4 offset;
        offset.yzw = float3(0.0, 0.0, 0.0);
        offset.x = sin(_Frequency * _Time.y + v.vertex.x * _InvWaveLength + v.vertex.y * _InvWaveLength + v.vertex.z * _InvWaveLength) * _Magnitude;
        v.vertex = v.vertex + offset;

        TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)

        return o;
    }

    fixed4 frag(v2f i) : SV_Target {
        SHADOW_CASTER_FRAGMENT(i)
    }
    ENDCG
}
```

效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/19/PFGwerLgQTdDJn7.jpg" width = "70%" height = "70%" alt="图61- 使用自定义的 ShadowCaster Pass 为变形物体绘制正确的阴影"/>
</div>

在自定义的阴影投射的 Pass 中，我们通常会使用 Unity 提供的内置宏 V2F_SHADOW_CASTER、TRANSFER_SHADOW_CASTER_NORMALOFFSET（旧版本中为 TRANSFER_SHADOW_CASTER）和 SHADOW_CASTER_FRAGMENT 来计算阴影投射时需要的各自变量。  

在上面代码中，首先在 v2f 结构体中使用了 V2F_SHADOW_CASTER 来定义阴影投射所需要定义的变量。在顶点着色器中计算偏移量，加到顶点位置变量中，剩下的交给 TRANSFER_SHADOW_CASTER_NORMALOFFSET。在片元着色器中，直接使用 SHADOW_CASTER_FRAGMENT 让 Unity 自动完成阴影投射的部分，把结果输出到深度图和阴影映射纹理中。

TRANSFER_SHADOW_CASTER_NORMALOFFSET 会使用名称 v 作为输入结构体，v 中需要包含顶点位置 v.vertex 和顶点法线 v.normal 的信息，我们也可以直接使用内置的 appdata_base 结构体，它包含了这些必需的顶点变量。