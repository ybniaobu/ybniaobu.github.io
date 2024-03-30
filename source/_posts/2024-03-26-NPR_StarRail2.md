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
float cotHalfFOV = unity_CameraProjection._m11;
float widthFactor = saturate(abs(vertexPositionInput.positionCS.w) * (1.0f / cotHalfFOV));
#if _OUTLINE_UV7_SMOOTH_NORMAL
    float3x3 tbn = float3x3(vertexNormalInput.tangentWS, vertexNormalInput.bitangentWS, vertexNormalInput.normalWS);
    positionWS += mul(input.uv7.rgb, tbn) * width * widthFactor;
#else
    positionWS += vertexNormalInput.normalWS * width * widthFactor;
#endif
```

这段代码也可以处理摄像机为正交投影的情况，因为正交投影矩阵的 _m11 是 Scale 的倒数，也是需要乘回去。

但是这样虽然不会近大远小了，但是镜头离模型太近了，描边又会感觉太小（相对于模型感觉太小，虽然宽度没变化），效果感觉真的一般。于是我登录游戏看了一下，发现米哈游的描边是有近大远小的，但是它变化的速率感觉没有最开始我们的近大远小的变化速率快，那么这个变化速率是由什么控制的？也是由 FOV 控制的，也就是说 FOV 控制的是近大远小的变化速率，当 FOV 为 0，tan(FOV/2) 也为 0，此时没有变化速率，不产生近大远小，也就变为了平行透视。FOV 越大，变化越快，所以我们需要改变描边宽度的 FOV 值。为了产生近大远小，我们还是要齐次除法的，所以不乘回深度值了，我们只改变 FOV 值，就取原 FOV 的一半：  

    float cotHalfFOV = unity_CameraProjection._m11;
    float HalfFOV = atan(1.0f / cotHalfFOV);
    float widthFactor = (1.0f / cotHalfFOV) / (tan(HalfFOV / 2));

这样的效果还是和米哈游的有点不一样，但是我尽力了，感觉还是缩小了近距离原本变大的宽度。莫非只是改变了参数的影响程度，还是改了回去，最终代码如下，别问我为什么，问就是感觉最舒服：  

    float cotHalfFOV = unity_CameraProjection._m11;
    float widthFactor = clamp(0, 2, (abs(vertexPositionInput.positionCS.w) * (1.0f / cotHalfFOV) + 1) / 4);

这样子其实效果已经不错了，但是我睡了一觉后突然想到，之前去除了透视投影矩阵以及齐次除法的代码的效果，是我们感觉拉近的时候描边变小得太快（相对于近大远小效果），但是拉远了又嫌描边变大得太快，我突然就想到了平方根函数，相对于 y = x，刚好是小于 1 时，变小的慢了，大于 1 时，变大得慢了，我悟了，修改代码如下：  

    float cotHalfFOV = unity_CameraProjection._m11;
    float widthFactor = pow(abs(vertexPositionInput.positionCS.w) * (1.0f / cotHalfFOV), 0.5f);

这个最终得描边效果我很满意，可以尝试改变距离和相机 FOV 看看效果，应该能满足需求了。

## 描边颜色
我们先把顶点着色器的内容的输出结构补充完整，新增代码如下：

    output.uv = TRANSFORM_TEX(input.uv, _BaseMap);
    output.fogFactor = ComputeFogFactor(vertexPositionInput.positionCS.z);

描边颜色使用 Ramp 贴图的颜色，同时对冷暖贴图采样出来的颜色进行混合，并使用 `_OutlineGamma` 参数对颜色调整（默认值为 16），最后加上 fogFactor 的影响，如下：  

``` C
float3 coolRamp = 0;
float3 warmRamp = 0;

#if _AREA_HAIR
{
    float2 outlineUV = float2(0, 0.5); //选择了一个颜色
    coolRamp = SAMPLE_TEXTURE2D(_HairCoolRamp, sampler_HairCoolRamp, outlineUV).rgb;
    warmRamp = SAMPLE_TEXTURE2D(_HairWarmRamp, sampler_HairWarmRamp, outlineUV).rgb;
}
#elif _AREA_UPPERBODY || _AREA_LOWERBODY
{
    float4 lightMap = 0;
    #if _AREA_UPPERBODY
        lightMap = SAMPLE_TEXTURE2D(_UpperBodyLightMap, sampler_UpperBodyLightMap, input.uv);
    #elif _AREA_LOWERBODY
        lightMap = SAMPLE_TEXTURE2D(_LowerBodyLightMap, sampler_LowerBodyLightMap, input.uv);
    #endif
    
    float materialEnum = lightMap.a;
    //float materialEnumOffset = materialEnum + 0.0425;
    //float outlineUVy = lerp(materialEnumOffset, materialEnumOffset + 0.5 > 1 ? materialEnumOffset + 0.5 - 1 : materialEnumOffset + 0.5, fmod((round(materialEnumOffset/0.0625) - 1)/2, 2));
    int rawIndex = (round((lightMap.a + 0.0425)/0.0625) - 1)/2;
    int rampRowIndex = lerp(rawIndex, rawIndex + 4 < 8 ? rawIndex + 4 : rawIndex + 4 - 8, fmod(rawIndex, 2));
    float outlineUVy = (2 * rampRowIndex + 1) * (1.0 / (8 * 2));
        
    float2 outlineUV = float2(0, outlineUVy);
    coolRamp = SAMPLE_TEXTURE2D(_BodyCoolRamp, sampler_BodyCoolRamp, outlineUV).rgb;
    warmRamp = SAMPLE_TEXTURE2D(_BodyWarmRamp, sampler_BodyWarmRamp, outlineUV).rgb;
}
#elif _AREA_FACE
{
    float2 outlineUV = float2(0, 0.0625); //选择了一个颜色
    coolRamp = SAMPLE_TEXTURE2D(_BodyCoolRamp, sampler_BodyCoolRamp, outlineUV).rgb;
    warmRamp = SAMPLE_TEXTURE2D(_BodyWarmRamp, sampler_BodyWarmRamp, outlineUV).rgb;
}
#endif

float3 ramp = lerp(coolRamp, warmRamp, 0.5);
float3 albedo = pow(saturate(ramp), _OutlineGamma);
    
float4 color = float4(albedo, 1);
color.rgb = MixFog(color.rgb, input.fogFactor);
return color;
```

头发和脸都是直接在 Ramp 图上找了一个颜色采样出来。而身体部分和之前渐变阴影部分是一样的，可以回去看看。不知道为什么，我这里的效果还是和视频 up 主有点不一样，我稍微改了改采样的点位和 `_OutlineGamma` 参数值，代码见最后面全部代码小节，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/29/nmxX6FV2CgwRotS.jpg" width = "100%" height = "100%" alt="图23 - 描边效果（渐变）"/>
</div>

为了更好的效果，可以不对 ramp 采样，可以设置多个颜色参数，根据 lightMap.a 去使用不同的颜色参数。这里就不处理了。

## 鼻子描边
鼻子描边在 FaceMap 的 B 通道上，我们要回到之前的 ToonForwardPass 去。我们希望鼻子描边效果越直视越明显，越斜视越不明显，所以还是使用头前向量和视角方向进行点乘，给点乘指数幂降低可视范围。使用 smoothstep() 函数来抬高最后颜色 lerp() 的参数。颜色使用 Ramp 贴图的一个点。代码如下：

```
float fakeOutlineEffect = 0;
float3 fakeOutlineColor = 0;

#if _AREA_FACE && _OUTLINE_ON
{
    float fakeOutline = faceMap.b;
    float3 headForward = normalize(_HeadForward);
    fakeOutlineEffect = smoothstep(0.0, 0.25, pow(saturate(dot(headForward, viewDirectionWS)), 20) * fakeOutline);

    float2 outlineUV = float2(0 , 0.0625);
    float3 coolRamp = SAMPLE_TEXTURE2D(_BodyCoolRamp, sampler_BodyCoolRamp, outlineUV).rgb;
    float3 warmRamp = SAMPLE_TEXTURE2D(_BodyWarmRamp, sampler_BodyWarmRamp, outlineUV).rgb;
    float3 ramp = lerp(coolRamp, warmRamp, 0.5);
    fakeOutlineColor = pow(ramp, _OutlineGamma);
}
#endif

...
albedo = lerp(albedo, fakeOutlineColor, fakeOutlineEffect);
```

这样就有鼻子正面时候的描边效果了，不展示图片了。

# 边缘光
边缘光要先把 URP 的 Depth Texture 打开，要使用到深度图。视频中使用的方法是屏幕空间深度边缘光，即取当前像素屏幕 uv，向该点法线方向偏移后采样深度，与当前像素深度进行比较，深度差大于某一阈值则为边缘。视频 up 主的代码如下：

``` C
float linearEyeDepth = LinearEyeDepth(input.positionCS.z, _ZBufferParams);
float3 normalVS = mul((float3x3)UNITY_MATRIX_V, normalWS);
float2 uvOffset = float2(sign(normalVS.x), 0) * _RimLightWidth / (1 + linearEyeDepth) / 100;
int2 loadTexPos = input.positionCS.xy + uvOffset * _ScaledScreenParams.xy;

loadTexPos = min(max(loadTexPos, 0), _ScaledScreenParams.xy - 1); //防止采样出界
        
float offsetScreenDepth = LoadSceneDepth(loadTexPos); //函数在 DeclareDepthTexture.hlsl 里
float offsetLinearEyeDepth = LinearEyeDepth(offsetScreenDepth, _ZBufferParams);
float rimLight = saturate(offsetLinearEyeDepth - (linearEyeDepth + _RimLightThreshold)) / _RimLightFadeout;
float3 rimLightColor = rimLight * mainLight.color.rgb;
rimLightColor *= _RimLightTintColor;
rimLightColor *= _RimLightBrightness;
...
albedo += rimLightColor * lerp(1, albedo, _RimLightMixAlbedo);
```

一开始我还以为视频 up 主的代码有问题，因为第一句代码：`float linearEyeDepth = LinearEyeDepth(input.positionCS.z, _ZBufferParams);` 。我一直记得 `LinearEyeDepth()` 函数应该传递进去的是范围在 (0 - 1) 的 NDC 坐标，而上面传递进去的是裁切空间下的 z 坐标，未被齐次除法，也未被重映射至 (0 - 1)。但是经过高强度的网上冲浪之后，我发现一个惊人的事实，那就是 positionCS.z 存储的就是范围在 (0 - 1) 的 NDC 坐标，准确的说是带 SV_POSITION 语义的 positionCS.z。也就是说带 SV_POSITION 语义的 positionCS 和不带的 positionCS 并不一样，这是源于 **SV_POSITION 语义**的特殊性而导致的。

> SV_POSITION 虽然是 DirectX 平台的 HLSL 语义，但不代表 Unity editor 内部计算风格习惯也是基于 DirectX。Unity 大部分内置函数还是基于 OpenGL 的，比如视角空间是右手坐标系（在 URP 源码我们也可以看到这一点），而 DirectX 的视角空间是左手坐标系。除了因 SV_POSITION 语义的特殊性而产生影响的相关函数，Unity 会考虑到这些影响，所以我们要注意区分。之前的《Unity Shader入门精要》在讲解数学逻辑时，也都是基于 OpenGL 的风格习惯来讲解的。

接下来我们先补充一点细节差异上的基础知识，再略微修改视频 up 主的代码为了更好的理解整个边缘光效果。

---

首先对几个名词进行说明：  
①**NDC 坐标**：Normalized Device Coordinates，即裁切空间齐次除法后的结果。区分为 z 的范围在 [-1, 1] 的 NDC 坐标，即 OpenGL 的风格；z 的范围在 [0, 1] 的 NDC 坐标，DirectX 风格；以及 Unity URP 中 ShaderVariablesFunctions.hlsl 的 `GetVertexPositionInputs()`函数获取到的 positionNDC，详见下面。（无论 OpenGL 还是 DirectX，NDC.xy 范围都是 [-1, 1]）。
②**屏幕坐标**：Sceen Space 或者说屏幕 uv。区分为归一化的屏幕坐标，也就是将 NDC.xy 重映射至范围在 [0, 1] 的坐标，在《Unity Shader入门精要》也称为视口坐标 viewport space；未归一化的屏幕坐标，也称为像素坐标，就是归一化的屏幕坐标乘上屏幕分辨率。

**SV_POSITION** 语义说明：  
在顶点着色器中，我们使用 SV_POSITION 作为裁剪空间坐标的输出值。但是在片元着色器中，SV_POSITION 的含义会发生改变：  
①**SV_POSITION.xy** 坐标表示当前像素在屏幕空间下的像素坐标加上 0.5 的位置偏移（因为一个像素点为 1 的话，其像素中心点为 0.5）。  
②**SV_POSITION.z** 的值等于 NDC Space 下的 z 值，也就是非线性深度值，因为是在 DirectX 中，其范围在 [0, 1]。这和 OpenGL 的 NDC 坐标的 z 的范围不同，OpenGL 范围在 [-1, 1]。SV_POSITION.z 也是写入 Depth buffer 的值，但有些平台会使用 reverse Z buffer（范围变为 [1, 0]），目的是为了防止 z-fight，有需要去额外了解。Depth buffer 的值其实就是写入 Depth Texture 的值。  
③**SV_POSITION.w** 的值，就是经过插值后的裁切空间的 w 值，等价于视角坐标下的 z 坐标值（注意负号），也就是未归一化的线性深度值。

> DirectX 的透视矩阵变换后，裁剪空间中 z 的范围是 [0, w] 而不是 [-w, w]。所以透视除法后得到的 NDC 坐标的 z 的范围是 [0, 1]。
>  
> OpenGL 类似于顶点着色器的 SV_POSTION 的语义为 gl_FragCoord。但 gl_FragCoord.w 的值，等于裁切空间的 w 值的倒数。不过在 Unity 中，为我们做了自动的转换，如果我们在顶点着色器中使用 SV_POSTION.w 值的时候，在 OpenGL 下，会自动转换成 1/gl_FragCoord.w，非常方便。

所以为了获取 screenUV，以便对深度纹理进行采样，一共有三种方式可以获取：  
①使用不带 SV_POSITION 语义的 positionCS（使用 TEXCOORDn 语义）：**`float2 screenUV = (input.positionCS.xy / input.positionCS.w) * 0.5 + 0.5;`**  
②使用带 SV_POSITION 语义的 positionCS：**`float2 screenUV = input.positionCS.xy / _ScaledScreenParams.xy;`**。_ScaledScreenParams.xy 就是当前的屏幕分辨率；（注：官方的示例中使用的代码就是这样，但是不知道为什么没考虑上面说的 0.5 的偏移）  
③使用 URP 内置的 `GetVertexPositionInputs()` 函数获取到的 positionNDC.xy：**`float2 screenUV = i.positionNDC.xy / i.positionNDC.w;`**。好了疑问又来了，为什么这样可以得到 screenUV。因为此 NDC 非彼 NDC，它是 Homogeneous Normalized Device Coordinates，并不是严格意义上的 NDC 坐标。`GetVertexPositionInputs()` 这个函数的逻辑和之前内置管线的 UnityCG.cginc 的 `ComputeScreenPos()` 函数是一样的，忘了可以去《Unity Shader入门精要》看看。也就是未经过齐次除法，但是做了重映射。总之 **positionNDC.x** 为 $\,clip_x / 2 + clip_w / 2\,$，**positionNDC.y** 为 $\,clip_y / 2 + clip_w / 2\,$，**positionNDC.z** 为 $\,clip_z\,$，**positionNDC.w** 为 $\,clip_w\,$。所以 **positionNDC.xy/positionNDC.w** 为 **(positionCS.xy / positionCS.w) * 0.5 + 0.5**，即屏幕坐标。（注，positionNDC 在内置函数中是通过 positionWS 转换到裁切空间后的 positionCS 获得的，所以不用关心 SV_POSITION 语义的影响）

---

回到我们的 Toon Shader，我就按照我略微改动后的代码来讲解，首先获取当前像素的深度：  

    //float linearEyeDepth = LinearEyeDepth(input.positionCS.z, _ZBufferParams);
    //float linearEyeDepth = input.positionCS.w;
    float2 screenUV = input.positionCS.xy / _ScaledScreenParams.xy;
    float depth = SampleSceneDepth(screenUV);
    float linearEyeDepth = LinearEyeDepth(depth, _ZBufferParams);

上面两个注释掉的方法也能获取到相同的线性深度值，第一个方法就是视频中的代码。然后就是对 screenUV 进行偏移，视频中的方法是对视角空间下的法线做个偏移的方向判断，然后只对 uv 的 u 方向上进行偏移，我选择了同时对 u 和 v 方向进行偏移，代码如下：  

    float3 normalVS = mul((float3x3)UNITY_MATRIX_V, normalWS);
    float2 uvOffset = sign(normalVS.xy) / _ScaledScreenParams.xy * _RimLightWidth * pow((1.0f / linearEyeDepth) * unity_CameraProjection._m11, 0.5f);
    float2 OffsetScreenUV = screenUV + uvOffset;

因为 uvOffset 其实也控制着边缘光的宽度，所以我们乘上了参数 `_RimLightWidth` 方便控制宽度，因为 `_RimLightWidth` 的默认值为 1，而 uv 范围是 [0, 1] 的，所以要除以 `_ScaledScreenParams.xy` 把像素宽度变为归一化后的屏幕坐标的宽度。至于为什么乘上 `pow((1.0f / linearEyeDepth) * unity_CameraProjection._m11, 0.5f);`？这个公式很眼熟，没错，就是我在描边中使用的方法，加上是为了产生近大远小，因为模型离屏幕近，我们希望边缘光宽度变大，反之亦然，至于开平方是因为不希望变得太大。若不乘上无论模型远近，采样的宽度不变，当很近时，采样的宽度相对于模型会变很小，不信可以试试。

接下来就是使用 OffsetScreenUV 对深度纹理进行采样，我使用的是 `SampleSceneDepth()` 函数，接受的是范围 [0, 1] 的 uv 坐标。而视频中采用的是 `LoadSceneDepth()` 函数，它接受的是像素的像素坐标，所以会有些许区别。我的代码如下：  

    float offsetDepth = SampleSceneDepth(OffsetScreenUV);
    float offsetLinearEyeDepth = LinearEyeDepth(offsetDepth, _ZBufferParams);

视频中还遇到了采样出界的问题，但是我没遇到，不知道是不是 `SampleSceneDepth()` 和 `LoadSceneDepth()` 的区别，所以我没那行代码。然后就是根据 offsetLinearEyeDepth 和 LinearEyeDepth 的差值判断出边缘光了，可以使用 step() 函数大于某个阈值才产生边缘光，这里简单点，就使用视频中使用的 saturate() 函数，并使用 `_RimLightThreshold` 参数控制偏移值，默认为 0.05，该值调整为负数可以增大边缘光范围。除以 `_RimLightFadeout` 可以减弱边缘光，默认值为 1，不知道为什么范围是 Range(0.01, 1)，建议修改一下。

    float rimLight = saturate(offsetLinearEyeDepth - (linearEyeDepth + _RimLightThreshold)) / _RimLightFadeout;

最后乘上主光颜色，`_RimLightTintColor` 颜色控制参数，默认白色，以及 `_RimLightBrightness` 亮度控制参数，默认为 1。

    float3 rimLightColor = rimLight * mainLight.color.rgb;
    rimLightColor *= _RimLightTintColor;
    rimLightColor *= _RimLightBrightness;
    
此时直接输出边缘光，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/30/sHtSPnZNL5qpCWm.jpg" width = "100%" height = "100%" alt="图25 - 边缘光"/>
</div>

这样效果只能算一般把，视频基本上就到这里了。我使用菲涅耳边缘光混合了一下：


_RimLightMixAlbedo 默认 0.9。