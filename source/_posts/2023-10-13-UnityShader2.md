---
title: 《Unity Shader入门精要》读书笔记（二）
date: 2023-10-13 21:58:01
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
top_img: /images/black.jpg
cover: https://s2.loli.net/2023/10/15/RZftaNSscWoLH1u.gif
mathjax: true
---

> 本读书笔记为初级篇，主要内容为 Unity Shader 的结构及 Unity 提供的便利、基础光照、基础纹理和透明效果。  
> 读书笔记是对知识的记录与总结，但是对比较熟悉的内容不会再行描述。

# 第四章 开始 Unity Shader 学习之旅
## 一个简单的顶点/片元着色器
### 顶点/片元着色器的基本结构
Unity Shader 的基本结构包含了 Shader、Properties、SubShader、Fallback 等语义块，顶点/片元着色器也是若此，结构如下：

``` C C for Graphics
Shader "MyShaderName" {
    Properties {
        // 属性
    }
    SubShader {
        // 针对显卡 A 的 SubShader
        Pass {
            // 设置渲染状态和标签

            // 开始 Cg 代码片段
            CGPROGRAM
            // 该代码片段的编译指令。例如：
            #pragma vertex vert
            #pragma fragment frag

            // Cg 代码写在这里

            ENDCG

            // 其他设置
        }
        // 其他需要的 Pass
    }
    SubShader {
        // 针对显卡 B 的 SubShader
    }

    // 上述 SubShader 都失败后用于回调的 Unity Shader
    Fallback "VertexLit"
}
```

其中最重要的部分是 Pass 语义块，绝大部分代码都写在这里面。

### 简单示例
***1. 案例常用操作说明***  
后续所有的案例代码编写都会涉及创建场景、创建 Shader、创建材质、关闭天空盒等操作，在此做一下说明。为统一规范，建议在新建项目的 Assets 根目录下创建如下几个文件夹：

①Scenes 存放场景 ②Shaders 存放着色器代码 ③Materials 存放材质 ④Textures 存放贴图 ⑤Models 存放案例需要用到的模型 ⑥Scripts 存放案例需要编写的 C# 代码 ⑦Prefabs 存放预制体。

关闭天空盒：为了更清楚地看到 Shader 实现的效果，我们一般选择关闭天空盒。选中对应场景后，在单击菜单栏 -> Window -> Rendering -> Lighting Settings -> Environment -> Skybox Material，选择为 None 即可。

***2. 简单的顶点/片元着色器示例***  
①新建一个场景，命名为 Scene_5_2 ，并关闭天空盒；  
②新建一个 Unity Shader，命名为 Chapter5-SimpleShader；  
③新建一个材质，命名为 SimpleShaderMat，并选择步骤 2 创建的 Shader 赋给它；  
④新建一个球体，并更换材质为刚刚创建的 SimpleShaderMat。
⑤双击打开步骤 2 创建的 Shader 。删除里面所有代码，替换为下面代码：

``` C C for Graphics
// 第一行，通过 Shader 语义，定义这个 Shader 的名字，名字中使用'/'可定义该Shader的路径（或分组）
// 回到之前创建的材质，选择 Shader，可以看到多了一个 Unity Shaders Book 目录，下面的子目录 Chapter 5 里面就有我们目前的 Shader
Shader "Unity Shaders Book/Chapter 5/Simple Shader" {
    // Properties 语义并不是必需的
    SubShader {
        // SubShader 中没有进行任何渲染设置和标签设置，所以该 SubShader 将使用默认的设置
        Pass {
            // Pass 中没有进行任何渲染设置和标签设置，所以该 Pass 将使用默认的设置

            // 由 CGPROGRAM 和 ENDCG 包围 CG 代码片段
            CGPROGRAM

            // 告诉 Unity，顶点着色器的代码在 vert 函数中 （格式：#pragma vertex name）
            #pragma vertex vert
            // 告诉 Unity，片元着色器的代码在 frag 函数中 （格式：#pragma fragment name）
            #pragma fragment frag

            // 顶点着色器代码，它是逐顶点执行的
            // 通过 POSITION 语义，告诉顶点着色器，输入 v 是这个顶点的模型坐标
            // SV_POSITION 语义表示返回的 float4 类型的变量，是当前顶点在裁剪空间中的坐标
            // 注意：这两个语义是不能省略的，着色器通过语义来辨识各个变量是用来做什么的，并用在不同的底层处理中
            float4 vert(float4 v : POSITION) : SV_POSITION {
                return UnityObjectToClipPos(v); 
                // 或者使用这个语句 return mul(UNITY_MATRIX_MVP, v); 会被自动替换为上面语句
            }

            // 片元着色器代码
            // frag 函数没有任何输入，输出是一个 fixed4 类型的变量，并且使用了 SV_Target 语义进行限定
            // 通过 SV_Target 语义，告诉渲染器，把用户的输出颜色存储到一个渲染目标中（比如默认的帧缓存中）
            // 颜色的 RGBA 每个分量范围在[0, 1]，所以使用 fixed4 类型
            fixed4 frag() : SV_Target {
                return fixed4(1.0, 1.0, 1.0, 1.0);
            }

            ENDCG
        }
    }
}
```

> pragma 的释义和渊源：A pragma (from the Greek word meaning action) is used to direct the actions of the compiler in particular ways, but has no effect on the semantics of a program (in general).The programming language Ada was quite possibly the first compiler to use pragma to specify preprocessor directives. The word was used as a shortened form of "pragmatic information". When the C programming language was designed it didn't initially have pragma directives, but was quickly added to the specification to support custom compiler features.   
>  
> sv_position 中的 sv 指 system value，系统值语义是 Direct3D 10 的新增功能。所有系统值都以 SV 前缀开头。详见：<https://learn.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics?redirectedfrom=MSDN#System_Value> 

### 使用结构体获取模型数据
上面的例子中，顶点着色器使用 POSITION 语义得到了模型的顶点坐标，除了需要当前顶点的坐标外，还需要诸如：法线、切线、纹理坐标等信息时，通常会定义一个结构体，在结构体声明多个变量，并赋予其语义，可将更多的顶点信息传入到顶点着色器中。修改后的代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 5/Simple Shader" {
    SubShader {
        Pass {
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            // 使用结构体作为顶点着色器的输入，可以包含更多模型数据
            // a2v 是当前结构体的名字，可自行定义（写法：struct [StructName]）
            // 这里 a2v 表示 application to vertex ，意思是：把数据从应用阶段传递到顶点着色器中
            struct a2v {
                // POSITION 语义告诉 Unity，用模型的顶点空间坐标填充 vertex 变量
                float4 vertex : POSITION;
                // Normal 语义告诉 Unity，用模型的法线方向填充 normal 变量
                float3 normal : NORMAL;
                // TEXCOORD0 语义告诉 Unity，用模型的第一套纹理坐标填充 texcoord 变量
                float4 texcoord : TEXCOORD0;
                // 上述语义中的数据由使用该材质的 Mesh Renderer 组件提供。在每帧调用 Draw Call 时，Mesh Renderer 组件会把它负责渲染的模型数据发送给 Unity Shader
            };

            // 使用结构体作为输入参数，不需要写语义，因为语义在结构体里已经声明了
            float4 vert(a2v v) : SV_POSITION {
                // 从结构体中访问当前顶点的模型空间坐标，将其转为裁剪空间下的坐标
                return UnityObjectToClipPos(v.vertex);
            }

            fixed4 frag() : SV_Target {
                return fixed4(1.0, 1.0, 1.0, 1.0);
            }

            ENDCG
        }
    }
}
```

①一个自定义结构体的格式如下：  
```
struct StructName {
    Type Name: Semantic;
    Type Name: Semantic;
    ......
};
```

②对于顶点着色器的输入，Unity 支持的语义有：POSITION 、 NORMAL 、 TANGENT 、TEXCOORD0 、 TEXCOORD1 、 TEXCOORD2 、 TEXCOORD3 、 COLOR 等  

### 顶点着色器和片元着色器的通信
因为我们往往希望从顶点着色器输出一些数据，例如把模型的法线、纹理坐标等传给片元着色器，此时将涉及顶点着色器与片元着色器之间的通信，通过定义新的结构体可以解决该问题，修改后的代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 5/Simple Shader" {
    SubShader {
        Pass {
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };

            // 使用一个结构体定义顶点着色器的输出
            struct v2f {
                // SV_POSITION 语义告诉 Unity，pos 存储了顶点在裁剪空间中的位置信息
                float4 pos : SV_POSITION;
                // COLOR0 语义用于存储颜色信息，当需要存储更多颜色时，可继续用 COLOR1 、 COLOR2 等
                fixed3 color : COLOR0;
            };

            v2f vert(a2v v) {
                // 声明输出结构
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                // 将法线的值转变成颜色值，呈现到模型上
                // 因为 v.normal 的顶点法线方向，各分量范围是[-1, 1]，为了让其转变到颜色的范围[0, 1]，故做如下运算：
                o.color = v.normal * 0.5 + fixed3(0.5, 0.5, 0.5);
                return o;
            }

            // 将顶点输出的结构体传入片元着色器中
            fixed4 frag(v2f i) : SV_Target {
                // 结构体里定义的 color 是 fixed3 类型
                // 输出的颜色为 fixed4 类型，所以需要补上第四个分量
                // 将插值后的 i.color 显示到屏幕上
                return fixed4(i.color, 1.0);
            }

            ENDCG
        }
    }
}
```

①此时保存代码后，回到 Unity 的 Scene 窗口，可以看到一个 RGB 色彩空间球体。  
②顶点着色器的输出结构中，必须包含一个变量，它的语义是 SV_POSITION。否则渲染器无法获取裁剪空间中的顶点坐标，也就无法把顶点渲染到屏幕上。  
③顶点着色器是逐顶点调用的，片元着色器是逐片元（像素）调用的，这意味着顶点着色器的调用次数远远小于片元着色器。为了让每个片元着色器都能有一个结构体的输入，Unity 会将非顶点像素的片元着色器的输入结构体的数据由顶点着色器的输出经过插值后得到结果。比如三角形面片中某个像素点 V：

$$V=\alpha V_A + \beta V_B + \gamma V_C$$
<center>$V_A, V_B, V_C$  can be positions, texture, coordinates, color, normal, depth, material attributes...</center>

###  如何使用属性
若将参数写在 Properties 语义块中，就可以在材质面板中快速调节 Unity Shader 中的参数。一个 Shader 通常会被多个材质使用，不同的材质会对 Shader 进行不同的设置，而属性则是材质和 Shader 之间的通道，允许材质传入不同的值给 Shader，以实现不同的视觉效果。继续修改之前的代码：

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 5/Simple Shader" {
    Properties {
        // 声明一个 Color 类型的属性
        _Color("Color Tint", Color) = (1.0, 1.0, 1.0, 1.0)
    }
    SubShader {
        Pass {
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            // CG 代码中，若需要使用属性数值，需要声明变量，变量名与属性名一致，一般以下划线开头
            fixed4 _Color;

            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };

            struct v2f {
                float4 pos : SV_POSITION;
                fixed3 color : COLOR0;
            };

            v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.color = v.normal * 0.5 + fixed3(0.5, 0.5, 0.5);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target {
                fixed3 color = i.color;
                // 使用 _Color 属性来影响法线转换成的颜色效果
                color *= _Color.rgb;
                return fixed4(color, 1.0);
            }

            ENDCG
        }
    }
}
```

①保存代码并回到 Unity，可以看到使用该 Shader 的材质，多了一个 Color Tint 属性，可以直接在上面更改。  
②ShaderLab 中属性的类型和 Cg 中变量的类型之间的匹配关系如下表：  

| ShaderLab 属性类型 | Cg 变量类型 |
| :---- | :---- |
| Color, Vector | float4, half4, fixed4 |
| Range, Float | float, half, fixed |
| 2D | sampler2D |
| 3D | sampler3D |
| Cube | samplerCube |

③关于 uniform 关键字，例如：`uniform fixed4 _Color;`。uniform 关键词是 Cg 语言中修饰变量和参数的一种修饰词，仅用于提供一些该变量的初始值是如何指定和存储的相关信息，在 Unity Shader 中可以省略。

## Unity 提供的内置文件和变量
为了方便开发者的编码过程，Unity 提供了很多内置文件，这些文件包括了很多提前定义的函数、变量和宏等。本节将给出这些文件和变量的概览。

### 内置的包含文件
**包含文件 include file**，是类似于 C++ 中头文件的一种文件。在 Unity 中，它们的文件后缀为 .cginc。使用 #include 指令引入，在 CGPROGRAM 和 ENDCG 之间编写，这样就可以使用 Unity 提供的一些有用的变量和帮助函数，例如：

    CGPROGRAM
    //...
    #include "UnityCG.cginc"
    //...
    ENDCG

内置文件一般在 Unity 安装路径下的 /Data/CGIncludes 中可以找到。下表给出一些 CGIncludes 中主要的包含文件和主要用处：  

| 文件名 | 描述 |
| :---- | :---- |
| UnityCG.cginc | 包含了最常用的帮助函数，宏和结构体等 |
| UnityShaderVariables.cginc | 在编译 Unity Shader 时会被自动包含进来，包含了许多内置全局变量，如 UNITY_MATRIX_MVP 等 |
| Lighting.cginc | 包含了各种内置光照模型，如果编写的是 Surface Shader，会被自动包含进来 |
| HLSLSupport.cginc | 在编译 Unity Shader 时会被自动包含进来，声明了许多用于跨平台编译的宏的定义 |
| UnityStandardBRDF.cginc <br> UnityStandardCore.cginc | 用于实现基于物理的渲染，后面章节会遇到 |

### UnityCG.cginc
UnityCG.cginc 是经常被使用的内置文件，里面预先定义了一些结构体和函数，可以直接使用作为顶点着色器的输入和输出。下表给出了 UnityCG.cginc 中一些常用的结构体和包含的变量：  

| 名称 | 描述 | 包含的变量 |
| :---- | :---- | :---- |
| appdata_base | 可用于顶点着色器的输入 | 顶点位置、顶点法线、第一组纹理坐标 |
| appdata_tan | 可用于顶点着色器的输入 | 顶点位置、顶点切线、顶点法线、第一组纹理坐标 |
| appdata_full | 可用于顶点着色器的输入 | 顶点位置、顶点切线、顶点法线、四组(或更多)纹理坐标 |
| appdata_img | 可用于顶点着色器的输入 | 顶点位置、第一组纹理坐标 |
| v2f_img | 可用于顶点着色器的输出 | 裁剪空间中的位置、纹理坐标 |

UnityCG.cginc 也预定义了一些帮助函数，使用频率较高的有：  

| 函数名 | 描述 |
| :---- | :---- |
| float3 WorldSpaceViewDir(float4 v) | 输入一个模型空间中的顶点位置，返回世界空间中从该点到摄像机的观察方向 |
| float3 ObjSpaceViewDir(float4 v) | 输入一个模型空间中的顶点位置，返回模型空间中从该点到摄像机的观察方向 |
| float3 WorldSpaceLightDir(float4 v) | 仅可用于前向渲染中，输入一个模型空间中的顶点位置，返回世界空间中从该点到光源的光照方向，没有被归一化 |
| float3 ObjSpaceLightDir(float4 v) | 仅可用于前向渲染中，输入一个模型空间中的顶点位置，返回模型空间中从该点到光源的光照方向，没有被归一化 |
| float3 UnityObjectToWorldNormal(float3 norm) | 把法线方向从模型空间转换到世界空间中 |
| float3 UnityObjectToWorldDir(float3 dir) | 把方向矢量从模型空间变换到世界空间中 |
| float3 UnityWorldToObjectDir(float3 Dir) | 把方向矢量从世界空间变换到模型空间中 |

建议在 UnityCG.cginc 中找到上述结构体的声明和函数的定义，并尝试去理解。

## Unity 提供的 CG/HLSL 语义
### 什么是语义
上面提到的 POSITION、SV_POSITION、COLOR0 这些都是 Cg/HLSL 提供的**语义 semantics**。可以在微软官网查看关于 DirectX 的文档中查看语义的详细说明。语义实际上是一个赋给 shader 输入和输出的字符串，该字符串表达了这个参数的含义，即这些语义可以让 shader 知道从哪里读取数据，并把数据输出到哪里。

DirectX 10 以后出现了一种新的语义类型，就是**系统数值语义 system-value semantics**。这些语义以 SV，即 system-value 系统数值开头，在渲染流水线中有特殊的含义。

例如，在上面的代码中 SV_POSITION 语义去修饰顶点着色器的输出变量 pos，那么就表示 pos 包含了可用于光栅化的变换后的顶点坐标(即齐次裁剪坐标系中的坐标)。用这些语义描述的变量是不可以随便赋值的，因为流水线需要使用它们来完成特定的目的，例如渲染引擎会把用 SV_POSITION 修饰的变量经过光栅化后显示在屏幕上。绝大多数平台上 SV_POSITION 和 POSITION 是等价的，一些 Shader 会使用 POSITION 而非 SV_POSITION 来修饰顶点着色器的输出。但是在某些平台上必须使用 SV_POSITION 来修饰顶点着色器的输出(例如索尼 PS4 )，为了让 Shader 有更好的跨平台性，对于一些有特殊含义的变量最好用 SV 开头进行修饰。

### Unity 支持的语义
①下表总结了从应用阶段传递模型数据给顶点着色器时 Unity 使用的常用语义。这些语义虽然没有使用 SV 开头，但 Unity 内部赋予了它们特殊的含义：  

| 语义 | 描述 |
| :---- | :---- |
| POSITION | 模型空间中的顶点位置，通常为 float4 类型 |
| NORMAL | 顶点法线，通常是 float3 类型 |
| TANGENT | 顶点切线，通常是 float4 类型 |
| TEXCOORDn | 该顶点的纹理坐标，TEXCOORD0 表示第一组纹理坐标，依次类推。通常是 float2 或 float4 类型 |
| COLOR | 顶点颜色，通常是 fixed4 或 float4 类型 |

其中，TEXCOORDn 中 n 的数目是和 Shader Model 有关的，例如一般在 Shader Model 2（Unity 默认编译到的 Shader Model 版本）和 Shader Model 3 中，n = 8，而在 Shader Model 4 和 5 中，n = 16，一个模型的纹理坐标组数一般不超过2，往往只使用 TEXCOORD0 和 TEXCOORD1，在 Unity 内置结构体 appdata_full 中，最多使用 6 个纹理坐标。

②下表总结了从顶点着色器到片元着色器阶段 Unity 支持的语义：  

| 语义 | 描述 |
| :---- | :---- |
| SV_POSITION | 裁剪空间中的顶点坐标，结构体中必须包含一个用该语义修饰的变量。等同于DiectX9中的 POSITION，但最好使用 SV_POSITION |
| COLOR0 | 通常用于输出第一组顶点颜色，但不是必需的 |
| COLOR1 | 通常用于输出第二组顶点颜色，但不是必需的 |
| TEXCOORD0~TEXCOORD7 | 通常用于输出纹理坐标，但不是必需的 |

除了 SV_POSITION 有特别含义外，其他语义没有明确要求，我们可以随意存储任意值到这些语义描述变量中，比如把一些自定义的数据从顶点着色器传递给片元着色器。

③下表给出了 Unity 中支持的片元着色器的输出语义

| 语义 | 描述 |
| :---- | :---- |
| SV_Target | 输出值将存储到渲染目标 render target 中。等同于 DirectX 9 中的 COLOR 语义，但最好使用 SV_Target |

> 一个语义可以使用的寄存器只能处理 4 个浮点数 float。因此，若想定义矩阵类型，比如 float3×4、float4×4 等变量就需要使用更多空间。一个方法就是把这些变量拆分为多个变量，比如拆分为 4 个 float4 类型的变量。  
>  
> 为什么一个 float4x4 还要拆开来存储，主要是因为在 GPU 中，寄存器的数量是有限的。为了最大限度地利用寄存器的空间，Unity Shader 将 float4x4 拆成四个 float4 存储。这样可以确保矩阵的每一列都能被存储在一个寄存器中，从而提高了 Shader 的运行效率。需要注意的是，即使寄存器可以存储数组，但是在实际开发中，应该尽量避免在 Shader 中使用大量的数组。因为数组的使用会占用大量的寄存器，导致 Shader 的性能下降。如果必须使用数组，可以尝试使用常量缓冲区来传递数组数据，以减少寄存器的占用量。

## Unity 内置的帧调试器
Unity 内置的**帧调试器 Frame Debugger** 可方便快捷地为我们呈现模型的每一帧渲染情况，单击菜单 - Window - Analysis - FrameDebuger，可打开帧调试器窗口。

单击 Frame Debug 窗口的左上角的 Enable 按钮，可看到渲染情况。单击前进后退按钮，可以重放渲染事件。这里不做详细介绍，有需求需额外了解。

## 渲染平台的差异
Unity 具有跨平台的特性，大多数的平台差异 Unity 底层都帮我们处理了，但是依然有些差异需要我们自己处理。这里不做摘抄，详见书第115-117页，遇到问题再解决。

## 规范 Shader 代码的建议
### 数值类型
Cg/HLSL 中三种精度的数值类型：  

| 类型 | 精度 |
| :---- | :---- |
| float | 最高精度的浮点值。通常使用 32 位来存储 |
| half | 中等精度的浮点值。通常使用 16 位来存储，精度范围是 -60,000 ~ +60,000 |
| fixed | 最低精度的浮点值。通常使用 11 位来存储，精度范围是 -2.0 ~ +2.0 |

不同的平台和 GPU 上，三者实际的精度可能和上面描述的不一致：  
①大多数现代桌面 GPU 会把所有计算按最高的浮点精度计算，即：float、half、fixed 在这些平台上是等价的，都等同于 float；  
②但在移动平台 GPU 上，它们的确有不同的精度范围，不同的精度浮点运算速度会有差异；  
③fixed 精度在现代大多数GPU上，被当做和 half 同等精度对待。

但是在书写上，我们还是要保证尽可能使用精度较低的类型，以此来优化 Shader 的性能，尤其在移动平台上。fixed 用来存储颜色和单位矢量；half 存储更大范围的数据；最差情况下再选择使用 float。

### 规范语法
DirectX 平台对 Shader 的语义有更加严格的要求。按最严格的要求书写，防止偏门的写法导致某些平台无法运行。比如：

    // 规范的写法
    float4 v = float4(0.0, 0.0, 0.0, 0.0);

    // 不规范的写法
    float4 v = float4(0.0);

### 避免不必要的计算
在 Shader 中进行过多的运算，可能会收到以下 Unity 的错误提示：  
① temporary register limit of 8 exceeded  
② Arithmetic instruction limit of 64 exceeded; 65 arithmetic instructions needed to complie program

这是因为需要的临时寄存器数目或指令数目超过了当前可支持的数目。不同的 Shader Target、不同的着色器阶段，我们可使用的寄存器数目和指令数目都是不同的。通常，可以通过指定更高等级的 Shader Target 来消除这些错误。下表给出了 Unity 目前支持的一些 Shader Target：  

| 指令 | 描述 |
| :---- | :---- |
| #pragma target 2.0 | 默认的 Shader Target 等级。相当于 Direct3D 9 上的 Shader Model 2.0，不支持对顶点纹理的采样，不支持显式的 LOD 纹理采样等 |
| #pragma target 3.0 | 相当于 Direct3D 9 上的 Shader Model 3.0，支持对顶点纹理的采样等 |
| #pragma target 4.0 | 相当于 Direct3D 10 上的 Shader Model 4.0，支持几何着色器等 |
| #pragma target 5.0 | 相当于 Direct3D 11 上的 Shader Model 5.0 |

Shader Model 是微软提出的一套规范，决定了 Shader 的各个特性的能力，比如能使用的运算指令数目、寄存器个数等等。

### 慎用分支和循环语句
对于分支和循环语句，GPU 的实现和 CPU 是不同的，GPU 处理这些语句会消耗更多的性能，降低 GPU 并行处理的速度。因此最好把计算往渲染管线的前面移动，比如把片元着色器计算放在顶点着色器中，或者直接在 CPU 进行预运算，再将结果传给 Shader。若无法避免使用分支语句，最好：  
①分支判断语句中使用的条件变量最好是常量；  
②每个分支中包含的操作指令数尽可能少；  
③分支的嵌套层数尽可能少。


# 第五章 Unity 中的基础光照
从宏观上说，渲染包含了两大部分：决定一个像素的可见性，决定这个像素上的光照计算。而光照模型就是用于决定着一个像素上进行怎样的光照计算。

本章着重描述光照模型的原理，因此实现的 Shader 往往不能直接应用到实现项目中，会缺少阴影、光照衰减等效果。

## 光的物理知识
通过模拟真实的光照环境来生成一张图像，需要考虑三种物理现象：  
①首先，光线从**光源 light source** 中被发射出来；  
②然后，光线和场景中的一些物体相交：一些光线被物体吸收了，而另一些光线被散射到其他方向；  
③最后，摄像机吸收了一些光，产生了一张图像。

### 光源
实时渲染中，光源通常被当成一个没有体积的点，用 $l$ 来表示它的方向。在光学中，用**辐照度 irradiance** 来量化光。

①对于平行光来说，辐照度可通过计算垂直于 $l$ 的单位面积上单位时间内穿过的能量得到。  
②对于不垂直于物体表面的光，辐照度可使用光源方向 $l$ 和表面法线 $n$ 之间的余弦值得到。  
如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/03/zOSb96wcEgG4FB5.jpg" width = "70%" height = "70%" alt="图13- 在左图中，光是垂直照射到物体表面，因此光线之间的垂直距离保持不变；而在右图中，光是斜着照射到物体表面，在物体表面光线之间的距离是d/cosθ，因此单位面积上接收到的光线数目要少于左图"/>
</div>

因为辐照度是和照射在物体表面时光线之间的距离 d/cosθ 成反比，因此辐照度和 cosθ 成正比。cosθ 可用光源方向 $l$ 和表面法线 $n$ 的点积得到，这就是使用点积来计算辐照度的由来。

### 吸收和散射
光线由光源发射出来后，就会与一些物体相交。通常，相交的结果只有两个，即**散射 scattering** 和**吸收 absorption**。  

①散射：只改变光线的方向，不改变光线的密度和颜色。光线在物体表面经过散射后，有两种方向：  
&emsp;&emsp; - 散射到物体内部：被称为**折射 refraction** 或**透射 transmission**；  
&emsp;&emsp; - 散射到物体外部：被称为**反射 reflection**。  
对于不透明物体，折射进入物体内部的光线会继续和内部的颗粒进行相交，其中一部分光线重新发射出物体表面，另一部分则被物体吸收。那些从物体表面重新发射出的光线具有和入射光线不同的方向分布和颜色。  
②吸收：只改变光线的密度和颜色，不改变光线的方向。

为了区分上述两种不同的 *散射* 方向，在光照模型中用了不同的部分来计算它们：  
①**高光反射 specular** 部分表示物体表面是如何反射光线的。  
②**漫反射 diffuse** 部分表示有多少光线会被折射、吸收和散射出表面。

根据入射光线的数量和方向，可以计算出出射光线的数量和方向，通常用**出射度 exitance** 来描述它。辐照度和出射度之间满足线性关系，它们之间的比值就是材质的漫反射和高光反射属性。

在本章中，假设漫反射部分是没有方向性的，即光线在所有方向上平均分布的，同时也只考虑在某一特定方向上的高光反射。

### 着色
**着色 shading** 指根据材质属性（如漫反射属性等）、光源信息（如光源方向、辐照度等），使用一个公式去计算沿某个观察方向的出射度的过程。这个公式称为**光照模型 lighting model**。不同光照模型有不同的目的，有些描述粗糙物体表面，有些描述金属表面等。

### BRDF 光照模型
BRDF，即**双向反射分布函数 Bidirectional Reflectance Distribution Function**，即给定入射光线的方向和辐照度，可以用它得到某个出射方向上的光照能量分布。

本文涉及的 BRDF 是对真实场景进行理想化或简化后的模型。也就是说，他们不能真实地反映问题和光线之间的交互，这些模型被称为经验模型。在实时渲染中，考虑到性能消耗，一般还是会使用经验模型。虽然他们不能很真实地反映光照情况，但是它们“看起来是对的”。如果希望更真实地模拟光和物体的交互，一般会使用基于物理的 BRDF 模型（后面章节会学到）。

## 标准光照模型
在 BRDF 理论被提出来之前，标准光照模型已经被广泛使用了。

**标准光照模型**，也称为 **Phong 光照模型**，由著名 CG 学者裴祥风 Bui Tuong Phong 于 1973 年提出。标准光照模型只关心直接光照，即直接从光源发射出来照射到物体表面后，经过物体表面地一次反射直接进入摄像机的光线。

其基本方法就是把进入到摄像机内的光线分为 4 个部分，每个部分使用一种方法计算它的贡献度：  
①**自发光 emissive** 部分，使用 $c_{emissive}$ 表示。描述给定一个方向时，一个表面本身会向该方向发射多少辐射量。注意：如果没有全局光照技术，自发光的表面并不会照亮周围物体，只是本身看起来更亮了。  
②**高光反射 specular** 部分，使用 $c_{specular}$ 表示。描述当光线从光源照射到模型表面时，该表面会在完全镜面反射方向散射多少辐射量。  
③**漫反射 diffuse** 部分，使用 $c_{diffuse}$ 表示。描述当光线从光源照射到模型表面时该表面会向每个方向散射多少能量。  
④**环境光 ambient** 部分，使用 $c_{ambient}$ 表示。用于描述其他所有间接光照。  

### 环境光
利用环境光来模拟所有的间接光照，场景中的所有物体都使用这个环境光，通常是一个全局变量：

$$c_{ambient} = g_{ambient}$$

> 间接光照指，光线经过多个物体反射后，即不止一次的物体反射，最后进入摄像机。比如红地毯导致沙发底部泛红色。

### 自发光
光线直接由由光源发射进入摄像机，不需要经过任何物体反射，所以直接使用该材质的自发光颜色来计算：  

$$c_{emissive} = g_{emissive}$$

通常在实时渲染中，自发光的表面往往不会照亮周围的表面，即这个物体不会被当成一个光源。Unity 中引入的全局光照系统可以模拟这类自发光物体对周围物体的影响。

### 漫反射
漫反射光照是用于对那些被物体表面随机散射到各个方向的辐射度进行建模的。在漫反射中，视角的位置不重要，因为反射完全随机，即可以认为在任何反射方向上的分布是均匀的。但是入射光线的角度很重要。  

漫反射光照符合**兰伯特定律 Lambert's law**：反射光线的强度与表面法线和光源方向之间夹角的余弦值成正比。

$$c_{diffuse} = (c_{light} \cdot m_{diffuse}) max(0, \hat n \cdot \hat l)$$

其中，$\hat n\,$ 是表面法线，$\hat l\,\,$ 是指向光源的单位矢量，$m_{diffuse}\,$ 是材质的漫反射颜色，$c_{light}\,$ 是光源颜色。取最大值是为了防止点乘结果为负值，可以防止物体被后面来的光照亮。

### 高光反射
这里的高光反射是一种经验模型，即并不完全符合真实世界中的高光反射现象。它可用于计算那些沿着完全镜面反射方向被反射的光线。

计算高光反射需要信息较多，如表面法线 n、视角方向 v、光源方向 l、反射方向 r 等。假设这些矢量都是单位矢量，反射方向可以通过法线方向和光源方向计算得到：  

$$\hat {r} = 2(\hat {n} \cdot \hat {l})\hat {n} - \hat {l}$$

利用 Phong 模型来计算高光反射的部分：  

$$c_{specular} = (c_{light} \cdot m_{specular}) max(0, \hat {v} \cdot \hat {r})^{m_{gloss}}$$

① $m_{gloss}\,$ 是材质的**光泽度 gloss**，也称为**反光度 shininess**，用于控制高光区域的亮点有多宽，$m_{gloss}\,$ 越大，亮点就越小。  
② $m_{specular}\,$ 是材质的高光反射颜色，用于控制该材质对于高光反射的强度和颜色。  
③ $c_{light}\,$ 是光源的颜色和强度。

<div  align="center">  
<img src="https://s2.loli.net/2023/11/06/tQmMHVJ4WfZSw3j.jpg" width = "50%" height = "50%" alt="图14- 使用 Phong 模型计算高光反射"/>
</div>

在 Phong 模型基础上，Blinn 简化了计算来得到类似的效果。为了避免计算反射方向 $\,\hat r\,$，Blinn 模型引入了一个新的矢量 $\,\hat h\,$ （**半角向量 halfway vector**），它是通过对 $\,\hat v\,$ 和 $\,\hat l\,\,$ 的取平均后再归一化得到的，即：  

$$\hat {h} = \cfrac {\hat {v} + \hat {l}} {|\hat {v} + \hat {l}|}$$

然后使用 $\,\hat n\,$ 和 $\,\hat h\,$ 之间的夹角进行计算，而非 $\,\hat v\,$ 和 $\,\hat r\,$ 之间的夹角：  

$$c_{specular} = (c_{light} \cdot m_{specular}) max(0, \hat {n} \cdot \hat {h})^{m_{gloss}}$$

<div  align="center">  
<img src="https://s2.loli.net/2023/11/06/8Ky4CHnVqdlcUXP.jpg" width = "50%" height = "50%" alt="图15- 使用 Blinn 模型计算高光反射"/>
</div>

在硬件实现时，如果摄像机和光源距离模型足够远，Blinn 模型会快于 Phong 模型，因为此时可以认为 v 和 l 都是定值，因此 h 是一个常量。但是当 v 和 l 不是定值时，Phong 模型反而可能更快。注意，这两种光照模型都是经验模型，不能认为 Blinn 模型是对“正确的” Phong 模型的近似。在一些情况下 Blinn 模型更符合实验结果。

### 逐像素还是逐顶点
①**逐像素光照 per-pixel lighting**：在片元着色器中计算，以每个像素为基础，得到它的法线（可以是对顶点法线插值得到的，也可以是从法线纹理中采样得到的），然后进行光照模型的计算。这种在面片之间对顶点法线进行插值的技术称为 **Phong 着色**，不同于与前面的 Phong 光照模型，也称 Phong 插值、法线插值着色技术。

②**逐顶点光照 per-vertex lighting**：也被称为**高洛德着色 Gouraud shading**。在顶点着色器中计算，在每个顶点上计算光照，然后在渲染图元内部进行线性插值，最后输出像素颜色。

由于顶点数往往小于像素数目，因此逐顶点光照的计算量往往小于逐像素光照。但是逐顶点光照依赖于线性插值来得到像素光照，因此，当光照模型中有非线性计算（比如：高光反射计算）时，逐顶点光照的效果会出现问题。而且由于逐顶点光照会在渲染图元内部对顶点颜色进行插值，这会导致渲染图元内部的颜色总是暗于顶点处的最高颜色值，某些情况下会产生明显的棱角现象。

### 总结
虽然 Blinn-Phong 模型并不完全符合真实世界中的光照现象，但由于易用性和计算速度快的特点，使它仍被广泛使用。

但该模型也有很多的局限性，例如**菲涅尔反射 Fresnel reflection** 等物理现象就无法用 Blinn-Phong 模型表现出来，其次，Blinn-Phong 模型是**各向同性 isotropic** 的，即当我们固定视角和光源方向旋转这个表面时，反射不会发生任何改变。但是某些物体表面是**各向异性 anisotropic** 的，比如拉丝金属、毛发等。


## Unity 中的环境光和自发光
在 Unity 中，场景中的环境光可以在 Window -> Rendering -> Lighting -> Environment -> Environment Lighting 中控制。在 Shader 中，我们可以调用 Unity 的内置变量 UNITY_LIGHTMODEL_AMBIENT 就可以得到环境光的颜色和强度信息。  

由于大多数物体没有自发光特性，因此在本书中绝大部分 Shader 都没有计算自发光 Shader。若需要，只需要在片元着色器输出最后的颜色之前，把材质的自发光颜色添加到输出颜色上即可。

## 在 Unity Shader 中实现漫反射光照模型
上面给出了基本光照模型中漫反射部分的计算公式，公式中 max 函数是为了防止点积结果为负值，Cg 语言中有另一个函数可以达到同样的目的，即 saturate 函数：

    // 参数 x：可以是标量或矢量，float、float2、float3 等类型
    // 如果是标量，将 x 截取在 [0, 1] 范围内；
    // 如果是矢量，会对矢量的每一个分量进行 [0, 1] 范围的限制；
    saturate(x)

### 逐顶点光照实践
***1. 准备工作***  
①新建名为 Scene_6_4 的场景，去掉天空盒子；  
②新建名为 DiffuseVertexLevelMat 的材质；  
③新建名为 Chapter6-DiffuseVertexLevel 的 Unity Shader，并赋给上一步创建的材质；  
④在场景中创建一个胶囊体，并将之前创建的材质赋给它；  
⑤保存场景；

***2. 编写 Shader***  
打开 Chapter6-DiffuseVertexLevel，删除里面的所有代码，从第一行开始跟着下面的代码一行一行往下写，注释中已详细地对关键代码作了说明：

``` C C for Graphics
// 给Shader命名
Shader "Unity Shaders Book/Chapter 6/Diffuse Vertex-Level"
{
    Properties
    {
        // 漫反射颜色的属性，默认设为白色
        _Diffuse("Diffuse", Color) = (1, 1, 1, 1)
    }
    
    SubShader
    {
        Pass
        {
            // LightMode 标签用于定义该 Pass 在 Unity 的光照流水线中的角色
            // 后续会详细地了解，这里有个概念即可
            // 只有定义了正确的 LightMode，才能得到一些 Unity 的内置光照变量，比如下面的 _LightColor0
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            // 为了使用 Unity 内置的一些变量，比如：_LightColor0
            #include "Lighting.cginc"

            // 需要定义和 Properties 语义块中声明的属性相匹配的变量，用于获取漫反射参数，别忘了
            fixed4 _Diffuse;

            struct a2v
            {
                float4 vertex : POSITION; // 模型空间中的顶点位置 
                float3 normal : NORMAL; // 模型顶点的法线信息
            };

            struct v2f
            {
                float4 pos : SV_POSITION; // 裁剪空间中的顶点坐标
                fixed3 color : COLOR; // 将顶点信息计算的光照颜色传递给片元着色器，这里不是必须使用 COLOR 语义，也可以使用 TEXCOORD0 语义
            };

            v2f vert (a2v v)
            {
                v2f o;
                // 将顶点坐标由模型空间转到裁剪空间
                o.pos = UnityObjectToClipPos(v.vertex);

                // 通过 Unity 内置变量得到环境光的颜色值
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

                // 见下面注
                fixed3 worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));

                // 通过内置变量 _WorldSpaceLightPos0 获取世界空间下的光源方向，并进行归一化
                // 因为本案例中光源只有一个并且是平行光，所以可以直接取该变量进行归一化，若还有其他类型光源，则该内置变量不能得到正确的结果
                fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);

                // 下面使用漫反射公式得到漫反射的颜色值
                // 通过内置变量 _LightColor0，来访问该 Pass 处理的光源的颜色和强度信息
                // 利用 _Diffuse.rgb 获取材质漫反射颜色
                // saturate，把取值范围截取到 [0, 1] 的范围内
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLight));

                // 最后对环境光和漫反射光部分相加，得到最终的光照结果
                o.color = ambient + diffuse;

                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // 将顶点输出的颜色值作为片元着色器的颜色输出，输出到屏幕上
                return fixed4(i.color, 1.0);
            }

            ENDCG
        }
    }

    // 使用内置的 Diffuse 作为该 Unity Shader 的回调 shader
    Fallback "Diffuse"
}
```

注：为了计算法线和光源方向的点积（即它们之间夹角的余弦值，因为是单位向量），需要把两者放置在同一坐标空间下，计算才有效，这里选择的是世界坐标空间。而 a2v 得到的顶点法线是在模型空间下，所以需要把法线转换到世界空间下。至于为什么法线的坐标空间转换需要用逆矩阵反向相乘，详见 Shader 数学基础的法线变换。法线变换矩阵为顶点变换矩阵的逆转置矩阵，所以使用模型空间到世界空间的逆矩阵 _World2Object，然后调换 mul 函数中的位置得到转置矩阵。由于法线是方向，所以只需要截取 _World2Object 的三行三列即可。（ _World2Object 因为版本问题更新为了 unity_WorldToObject）

对于细分程度较高的模型，逐顶点光照已经可以得到较好的光照效果了。但是对于细分程度较低的模型，逐顶点光照会出现一些视觉问题，比如上述代码的胶囊体的背光面和向光面交界处会有一些锯齿，图见后面。

### 逐像素光照实践
***1. 准备工作***  
①新建名为 DiffusePixelLevelMat 的材质；  
②新建名为 Chapter6-DiffusePixelLevel 的 Unity Shader，并赋给上一步创建的材质；  
③在原来的场景中创建一个新的胶囊体，将刚刚创建的材质赋给它；  
④保存场景；

***2. 编写 Shader***  
打开 Chapter6-DiffusePixelLevel，删除里面的所有代码，并复制粘贴上一节编写的 Shader 代码，做部分修改。下面是逐像素光照的 Shader 代码，修改部分已用注释说明：  

``` C C for Graphics
// 修改着色器的名字
Shader "Unity Shaders Book/Chapter 6/Diffuse Pixel-Level"
{
    Properties
    {
        _Diffuse("Diffuse", Color) = (1, 1, 1, 1)
    }
    
    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Diffuse;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                fixed3 worldNormal : TEXCOORD0; // 顶点着色器输出的世界空间下的法线，用于在片元着色器中编写光照计算逻辑
            };

            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                // 顶点着色器只需要计算世界空间下的法线矢量，并传递给片元着色器即可
                o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
                return o;
            }

            //顶点着色器计算漫反射光照模型：
            fixed4 frag (v2f i) : SV_Target 
            {
                // 获取环境光颜色
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
                
                // 将世界空间下的法线矢量进行归一化
                fixed3 worldNormal = normalize(i.worldNormal);
                // 通过内置变量 _WorldSpaceLightPos0 获取世界空间下的光照方向，并进行归一化
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

                // 根据漫反射公式计算漫反射颜色值
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));

                // 将环境光和漫反射光相加，输出到屏幕
                fixed3 color = ambient + diffuse;
                return fixed4(color, 1.0);
            }

            ENDCG
        }
    }

    Fallback "Diffuse"
}
```

逐像素光照可以的得到更加平滑的效果，图见后面。光照无法到达的区域模型外观通常是全黑的，使模型背光的区域看起来和一个平面一样，失去了模型的细节表现，如果从完全背对光的一面看模型，模型几乎看不到立体的效果。对于此问题，使用**半兰伯特 Half Lambert** 光照模型改善。

### 半兰伯特模型
之前使用的漫反射光照模型也被称为兰伯特光照模型，符合兰伯特定律，即在平面某点漫反射光的光强和该反射点的法线和入射角度的余弦值成正比。

为了改善上一小节提到的问题，Valve 公司在开发半条命时提出了**半兰伯特光照模型**，对原模型进行了简单的修改。广义的半兰伯特光照模型的公式如下：  

$$c_{diffuse} = (c_{light} \cdot m_{diffuse}) (α(\hat n \cdot \hat l) + β)$$

半兰伯特光照模型没有使用 max 操作防止点乘出现负值，而是对其结果进行了 α 倍的缩放再加上 β 的偏移，绝大多数情况下 α 和 β 的值均为 0.5 。即公式为：  

$$c_{diffuse} = (c_{light} \cdot m_{diffuse}) (0.5(\hat n \cdot \hat l) + 0.5)$$

这样可以将两个矢量点积的结果范围由 [-1, 1] 映射到 [0, 1]，也就实现了：即使观察点在背光面，原模型的点积都是 0，而新模型中，背面也将会有明暗变化。

***1. 准备工作***  
①新建名为 HalfLambertMat 的材质；  
②新建名为 Chapter6-HalfLambert 的 Unity Shader，并赋给上一步创建的材质；  
③在原来场景中创建一个新的胶囊体，并将第一步创建的材质赋给它；  
④保存场景；

***2. 编写 Shader***  
打开 Chapter6-HalfLambert，删除里面的所有代码，并复制粘贴 Chapter6-DiffusePixelLevel 的代码，并使用半兰伯特公式修改片元着色器中计算漫反射光照的部分（别忘了修改着色器名字）：  

``` C C for Graphics
fixed4 frag(v2f i) : SV_Target
{
    ...

    // 计算漫反射的半兰伯特值
    fixed halfLambert = dot(worldNormal, worldLightDir) * 0.5 + 0.5;
    // 将公式修改为半兰伯特模型的公式
    fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * halfLambert;

    fixed3 color = ambient + diffuse;
    return fixed4(color, 1.0);
}
```

下图是上面三种光照的对比效果，从左到右分别是逐顶点漫反射光照、逐像素漫反射光照、半兰伯特光照，可以看到逐顶点会有一些锯齿，半兰伯特光照太白是因为 0.5 的偏移值太大了：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/07/9OnPoypKNL8Yi7g.png" width = "70%" height = "70%" alt="图16- 逐顶点漫反射光照、逐像素漫反射光照、半兰伯特光照的对比效果"/>
</div>


## 在 Unity Shader 中实现高光反射光照模型
之前给出了基本光照模型中高光反射部分的计算公式：  

$$c_{specular} = (c_{light} \cdot m_{specular}) max(0, \hat {v} \cdot \hat {r})^{m_{gloss}}$$

计算高光反射需要知道 4 个参数：入射光线的颜色和强度 $c_{light}$，材质的高光反射系数 $m_{specular}$，视角方向 $\,\hat v\,$ 以及反射方向 $\,\hat r\,$。其中，反射方向 $\,\hat r\,$ 由表面法线 $\,\hat n\,$ 和光源方向 $\,\hat l\,$ 计算而得：  

$$\hat {r} = \hat {l} - 2(\hat {n} \cdot \hat {l})\hat {n}$$

> 这里的公式和之前是反过来的，原因是光源方向和之前相反（点乘结果为负数），见下图。

Cg 提供了计算反射方向的函数 `reflect(i , n)`，其中 i 为**入射方向**（与之前指向光源的方向相反），n 是法线方向，可以是 float，float2，float3 等类型。

<div  align="center">  
<img src="https://s2.loli.net/2023/11/08/D4gSraRV1XhTG2K.jpg" width = "50%" height = "50%" alt="图17- Cg 的 reflect 函数"/>
</div>

### 逐顶点光照实践
***1. 准备工作***  
①新建名为 Scene_6_5 的场景，并去掉天空盒子；  
②新建名为 SpecularVertexLevelMat 的材质；  
③新建名为 Chapter6-SpecularVertexLevel 的 Unity Shader，并赋给刚刚创建的材质；  
④在场景中创建一个胶囊体，并将刚刚创建的材质赋给赋给它；  
⑤保存场景；

***2. 编写 Shader***  
打开 Chapter6-SpecularVertexLevel，删除里面的所有代码，编写如下代码：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 6/Specular Vertex-Level"
{
    Properties
    {
        _Diffuse ("Diffuse", Color) = (1, 1, 1, 1) // 控制漫反射颜色
        _Specular ("Specular", Color) = (1, 1, 1, 1) // 控制高光反射颜色
        _Gloss ("Gloss", Range(8.0, 256)) = 20 // 控制高光区域大小，值越小，区域越大，因为点乘结果小于 1，指数越大，结果越小
    }

    SubShader
    {
        Pass
        {
            // 为了正确地得到一些 Unity 的内置光照变量，如_LightColor0
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            // 获取 Properties 语义中声明的属性
            fixed4 _Diffuse;
            fixed4 _Specular;
            float _Gloss; // 光泽度数值范围较大，使用 float 精度

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                fixed3 color : COLOR;
            };

            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);

                // 获取环境光
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

                fixed3 worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

                // 计算出漫反射光
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
                
                // 计算反射方向
                // 使用 reflect 函数求出入射光线关于表面法线的反射方向，并进行归一化
                // 因为 reflect 函数的入射方向要求由光源指向交点处（worldLightDir 是交点处指向光源），所以需要取反
                fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));

                // 计算视角方向
                // 通过 Unity 内置变量 _WorldSpaceCameraPos 得到世界空间中的相机位置
                // 通过与世界空间中的顶点坐标进行相减，得到世界空间下的视角方向
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - mul(unity_ObjectToWorld, v.vertex).xyz);

                // 计算出高光反射光
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
                
                // 环境光 + 漫反射 + 高光反射
                o.color = ambient + diffuse + specular;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                return fixed4(i.color, 1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```

使用逐顶点的方法得到的高光效果会有较大问题，因为高光反射的计算是非线性的，而顶点着色器中计算光照再进行插值的过程是线性的，破坏了原计算的非线性关系，就会有较大的视觉问题。效果图见后面。

### 逐像素光照实践
***1. 准备工作***  
①新建名为 SpecularPixelLevelMat 的材质；  
②新建名为 Chapter6-SpecularPixelLevel 的 Unity Shader，并赋给上一步的材质；  
③在原来的场景新建一个胶囊体，并将刚刚创建的材质赋给它；  
④保存场景；

***2. 编写 Shader***  
打开 Chapter6-SpecularPixelLevel，删除里面所有代码，并复制粘贴上一节编写 Chapter6-SpecularVertexLevel 代码，做部分修改，修改后的代码如下，修改部分已用注释说明：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 6/Specular Pixel-Level"
{
    Properties
    {
        _Diffuse("Diffuse", Color) = (1, 1, 1, 1)
        _Specular("Specular", Color) = (1, 1, 1, 1)
        _Gloss("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

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
                float3 worldNormal : TEXCOORD0; //顶点着色器输出的世界空间下的法线
                float3 worldPos : TEXCOORD1; //顶点着色器输出的世界空间下的坐标
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);

                // 将法线由模型空间转到世界空间
                o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
                // 将顶点坐标由模型空间转到世界空间
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                // 获取环境光
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

                // 对法线进行归一化
                fixed3 worldNormal = normalize(i.worldNormal);
                // 对光照方向进行归一化
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);

                // 计算漫反射
                fixed3 diffuse = _LightColor0.rgb * _Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));

                // 计算反射方向并进行归一化
                fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
                // 计算视角方向并进行归一化
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
                // 计算高光反射
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
                
                return fixed4(ambient + diffuse + specular, 1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```

逐像素处理光照可以得到更加平滑的高光效果，效果见后面图。至此，实现了一个完整的 Phong 光照模型。

### Blinn-Phong 光照模型
之前提过 Blinn 光照模型，不使用反射方向，而是引入了一个新的矢量 $\,\hat h\,$ ，它是通过对 $\,\hat v\,$ 和 $\,\hat l\,\,$ 的取平均后再归一化得到的，即：  

$$\hat {h} = \cfrac {\hat {v} + \hat {l}} {|\hat {v} + \hat {l}|}$$

而 Blinn 光照模型计算高光反射的公式如下：  

$$c_{specular} = (c_{light} \cdot m_{specular}) max(0, \hat {n} \cdot \hat {h})^{m_{gloss}}$$

***1. 准备工作***  
①新建名为 BlinnPhongMat 的材质；  
②新建名为 Chapter6-BlinnPhong 的 Unity Shader，并赋给上一步创建的材质；  
③在原来的场景中新建一个胶囊体，并将第一步创建的材质赋给它；  
④保存场景；

***2. 编写 Shader***  
打开 Chapter6-BlinnPhong，删除里面的所有代码，将上一节 Chapter6-SpecularPixelLevel 代码复制粘贴进去，做部分修改：  

``` C C for Graphics
fixed4 frag(v2f i) : SV_Target
{
    ...

    fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
    // 通过归一化入射方向 + 视角方向，得到半角向量 h
    fixed3 halfDir = normalize(worldLightDir + viewDir);
    // 根据 Blinn 光照模型公式计算高光反射部分
    fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(saturate(dot(worldNormal, halfDir)), _Gloss);

    return fixed4(ambient + diffuse + specular, 1.0);
}
```

可以看出，Blinn-Phong 的高光反射部分看起来更大更亮，实际渲染中绝大多数情况会选择 Blinn-Phong 光照模型。注意，上述模型都是经验模型。

下图是上面三种光照的对比效果（为了更清晰看到高光效果，把漫反射颜色改为了灰色），从左到右分别是逐顶点高光反射光照、逐像素高光反射光照、Blinn-Phong 光照：

<div  align="center">  
<img src="https://s2.loli.net/2023/11/13/YSbDIz87glZePkV.png" width = "70%" height = "70%" alt="图18- 逐顶点的高光反射光照、逐像素的高光反射光照（Phong 光照模型）和 Blinn-Phong 高光反射光照的对比结果"/>
</div>

## 使用 Unity 内置的函数
在之前的案例中，我们手动计算了光源方向和视角方向信息。光源方向是基于平行光的实现，如果遇到更复杂的光照类型（比如：点光源、聚光灯），使用上面的代码会得到错误的结果。这需要我们在代码中先判断光源类型，在计算其他光源信息，在后面会讲到。

手动计算光源信息的过程比较麻烦，Unity 则提供了一些内置函数来帮助我们计算这些信息。详见第五章的 UnityCG.cginc 的内容。表格中的 9 个函数中，有 5 个我们已经掌握了其内部实现，甚至在案例中已经写过，例如 WorldSpaceViewDir 函数实现如下：  

    inline float3 UnityWorldSpaceViewDir(float3 worldPos)
    {
        return _WorldSpaceCameraPos.xyz - worldPos;
    }

可以看出，这和之前计算视角方向的方法一致，**但要注意的是，使用前要先归一化**。  

与计算光源方向相关的 3 个函数：WorldSpaceLightDir、UnityWorldSpaceLightDir 和 ObjSpaceLightDir，它们的内部逻辑会稍微复杂一些，因为 Unity 要针对不同种类的光源做不同的逻辑。需要注意的是，这 3 个函数仅可用于前向渲染（也就是之前 Pass 中 Tags 写的 "LightMode" = "ForwardBase"，后续会详细了解），因为函数内使用的一些内置变量，如 _WorldSpaceLightPos0 等，只有在前向渲染中才会被正确赋值。

接下来基于前面写的 Blinn-Phong 案例代码，使用内置函数进行调整：  
①在顶点着色器中，使用内置的 UnityObjectToWorldNormal 来计算世界空间下的法线方向：`o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);` 改为 `o.worldNormal = UnityObjectToWorldNormal(v.normal);`  
②在片元着色器中，使用内置的 UnityWorldSpaceLightDir 计算世界坐标下光照方向，使用内置的 UnityWorldSpaceViewDir 计算世界坐标下的视角方向：`fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);` 改为 `fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));`，  
以及 `fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);` 改为 `fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));`


# 第六章 基础纹理
纹理，即使用**纹理映射 texture mapping** 技术来使用图片控制模型的外观，逐**纹素 texel** (用于和像素区分)地控制模型颜色。美术建模时，通常会在建模软件中利用纹理展开技术把纹理映射坐标存储在每个顶点上。**纹理映射坐标**定义了该顶点在纹理中对应的 2D 坐标。通常使用二维变量 (u, v) 来表示，u 是横向坐标，v 是纵向坐标，因此纹理映射坐标也被称为 **UV 坐标**。

尽管纹理大小各不相同，但是顶点 UV 坐标的范围通常都被归一化为 [0, 1] 范围内。但，纹理采样使用的纹理坐标不一定在 [0, 1] 范围内。实际上，不在 [0, 1] 范围内的纹理坐标有时很有用，与之关系紧密的是纹理的平铺模式，详见后面章节。

OpenGL 的纹理空间的原点位于左下角；DirectX 的纹理空间的原点位于左上角；Unity 在绝大多数情况下会帮我们处理好差异。但 Unity 本身使用的是符合 OpenGL 传统的，即原点位于左下角。

## 单张纹理
本节学习使用一张纹理代替物体的漫反射颜色。

### 实践
本例中仍然使用 Blinn-Phong 光照模型来计算光照。

***1. 准备工作***  
①新建名为 Scene_7_1 的场景，并去掉天空盒子；  
②新建名为 SingleTextureMat 的材质；  
③新建名为 Chapter7-SingleTexture 的 Shader，并赋给上一步创建的材质；  
④在场景中新建一个胶囊体，并把第二步的材质赋给它；  
⑤保存场景；

***2. 编写 Shader***  
打开 Chapter7-SingleTexture，删除里面所有代码，编写如下代码：

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 7/Single Texture"
{
    Properties
    {
        //使用纹理替代了漫反射颜色，漫反射颜色由 Color 和纹理颜色共同作用
        _Color ("Color Tint", Color) = (1, 1, 1, 1) //控制物体整体色调
        _MainTex ("Main Tex", 2D) = "white" {} //之前提到过，2D 是纹理属性的声明方式，"white" 是内置纹理的名字
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _Gloss ("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase"}

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            // 纹理和其他属性类型不同，还需要为纹理类型的属性声明一个 float4 类型的变量 _MainTex_ST。这个名字是因为在 Unity 中需要使用 纹理名_ST 的方式来声明某个纹理的属性
            // ST 指纹理缩放 scale 和平移 translation，_MainTex_ST.xy 存储缩放值，_MainTex_ST.zw 存储偏移值，在材质面板的纹理属性中可以调节（Tiling 平铺即缩放，Offset 偏移即平移）
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0; // 存储模型的第一组纹理坐标
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                // 存储纹理坐标的 uv 值，以便在片元着色器中使用该坐标进行纹理采样
                float2 uv : TEXCOORD2;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                // 通过缩放和平移后的纹理 uv 值
                o.uv = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                // 也可以调用 Unity 提供的内置宏 TRANSFORM_TEX，得到缩放和平移后的纹理 uv 值，与上面的计算逻辑是一致的，可以在 UnityCG.cginc 中找到该内置宏的定义：
                // #define TRANSFORM_TEX(tex, name) (tex.xy * name##_ST.xy + name##_ST.zw)
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

                return o;
            }

            float4 frag(v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

                // 通过 Cg 的 tex2D 函数对纹理进行采样。第一个参数是被采样的纹理，第二个参数是一个 float2 类型的纹理坐标，函数返回计算得到的纹素值
                // 使用采样结果和颜色属性的乘积作为材质的反射率颜色 albedo
                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;

                // 环境光照和反射率相乘得到环境光部分
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                
                // 使用反射率颜色 albedo 来计算漫反射光照的结果
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));

                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);

                return fixed4(ambient + diffuse + specular, 1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```

对材质的纹理进行赋值后，效果如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/13/ZPDyJmzEOpkVNg9.png" width = "70%" height = "70%" alt="图19- 使用单张纹理"/>
</div>

### 纹理的属性
向 Unity 导入一张纹理资源后，可以在其 Inspector 面板中调整其属性。第一个属性是纹理类型，在后面的法线纹理一节中，会使用 Normal Map 类型，之后会看到更多高级纹理。

纹理的类型默认为 Default，Alpha Source 属性对应的下拉框会有 From Gray Scale，如果选择它，透明通道的值将会由每个像素的灰度值生成。透明效果会在第 7 章讲到，这里先不需要勾选它。

**Wrap Mode** 属性比较重要，决定了当纹理坐标超过 [0, 1] 范围后会如何被平铺：  
① Repeat 模式：在这模式下，若纹理坐标超过了 1，那么它的整数部分会被截断，直接使用小数部分进行采样，这样的结果就是纹理会不断重复；  
② Clamp 模型：在这模式下，若纹理坐标超过了 1，那么将会截取到 1，若小于 0，则截取到 0。

**Filter Mode** 属性决定了当纹理由于变换而产生拉伸时，采用哪种滤波模式。Filter Mode 支持 3 种模式：Point，Bilinear 以及 Trilinear。它们得到的图片滤波效果依次提升，但性能消耗也依次增大，纹理滤波会影响放大或缩小纹理时得到的图片质量。  
① Point 模式：使用**最近邻 nearest neighbor** 滤波，在放大缩小时，采样像素数目通常只有一个，所以图像呈现像素风格的效果。若需要像素风格，使用这个；  
② Bilinear 模式：使用线性滤波，对每个目标像素，会找到 4 个邻近像素，对其线性插值混合后得到最终像素，所以图像看起来被模糊了；  
③ Trilinear 模式：原理和 Bilinear 一样，只是 Trilinear 会在多级渐远纹理（下面讲）之间进行混合，如果一张纹理没有使用多级渐远纹理技术，Trilinear 得到的结果和 Bilinear 就是一样的。

纹理缩小的同时还要解决抗锯齿问题（即会出现一个个很明显的像素边界），常用的方法是使用**多级渐远纹理 mipmapping** 技术。这是一种用空间换取时间的办法，提前将原纹理用滤波器处理来得到更多的小图像，并使用一定的空间用于存放它们，而当物体远离摄像机时，可直接使用较小的纹理进行代替。但通常会多占用 33% 的内存空间。在 Unity 中，在纹理面板中，展开 Advenced，再勾选 Generate Mip Maps 即可开启多级渐远纹理技术。同时可以选择生成多级渐远纹理时是否使用线性空间（用于伽马校正，见第 17 章）以及采用的滤波器等。

**Max Size** 属性：为不同的平台发布游戏时，需要考虑目标平台的纹理尺寸和质量问题，Unity 允许不同的平台进行不同的设置。若导入的纹理大小超过了 Max Size 的设置值，Unity 会把该纹理缩放为这个最大分辨率。一般导入的纹理可以是非正方形的，但是长宽需要是 2 的幂，例如：2、4、8、16等，如果使用了非 2 的幂 **Non Power of Two, NPOT** 大小的纹理，那么这些纹理往往会占用更多的内存空间，GPU 读取这些纹理的速度也会下降，。有一些平台甚至不支持 NPOT 纹理，虽然 Unity 内部会把它缩放到最近的 2 的幂大小，但是尽量使用 2 的幂大小的纹理。

**Format** 属性决定了 Unity 内部使用哪些格式来存储该纹理。使用的纹理格式精度越高，占用的内存越大，得到的效果也越好。可以在纹理面板下方的预览中看到该纹理需要占用的内存空间及相关信息（如果开启了多级渐远纹理技术，也会增加纹理的内存占用）。

## 凹凸映射
**凹凸映射 bump mapping**，即使用一张纹理来修改模型表面的法线，以便为模型提供更多的细节。该方法不会真的改变模型的顶点位置，只是让模型看起来像是“凹凸不平”，但可以从模型的轮廓看出“破绽”。

有两种主要方法可以进行凹凸映射：  
①使用一张**高度纹理 hight map** 来模拟**表面位移 displacement**，然后得到一个修改后的法线值，该方法被称为**高度映射 height mapping**；  
②使用一张**法线纹理 normal map** 来直接存储表面法线，这种方法被称为**法线映射 normal mapping**。

### 高度纹理
高度图中存储的是强度值，它用于表示模型表面局部的海拔高度。颜色越浅表明该位置的表面越向外凸起，颜色越深表示越往里凹。我们可以通过高度图中明确的知道一个模型表面的凹凸情况，缺点是计算更加复杂，在实时计算中不能直接得到表面法线，而是需要通过由像素的灰度值计算得出，从而消耗更多的性能。

高度图通常会和法线映射一起使用，用于给出表面凹凸的额外信息，开发者通常会使用法线映射来修改光照。

### 法线纹理
法线纹理中存储的就是表面的法线方向。由于法线方向各分量范围是 [-1, 1] ，而像素分量范围是 [0, 1]，因此需要做一个映射：

$$pixel = \cfrac {normal + 1} {2}$$

这就要求我们在 Shader 中对法线纹理进行纹理采样后，还需要对结果进行一次反映射过程，以得到原先的法线方向，即上面的逆函数：  

$$normal = pixel \times 2 - 1$$

因为模型顶点自带的法线是定义在模型空间中，从而可以将修改后的模型空间中的表面法线存储在一张纹理中，这种纹理被称为**模型空间的法线纹理 object-space normal map**。而在实际制作中，往往会采用另一种坐标空间，即模型顶点的**切线空间 tangent space** 来存储法线。对于模型的顶点，它都有一个属于自己的切线空间，这个切线空间的原点是该顶点本身，而 z 轴是顶点的法线方向 n，x 轴是顶点的切线方向 t，y 轴可由法线和切线叉积而得，也被称为副切线 bitangent 或副法线。该纹理被称为是**切线空间的法线纹理 tangent-space normal map**。

<div  align="center">  
<img src="https://s2.loli.net/2023/11/14/kJdQsIieGKUVNRa.jpg" width = "30%" height = "30%" alt="图20- 模型顶点的切线空间。其中，原点对应了顶点坐标，x 轴是切线方向 t，y 轴是副切线方向 b，z 轴是法线方向 n"/>
</div>

①**模型空间下的法线纹理看上去是五颜六色的**，这是因为，所有法线所在的坐标空间是同一个坐标空间，即模型空间，每个点存储的法线方向是各异，有的是（0，1，0），经过映射后存储到纹理中就对应了 RGB（0.5，1，0.5）浅绿色，有的是（0，-1，0），经过映射后存储到纹理中就对应了 RGB（0.5，0，0.5）紫色。  
②**切线空间下的法线纹理看起来几乎全部是浅蓝色**，这是因为，每个法线方向所在的坐标空间不一样，即表面每点各自的切线空间。这种法线纹理其实是存储了每个点在各自切线空间中的法线扰动方向。也就是说，若每个点的法线方向不变，那么在它的切线空间中，法线方向就是 z 轴方向，即值为（0，0，1），经过映射后存储在纹理中就对应了 RGB（0.5，0.5，1）浅蓝色。由于凹凸效果一般只要对原法线做微小的变化，所以切线空间下的法线纹理一般都有大面积的蓝色。

<div  align="center">  
<img src="https://s2.loli.net/2023/11/14/t3Ku2JL1OXMBwWN.jpg" width = "50%" height = "50%" alt="图21- 左图：模型空间下的法线纹理。右图：切线空间下的法线纹理"/>
</div>

①模型空间存储法线的优缺点  
&emsp;&emsp; - 计算更少，生成简单；  
&emsp;&emsp; - 可以提供平滑的边界，在纹理坐标缝合处和尖锐的边角部分，可见的突隙较少。因为模型空间的法线纹理是同一个坐标系下，边界处通过插值得到的法线可以平滑变换；而切线空间需要依赖纹理坐标方向得到，边缘处或尖锐部分会造成更多可见的缝合迹象。  
②切线空间存储法线的优缺点  
&emsp;&emsp; - 自由度很高，模型空间下的法线纹理记录的是绝对法线信息，仅可用于创建它时的模型，而应用到其他模型上效果就完全错误的。切线空间下的法线纹理记录的是相对法线信息。即把该纹理应用到一个完全不同的网络上，也可以得到一个合理的效果；  
&emsp;&emsp; - 可进行 UV 动画。如可移动一个纹理的 UV 坐标来实现一个凹凸移动的效果，但使用模型空间下的法线纹理会得到完全错误的结果。UV 动画在水或火山熔岩这种类型上的物体会经常用到；  
&emsp;&emsp; - 可重用法线纹理，如一个砖块，仅使用一张法线纹理就可以用到所有的 6 个面上了；  
&emsp;&emsp; - 可压缩。由于切线空间下的法线纹理中法线的 Z 方向总是正方向，因此开发者可以仅存 XY 方向，而推导 Z 方向。而模型空间下的法线纹理由于每个方向都是可能的，从而必须存储 3 个方向的值，不可压缩。

由以上的优点分析，可见切线空间的灵活性和可重用性让它很多情况下都优于模型空间下的法线纹理，因此大多数情况下会使用切线空间下的纹理坐标，原书使用的也是切线空间下的法线纹理。

### 实践
在计算光照模型中需要统一各个方向矢量所在的坐标空间，法线纹理中存储的是切线空间下的法线方向，处理光照有两种选择：  
①在切线空间下计算光照，将光照方向、视角方向变换到切线空间下；  
②在世界空间下计算光照，将采样得到的法线方向变换到世界空间下，再和世界空间下的光照方向和视角方向进行计算；  

从效率上讲，前一种更优，因为前者可在顶点着色器上完成对光照方向和视角方向的变换，而后者因为光照计算在采样获得法线方向之后，所以光照方向和视角方向的变换过程必须在片元着色器上实现；从通用性上将，后一种更优，因为有时候需要在世界空间中进行其他的计算，比如使用 Cubemap 进行环境映射时，需要使用世界空间下的反射方向对 Cubemap 进行采样（见第九章）。

#### 在切线空间下计算
***1. 准备工作***  
①新建名为 Scene_7_2_3 的场景，并去掉天空盒子；  
②新建名为 NormalMapTangentSpaceMat 的材质；  
③新建名为 Chapter7-NormalMapTangentSpace 的 Shader，并赋给上一步创建的材质；  
④在场景中新建一个胶囊体，并将第二步创建的材质赋给它；  
⑤保存场景；

***2. 编写 Shader***  
在切线空间下计算光照模型的基本思路就是：在片元着色器中通过纹理采样得到切线空间下的法线，然后再与切线空间下的视角方向、光照方向等进行计算，得到最终的光照效果。

因此我们需要知道从模型空间到切线空间的变化矩阵。这个变化矩阵的逆矩阵，即从切线空间到模型空间的变换矩阵，非常容易求：按切线（x 轴）、副切线（y 轴）、法线（z 轴）的顺序，以它们在模型空间的坐标**按列**排列即可（数学原理忘了详见数学基础章节）。因为从模型空间到切线空间的变换仅存在平移、旋转和缩放变换，而我们只关心方向，并假设不存在非统一缩放，故截取变换矩阵前三列前三行的矩阵是旋转矩阵，而旋转矩阵是正交矩阵，那么这个变换的逆矩阵就等于它的转置矩阵。因此，从模型空间到切线空间的变换矩阵就是从切线空间到模型空间的变换矩阵的转置矩阵，把切线（x 轴）、副切线（y 轴）、法线（z 轴）的顺序**按行**排列即可。

> 原书中这一段存在些许错误，上面是我更正后的版本，了解逻辑比较重要，故保留书中原代码。作者冯乐乐在 Github 中对该错误进行了修正和解释，可以去看看，下面代码（原书代码）只适用于只存在统一缩放的情形下，不然会出现视觉问题。若存在非统一缩放，就必须求逆，但 Unity 不支持 Cg 的 inverse 函数，需要自己定义一个（冯乐乐的 github 中有写），比较麻烦，建议直接使用转换到世界空间的方法计算，会比较简单，不用求逆（因为不需要将 XX 向量转换到切线空间）。  
> 
> 实际上，在 Unity 4.x 版本及其之前的版本中，内置的 shader 一直是原来书上那种不严谨的转换方法，这是因为 Unity 5 之前，如果我们对一个模型 A 进行了非统一缩放，Unity 内部会重新在内存中创建一个新的模型 B，模型 B 的大小和缩放后的 A 是一样的，但是它的缩放系数是统一缩放。换句话说，在 Unity 5 以前，实际上我们在 Shader 中根本不需要考虑模型的非统一缩放问题，因为在 Shader 阶段非统一缩放根本就不存在了。但从 Unity 5 以后，我们就需要考虑非统一缩放的问题了。  
> 
> **注意：**区分模型顶点的真实法线和渲染法线（贴图上的法线）。对于模型顶点的真实法线，即切线空间中的法线，不论处于任何空间之时，它必须与模型表面垂直。即切线空间的三条线，切线 Tagent，副切线 Bitangent，法线 Normal，相互垂直。这三条线构成的变换矩阵，称为 **TBN 矩阵**（分为到模型空间和世界空间两种）。注：曲面上某点的切线理论上有无数条，建模软件计算切线时，会选择和 UV 展开方向相同的方向作为切线方向。法线纹理的原理就是修改了法线的方向，来得到看似凹凸的效果，故需要用真实法线的 TBN 矩阵把贴图记录的法线（渲染法线）转换到其他空间上去，以使用法线的偏移信息。渲染法线（用于光照计算）和切线不需要相互垂直，想想 Blender 里面对法线的修改产生不一样的视觉效果。

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 7/Normal Map In Tangent Space"
{
    Properties
    {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Tex", 2D) = "white" {}
        _BumpMap ("Normal Map", 2D) = "bump" {} // 法线纹理，默认使用 "bump" ，是 Unity 内置的法线纹理，当没有提供任何法线纹理时，"bump" 就对应模型自带的法线信息
        _BumpScale ("Bump Scale", Float) = 1.0 // 控制凹凸程度，当它为 0 时，表示该法线纹理不会对光照产生任何影响
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _Gloss ("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST; //同样，为了得到纹理的平铺和偏移属性
            float _BumpScale;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT; // 切线方向，使用 float4 类型（不同于法线的 float3），因为需要 tangent.w 分量来决定切线空间的副切线的方向性
                float4 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float4 uv : TEXCOORD0; // UV 使用 float4 类型，xy 存储主纹理坐标，zw 存储法线纹理坐标，出于减少插值寄存器的使用数量的目的
                float3 lightDir : TEXCOORD1; //输出切线空间下光照方向
                float3 viewDir : TEXCOORD2; //输出切线空间下视角方向
            };


            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);

                // 存储主纹理的 uv 值
                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                // 存储法线纹理的 uv 值
                o.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;

                // 计算副切线，乘以 w 分量确定副切线的方向性（与法线、切线垂直的有两个方向）
                // float3 binormal = cross(normalize(v.normal), normalize(v.tangent.xyz)) * v.tangent.w;
                // Cg 中 float4×4 是按行排列的，所以获取从从模型空间到切线空间的变换矩阵直接按行排列前后即可
                // float3x3 rotation = float3x3(v.tangent.xyz, binormal, v.normal);
                
                // 直接使用下述内置宏（在 UnityCG.cginc 中被定义)即可，它的实现和上述代码完全一样
                TANGENT_SPACE_ROTATION;

                // 将光照、视角方向由模型空间转到切线空间
                o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
                o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;

                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 tangentLightDir = normalize(i.lightDir);
                fixed3 tangentViewDir = normalize(i.viewDir);

                // 对法线纹理进行采样，获取纹素值
                fixed4 packedNormal = tex2D(_BumpMap, i.uv.zw);
                fixed3 tangentNormal;

                // 如果纹理没有被标记为 Normal map ，则需要手动反映射得到法线方向
                // tangentNormal.xy = packedNormal.xy * 2 - 1;

                // 如果纹理被标记为 Normal map，Unity 就会根据不同的平台来选择不同的压缩方法，需要调用 UnpackNormal 来进行反映射。如果这时再手动计算反映射就会出错，因为 _BumpMap 的 rgb 分量不再是切线空间下的法线方向 xyz 值了。本大节第四小节会解释
                tangentNormal = UnpackNormal(packedNormal);

                tangentNormal.xy *= _BumpScale; //控制凹凸程度

                // 因为贴图存储的法线为单位向量，且 z 分量为正。所以我们可以通过勾股定理计算 z 分量的值，下面通过点乘计算 x 的平方加上 y 的平方
                tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));

                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));

                fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(tangentNormal, halfDir)), _Gloss);

                return fixed4(ambient + diffuse + specular, 1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```

效果见在世界空间下计算的后面。

#### 在世界空间下计算
***1. 准备工作***  
①新建名为 NormalMapWorldSpaceMat 的材质；  
②新建名为 Chapter7-NormalMapWorldSpace 的 Shader，并赋给上一步创建的材质；  
③在原来的场景中新建一个胶囊体，并将第一步创建的材质赋给它；  
④保存场景；

***2. 编写 Shader***  
在世界空间下计算光照模型的基本思路就是：在顶点着色器中计算从切线空间到世界空间的变换矩阵，传递给片元着色器，然后在片元着色器中将贴图的法线方向从切线空间变换到世界空间：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 7/Normal Map In World Space"
{
    Properties
    {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Tex", 2D) = "white" {}
        _BumpMap ("Normal Map", 2D) = "bump" {}
        _BumpScale ("Bump Scale", Float) = 1.0
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _Gloss ("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            float _BumpScale;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                float4 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float4 uv : TEXCOORD0;

                // 矩阵变量按行拆分存储，因为一个插值寄存器最多只能存储 float4 大小的变量故将切线空间到世界空间的变换矩阵的每一行拆分到对应的 float4 变量中
                // 矩阵是 3X3 大小，但是为了充分利用存储空间，float4的 w 分量用于分别存储世界空间下的顶点位置的 x、y、z 分量
                float4 TtoW0 : TEXCOORD1;
                float4 TtoW1 : TEXCOORD2;
                float4 TtoW2 : TEXCOORD3;
            };


            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);

                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                o.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;

                
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz; //这里最好不要使用 UnityObjectToWorldDir 这个帮助函数，因为这个帮助函数会归一化向量，详见源码，这样存储的原点位置会导致片元着色器的计算有点小问题

                //这里使用归一化后的向量可行的原因是切线空间到世界空间的变换不存在缩放，必然是一个旋转或者镜像，所以乘上 v.tangent.w。上面的在切线空间下计算的方法存在缩放问题，是因为模型空间到世界空间可能存在非统一缩放，即模型存在非统一缩放，所以当切线空间转换到模型空间也可能存在非统一缩放。不知道对不对，先写着，否则没法解释被归一化的问题
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
                fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w; //单位向量与单位向量叉乘得到的也是单位向量

                // 计算切线空间到世界空间的矩阵，并存储到 TtoWX 变量中，将切线、副切线、法线按列拜访，w 分量用来存储顶点在世界空间下的坐标（充分利用插值寄存器的存储空间）
                o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);

                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                // 解析 w 分量，得到世界空间下的当前坐标
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                // 使用内置函数计算世界空间下的光照、视角方向（世界坐标被标准化了，计算出的方向就会出问题，所以不能标准化）
                fixed3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos));
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                // 采样并反映射得到法线信息，UnpackNormal 函数需要法线纹理的格式为 Normal map，别忘了
                fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv.zw));
                bump.xy *= _BumpScale;
                bump.z = sqrt(1.0 - saturate(dot(bump.xy, bump.xy)));

                // 将法线由切线空间转到世界空间，为什么这么乘，想想矩阵乘法
                bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));

                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(bump, lightDir));

                fixed3 halfDir = normalize(lightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(bump, halfDir)), _Gloss);

                return fixed4(ambient + diffuse + specular, 1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```

对材质的纹理进行赋值后，效果如下图（注意叉乘的顺序，会改变凹凸的方向）：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/15/GcNQ9geDMTlWbmy.jpg" width = "70%" height = "70%" alt="图22- 使用 Bump Scale 属性来调整模型的凹凸程度"/>
</div>

### Unity 中的法线纹理类型
#### 纹理压缩
前面代码中提到，必须将法线纹理标识为 Normal map，使用 Unity 的内置函数 UnpackNormal 才能得到正确的法线方向（Unity 会在材质面板中提醒你修正这个问题）。

这样做可以让 Unity 根据不同平台对纹理进行压缩（例如使用 DXT5nm 格式），再通过 UnpackNormal 函数来针对不同的压缩格式对法线纹理进行正确的采样。UnityCG.cginc 里 UnpackNormal 函数的内部实现（最新版的有细微变化）：  

    inline fixed3 UnpackNormalDXT5nm(fixed4 packednormal)
    {
        fixed3 normal;
        normal.xy = packnormal.wy * 2 - 1;
        normal.z = sqrt(1 - saturate(dot(normal.xy, normal.xy)));
        return normal;
    }

    inline fixed3 UnpackNormal(fixed4 packednormal)
    {
    #if defined(UNITY_NO_DXT5nm)
        return packednormal.xyz * 2 - 1;
    #else
    	return UnpackNormalDXT5nm(packednormal);
    #endif
    }

在 DXT5nm 格式的法线纹理中，纹理的 a 通道对应了法线的 x 分量，g 通道对应了法线的 y 分量，而纹理的 r 和 b 通道则会被舍弃，法线的 z 分量可以由 xy 分量推导而得。这种压缩方式可以减少法线纹理占用的内存空间。

#### 高度图的纹理类型
若要从高度图生成法线纹理，因为高度图本身记录的是相对高度，是一张灰度图，白色相对高，黑色相对低，所以把一张高度图导入 Unity 之后，除了需要将纹理类型设置为 Normal map 外，还需要勾选 **Create from Grayscale**，然后 Unity 会根据高度图来生成一张切线空间下的法线纹理（灰色变蓝色）。

勾选 Create from Grayscale 后，会多出 Bumpiness 和 Filtering 两个选项。Bumpiness 用于控制凹凸程度；Filtering 决定了使用什么方式计算凹凸程度，选择 Smooth，法线纹理会比较平滑，选择 Sharp，会使用 Sobel 滤波（一种边缘检测时使用的滤波器）来生成法线。

Sobel 滤波的实现是在一个 3×3 的滤波器中计算 x 和 y 方向上的导数，然后从中得到法线即可。具体方法是：对于高度图中的每个像素，考虑它与水平方向和竖直方向上的像素差，把它们的差当成该点对应的法线在 x 和 y 方向上的位移，然后使用映射函数存储成法线纹理的 r 和 g 分量即可。


## 渐变纹理
最开始纹理只是为了定义物体的颜色，后来发现，纹理可以存储任何表面属性。其中一种用法就是使用渐变纹理来控制漫反射光照的结果。这种技术在游戏《军团要塞 2》（Team Fortress 2）中流行起来，它也是由 Valve 公司提出来的，他们使用这种技术来渲染游戏中具有插画风格的角色。Valve 发表了一篇著名的论文来专门讲述在制作《军团要塞 2》时使用的技术。

这项技术最初由 Gooch 等人在 1998 年发表的一篇著名的论文《A Non-Photorealistic Lighting Model For Automatic Technical Illustration》中被提出，在这篇论文中，作者提出一种基于**冷到暖色调 cool-to-warm tones** 的着色技术，用来得到一种插画风格的渲染效果。使用这种技术，可以保证物体的轮廓线相比传统的漫反射光照更加明显，而且能够提供多种色调变化。现在，很多卡通风格的渲染中都使用了这种技术，在第十三章，会专门学习如何编写卡通风格的 Unity Shader。

### 实践
***1. 准备工作***  
①新建名为 Scene_7_3 的场景，并关闭天空盒子；  
②新建名为 RampTextureMat 的材质；  
③新建名为 Chapter7-RampTexture 的 Shader，并赋给上一步创建的材质；  
④往场景中拖拽一个名为 Suzanne 的模型，并把第二步创建的材质赋给它，模型可以从文章开头的 Git 仓库中找，路径为 Assets/Models/Chap7/；  
⑤保存场景

***2. 编写 Shader***  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 7/Ramp Texture"
{
    Properties
    {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _RampTex ("Ramp Tex", 2D) = "white" {} // 渐变纹理
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _Gloss ("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            sampler2D _RampTex;
            float4 _RampTex_ST;
            fixed3 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                float2 uv : TEXCOORD2;
            };

            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz; //不能使用 UnityObjectToWorldDir ，该帮助函数会把坐标点标准化，适用于方向，不使用于位置
                o.uv = TRANSFORM_TEX(v.texcoord, _RampTex); //使用内置宏计算经过平铺偏移后的纹理坐标
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;

                //半兰伯特模型
                fixed halfLambert = 0.5 * dot(worldNormal, worldLightDir) + 0.5;
                //见代码后面补充
                fixed3 diffuseColor = tex2D(_RampTex, fixed2(halfLambert, halfLambert)).rgb * _Color.rgb;
                fixed3 diffuse = _LightColor0.rgb * diffuseColor;

                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
                
                return fixed4(ambient + diffuse + specular, 1.0);
            }

            ENDCG
        }
    }
    Fallback "Specular"
}
```

补充：为什么这样计算可以使用渐变纹理，即使用 halfLambert 来构建纹理坐标。首先渐变纹理本质上是一个一维纹理（它在纵轴方向上颜色不变），所以渐变纹理本身不是和模型的纹理坐标一一对应的，而是和顶点的法线方向和光照之间的角度，一一对应。首先，halfLambert 被映射到了 [0, 1] 之间，假设顶点法线和光照方向相反，即点乘为 -1 ，halfLambert 值为零，这样得到的是渐变纹理最左边的颜色（一般为深颜色，可以看下面效果图）；若顶点法线和光照方向一致，即点乘为 1，halfLambert 值为 1，这样得到的是渐变纹理最右边的颜色（一般为浅色）。所以通过这样的方法，可以获得一个根据角度进行颜色渐变的模型。这也是为什么按 halfLambert 对渐变纹理进行采样，也就是 diffuseColor 的变量本质上已经反应了半兰伯特模型对光照的修改，所以 diffuse 变量不需要再乘上 halfLambert。

使用不同的渐变纹理的效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/16/nAixmBu37kfQRYg.jpg" width = "80%" height = "80%" alt="图23- 使用不同的渐变纹理控制漫反射光照，左下角给出了每张图使用的渐变纹理"/>
</div>

需要注意的是，渐变纹理的 WrapMode 需要设置为 Clamp，否则采样时由于浮点数精度造成问题。使用 Repeat 模式在高光区域会有些黑点，因为使用 halfLambert 采样时，理论上范围在 [0, 1]，但实际上会出现 1.00001 这样的值出现，使用 Repeat 模型，会舍弃整数部分，保留 0.00001 ，对应渐变图的最左边，即黑色。


## 遮罩纹理
**遮罩纹理 mask texture** 允许我们保护某些区域，使其免于被修改。例如之前，我们都是把高光反射应用到模型表面的所有的地方，即所有的像素都使用同样大小的高光强度和高光指数，但有时，我们希望模型表面某些区域的反光强烈一些，而某些区域弱一些。为了得到更细腻的效果，我们使用一张遮罩纹理来控制光照。另一个常见的应用就是在制作地形材质时，需要混合多张图片，例如草地纹理、石子纹理、土地纹理等，使用遮罩纹理可以控制如何混合这些纹理。

使用遮罩纹理的一般流程：采样得到遮罩纹理的纹素值，然后取纹素值的某个（或某几个）通道的值（如：r，rg）与某种表面属性相乘。这样，若该通道的值为 0，可以保护表面不受该属性影响。总之，遮罩纹理可以让美术人员更加精准（像素级别）地控制模型表面的各种性质。

### 实践
***1. 准备工作***  
①新建名为 Scene_7_4 的场景，并去掉天空盒子；  
②新建名为 MaskTextureMat 的材质；  
③新建名为 Chapter7-MaskTexture 的 Shader，并赋给上一步创建的材质；  
④在场景中创建一个胶囊体，并将第二步的材质赋给它；  
⑤保存场景；

***2. 编写 Shader***  
原书中用的是在切线空间下计算的方法，我把它改为了在世界空间下计算：

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 7/Mask Texture"
{
    Properties
    {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Tex", 2D) = "white" {}
        _BumpMap ("Normal Map", 2D) = "bump" {}
        _BumpScale ("Bump Scale", Float) = 1.0
        _SpecularMask ("Specular Mask", 2D) = "white" {} //高光反射遮罩纹理
        _SpecularScale ("Specular Scale", Float) = 1.0 //控制遮罩影响程度的系数
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _Gloss ("Gloss", Range(8.0, 256)) = 20
    }

    SubShader
    {
        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST; // _MainTex、_BumpMap 和 _SpecularMask 共用同一套纹理属性变量 _MainTex_ST，这样意味着修改主纹理的平铺系数和偏移系数，会同时影响 3 个纹理的采样，这样可以节省存储的纹理坐标数，减少差值寄存器的使用。
            sampler2D _BumpMap;
            float _BumpScale;
            sampler2D _SpecularMask; 
            float _SpecularScale;
            fixed4 _Specular;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                float4 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float4 uv : TEXCOORD0;
                float4 TtoW0 : TEXCOORD1;
                float4 TtoW1 : TEXCOORD2;
                float4 TtoW2 : TEXCOORD3;
            };


            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;

                
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
                fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w;

                o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos));
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv));
                bump.xy *= _BumpScale;
                bump.z = sqrt(1.0 - saturate(dot(bump.xy, bump.xy)));
                bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));

                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(bump, lightDir));

                fixed3 halfDir = normalize(lightDir + viewDir);
                // 先对遮罩纹理进行采样，本书使用的遮罩纹理的每个纹素的 rgb 分量是一样的，表示该点对应的高光强度，这里选择 r 分量来计算掩码值。然后乘上 _SpecularScale 来控制高光反射的强度
                fixed specularMask = tex2D(_SpecularMask, i.uv).r * _SpecularScale;
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0, dot(bump, halfDir)), _Gloss) * specularMask;

                return fixed4(ambient + diffuse + specular, 1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```

效果如下（为了效果明显一点，把 _Gloss 属性调成了 8）：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/17/B52lDu4dftmwkg8.png" width = "70%" height = "70%" alt="图24- 使用高光遮罩纹理。从左到右：只包含漫反射，未使用遮罩的高光反射，使用遮罩的高光反射"/>
</div>

### 其他遮罩纹理
我们可以充分利用一张纹理 RGBA 四个通道，分别存储各种不同的属性来控制模型表面的渲染效果。例如，R 通道存储高光反射的强度；G 通道存储边缘光照的强度；B 通道存储高光反射的参数；A 通道存储自发光强度。

在《DOTA 2》的开发中，开发人员为每个模型使用了 4 张纹理：一张用于定义模型颜色，一张用于定义表面法线，另外两张则都是遮罩纹理，可提供 8 种额外的表面属性。可以在官网上找到详细的制作资料，是非常好的学习资料。


# 第七章 透明效果
实时渲染中，通过控制渲染模型的**透明通道 Alpha Channel** 来实现透明效果。当开启透明混合后，当一个物体被渲染到屏幕上时，每个片元除了颜色值和深度值，还有透明值属性。

在 Unity 中，我们通常使用两种方法来实现透明效果：第一种为使用**透明度测试 Alpha Test**，这种方式无法得到真正的半透明效果；另一种是**透明度混合 Alpha Blending**。

当场景中存在多个模型时，渲染顺序问题非常重要。实际上，对于不透明 opaque 物体，不考虑它们的渲染顺序也能得到正确的排序效果，这是由于**深度缓冲 depth buffer**，也称 **z-buffer**，的存在。在实时渲染中，深度缓冲是用于解决可见性问题的。它的基本思路就是：根据深度缓冲中的值来判断该片元距离摄像机的距离，当渲染一个片元时，需要把它的深度值和已经存在于深度缓冲中的值进行比较（如果开启了深度测试），如果它的值距离摄像机更远，那么说明这个片元不应该被渲染到屏幕上（即有物体挡住了它）；否则，这个片元应该覆盖掉此时颜色缓冲中的像素值，并把它的深度值更新到深度缓冲中（如果开启了深度写入）。使用深度缓冲，可以不用关心不透明物体的渲染顺序，但如果要实现透明效果，就不那么简单了，因为要关闭深度写入。

①**透明度测试**：只要一个片元的透明度不满足条件（通常是小于某个阈值），则该片元就会被舍弃，否则就按不透明的物体进行处理，即进行深度测试、深度写入等。即透明度测试不需要关闭深度写入，虽然简单，但是它的效果，要么看不见，要么完全不透明。  

②**透明度混合**：可以得到半透明效果。使用当前片元的透明度作为混合因子，与已经存储在颜色缓冲中的颜色值进行混合，得到新的颜色。但是，透明度混合需要关闭深度写入（后面讲为什么），这使得要特别小心物体的渲染顺序。注意，透明度混合只关闭了深度写入（即对于透明度混合来说，深度缓冲是只读地），没关闭深度测试。即当不透明物体在透明物体前面，还是可以正常地遮挡。

## 渲染顺序的重要性
对于透明度混合，关闭深度写入是必要的：如果不关闭深度写入，一个半透明表面背后的表面本来是可以透过它被看到的，但是由于深度测试时判断是该半透明体离摄像机更近，导致后面的表面会被剔除。而关闭深度写入，会导致渲染顺序变得非常重要。

假设场景里存在半透明物体 A，和不透明物体 B，A 在 B 前面（A 离摄像头更近），此时不同的渲染顺序会有不同的结果：  
①先渲染 B，再渲染 A：由于不透明物体开启了深度测试和深度写入，B 会首先写入颜色缓冲和深度缓冲。随后渲染 A，透明物体仍然会进行深度测试，因为 A 离摄像头更近，因此使用 A 的透明度和颜色缓冲中 B 的颜色进行混合就可以得到正确的半透明效果。  
②先渲染 A，再渲染 B：渲染 A 时，深度缓冲区没有任何有效数据，因此 A 直接写入颜色缓冲，但由于对半透明物体关闭了深度写入，因此 A 不会修改深度缓冲。等到渲染 B 时，B 进行深度测试，由于深度缓冲中没有任何有效数据，B 就会写入颜色缓冲，结果就是 B 会直接覆盖 A 的颜色。

假设场景中 A 和 B 都是半透明物体，半透明物体之间也要符合一定的渲染顺序：  
①先渲染 B，再渲染 A：B 会正常写入颜色缓冲，然后 A 和颜色缓冲中的 B 颜色进行混合，得到正确的半透明效果。  
②先渲染 A，再渲染 B：A 先正常写入颜色缓冲，随后 B 会和颜色缓冲中的 A 进行混合，这样混合结果会完全反过来，看起来好像 B 在 A 的前面，得到错误的半透明结构。

基于上面两个例子，渲染引擎一般会先对物体进行排序，再渲染。常用的方法是：  
①先渲染所有不透明物体，并开启它们的深度测试和深度写入；  
②把透明物体按照它们距离摄像机的远近进行排序，然后按照从后往前的顺序渲染这些半透明物体，开启它们的深度测试，但关闭深度写入。

但是仍有问题没被解决，因为物体级别的顺序和像素级别的顺序会存在出入。由于某些情况下物体的排序存在循环重叠的情况（比如三个棒子，交叉循环），会导致渲染无法得到正确的结果。即使使用物体网格的相同顶点进行排序，仍然可能存在，A 的顶点在 B 的顶点前，但实际上，B 在 A 前的情况，如下图：

<table><tr>
<td><img src='https://s2.loli.net/2023/11/21/mP74J5snRhDW9MA.jpg' width="200" alt="图25- 循环重叠的半透明物体总是无法得到正确的半透明效果"></td>
<td><img src='https://s2.loli.net/2023/11/21/ifpBgNaZwVH9CPW.jpg' width="200" alt="图26- 使用哪个深度对物体进行排序。红色点分别标明了网格上距离摄像机最近的点、最远的点以及网格中点"></td>
</tr></table>

尽管会有一些特殊情况，大多数游戏引擎仍然使用了这样的方法。为了减少错误顺序的情况，开发者尽可能让模型是凸面体，并考虑将复杂的模型拆分成可以独立排序的多个子模型。而若不想分割网格可以将透明通道更加柔和，使穿插不那么明显，同时也可以使用开启深度写入的半透明效果来近似模拟物体的半透明（见本章第 5 节）。


## Unity Shader 的渲染顺序
Unity 为了解决渲染顺序的问题提供了**渲染队列 render queue** 这一解决方案。可以使用 SubShader 的 **Queue** 标签来决定我们的模型将归于哪个渲染队列。Unity 在内部使用一系列整数索引来表示每个渲染队列，且索引号越小表示越早被渲染。下表给出了 Unity 提前定义的渲染队列以及它们的描述（在每个队列中间可以使用其他队列）：  

| 名称 | 队列索引号 | 描述 |
| :---- | :---- | :---- |
| Background | 1000 | 会在任何其他队列之前渲染，通常使用该队列来渲染那些需要绘制在背景上的物体 |
| Geometry | 2000 | 默认的渲染队列，大多数物体使用这个队列，不透明物体使用这个队列 |
| AlphaTest | 2450 | 需要透明度测试的物体使用这个队列。Unity 5 中将它从 Geometry 中分离出来，因为在所有不透明物体渲染之后再渲染需要透明度测试的物体更加有效 |
| Transparent | 3000 | 这个队列的物体会在所有的 Geometry 和 AlphaTest 物体渲染后，再按**从后往前**的顺序进行渲染。任何使用了透明底混合(例如关闭了深度写入的 Shader)的物体都应该使用该队列 |
| Overlay | 4000 | 用于实现一些叠加效果。任何需要在最后渲染的物体都使用该队列 |

想要通过透明度测试实现透明效果，代码应该包含类似下面的代码：  

    SubShader {
    	Tags {"Queue" = "AlphaTest"}
    	Pass {
    		...
    	}
    }

想要通过透明度混合实现透明效果，代码应该包含类似下面的代码：  

    SubShader {
	    Tags {"Queue" = "Transparent"}
	    Pass { 
		    ZWrite Off
		    ...
	    }
    }

其中，**ZWrite Off** 用于关闭深度写入，这里选择写在 Pass 中。也可以写在 SubShader 中，意味着 SubShader 下的所有 Pass 都会关闭深度写入。


## 透明度测试
只要一个片元的透明度不满足条件（通常是小于某个阈值），则这个片元就会被舍弃，不进行任何处理，不会对颜色缓冲产生任何影响。否则按照普通的不透明物体的处理方式进行处理。通常在片元着色器中使用 `clip` 函数进行透明度测试，clip 是 Cg 中的一个函数，它的定义如下：  

    // 函数的参数是裁剪时使用的标量或矢量条件
    void clip(float4 x);
    void clip(float3 x);
    void clip(float2 x);
    void clip(float1 x);
    void clip(float x);

    // 如果给定参数的任何一个分量是负数，就会舍弃当前像素的输出颜色
    void clip(float4 x)
    {
        if (any(x < 0))
            discard;
    }

### 实践
***1. 准备工作***  
①新建名为 Scene_8_3 的场景，并去掉天空盒子；  
②新建名为 AlphaTestMat 的材质；  
③新建名为 Chapter8-AlphaTest 的 Shader，并赋给上一步创建的材质；  
④在场景中创建一个立方体，并把第二步的材质赋给它；  
⑤保存场景。

***2. 编写 Shader***  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 8/Alpha Test"
{
    Properties
    {
        _Color ("Main Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Tex", 2D) = "white" {}
        _Cutoff ("Alpha Cutoff", Range(0, 1)) = 0.5 // 透明度测试的阈值
    }
    SubShader
    {
        // 透明度测试常用的三个标签
        Tags {
            // 设置渲染队列为 AlphaTest 队列
            "Queue" = "AlphaTest"
            // 让当前 Shader 不会受到投射器的影响
            "IgnoreProjector" = "True"
            // 让 Unity 把当前 Shader 归入到提前定义的组，这里指 TransparentCutout 组，以指明该 Shader 是一个使用了透明度测试的 Shader
            "RenderType" = "TransparentCutout"
        }

        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed _Cutoff;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                float2 uv : TEXCOORD2;
            };

            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

                fixed4 texColor = tex2D(_MainTex, i.uv);

                // 透明度测试：透明度小于 _Cutoff 的值，则当前片元会被丢弃
                clip(texColor.a - _Cutoff);

                // 等同于
                // if ((texColor.a - _Cutoff) < 0.0) {
                //     discard;
                // }

                fixed3 albedo = texColor.rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
                return fixed4(ambient + diffuse, 1.0);
            }
            ENDCG
        }
    }

    // 使用内置的 VertexLit Shader 作为回调 Shader。保证该 SubShader 无法在当前显卡工作时可以有合适的代替 shader，还保证使用透明度测试的物体可以正确地向其他物体投射阴影
    Fallback "Transparent/Cutout/VertexLit"
}
```

使用下图的半透明纹理来实现透明度测试。在 Github 中 /Assets/Textures/Chapter8 文件夹下的 transparent_texure.psd：

<div  align="center">  
<img src="https://s2.loli.net/2023/11/21/VQ2bGYOuZXqWopi.jpg" width = "30%" height = "30%" alt="图27- 一张透明纹理，其中每个方格的透明度都不同"/>
</div>

效果如下（从左到右分别给 _Cutoff 值设置 0.6、0.7、0.8）：

<div  align="center">  
<img src="https://s2.loli.net/2023/11/21/MsIvklBULaj94pD.png" width = "70%" height = "70%" alt="图28- 随着 Alpha cutoff 参数的增大，更多的像素由于不满足透明度测试条件而被剔除"/>
</div>

可以看出，透明度测试得到的效果，要么完全透明，要么完全不透明。而且，得到的透明效果在边缘处往往参差不齐，有锯齿，这是因为边界处纹理的透明度的变化精度问题。


## 透明度混合
使用当前片元的透明度作为混合因子，与已经存储在颜色缓冲中的颜色值进行混合得到新的颜色。但是需要关闭深度写入，所以我们需要非常小心物体的渲染顺序。为了进行混合，Unity 为我们提供了混合命令 —— **Blend**。Blend 是 Unity 提供的设置混合模式的命令。下表中给出了 Blend 命令的语义：  

| 语义 | 描述 |
| :---- | :---- |
| Blend Off | 关闭混合 |
| Blend SrcFactor DstFactor | 开启混合，并设置混合因子。源颜色（当前片元的颜色）x SrcFactor + 目标颜色（已经存在于颜色缓冲中的颜色） x DstFactor，得到的最终颜色存入颜色缓冲中 |
| Blend SrcFactor DstFactor, SrcFactorA DstFactorA | 与上面几乎一样，使用不同的因子来混合透明通道 |
| BlendOp BlendOperation | 并非源颜色和目标颜色的简单相加后混合，而使用 BlendOperation 对它们进行其他操作 |

在本节中，会使用第二种语义，即 Blend SrcFactor DstFactor 来进行混合（更多混合语义见本章第 6 节）。这个命令在设置混合因子的同时也开启了混合模式。Blend 命令设置混合因子的同时，Unity 会自动帮我们开启混合模式。

我们把源颜色的混合因子 SrcFactor 设为 SrcAlpha，目标颜色的混合因子 DstFactor 设为 OneMinusSrcAlpha，那么混合后的新颜色是：

$$ DstColor_{new} = SrcAlpha \times SrcColor + (1-SrcAlpha) \times DstColor_{old} $$

Src 前缀即 source，Dst 前缀即 destination。OneMinusSrcAlpha （字面意思）就是对应 1 - SrcAlpha。

### 实践
***1. 准备工作***  
①新建名为 Scene_8_4 的场景，并去掉天空盒子；  
②新建名为 AlphaBlendMat 的材质；  
③新建名为 Chapter8-AlphaBlend 的 Shader，并赋给上一步创建的材质；  
④在场景中创建一个立方体，将第二步创建的材质赋给它；  
⑤保存场景。

***2. 编写 Shader***  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 8/Alpha Blend"
{
    Properties
    {
        _Color ("Main Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Tex", 2D) = "white" {}
        _AlphaScale ("Alpha Scale", Range(0, 1)) = 1 // 用于在透明纹理的基础上控制整体的透明度
    }

    SubShader
    {
        // 透明度混合常用的 3 个标签
        Tags {
            // 将渲染队列修改为透明混合队列 Transparent
            "Queue" = "Transparent"
            "IgnoreProjector" = "True"
            // 将当前 Shader 归入提前定义的组， Transparent 组。用来指明该 Shader 是一个使用了透明度混合的 Shader
            "RenderType" = "Transparent"
        }

        Pass
        {
            Tags { "LightMode" = "ForwardBase" }

            // 关闭深度写入
            ZWrite Off
            // 将源颜色（该片元着色器产生的颜色）的混合因子设为 SrcAlpha
            // 将目标颜色（已存在颜色缓冲中的颜色）的混合因子设为 OneMinusSrcAlpha
            Blend SrcAlpha OneMinusSrcAlpha

            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            fixed _AlphaScale;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                float2 uv : TEXCOORD2;
            };

            v2f vert (a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

                fixed4 texColor = tex2D(_MainTex, i.uv);

                // 这里移除了透明度测试的代码

                fixed3 albedo = texColor.rgb * _Color.rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));

                // 在返回值中设置透明通道
                return fixed4(ambient + diffuse, texColor.a * _AlphaScale);
            }
            ENDCG
        }
    }
    // 修改回调的 Shader
    Fallback "Transparent/VertexLit"
}
```

不同的 Alpha Scale 参数下的半透明效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/21/Xz57i9gSQTWs2PG.jpg" width = "70%" height = "70%" alt="图29- 随着 Alpha Scale 参数的增大，模型变得越来越透明"/>
</div>

之前解释过由于关闭深度写入带来的各自问题，即当模型本身有复杂的遮挡关系或是包含了复杂的非凸网格时，会因为排除错误而导致错误的透明效果。下图给出了使用上面的 Shader 渲染 Knot 模型出现的问题：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/21/nUf5Yat6L21Qdgs.jpg" width = "30%" height = "30%" alt="图30- 当模型网格之间有互相交叉的结构时，往往会得到错误的半透明效果"/>
</div>

这都是由于关闭了深度写入，无法对模型进行像素级别的深度排序。对此，可以通过分割网格或者重新利用深度写入，从而让模型可以像半透明模型一样淡入淡出。


## 开启深度写入的半透明效果
为了解决上面描述的问题，一种解决办法是**使用两个 Pass** 来渲染模型：第一个 Pass 开启深度写入，但不输出颜色，仅仅是为了把模型的深度值写入深度缓冲；第二个 Pass 进行正常的透明度混合，由于上一个 Pass 已经得到了逐像素的正确的深度值，该 Pass 就可以按照像素级别的深度排序进行透明渲染。缺点就是 Pass 多了影响性能。

### 实践
***1. 准备工作***  
①新建名为 Scene_8_5 的场景，并关闭天空盒子；  
②新建名为 AlphaBlendZWriteMat 材质；  
③新建名为 Chapter8-AlphaBlendZWrite 的 Shader，并赋给上一步创建的材质；  
④在场景中创建一个立方体，并将第二步创建的材质赋给它；  
⑤保存场景。

***2. 编写 Shader***  

这里的代码跟上一节的几乎一模一样，只需要在原来的 Pass 上面再增加一个新的 Pass 即可：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 8/Alpha Blending ZWrite" {
    Properties {...}
    SubShader {
        Tags {...}

        // 额外的 pass
        Pass {
            ZWrite On
            ColorMask 0
        }

        Pass {
            // 上一节的 pass
        }
    }
    Fallback "Diffuse"
}
```

新添加的 Pass 的目的仅仅是将模型的深度信息写入深度缓冲中，从而剔除被遮挡的片元。Pass 的第一行开启了深度写入。第二行，使用了新的渲染命令 ColorMask，在 ShaderLab 中，ColorMask 用于设置颜色通道的写掩码 write mask。语义为 `ColorMask RGB | A | 0 | 其他任何R、G、B、A的组合` ，当设为 0 时，表示该 Pass 不写入任何颜色通道。与上一节，效果对比如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/21/xj5VIvprgBOuo8S.png" width = "70%" height = "70%" alt="图31- 开启了深度写入的半透明效果（右图）和没有开启深度写入的效果（左图）的对比"/>
</div>

可以看出有深度写入的模型内部有遮挡关系，而没有深度写入的模型看不出前后遮挡关系。但有深度写入的物体的内部没有半透明颜色的融合效果。

## ShaderLab 的混合命令
混合和两个操作数有关：**源颜色 source color** 用 S 表示，指由片元着色器产生的颜色值；**目标颜色 destination color** 用 D 表示，指从颜色缓冲中读取到的颜色值。对它们进行混合后得到的**输出颜色**，用 O 表示，重新写入颜色缓冲中。需要注意，当谈及混合中的源颜色、目标颜色和输出颜色时，它们都包含了 RGBA 四个通道的值。

想要使用混合，我们必须首先开启它。在 Unity 中，当我们使用 Blend 命令时，除了设置混合状态外也开启了混合。但在其他图形 API 中需要手动开启，比如在 OpenGL 中，需要使用 glEnable(GL_BLEND) 来开启混合。

### 混合等式和参数
混合是一个逐片元的操作，且是不可编程的，却是高度可配置的。我们可以设置混合时使用的运算操作、混合因子等来影响混合。

从两个操作数 S 和 D，得到输出颜色 O 的等式称为**混合等式 blend equation**。当进行混合时，我们需要使用两个混合等式：一个用于混合 RGB 通道，一个用于混合 A 通道。当设置混合状态时，实际上设置的就是混合等式中的**操作**和**因子**。默认情况下，混合等式使用的操作都是加操作，我们只需要再设置一下混合因子即可。由于需要两个等式，每个等式有两个因子，因此一共需要四个因子。

`Blend SrcFactor DstFactor` 这个命令只提供了两个因子，意味着使用同样的混合因子来混合 RGB 通道和 A 通道，即：  

$$ O_{rgb} = SrcFactor \times S_{rgb} + DstFactor \times D_{rgb} $$
$$ O_{a} = SrcFactorA \times S_{a} + DstFactorA \times D_{a} $$

这些混合因子可以有以下值，下表给出了 ShaderLab 支持的几种混合因子：  

| 参数 | 描述 |
| :---- | :---- |
| One | 因子为 1 |
| Zero | 因子为 0 |
| SrcColor | 因子为源颜色值。用于混合 RGB 时，使用 SrcColor 的 RGB 分量作为混合因子。用于混合 A 时，使用 SrcColor 的 A 分量作为混合因子 |
| SrcAlpha | 因子为源颜色的透明度值（A 通道） |
| DstColor | 因子为目标颜色值。用于混合 RGB 时，使用 DstColor 的 RGB 分量作为混合因子。用于混合 A 时，使用 DstColor 的 A 分量作为混合因子 |
| DstAlpha | 因子为目标颜色的透明度值（A 通道） |
| OneMinusSrcColor | 因子为（1-源颜色）。用于混合 RGB 时，使用计算结果的 RGB 分量作为混合因子。用于混合 A 时，使用计算结果的 A 分量作为混合因子 |
| OneMinusSrcAlpha | 因子为（1-源颜色的透明度） |
| OneMinusDstColor | 因子为（1-目标颜色）。用于混合 RGB 时，使用计算结果的 RGB 分量作为混合因子。用于混合 A 时，使用计算结果的 A 分量作为混合因子 |
| OneMinusDstAlpha | 因子为（1-目标颜色的透明度） |

如果想使用不同的参数混合 RGB 通道和 A 通道，可以使用 `Blend SrcFactor DstFactor, Srcfactor DstFactorA` 指令。例如，下面的命令在混合后，输出颜色的透明值就是源颜色的透明值：`Blend SrcAlpha OneMinusSrcAlpha, One Zero`

### 混合操作
上面涉及的两个混合等式中，都是加操作。若要使用其他混合操作，可以使用 ShaderLab 的 `BlendOp BlendOperation` 命令，即混合操作命令。下表给出了支持的混合操作：  

| 操作 | 描述 |
| :---- | :---- |
| Add | 将混合后的颜色相加，默认的混合操作。使用的混合等式是： <br> $$ O_{rgb} = SrcFactor \times S_{rgb} + DstFactor \times D_{rgb} $$ <br> $$ O_{a} = SrcFactorA \times S_{a} + DstFactorA \times D_{a} $$ |
| Sub | 用混合后的源颜色减去混合后的目标颜色。使用的混合等式是： <br> $$ O_{rgb} = SrcFactor \times S_{rgb} - DstFactor \times D_{rgb} $$ <br> $$ O_{a} = SrcFactorA \times S_{a} - DstFactorA \times D_{a} $$ |
| RevSub | 用混合后的目标颜色减去混合后的源颜色。使用的混合等式是： <br> $$ O_{rgb} = DstFactor \times D_{rgb} - SrcFactor \times S_{rgb} $$ <br> $$ O_{a} = DstFactorA \times D_{a} - SrcFactorA \times S_{a} $$ |
| Min | 使用源颜色和目标颜色中较小的值，是逐分量比较的。使用的混合等式是： <br> $$ O_{rgba} = (min(S_r, D_r),min(S_g, D_g),min(S_b, D_b),min(S_a, D_a))$$ |
| Max | 使用源颜色和目标颜色中较大的值，是逐分量比较的。使用的混合等式是： <br> $$ O_{rgba} = (max(S_r, D_r),max(S_g, D_g),max(S_b, D_b),max(S_a, D_a))$$ |
| 其他逻辑操作 | 仅在 DirectX 11.1 中支持 |

混合操作命令通常是与混合因子命令一起工作的。但当使用 Min 或 Max 混合操作时，混合因子实际上不起任何作用，它们仅会判断原始的源颜色和目标颜色之间的比较结果。

### 常见的混合类型
通过混合操作和混合因子命令的组合，可以得到一些类似 Photoshop 混合模式中的混合效果：  

    // 正常（Normal），即透明度混合
    Blend SrcAlpha OneMinusSrcAlpha

    // 柔和相加（Soft Additive）
    Blend OneMinusDstColor One

    // 正片叠底（Multiply），即相乘
    Blend DstColor Zero

    // 两倍相乘（2x Multiply）
    Blend DstColor SrcColor

    // 变暗（Darken）
    BlendOp Min
    Blend One One

    // 变亮（Lighten）
    BlendOp Max
    Blend One One

    // 滤色（Screen）
    Blend OneMinusDstColor One
    // 等同于
    Blend One OneMinusSrcColor

    // 线性减淡（Linear Dodge）
    Blend One One

<div  align="center">  
<img src="https://s2.loli.net/2023/11/22/ikua92s7pQZ3Mbf.jpg" width = "70%" height = "70%" alt="图32- 不同混合状态设置得到的效果"/>
</div>

需要注意的是，虽然上面使用 Min 和 Max 混合操作时仍然设置了混合因子，但实际上它们并不会对结果产生影响。另一点是，虽然上面有些混合模式没有设置混合操作的种类，但是默认是加法操作，相当于设置了 `BlendOp Add`。

## 双面渲染的透明效果
在前面实现的透明效果中，无论透明度测试还是混合，都无法观察到正方体内部及其背部的形状。这是因为，默认情况下，渲染引擎剔除了物体背面（相对于摄像机的方向）的渲染图元。如果想要得到双面渲染的效果，可以使用 **Cull** 指令来控制需要剔除哪个面的渲染图元。在 Unity 中，Cull 指令的语法如下：  

    Cull Back | Front | Off

如果设置为 Back，背对摄像机的渲染图元不会被渲染，这是默认的剔除状态；如果设置为 Front，朝向摄像机的渲染图元不会被渲染；如果设置为 Off，会关闭剔除功能。

### 透明度测试的双面渲染
只要在 Pass 的渲染设置中使用 Cull 指令来关闭剔除即可。新建名为 Scene_8_7_1 的场景，并去掉天空盒子；新建名为 AlphaTestBothSidedMat 的材质；新建名为 Chapter8-AlphaTestBothSided 的 Shader，赋给上一步创建的材质；在场景中创建一个立方体，将第二步的材质赋给它。代码和 Chapter8-AlphaTest 几乎完全一样，只需添加一行代码：  

    Pass {
        Tags { "LightMode" = "ForwardBase" }

        // 关闭剔除功能
        Cull Off

效果如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/22/S5JLAgPENwdWqy8.png" width = "70%" height = "70%" alt="图33- 双面渲染的透明度测试的物体"/>
</div>

### 透明度混合的双面渲染
透明度混合实现双面渲染会更复杂，因为透明度混合要关闭深度写入，直接关闭剔除功能，无法保证同一个物体的正面和背面图元的渲染顺序，就可能得到错误的半透明效果。

为此，需要把双面渲染的工作分为两个 Pass，第一个 Pass 只渲染背面，第二个 Pass 只渲染正面，Unity 会顺序执行 Subshader 中的各个 Pass，可以保证背面在正面被渲染之前渲染。

新建名为 Scene_8_7_2 的场景；新建名为 AlphaBlendBothSidedMat 的材质；新建名为 Chapter8-AlphaBlendBothSided 的 shader，并赋给上一步创建的材质；在场景中创建一个立方体，将第二步创建的材质赋给它；代码和 Chapter8-AlphaBlend 相比较，只要做出两个改动：  
①复制原 Pass 的代码，得到另一个 Pass；  
②在两个 Pass 中分别使用 Cull 指令剔除不同朝向的渲染图元：

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 8/Alpha Blend With Both Side" {
    Properties {...}
    SubShader {
        Tags {...}

        Pass {
            Tags {...}

            // 第一个 Pass 只渲染背面
            Cull Front

            //和之前一样的代码
        }

        Pass {
            Tags {...}

            // 第二个 Pass 只渲染前面
            Cull Back

            //和之前一样的代码
        }
    }
}
```

效果如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2023/11/22/8kSUniB62ZGx5EM.png" width = "70%" height = "70%" alt="图34- 双面渲染的透明度混合的物体"/>
</div>

