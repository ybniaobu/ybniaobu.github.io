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
| float3 ShadeVertexLightsFull(float4 vertex, float3 normal, int lightCount, bool spotLight) | 输入模型空间中的顶点位置和法线，计算 lightCount 个光源的光照以及环境光，如果spotLight 值为 true，那么这些光源会被当成聚光灯来处理，虽然结果更精确，但计算更加耗时，否则按点光源处理 |

### 延迟渲染路径
