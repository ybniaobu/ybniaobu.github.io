---
title: 《Unity Shader入门精要》读书笔记（四）
date: 2023-12-19 19:35:53
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
top_img: /images/black.jpg
cover: https://s2.loli.net/2023/12/20/9Ah5ugiIpONK1cX.gif
mathjax: true
---

> 本读书笔记为高级篇，主要内容为XXXXXXXXXXX。  
> 读书笔记是对知识的记录与总结，但是对比较熟悉的内容不会再行描述。

# 第十一章 屏幕后处理效果
## 建立一个基本的屏幕后处理脚本系统
**屏幕后处理效果 screen post-processing effects** 通常指在渲染完整个场景得到屏幕图像后，再对图像进行一系列操作，实现各种屏幕特效。使用这种技术，可以为游戏画面添加更多的艺术效果，例如**景深 Depth of Field**、**运动模糊 Motion Blur** 等。

因此，想要实现屏幕后处理首先要得到渲染后的屏幕图像，即抓取屏幕，Unity 为我们提供了一个方便的接口 `OnRenderImage` 函数，声明如下：  

    MonoBehaviour.OnRenderImage(RenderTexture src, RenderTexture dest)

Unity 会把当前渲染得到的图像存储在第一个参数对应的源渲染纹理中，通过函数中的一系列操作后，再把目标渲染纹理，即第二个参数对应的渲染纹理显示到屏幕上。在 OnRenderImage 函数中，通常利用 `Graphics.Blit` 函数来完成对渲染纹理的处理。它有 3 中函数声明：  

    public static void Blit(Texture src, RenderTexture dest);
    public static void Blit(Texture src, RenderTexture dest, Material mat, int pass = -1);
    public static void Blit(Texture src, Material mat, int pass = -1);

参数 src 即源纹理，参数 dest 为目标纹理，若 dest 为 null 就会直接将结果显示在屏幕上。参数 mat 是我们使用的材质，在这个材质使用的 Unity Shader 里进行各种屏幕后处理操作，而 src 纹理将会被传递给 Shader 中名为 \_MainTex 的纹理属性。参数 pass 的默认值为 -1，表示将会依次调用 Shader 内的所有 Pass。否则，只会调用给定索引的 Pass。

---

在默认情况下，OnRenderImage 函数会在所有不透明和透明的 Pass 执行完毕后被调用，以便对场景中所有游戏对象都产生影响。但有时，会希望在不透明的 Pass（RenderQueue <= 2500，内置的 Background、Geometry 和 AlphaTest 渲染队列均在此范围内）执行完毕后立即调用 OnRenderImage 函数，从而不对透明物体产生任何影响。可在 OnRenderImage 函数前添加 ImageEffectOpaque 属性实现这样的目的（见第 12 章，利用深度和法线纹理进行边缘检测从而实现描边效果，但是不希望透明物体也被描边）。

---

因此，要在 Unity 中实现屏幕后处理效果，过程通常如下：  
①在摄像机中添加一个用于屏幕后处理的脚本，用于实现 OnRenderImage 函数；  
②调用 Graphics.Blit 函数使用特定的 Unity Shader 对当前图像进行处理，再把返回的渲染纹理显示到屏幕上。对于一些复杂的屏幕特效可多次调用 Graphics.Blit 函数对上一步的输出结果做进一步处理。  

但是，在进行屏幕后处理之前，需要检查一系列条件是否满足，例如当前平台是否支持渲染纹理和屏幕特效，是否支持当前使用的 Unity Shader 等。为此，我们创建了一个用于屏幕后处理效果的基类，在实现各种屏幕特效时，我们只需要继承自该基类，再实现派生类中不同的操作即可。代码如下：  

``` C#
using UnityEngine;
using System.Collections;

[ExecuteInEditMode]
[RequireComponent (typeof(Camera))] //屏幕效果需要绑定在某个摄像机中
public class PostEffectsBase : MonoBehaviour {

    //为了提前检查各种资源和条件是否满足，在 Start 函数调用 CheckResources 函数
    protected void CheckResources() {
        bool isSupported = CheckSupport();
        
        if (isSupported == false) {
            NotSupported();
        }
    }

    protected bool CheckSupport() {
        if (SystemInfo.supportsImageEffects == false || SystemInfo.supportsRenderTextures == false) {
            Debug.LogWarning("This platform does not support image effects or render textures.");
            return false;
        }
        return true;
    }

    protected void NotSupported() {
        enabled = false;
    }
    
    protected void Start() {
        CheckResources();
    }

    //因为每一个屏幕处理效果通常都需要制定一个 shader 来创建一个用于处理渲染纹理的材质，从而基类中也提供了该方法。该函数首先检查 Shader 的可用性，检查通过后就返回一个使用了该 shader 的材质
    protected Material CheckShaderAndCreateMaterial(Shader shader, Material material) {
        if (shader == null) {
            return null;
        }
        
        if (shader.isSupported && material && material.shader == shader)
            return material;
        
        if (!shader.isSupported) {
            return null;
        }
        else {
            material = new Material(shader);
            material.hideFlags = HideFlags.DontSave;
            if (material)
                return material;
            else 
                return null;
        }
    }
}
```

## 调整屏幕的亮度、饱和度和对比度
准备工作如下：  
①新建名为 Scene_12_2 的场景，并去掉天空盒子；  
②导入一张图片（案例图路径：Assets/Textures/Chapter12/Sakura0.jpg），调整图片纹理类型为 Sprite (2D and UI)，并拖拽到场景中，使其生成一个 Sprite；  
③新建名为 BrightnessSaturationAndContrast 的脚本，并拖拽到相机上；  
④新建名为 Chapter12-BrightnessSaturationAndContrast 的 Unity Shader；

BrightnessSaturationAndContrast 脚本代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class BrightnessSaturationAndContrast : PostEffectsBase { //继承 PostEffectsBase 基类

    public Shader briSatConShader; //声明该效果需要的 shader
    private Material briSatConMaterial; //并创建相应的材质
    public Material material {  
        get {
            briSatConMaterial = CheckShaderAndCreateMaterial(briSatConShader, briSatConMaterial);
            return briSatConMaterial;
        }  
    }

    //提供调整亮度、饱和度和对比度的参数
    [Range(0.0f, 3.0f)]
    public float brightness = 1.0f;

    [Range(0.0f, 3.0f)]
    public float saturation = 1.0f;

    [Range(0.0f, 3.0f)]
    public float contrast = 1.0f;

    void OnRenderImage(RenderTexture src, RenderTexture dest) {
        if (material != null) { //如果材质不为空，则把参数传递给材质，若不可用，则直接把源图像显示到屏幕上，不做任何处理
            material.SetFloat("_Brightness", brightness);
            material.SetFloat("_Saturation", saturation);
            material.SetFloat("_Contrast", contrast);

            Graphics.Blit(src, dest, material);
        }
        else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter12-BrightnessSaturationAndContrast 的 Unity Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 12/Brightness Saturation And Contrast" {
    Properties {
        //Graphics.Blit 把第一个参数传递给名为 _MainTex 的属性，所以必需声明 _MainTex
        _MainTex ("Base (RGB)", 2D) = "white" {} 
        //下面三个属性的值由脚本传递而得，事实上，这些属性声明可以省略，Properties 中声明属性仅仅是为了显示在材质面板里，但对于屏幕特效来说，使用的材质是临时创建的，不需要在材质面板上调整参数
        _Brightness ("Brightness", Float) = 1
        _Saturation("Saturation", Float) = 1
        _Contrast("Contrast", Float) = 1
    }
    
    SubShader {
        Pass {
            //屏幕后处理实际上是在场景中绘制了一个与屏幕同宽同高的四边形面片，为了防止它对物体产生影响，需要设置相关的渲染状态。关闭深度写入为了防止挡住其他后面被渲染的物体。如果当前 OnRenderImage 函数在所有不透明的 Pass 执行完毕后立刻被调用，不关闭深度写入就会影响后面透明的 Pass 渲染，该设置可以看作是用于屏幕后处理的 shader 的标配
            ZTest Always Cull Off ZWrite Off
            
            CGPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
              
            #include "UnityCG.cginc"  
             
            sampler2D _MainTex;  
            half _Brightness;
            half _Saturation;
            half _Contrast;
              
            struct v2f {
                float4 pos : SV_POSITION;
                half2 uv: TEXCOORD0;
            };

            //使用 Unity 内置的 appdata_img 结构体作为顶点着色器的输入，其只包含图像处理时必须的顶点坐标和纹理坐标等变量。屏幕特效使用的顶点着色器通常比较简单，只需要进行必要的顶点变换，把正确的纹理坐标传递给片元着色器，以便对屏幕图像进行正确的采样
            v2f vert(appdata_img v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = v.texcoord;
                return o;
            }
        
            fixed4 frag(v2f i) : SV_Target {
                fixed4 renderTex = tex2D(_MainTex, i.uv); //原屏幕图像采样
                
                fixed3 finalColor = renderTex.rgb * _Brightness; //利用 _Brightness 来调整亮度，亮度调整只需要把原颜色乘以亮度系数即可

                //luminance 亮度值：通过对每个颜色分量乘以一个特定的系数再相加即可得到。我们使用该亮度值创建一个饱和度为 0 的颜色值，并使用 _Saturation 和上一步得到的颜色值进行插值从而得到希望的饱和度颜色
                fixed luminance = 0.2125 * renderTex.r + 0.7154 * renderTex.g + 0.0721 * renderTex.b;
                fixed3 luminanceColor = fixed3(luminance, luminance, luminance);
                finalColor = lerp(luminanceColor, finalColor, _Saturation);
                
                //对比度：首先创建对比度为 0 的颜色值（各分量均为 0.5），使用 _Contrast 和上一步得到的颜色值进行插值从而得到最终的处理结果
                fixed3 avgColor = fixed3(0.5, 0.5, 0.5);
                finalColor = lerp(avgColor, finalColor, _Contrast);
                
                return fixed4(finalColor, renderTex.a);  
            } 
            ENDCG
        }  
    }
    Fallback Off
}
```

> 补充：  
> ①亮度：很简单，用 \_Brightness 对 RGB 进行统一缩放，但各自混合比例不变，保证了颜色不变，但亮度变大。  
> ②饱和度：饱和度指色彩的纯净程度，在同一色相中添加白色、黑色或灰色会降低纯度。因为有 Cg 的 Lerp 函数的存在，只要得到饱和度为 0 时的 RGB 值，当 \_Saturation > 1 时，就可以增大颜色饱和度。由上面的饱和度概念可推，往一种颜色添加大量黑白灰色，即该颜色的灰度图就是该颜色的饱和度为 0 时的 RGB 值。因此当一个画面饱和度为 0 时，得到的是该画面对应的灰度图。  
> ③上述代码的 luminance 值的计算公式就是求 RGB 颜色值对应的灰度值，这个公式是 RGB 转 YUV 的 BT709 明亮度转换公式，YUV 颜色模型起源于解决彩色电视和黑白电视的兼容性问题，其中 Y 值即 Luminance 就是灰阶值。至于三个系数不统一，是因为人眼对 RGB 三原色的敏感程度不同，有兴趣了解色度学（将主观颜色感知和客观物理测量联系的科学）。  
> ④对比度：对比度指一个画面显示的 RGB 值最大的像素和最小的像素之间的差值的大小，即最亮与最暗的像素之间的差值为对比度。使用（0.5, 0.5, 0.5）对原 RGB 值进行插值。所以 \_Contrast 值越小，画面中像素的 RGB 的值都会越来越接近 0.5，对比度就会越小；值越大，特别是 > 1 时，画面中像素的 RGB 的值都会越来越远离 0.5，对比度就会越大。

将编写好的 Shader 文件拖拽到摄像机 C# 脚本组件的 Bri Sat Con Shader 属性中，调整参数，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/20/mytDJOCMLgZro56.png" width = "70%" height = "70%" alt="图62-  左图：原效果。右图：调整了亮度（值为 1.2）、饱和度（值为 1.5）和对比度（值为 1.1）后的效果"/>
</div>

## 边缘检测
边缘检测是描边效果的一种实现方法，其原理是利用一些边缘检测算子对图像进行**卷积 convolution** 操作。

### 什么是卷积
在图像处理中，卷积操作指的是使用一个**卷积核 kernel** 对一张图像中的每个像素进行一系列操作。卷积核通常是一个四方形网格结构（例如：2x2、3x3的方形区域），该区域内每个方格都有一个权重值。当对图形中的某个像素进行卷积时，我们会把卷积核的中心放置到该像素上，如下图所示，翻转核之后再依次计算核中每个元素和其覆盖的图像像素值的乘积并求和，得到的结果就是该位置的新像素值。

<div  align="center">  
<img src="https://s2.loli.net/2023/12/20/3t2MPVUadi5zWno.png" width = "70%" height = "70%" alt="图63-  卷积核与卷积。使用一个3×3大小的卷积核对一张5×5大小的图像进行卷积操作，当计算图中红色方块对应的像素的卷积结果时，我们首先把卷积核的中心放置在该像素位置，翻转核之后再依次计算核中每个元素和其覆盖的图像像素值的乘积并求和，得到新的像素值"/>
</div>

卷积操作可用于图像模糊、边缘检测、锐化等效果，例如：均值模糊，可以使用 3x3 的卷积核，核内每个元素的值均为 1/9。

### 常见的边缘检测算子
卷积操作的神奇之处在于选择的卷积核。用于边缘检测的卷积核，也被称为**边缘检测算子**。

如果相邻像素之间存在差别明显的颜色、亮度、纹理等属性，可以认为他们之间有一条边界，这种相邻像素之间的差异用**梯度 gradient** 来表示，边缘处的梯度绝对值会比较大。基于这样的理解，有几种不同的边缘检测算子被先后提出：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/20/ZrSGUK6ya3WeFJ4.png" width = "70%" height = "70%" alt="图64-  三种常见的边缘检测算子"/>
</div>

上图的三种常见的边缘检测算子都包含了 2 个方向的卷积核，分别用于检测水平方向和竖直方向上的边缘信息。边缘检测时，需要对每个像素分别进行一次卷积计算，得到 2 个方向上的梯度值 $\,G_x\,$ 和 $\,G_y\,$，而整体的梯度可按如下公式计算得到：  

$$ G = \sqrt {G_x^2 + G_y^2}$$

考虑到性能，有时会用绝对值操作代替开根号操作：  

$$ G = \lvert G_x \rvert + \lvert G_y \rvert $$

得到梯度 G 后，可据此判断哪些像素对应了边缘（梯度值越大，越有可能是边缘点）。

### 实现
准备工作如下：  
①新建名为 Scene_12_3 的场景，并去掉天空盒子；  
②导入一张图片（案例图路径：Assets/Textures/Chapter12/Sakura0.jpg），调整图片纹理类型为Sprite (2D and UI)，并拖拽到场景中，使其生成一个 Sprite；  
③新建名为 EdgeDetection 的 C#脚本，并拖拽到相机上；  
④新建名为 Chapter12-EdgeDetection 的 Unity Shader。

首先是 C# 脚本文件，代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class EdgeDetection : PostEffectsBase {
    public Shader edgeDetectShader;
    private Material edgeDetectMaterial = null;
    public Material material {  
        get {
            edgeDetectMaterial = CheckShaderAndCreateMaterial(edgeDetectShader, edgeDetectMaterial);
            return edgeDetectMaterial;
        }  
    }

    //提供调整边缘性强度、描边颜色以及背景颜色的参数
    [Range(0.0f, 1.0f)]
    public float edgesOnly = 0.0f; //当其为 0 时，边缘将会叠加在原渲染图像上；当为 1 时，则只会显示边缘，不显示原渲染图像
    public Color edgeColor = Color.black;
    public Color backgroundColor = Color.white;

    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            material.SetFloat("_EdgeOnly", edgesOnly);
            material.SetColor("_EdgeColor", edgeColor);
            material.SetColor("_BackgroundColor", backgroundColor);

            Graphics.Blit(src, dest, material);
        } 
        else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter12-EdgeDetection 的 Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 12/Edge Detection" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _EdgeOnly ("Edge Only", Float) = 1.0
        _EdgeColor ("Edge Color", Color) = (0, 0, 0, 1)
        _BackgroundColor ("Background Color", Color) = (1, 1, 1, 1)
    }

    SubShader {
        Pass {  
            ZTest Always Cull Off ZWrite Off
            
            CGPROGRAM
            
            #include "UnityCG.cginc"
            
            #pragma vertex vert  
            #pragma fragment fragSobel
            
            sampler2D _MainTex;  
            uniform half4 _MainTex_TexelSize; //xxx_TexelSize 是 Unity 为开发者提供的访问 xxx 纹理对应的每个纹素的大小（一张 512 × 512 大小的纹理，该值为 1/512）；因为卷积需要对相邻区域内的纹理进行采样，因此需要利用 _Main_TexelSize 来计算各个相邻区域的纹理坐标。
            fixed _EdgeOnly;
            fixed4 _EdgeColor;
            fixed4 _BackgroundColor;
            
            struct v2f {
                float4 pos : SV_POSITION;
                half2 uv[9] : TEXCOORD0; //维数为 9 的纹理数组，对应算子需求的 9 个纹理坐标
            };

            //在顶点着色器中计算采样纹理坐标减少性能消耗。由于从顶点着色器到片元着色器的插值是线性的，因此不会影响纹理坐标的计算结果
            v2f vert(appdata_img v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                
                half2 uv = v.texcoord;
                o.uv[0] = uv + _MainTex_TexelSize.xy * half2(-1, -1);
                o.uv[1] = uv + _MainTex_TexelSize.xy * half2(0, -1);
                o.uv[2] = uv + _MainTex_TexelSize.xy * half2(1, -1);
                o.uv[3] = uv + _MainTex_TexelSize.xy * half2(-1, 0);
                o.uv[4] = uv + _MainTex_TexelSize.xy * half2(0, 0);
                o.uv[5] = uv + _MainTex_TexelSize.xy * half2(1, 0);
                o.uv[6] = uv + _MainTex_TexelSize.xy * half2(-1, 1);
                o.uv[7] = uv + _MainTex_TexelSize.xy * half2(0, 1);
                o.uv[8] = uv + _MainTex_TexelSize.xy * half2(1, 1);
                
                return o;
            }
            
            fixed luminance(fixed4 color) {
                return  0.2125 * color.r + 0.7154 * color.g + 0.0721 * color.b; 
            }
            
            half Sobel(v2f i) {
                const half Gx[9] = {-1,  0,  1, //定义了水平方向的卷积核 Gx
                                    -2,  0,  2,
                                    -1,  0,  1};
                const half Gy[9] = {-1, -2, -1, //定义了竖直方向的卷积核 Gy
                                    0,  0,  0,
                                    1,  2,  1};        
                
                half texColor;
                half edgeX = 0;
                half edgeY = 0;
                for (int it = 0; it < 9; it++) {
                    texColor = luminance(tex2D(_MainTex, i.uv[it])); //依次对 9 个像素进行采样，计算他们的亮度值
                    edgeX += texColor * Gx[it]; //再与 Gx 和 Gy 对应的权重进行相乘，得到各自的梯度值
                    edgeY += texColor * Gy[it];
                }
                
                half edge = 1 - abs(edgeX) - abs(edgeY); //从 1 中减去水平方向和竖直方向的梯度值的绝对值，得到 edge，edge 值越小则表明该位置越可能是一个边缘点
                
                return edge;
            }
            
            fixed4 fragSobel(v2f i) : SV_Target {
                half edge = Sobel(i); //调用 Soble 函数计算当前像素的梯度值 edge

                //利用 edge 分别计算了背景为原图和纯色下的颜色值
                fixed4 withEdgeColor = lerp(_EdgeColor, tex2D(_MainTex, i.uv[4]), edge);
                fixed4 onlyEdgeColor = lerp(_EdgeColor, _BackgroundColor, edge);
                //利用 _EdgeOnly 在两者之间插值得到最终的像素值
                return lerp(withEdgeColor, onlyEdgeColor, _EdgeOnly);
             }
            ENDCG
        } 
    }
    FallBack Off
}
```

将编写好的 Shader 文件拖拽到 C# 组件的 Edge Detect Shader 属性中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/20/F1qZG5ioYhjOKmk.png" width = "100%" height = "100%" alt="图65-  左图：原效果。中图：Edges Only 为 0。右图：Edges Only 为 1。"/>
</div>

需要注意的是，本节实现的边缘检测仅仅利用了屏幕颜色信息，而在实际的应用中，物体的纹理、阴影等信息都会影响到边缘检测的结果。为了得到更加准确的边缘信息，往往会在屏幕的深度纹理和法线纹理上进行边缘检测，在 第 12 章实现这种方法。

## 高斯模糊


