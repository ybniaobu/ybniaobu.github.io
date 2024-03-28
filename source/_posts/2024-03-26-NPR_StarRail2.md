---
title: 基于星穹铁道的卡通渲染（二）
date: 2024-03-26 15:25:22
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
  - NPR
mathjax: true
top_img: /images/black.jpg
cover: https://s2.loli.net/2024/03/26/kxdT3sE6wS9LXbp.gif
description: 本篇主要内容为 Toon Shader 的丝袜、边缘光、描边等效果的实现。本文以米哈游的崩坏星穹铁道的卡通人物渲染来切入 NPR (Non-Photorealistic Rendering) ，从而对卡通渲染有个初步的了解，并加深对渲染的理解，为做出自己的风格化渲染打下基础。
---

> 本文主要借鉴了 b 站 up 主给我柠檬养乐多你会跟我玩吗的视频，bv 号：BV1CN411C7qx。  
> 书接上文的光照部分，也就是间接光 + 直接光 + 高光。

# 丝袜效果
黑丝就是越直视，越容易看到丝袜下的皮肤；越边缘越不容易看到皮肤，越容易看到黑色。所以我们将法线点乘视角向量来模拟上述效果，使用指数参数 `_StockingsTransitionPower` 来增强效果，该值默认为 1。新增代码如下：  

``` C
float3 stockingsEffect = 1;

#if _AREA_UPPERBODY || _AREA_LOWERBODY
{
    float NdotV = dot(normalWS, viewDirectionWS);
    float fac = NdotV;
    fac = pow(saturate(fac), _StockingsTransitionPower);
}
#endif
```

使用参数 `_StockingsTransitionHardness` 来调整亮暗过渡的硬度，该值默认为 0.4：

    fac = saturate((fac - _StockingsTransitionHardness/2) / (1 - _StockingsTransitionHardness));

此时效果如下，我们主要观察腿部，因为还没使用丝袜贴图遮罩，故整个上下身都是该效果。可以尝试改变上述参数看看效果：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/26/AGKJhzVMEYQRoyd.jpg" width = "100%" height = "100%" alt="图20 - 丝袜效果"/>
</div>

接下来就是加上丝袜贴图了，我们观察丝袜贴图，因为 B 通道的纹理密度不够，所以我们将 UV 乘上 20 倍，同时把 RG 通道和 B 通道分开，新增代码如下：  

``` C
#if _AREA_UPPERBODY || _AREA_LOWERBODY
{
    float2 stockingsMapRG = 0;
    float stockingsMapB = 0;
    
    #if _AREA_UPPERBODY
        stockingsMapRG = SAMPLE_TEXTURE2D(_UpperBodyStockings, sampler_UpperBodyStockings, input.uv).rg;
        stockingsMapB = SAMPLE_TEXTURE2D(_UpperBodyStockings, sampler_UpperBodyStockings, input.uv * 20).b;
    #elif _AREA_LOWERBODY
        stockingsMapRG = SAMPLE_TEXTURE2D(_LowerBodyStockings, sampler_LowerBodyStockings, input.uv).rg;
        stockingsMapB = SAMPLE_TEXTURE2D(_LowerBodyStockings, sampler_LowerBodyStockings, input.uv * 20).b;
    #endif

    ...
}
#endif
```

然后就是把贴图的 B 通道，也就是丝袜纹理，和丝袜效果进行混合，使用 `_StockingsTextureUsage` 参数进行混合，该值默认为 0.1，下面代码其实就是 `lerp()` 函数拆出来：

    fac = fac * (stockingsMapB * _StockingsTextureUsage + 1 - _StockingsTextureUsage);

再使用贴图的 G 通道混合上花纹效果：  

    fac = lerp(fac, 1, stockingsMapRG.g);

其实此时已经可以得到一个不错的效果了，但是为了更不错的效果，我们使用颜色渐变。因为 URP 没有内置的颜色渐变的函数，但是 Shader Graph 有，我们可以在 Shader Graph 使用颜色渐变，然后生成一个 Shader 并复制其中代码，如下：  

``` C
struct Gradient
{
    int colorsLength;
    float4 colors[8];
};

Gradient GradientConstruct()
{
    Gradient g;
    g.colorsLength = 2; //默认 2 个颜色，首尾各一个
    g.colors[0] = float4(1, 1, 1, 0); //a 或 w 值表示的是颜色的位置，看看 Shader Graph 的颜色渐变节点就懂了
    g.colors[1] = float4(1, 1, 1, 1);
    g.colors[2] = float4(0, 0, 0, 0);
    g.colors[3] = float4(0, 0, 0, 0);
    g.colors[4] = float4(0, 0, 0, 0);
    g.colors[5] = float4(0, 0, 0, 0);
    g.colors[6] = float4(0, 0, 0, 0);
    g.colors[7] = float4(0, 0, 0, 0);
    return g;
}

float3 SampleGradient(Gradient Gradient, float Time) // Time 是 0 - 1 的一个值，控制要选择的渐变的点位。具体看官方文档
{
    float3 color = Gradient.colors[0].rgb;
    for (int c = 1; c < Gradient.colorsLength; c++)
    {
        float colorPos = saturate((Time - Gradient.colors[c - 1].w) / (Gradient.colors[c].w - Gradient.colors[c - 1].w)) * step(c, Gradient.colorsLength - 1);
        color = lerp(color, Gradient.colors[c].rgb, colorPos);
    }
    #ifdef UNITY_COLORSPACE_GAMMA
    color = LinearToSRGB(color);
    #endif
    return color;
}
```

我们使用上述函数创建颜色渐变，我们之前设置了三个丝袜颜色参数分别为 `_StockingsDarkColor` 默认为 (0, 0, 0)；`_StockingsLightColor` 默认为 (1.8, 1.48299, 0.856821)；`_StockingsTransitionColor` 默认为 (0.360381, 0.242986, 0.358131)。首先构建 Gradient，然后把上面三个参数赋值给 Gradient，中间的颜色的位置使用  `_StockingsTransitionThreshold` 参数控制。最后使用 fac 参数作为采样的点，来选择渐变颜色，代码如下：  

    Gradient curve = GradientConstruct();
    curve.colorsLength = 3;
    curve.colors[0] = float4(_StockingsDarkColor, 0);
    curve.colors[1] = float4(_StockingsTransitionColor, _StockingsTransitionThreshold);
    curve.colors[2] = float4(_StockingsLightColor, 1);
    float3 stockingsColor = SampleGradient(curve, fac);

最后使用丝袜贴图的 R 通道来进行遮罩：

    stockingsEffect = lerp(1, stockingsColor, stockingsMapRG.r);
    ...
    albedo *= stockingsEffect; //注意是乘上，因为没有额外的光

布洛妮娅的腿部的效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/26/rdmt9epKINo4uYJ.png" width = "30%" height = "30%" alt="图21 - 布洛妮娅丝袜效果"/>
</div>


# 描边 Pass
## 新增 Pass 和 HLSL 文件
描边效果需要使用另外一个 Pass，使用 `_OUTLINE_ON` 控制是否开启描边效果，开启就引入 ToonOutlinePass.hlsl 文件，不开启就输出 0：

``` C
Pass
    {
        Name "OutLine"
        
        Tags { "LightMode" = "SRPDefaultUnlit" }
        
        Cull Front //正面剔除
        ZWrite [_ZWrite]
        
        HLSLPROGRAM
        #pragma vertex ToonOutLineVert
        #pragma fragment ToonOutLineFrag

        #pragma multi_compile_fog

        #if _OUTLINE_ON
        #include "Assets/ShaderLibrary/ToonInput.hlsl"
        #include "Assets/ShaderLibrary/ToonOutlinePass.hlsl"

        #else
        struct Attributes {};
        struct Varyings
        {
            float4 positionCS : SV_POSITION;
        };
            
        Varyings ToonOutLineVert(Attributes input)
        {
            return (Varyings)0;
        }
            
        float4 ToonOutLineFrag(Varyings input) : SV_TARGET
        {
            return 0;
        }
        #endif
            
        ENDHLSL
    }
```

ToonOutlinePass.hlsl 基础参数及代码如下：  

``` C
#ifndef CUSTOM_TOON_OUTLINE_PASS_INCLUDED
#define CUSTOM_TOON_OUTLINE_PASS_INCLUDED

struct Attributes
{
    float3 positionOS   : POSITION;
    float3 normalOS     : NORMAL;
    float4 tangentOS    : TANGENT;
    float4 color        : COLOR;
    float2 uv           : TEXCOORD0;
    float4 uv7          : TEXCOORD7;
};

struct Varyings
{
    float2 uv                   : TEXCOORD0;
    float fogFactor             : TEXCOORD1; //w: vertex fog factor
    float4 color                : TEXCOORD2;
    float4 positionCS           : SV_POSITION;
};

Varyings ToonOutLineVert(Attributes input)
{
    Varyings output = (Varyings)0;

    VertexPositionInputs vertexPositionInput = GetVertexPositionInputs(input.positionOS);
    VertexNormalInputs vertexNormalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

    return output;
}

float4 ToonOutLineFrag(Varyings input) : SV_TARGET
{
    float4 color = float4(0, 0, 0, 1);
    return color;
}

#endif
```

## 描边顶点着色器
使用的描边方法就是法线外扩，只渲染模型的背面，把背面的顶点沿着法线往外移动一段距离。首先是获取外扩的宽度 `_OutlineWidth`，该值默认为 1，故乘上 0.001：  

    float width = _OutlineWidth;
    width *= 0.001;

然后就是使用它来对顶点沿着法线方向进行偏移：  

    float3 positionWS = vertexPositionInput.positionWS;
    positionWS += vertexNormalInput.normalWS * width;

    output.positionCS = TransformWorldToHClip(positionWS);

这样一个简单的描边效果就出现了，但是你也懂的，圆形区域还可以，矩形或尖锐区域就会出现描边断裂现象。所以需要对法线进行平滑，从而产生更好更合理的描边效果。

## 法线平滑
视频中原本采用的平滑法线的方法是利用 Blender 脚本平滑法线，并借用顶点色存储切线空间下平滑后法线的信息，然后使用 `_OUTLINE_VERTEX_COLOR_SMOOTH_NORMAL` 参数去控制相关代码。但是后来 up 主又在 b 站上发布了一个更方便的方法，直接使用 Unity 脚本。脚本代码如下：  

``` C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class SmoothNormals : MonoBehaviour
{
    public struct NormalWeight {
        public Vector3 normal;
        public float weight;
    }

#if UNITY_EDITOR
    void OnValidate()
    {
        void SmoothNormals(Mesh mesh)
        {
            Dictionary<Vector3, List<NormalWeight>> normalDict = new Dictionary<Vector3, List<NormalWeight>>(); //存储顶点以及对应的法线和顶点权重，一个顶点对应多个法线和顶点权重
            var triangles = mesh.triangles;
            var vertices = mesh.vertices;
            var normals = mesh.normals;
            var tangents = mesh.tangents;
            var smoothNormals = mesh.normals;

            //该循环填充 normalDict 的数据
            for (int i = 0; i <= triangles.Length - 3; i += 3)
            {
                int[] triangle = new int[] {triangles[i], triangles[i+1], triangles[i+2]}; //三角形，包含三个顶点索引
                for (int j = 0; j < 3; j++) //遍历一个三角形的三个顶点
                {
                    int vertexIndex = triangle[j];
                    Vector3 vertex = vertices[vertexIndex]; //根据顶点索引获取顶点坐标
                    if (!normalDict.ContainsKey(vertex)) //避免重复的顶点
                    {
                        normalDict.Add(vertex, new List<NormalWeight>());
                    }

                    NormalWeight nw;
                    Vector3 lineA = Vector3.zero;
                    Vector3 lineB = Vector3.zero;
                    if (j == 0)
                    {
                        lineA = vertices[triangle[1]] - vertex;
                        lineB = vertices[triangle[2]] - vertex;
                    }
                    else if (j == 1)
                    {
                        lineA = vertices[triangle[2]] - vertex;
                        lineB = vertices[triangle[0]] - vertex;
                    }
                    else
                    {
                        lineA = vertices[triangle[0]] - vertex;
                        lineB = vertices[triangle[1]] - vertex;
                    }
                    lineA *= 10000.0f; //计算精度问题
                    lineB *= 10000.0f;
                    float angle = Mathf.Acos(Mathf.Max(Mathf.Min(Vector3.Dot(lineA, lineB)/(lineA.magnitude * lineB.magnitude), 1), -1)); //利用点乘计算夹角
                    nw.normal = Vector3.Cross(lineA, lineB).normalized; //计算出真实法线
                    nw.weight = angle; //夹角即顶点权重
                    normalDict[vertex].Add(nw); //将真实法线、权重信息加入字典里的 List<NormalWeight>
                }
            }


            for (int i = 0; i < vertices.Length; i++) {
                Vector3 vertex = vertices[i];

                if (!normalDict.ContainsKey(vertex)) {
                    continue;
                } //防止一些不构成三角形的顶点出现

                List<NormalWeight> normalList = normalDict[vertex];

                Vector3 smoothNormal = Vector3.zero;
                float weightSum = 0;

                for (int j = 0; j < normalList.Count; j++)
                {
                    NormalWeight nw = normalList[j];
                    weightSum += nw.weight;
                }

                for (int j = 0; j < normalList.Count; j++)
                {
                    NormalWeight nw = normalList[j];
                    smoothNormal += nw.normal * nw.weight/weightSum; //根据夹角比例计算平滑后法线
                }

                smoothNormal = smoothNormal.normalized;
                smoothNormals[i] = smoothNormal; //在法线数据中存储平滑法线信息

                var normal = normals[i];
                var tangent = tangents[i];
                var binormal = (Vector3.Cross(normal, tangent) * tangent.w).normalized;
                var tbn = new Matrix4x4(tangent, binormal, normal, Vector3.zero);
                tbn = tbn.transpose;
                smoothNormals[i] = tbn.MultiplyVector(smoothNormals[i]).normalized; //模型空间转换到切线空间，默认统一缩放或无缩放
            }
            mesh.SetUVs(7, smoothNormals);
        }

        foreach (var item in GetComponentsInChildren<MeshFilter>())
        {
            SmoothNormals(item.sharedMesh);
        }
        foreach (var item in GetComponentsInChildren<SkinnedMeshRenderer>())
        {
            SmoothNormals(item.sharedMesh);
        }
    }
#endif
}
```

首先从 Unity 的 mesh 的数据结构开始讲起，一个 mesh 包含了以下属性：**顶点 Vertices**；**拓扑结构 Topology**；**索引结构 Indices**；**Blend shapes 混合形状、变形器**；**Bind poses 绑定姿势或 T-pose**：  
①Vertices 又包含了以下属性：Position、Normal、Tangent、Color、UVs、 blend indices and bone weights。  
&emsp;&emsp; - Position：模型空间的顶点坐标；  
&emsp;&emsp; - Normal：顶点法线；  
&emsp;&emsp; - Tangent：顶点切线，指向 uv 的 u 方向，w 值存储它的反向；  
&emsp;&emsp; - UVs：最多 8 套 uv，即 UV0 ~ UV7；  
&emsp;&emsp; - Color 也就是所谓的**顶点色**，独立于贴图存储的模型颜色，可选的属性，也可以用于存储其他信息；  
&emsp;&emsp; - Blend indices 存储那些骨骼影响一个顶点，bone weights 存储骨骼影响顶点的程度。  
②Topology 决定了多少个顶点决定了一个面，Unity 支持以下的 mesh 拓扑结构：Triangle、Quad、Lines 独立线段、LineStrip 不闭合的线、Points；  
③Indices：顶点的索引数组决定了一个面，数组内元素个数取决于 Topology。比如一个长方形的 Mesh 由四个顶点组成，这四个顶点的索引分别是 0，1，2，3。那么长方形的 triangles 数组就可以表示成 {0，1，2，2，3，1}，如果想拿到第一个三角形的三个顶点在 vertices 下的索引，就是 triangles[0]，triangles[1]，triangles[2]。顶点是可以复用的，同个顶点可以出现在不同的顶点数组。然后顶点的排列顺序决定了一个面的方向，即 **winding order**，顺时针排列是 front-facing，逆时针是 back-facing；  
④Blend shapes 存储 mesh 的不同形状变体，常用于脸部动画；  
⑤Bind poses 存储骨骼默认的位置坐标，游戏动画界常用 T-pose 作为默认姿势。

---

接下来就是讲解上面的平滑法线脚本了：  
①首先用一个字典来存储顶点以及其对应的法线和权重列表：`Dictionary<Vector3, List<NormalWeight>> normalDict = new Dictionary<Vector3, List<NormalWeight>>();` 为什么是列表，因为一个顶点可以有多条法线，看这个顶点被多少个面共用，一个面可以算出一个顶点法线以及一个顶点角度权重，故使用列表存储多个法线和顶点角度权重。

> 注意区分真实法线、拆分法线 Split Normals（面拐法线）、以及法线贴图的法线。拆分法线是为了模型平滑着色效果，而修改和自定义的法线，当然有些模型会不修改，此时拆分法线等于模型真实法线。而真实法线，一般建模软件都会自动计算，由顶点真实法线决定，顶点真实法线始终与面法线同向。我的理解是 Mesh 中的 Normal 记录的就是自定义的法线，即拆分法线，在建模软件都可以自定义修改，只不过默认情况下为模型的真实法线。若我们使用的 tbn 矩阵是基于拆分法线的切线空间，再去使用基于真实法线的法线贴图可能会出现渲染错误。

②第一个 for 循环的目的就是填充上述字典，记录 mesh 的所有的三角形的每个顶点的多个真实法线和权重信息。`mesh.triangles` 的结构就是 `{0, 2, 1, 2, 3, 1 };` 这样的，每三个定义一个三角形。所以获取每个三角形，再遍历它们的顶点。计算三角形每条线的向量，叉乘出顶点法线，并计算该顶点法线对应的角权重。

③第二个 for 循环目的就是根据每个顶点记录的角权重来加权平均出顶点的平滑法线，然后把平滑法线转换到模型拆分法线的切线空间。因为我们在 Shader 还需要将平滑法线用拆分法线的 TBN 矩阵转换回世界空间，所以不用在意使用的是拆分法线的切线空间还是真实法线的切线空间，只有来回转换的 TBN 矩阵统一就行。

---

回到我们的 Shader 上来，将脚本挂在布洛妮娅上来。因为我们改变了方案，先将 `_OUTLINE_VERTEX_COLOR_SMOOTH_NORMAL` 属性改为 `_OUTLINE_UV7_SMOOTH_NORMAL`，并使用世界坐标系的 TBN 矩阵将切线空间下的平滑法线转换至世界坐标，并使用平滑法线的方向偏移顶点来渲染描边：  

    #if _OUTLINE_UV7_SMOOTH_NORMAL
        float3x3 tbn = float3x3(vertexNormalInput.tangentWS, vertexNormalInput.bitangentWS, vertexNormalInput.normalWS);
        positionWS += mul(input.uv7.rgb, tbn) * width;
    #else
        positionWS += vertexNormalInput.normalWS * width;
    #endif

接下来在脸部、上衣、下衣、头发的材质中开启 `Use UV7 smooth normal` 选项，因为默认是关闭的，同时关闭牙舌口、眉毛、颜色的 `Use Outline` 选项，默认是开启状态。效果如下（我把描边宽度设置为了 2 ）：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/28/5Hun9jl862BXDTt.jpg" width = "100%" height = "100%" alt="图22 - 描边效果（黑）"/>
</div>

但是现在的描边，若拉近了看，描边会变得很粗，拉远了描边会变得很细。我们希望无论拉远拉近，描边的粗细都不会变化。视频 up 主采用的 github 上 ColinLeung-NiloCat 的方法，详见 https://github.com/ColinLeung-NiloCat/UnityURPToonLitShaderExample ，up 主复制了 github 里的 Outline 方法。该方法的效果并不是很好，因为它简单地乘上相机距离（深度），再乘上相机的 FOV，从而保证不同 FOV 下和深度下，描边宽度不变。我感觉乘上 Tan(FOV/2) 更为合理，因为远近的影响乘上 Tan(FOV/2) 才是屏幕平面上的影响，换个思路想也能得到类似的答案：  

因为我们希望描边宽度不产生近大远小，而近大远小产生的原因是透视投影以及透视投影后的齐次除法产生的，当然如果是正交投影摄像机只是个放大放小的作用。所以为了不产生描边宽度变化，我们需要去除掉透视投影矩阵以及齐次除法的影响，也就是把描边宽度还原至平行投影的状态，这样就不会近大远小。而齐次除法除去的正是深度信息（相机距离），所以我们需要乘回去。接下来我们回忆透视投影矩阵：  

$$ M_{ortho} = \begin{bmatrix} \cfrac{cot \cfrac {FOV}{2}}{Aspect} & 0 & 0 & 0 \\ 0 & cot \cfrac {FOV}{2} & 0 & 0 \\ 0 & 0 & -\cfrac {2}{Far - Near} & -\cfrac {Far + Near}{Far - Near} \\ 0 & 0 & 0 & 1 \end{bmatrix} $$

很好我们找到了 Tan(FOV/2)，Cot(FOV/2) 是 Tan 的倒数，透视投影除以了 Tan(FOV/2)，所以我们乘回去，这很合理。Aspect 的影响我们并不需要考虑，因为从裁切空间转换至屏幕坐标时，Unity 已经考虑了 Aspect 的影响。而透视投影矩阵对 z 轴的影响我们不需要考虑，裁切空间下的 z 轴只是一个非线性的深度，我们其实已经在齐次除法考虑了深度影响。我修改后的代码如下（加 saturate 是为了太远的时候，描边相对于于模型太大了，一坨黑色很难看。也就是拉近，模型小了，给描边宽度缩小；拉太远了，不给描边宽度放大）：  

``` C
    #if _OUTLINE_UV7_SMOOTH_NORMAL
        float cotHalfFOV = unity_CameraProjection._m11;
        float3x3 tbn = float3x3(vertexNormalInput.tangentWS, vertexNormalInput.bitangentWS, vertexNormalInput.normalWS);
        positionWS += mul(input.uv7.rgb, tbn) * width * saturate(abs(vertexPositionInput.positionCS.w) * (1.0f / cotHalfFOV));
    #else
        positionWS += vertexNormalInput.normalWS * width;
    #endif
```

## 描边颜色
