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

> 本读书笔记为中级篇，主要内容为XXXXXXXXXXXXXXX。  
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

我们新建一个 Scene_9_4_5_b 的场景，新建 AlphaBlendWithShadowMat 的材质和 Chapter9-AlphaBlendWithShadow 的 Unity Shader，将 Chapter8-AlphaTestBothSided 复制进去，然后添加阴影的计算，并且它的 Fallback 是内置的 Transparent/VertexLit。效果如下：  

