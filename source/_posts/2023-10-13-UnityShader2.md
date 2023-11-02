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

