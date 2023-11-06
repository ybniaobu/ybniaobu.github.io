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

> 本读书笔记主要内容为XXXXXXXXXXXXXXXXXXXX。
> 读书笔记是对知识的记录与总结，但是对比较熟悉的内容不会再行描述。

# 第五章 开始 Unity Shader 学习之旅
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


# 第六章 Unity 中的基础光照
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

$$c_{diffuse} = (c_{light} \cdot m_{diffuse}) max(0, \hat {n} \cdot \hat {l})$$

其中，$\hat {n}$ 是表面法线，$ \hat {l} $ 是指向光源的单位矢量，$m_{diffuse}$ 是材质的漫反射颜色，$c_{light}$ 是光源颜色。取最大值是为了防止点乘结果为负值，可以防止物体被后面来的光照亮。

### 高光反射
这里的高光反射是一种经验模型，即并不完全符合真实世界中的高光反射现象。它可用于计算那些沿着完全镜面反射方向被反射的光线。

计算高光反射需要信息较多，如表面法线 n、视角方向 v、光源方向 l、反射方向 r 等。假设这些矢量都是单位矢量，反射方向可以通过法线方向和光源方向计算得到：  

$$\hat {r} = 2(\hat {n} \cdot \hat {l})\hat {n} - \hat {l}$$

利用 Phong 模型来计算高光反射的部分：  

$$c_{specular} = (c_{light} \cdot m_{specular}) max(0, \hat {v} \cdot \hat {r})^{m_{gloss}}$$

① $m_{gloss}$ 是材质的**光泽度 gloss**，也称为**反光度 shininess**，用于控制高光区域的亮点有多宽，$m_{gloss}$ 越大，亮点就越小。  
② $m_{specular}$ 是材质的高光反射颜色，用于控制该材质对于高光反射的强度和颜色。  
③ $c_{light}$ 是光源的颜色和强度。

<div  align="center">  
<img src="https://s2.loli.net/2023/11/06/tQmMHVJ4WfZSw3j.jpg" width = "50%" height = "50%" alt="图14- 使用 Phong 模型计算高光反射"/>
</div>

在 Phong 模型基础上，Blinn 简化了计算来得到类似的效果。为了避免计算反射方向 $\hat r$，Blinn 模型引入了一个新的矢量 $\hat h$，它是通过对 $\hat v$ 和 $\hat l$ 的取平均后再归一化得到的，即：  

$$\hat {h} = \cfrac {\hat {v} + \hat {l}} {|\hat {v} + \hat {l}|}$$

然后使用 $\hat n$ 和 $\hat h$ 之间的夹角进行计算，而非 $\hat v$ 和 $\hat r$ 之间的夹角：  

$$c_{specular} = (c_{light} \cdot m_{specular}) max(0, \hat {n} \cdot \hat {h})^{m_{gloss}}$$

<div  align="center">  
<img src="https://s2.loli.net/2023/11/06/8Ky4CHnVqdlcUXP.jpg" width = "50%" height = "50%" alt="图15- 使用 Blinn 模型计算高光反射"/>
</div>

在硬件实现时，如果摄像机和光源距离模型足够远，Blinn 模型会快于 Phong 模型，因为此时可以认为 v 和 l 都是定值，因此 h 是一个常量。但是当 v 和 l 不是定值时，Phong 模型反而可能更快。注意，这两种光照模型都是经验模型，不能认为 Blinn 模型是对“正确的” Phong 模型的近似。在一些情况下 Blinn 模型更符合实验结果。

### 逐像素还是逐顶点
