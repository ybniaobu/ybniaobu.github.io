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
Shader "Unity Shaders Book/Chapter 5/Simple Shader"
{
    // Properties 语义并不是必需的
    SubShader
    {
        // SubShader 中没有进行任何渲染设置和标签设置，所以该 SubShader 将使用默认的设置
        Pass
        {
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
            float4 vert(float4 v : POSITION) : SV_POSITION
            {
                return UnityObjectToClipPos(v); 
                // 或者使用这个语句 return mul(UNITY_MATRIX_MVP, v); 会被自动替换为上面语句
            }

            // 片元着色器代码
            // frag 函数没有任何输入，输出是一个 fixed4 类型的变量，并且使用了 SV_Target 语义进行限定
            // 通过 SV_Target 语义，告诉渲染器，把用户的输出颜色存储到一个渲染目标中（比如默认的帧缓存中）
            // 颜色的 RGBA 每个分量范围在[0, 1]，所以使用 fixed4 类型
            fixed4 frag() : SV_Target
            {
                return fixed4(1.0, 1.0, 1.0, 1.0);
            }

            ENDCG
        }
    }
}
```

