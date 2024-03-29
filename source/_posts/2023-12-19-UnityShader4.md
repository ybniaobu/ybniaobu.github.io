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

> 本读书笔记为高级篇的前 4 章，主要内容为屏幕后处理的边缘检测、高斯模糊、Bloom 效果和运动模糊；使用深度法线纹理的屏幕后处理的运动模糊、全局雾效和边缘检测；非真实感渲染的卡通渲染、素描风格；使用噪声的消融、水波、全局雾效效果。
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

在默认情况下，OnRenderImage 函数会在所有不透明和透明的 Pass 执行完毕后被调用，以便对场景中所有游戏对象都产生影响。但有时，会希望在不透明的 Pass（RenderQueue <= 2500，内置的 Background、Geometry 和 AlphaTest 渲染队列均在此范围内）执行完毕后立即调用 OnRenderImage 函数，从而不对透明物体产生任何影响。可在 OnRenderImage 函数前添加 `ImageEffectOpaque` 属性实现这样的目的（见第 12 章，利用深度和法线纹理进行边缘检测从而实现描边效果，但是不希望透明物体也被描边）。

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
常见的使用卷积的模糊有均值模糊和中值模糊。均值模糊使用的卷积核各个元素都相等，且相加等于1，即卷积后得到的像素值是其邻域内各个像素值的平均值。中值模糊，则是选择邻域内所有像素排序后的中值替换掉原颜色。而**高斯模糊**是一个更高级的模糊方式。

### 高斯滤波
高斯模糊同样利用了卷积计算，它使用的卷积核名为**高斯核**。高斯核是一个正方形大小的滤波核，其中每个元素都是基于下面的高斯方程：  

$$ G(x,y) = \cfrac {1}{2 \pi \sigma ^ 2} e ^ {- \cfrac {x^2 + y^2} {2 \sigma ^2} } $$

其中，$\,\sigma\,$ 是标准方差，一般取值为 1；$\,x\,$  和 $\,y\,$ 分别对应了当前位置到卷积核中心的整数距离。要构建高斯核，只需要计算高斯核中各个位置对应的高斯值。为了保证滤波后的图像不会变暗，需要对高斯核中的权重进行归一化，即让每一个权重除以所有权重的和，以保证所有权重的和为 1，因此高斯函数中的 $\,e\,$  前面的系数实际不会对结果有任何影响。

高斯方程很好地模拟了邻域每个像素对当前处理像素的影响程度 —— 距离越近，影响越大。高斯核的维数越高，模糊程度越大。使用 N x N 的高斯核对图像进行卷积滤波，需要进行 N x N x W x H（W 和 H分别是图像的宽和高）次纹理采样。N 不断增大，采样次数会变得非常巨大。

为了节省性能，可以把二维高斯函数可以拆分成两个一维函数。使用两个一维的高斯核先后对图像进行滤波，得到的结果和直接使用二位高斯核是一样的，但采样次数只需要 2 x N x W x H。对于一个大小为 5 的一维高斯核，只需要记录 3 个权重值即可，因为左右权重有重复的值。如下图：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/21/XCzRgKkvqG6Q7a5.png" width = "70%" height = "70%" alt="图66-  一个 5×5 大小的高斯核。左图显示了标准方差为 1 的高斯核的权重分布。我们可以把这个二维高斯核拆分成两个一维的高斯核（右图）。"/>
</div>

### 实现
接下来使用上述的 5 x 5 的高斯核对原图像进行高斯模糊。先后调用两个 Pass，第一个 Pass 将会使用竖直方向的一维高斯核对图像进行滤波，第二个 Pass 再使用水平方向的一维高斯核对图像进行滤波，得到最终的目标图像。在现实中还会利用图像缩放来进一步提高性能，通过调整高斯滤波的应用次数来控制模糊程度（次数越多，图像越模糊）。

准备工作如下：  
①新建名为 Scene_12_4 的场景，并去掉天空盒子；  
②导入一张图片（资源路径：Assets/Textures/Chapter12/Sakura1.jpg），调整图片纹理类型为 Sprite (2D and UI)，并拖拽到场景中，使其生成一个 Sprite；  
③新建名为 GaussianBlur 的 C# 脚本，将脚本拖拽到相机；  
④新建名为 Chapter12-GaussianBlur 的 Unity Shader。

GaussianBlur 的 C# 脚本的代码如下：

``` C#
using UnityEngine;
using System.Collections;

public class GaussianBlur : PostEffectsBase {

    public Shader gaussianBlurShader;
    private Material gaussianBlurMaterial = null;

    public Material material {  
        get {
            gaussianBlurMaterial = CheckShaderAndCreateMaterial(gaussianBlurShader, gaussianBlurMaterial);
            return gaussianBlurMaterial;
        }  
    }

    [Range(0, 4)]
    public int iterations = 3; //高斯模糊迭代次数
    
    [Range(0.2f, 3.0f)]
    public float blurSpread = 0.6f; //模糊范围，计算得到 shader 中的 _BlurSize。_BlurSize 越大，模糊程度越大，但采样数不会受到影响，过大的 _BlueSize 值会造成虚影
    
    [Range(1, 8)]
    public int downSample = 2; //缩放图像降采样的系数，downSample 越大，需要处理的像素数越少，同时也可以进一步提高模糊程度，但过大的 downSample 会造成图像像素化
    
// 第一个版本：简单的 OnRenderImage 实现，不使用上面 3 个字段。首先利用 RenderTexture.GetTemporary 函数分配一块与屏幕图像大小相同的缓冲区。因为模糊需要调用两个Pass，从而需要使用一块中间缓存来存储第一个 Pass 执行完毕后得到的模糊结果。
//    void OnRenderImage(RenderTexture src, RenderTexture dest) {
//        if (material != null) {
//            int rtW = src.width;
//            int rtH = src.height;
//            RenderTexture buffer = RenderTexture.GetTemporary(rtW, rtH, 0);        
//
//            Graphics.Blit(src, buffer, material, 0); //参数为 0，即使用 Shader 中的第一个 Pass（即使用竖直方向的一维高斯核进行滤波）对 src 进行处理，并将结果存储在 buffer中
//
//            Graphics.Blit(buffer, dest, material, 1); //然后利用第二个 pass 对 buffer 进行处理，返回最终的屏幕图像
//
//            RenderTexture.ReleaseTemporary(buffer); //调用这个释放之前分配的缓存
//        } else {
//            Graphics.Blit(src, dest);
//        }
//    } 

// 第二个版本：利用缩放系数对图像进行降采样，从而减少需要处理的像素个数，提高性能
//    void OnRenderImage (RenderTexture src, RenderTexture dest) {
//        if (material != null) {
//            int rtW = src.width/downSample;
//            int rtH = src.height/downSample;
//            RenderTexture buffer = RenderTexture.GetTemporary(rtW, rtH, 0);
//            buffer.filterMode = FilterMode.Bilinear; //声明缓冲区的大小时，使用了小于原屏幕分辨率的尺寸，并将该临时渲染纹理的滤波模式设置为双线性，从而不仅减少需要处理的像素个数，同时更提高了性能，适当的降采样还可以得到更好的模糊效果，但过大的 downSample 会造成图像像素化
//
//            Graphics.Blit(src, buffer, material, 0);
//            Graphics.Blit(buffer, dest, material, 1);
//            RenderTexture.ReleaseTemporary(buffer);
//        } else {
//            Graphics.Blit(src, dest);
//        }
//    }
//
// 第三个版本相比第二个版本增加了迭代次数和模糊范围，下面代码显示了如何利用两个临时缓存在迭代之间进行交替的过程
    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            int rtW = src.width/downSample;
            int rtH = src.height/downSample;

            //首先定义第一个缓存 buffer0，并直接把 scr 的图像缩放后存储到 buffer0 中
            RenderTexture buffer0 = RenderTexture.GetTemporary(rtW, rtH, 0);
            buffer0.filterMode = FilterMode.Bilinear;
            Graphics.Blit(src, buffer0);

            //在迭代过程中，定义第二个缓存 buffer1。在执行第一个 Pass 时，输入的是 buffer0，输出的是 buffer1，完毕之后，释放 buffer0，将 buffer1 存储到 buffer0 中，重新分配 buffer1，再调用第二个 Pass，不断重复
            for (int i = 0; i < iterations; i++) {
                material.SetFloat("_BlurSize", 1.0f + i * blurSpread);
            
                RenderTexture buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 0); //第一个 Pass
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
                buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 1); //第二个 Pass
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
            }

            Graphics.Blit(buffer0, dest);        //把结果显示在屏幕上
            RenderTexture.ReleaseTemporary(buffer0);    
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter12-GaussianBlur 的 Unity Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 12/Gaussian Blur" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _BlurSize ("Blur Size", Float) = 1.0
    }
    SubShader {
        //第一次使用 CGINCLUDE，该代码不需要包含任何 Pass 语义块，在使用时只需要在 Pass 中直接指定需要使用的顶点着色器和片元着色器函数名即可，类似于 C++ 中的头文件，这样可以避免编写相同的片元着色器
        CGINCLUDE
        
        #include "UnityCG.cginc"
        
        sampler2D _MainTex;  
        half4 _MainTex_TexelSize;
        float _BlurSize;
          
        struct v2f {
            float4 pos : SV_POSITION;
            half2 uv[5]: TEXCOORD0; //因为可以拆分成 2 个大小为 5 的一维高斯核，从而只需要计算 5 个纹理坐标即可
        };

        //竖直方向的顶点着色器代码
        v2f vertBlurVertical(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            half2 uv = v.texcoord;

            //利用与属性 _BlurSize 相乘来控制采样距离。在高斯核维数不变的情况下，_BlurSize 越大，模糊程度越高，但采样数不受到影响
            o.uv[0] = uv;
            o.uv[1] = uv + float2(0.0, _MainTex_TexelSize.y * 1.0) * _BlurSize;
            o.uv[2] = uv - float2(0.0, _MainTex_TexelSize.y * 1.0) * _BlurSize;
            o.uv[3] = uv + float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
            o.uv[4] = uv - float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
            return o;
        }

        //水平方向的顶点着色器代码
        v2f vertBlurHorizontal(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            half2 uv = v.texcoord;
            
            o.uv[0] = uv;
            o.uv[1] = uv + float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
            o.uv[2] = uv - float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
            o.uv[3] = uv + float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
            o.uv[4] = uv - float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;   
            return o;
        }

        //两个 Pass 共用的片元着色器
        fixed4 fragBlur(v2f i) : SV_Target {
            float weight[3] = {0.4026, 0.2442, 0.0545}; //高斯核的三个不重复的权重
            
            fixed3 sum = tex2D(_MainTex, i.uv[0]).rgb * weight[0]; //权重和 sum 初始化为当前的像素值乘以它的权重值

            //根据对称性，进行两次迭代，每次迭代包含了两次纹理采样
            for (int it = 1; it < 3; it++) {
                sum += tex2D(_MainTex, i.uv[it*2-1]).rgb * weight[it];
                sum += tex2D(_MainTex, i.uv[it*2]).rgb * weight[it];
            }
            
            return fixed4(sum, 1.0); //返回滤波结果sum
        }
            
        ENDCG
        
        ZTest Always Cull Off ZWrite Off

        //定义第一个竖直的 Pass
        Pass {
            //使用 NAME 语义定义了它们的名字，为 Pass 定义名字可以在其他 shader 中直接通过它们的名字来使用该 Pass，而不需要重新写代码
            NAME "GAUSSIAN_BLUR_VERTICAL"
            
            CGPROGRAM
              
            #pragma vertex vertBlurVertical //告诉 Unity，顶点着色器的代码在 vertBlurVertical 函数中
            #pragma fragment fragBlur //告诉 Unity，片元着色器的代码在 fragBlur 函数中
              
            ENDCG  
        }
        
        Pass {  
            NAME "GAUSSIAN_BLUR_HORIZONTAL"
            
            CGPROGRAM  
            
            #pragma vertex vertBlurHorizontal  
            #pragma fragment fragBlur
            
            ENDCG
        }
    } 
    FallBack "Diffuse"
}
```

将编写好的 Shader 文件拖拽到 C# 脚本组件的 Gaussian Blur Shader 属性中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/21/6IuoeZQXA2YzkGt.png" width = "70%" height = "70%" alt="图67-  左图：原效果。右图：高斯模糊后的效果（Iterations 为 3，Blur Spread 为 0.6，Down Sample 为 2）。"/>
</div>

## Bloom 效果
Bloom 特效是游戏中常见的一种屏幕效果，可以模拟真实相机的一种图像效果，它让画面中较亮的区域“扩散”到周围的区域中，产生一种朦胧的效果。

Bloom 的实现原理非常简单：  
①根据一个阈值提取出图像中的较亮区域，把它们存储在一张渲染纹理中；  
②再利用高斯模糊对这张纹理进行模糊处理，模拟光线扩散的效果；  
③最后与原图像进行混合，得到最终效果。

准备工作如下：  
①新建名为 Scene_12_5 的场景，并去掉天空盒子；  
②导入一张图片（资源图路径：Assets/Textures/Chapter12/Sakura1.jpg），调整图片纹理类型为 Sprite (2D and UI)，并拖拽到场景中，使其生成一个 Sprite；  
③新建名为 Bloom 的 C# 脚本；  
④新建名为 Chapter12-Bloom 的 Unity Shader。

Bloom.cs 的脚本代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class Bloom : PostEffectsBase {

    public Shader bloomShader;
    private Material bloomMaterial = null;
    public Material material {  
        get {
            bloomMaterial = CheckShaderAndCreateMaterial(bloomShader, bloomMaterial);
            return bloomMaterial;
        }  
    }

    [Range(0, 4)]
    public int iterations = 3;
    
    [Range(0.2f, 3.0f)]
    public float blurSpread = 0.6f;

    [Range(1, 8)]
    public int downSample = 2;

    //增加 luminanceThreshold 来控制提取较亮的区域时使用的阈值大小。尽管在绝大多数的情况下，图像的亮度值不会超过 1，但如果开启了 HDR，硬件会允许把颜色值存储在一个更高精度范围的缓冲中，此时像素的亮度值可能会超过 1，这里把 luminanceThreshold 的值规定在 [0,4] 中
    [Range(0.0f, 4.0f)]
    public float luminanceThreshold = 0.6f;

    //Bloom 建立在高斯模糊之上实现，从而与高斯模糊的代码有大量重复的地方
    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            material.SetFloat("_LuminanceThreshold", luminanceThreshold); 

            int rtW = src.width/downSample;
            int rtH = src.height/downSample;

            //先使用 Shader 中的第一个 Pass 提取图像中的较亮区域，存储到 buffer0 中
            RenderTexture buffer0 = RenderTexture.GetTemporary(rtW, rtH, 0);
            buffer0.filterMode = FilterMode.Bilinear;
            Graphics.Blit(src, buffer0, material, 0);

            //和之前相同的高斯模糊迭代处理，只不过 Pass 对应的是 Shader 的第二个和第三个 Pass
            for (int i = 0; i < iterations; i++) {
                material.SetFloat("_BlurSize", 1.0f + i * blurSpread);
                
                RenderTexture buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                Graphics.Blit(buffer0, buffer1, material, 1);
                RenderTexture.ReleaseTemporary(buffer0); 
                buffer0 = buffer1;
                buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0); 
                Graphics.Blit(buffer0, buffer1, material, 2);
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1; 
            }

            //将 buffer0 传递给材质中的 _Bloom 纹理属性，使用第四个 pass 进行最后的混合，并存储到目标渲染纹理 dest 中
            material.SetTexture ("_Bloom", buffer0);
            Graphics.Blit (src, dest, material, 3);
            RenderTexture.ReleaseTemporary(buffer0);
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter12-Bloom 的 Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 12/Bloom" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _Bloom ("Bloom (RGB)", 2D) = "black" {} //高斯模糊后较亮的区域的输出纹理
        _LuminanceThreshold ("Luminance Threshold", Float) = 0.5 //提取较亮区域的阈值
        _BlurSize ("Blur Size", Float) = 1.0 //控制不同迭代之间高斯模糊的模糊区域范围
    }

    SubShader {
        CGINCLUDE
        
        #include "UnityCG.cginc"

        sampler2D _MainTex;
        half4 _MainTex_TexelSize;        
        sampler2D _Bloom;
        float _LuminanceThreshold;
        float _BlurSize;

        //下面先定义提取较亮区域需要使用的结构体、顶点着色器和片元着色器
        struct v2f {
            float4 pos : SV_POSITION; 
            half2 uv : TEXCOORD0;
        };    

        v2f vertExtractBright(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            o.uv = v.texcoord;
            return o;
        }
        
        fixed luminance(fixed4 color) {
            return  0.2125 * color.r + 0.7154 * color.g + 0.0721 * color.b; 
        }
        
        fixed4 fragExtractBright(v2f i) : SV_Target {
            fixed4 c = tex2D(_MainTex, i.uv);
            fixed val = clamp(luminance(c) - _LuminanceThreshold, 0.0, 1.0); //采样得到的亮度值减去阈值 _LuminanceThreshold，并把结果截取在 0～1 之间
            return c * val; //把该值和愿像素相乘，得到提取后的亮度区域
        }

        //定义混合亮部图像和原图像时使用的结构体、顶点着色器和片元着色器
        struct v2fBloom {
            float4 pos : SV_POSITION; 
            half4 uv : TEXCOORD0;
        };

        v2fBloom vertBloom(appdata_img v) {
            v2fBloom o;
            o.pos = UnityObjectToClipPos (v.vertex);
            //定义两个纹理坐标，并存储在同一个类型为 half4 的变量 uv 中，xy 对应 _MainTex，即原图像的纹理坐标；zw 对应 _Bloom，即对应模糊后的较亮区域的纹理坐标
            o.uv.xy = v.texcoord;
            o.uv.zw = v.texcoord;

            //平台差异化处理的代码
            #if UNITY_UV_STARTS_AT_TOP            
            if (_MainTex_TexelSize.y < 0.0)
                o.uv.w = 1.0 - o.uv.w;
            #endif
                            
            return o; 
        }
        
        fixed4 fragBloom(v2fBloom i) : SV_Target {
            return tex2D(_MainTex, i.uv.xy) + tex2D(_Bloom, i.uv.zw);
        } 
        
        ENDCG
        
        ZTest Always Cull Off ZWrite Off

        //定义 Bloom 效果需要的 4 个 pass
        Pass {  
            CGPROGRAM  
            #pragma vertex vertExtractBright  
            #pragma fragment fragExtractBright  
            ENDCG  
        }

        //第二、第三个 pass 使用上一节的高斯模糊定义的两个 pass，该是通过 UsePass 语义来指明所使用的 pass，注意：Unity 内部会把 shader 名字全部转换为大写字母，从而使用时需要大写字母
        
        UsePass "Unity Shaders Book/Chapter 12/Gaussian Blur/GAUSSIAN_BLUR_VERTICAL"
        
        UsePass "Unity Shaders Book/Chapter 12/Gaussian Blur/GAUSSIAN_BLUR_HORIZONTAL"
        
        Pass {  
            CGPROGRAM  
            #pragma vertex vertBloom  
            #pragma fragment fragBloom  
            ENDCG  
        }
    }
    FallBack Off
}
```

将编写好的 Shader 文件拖拽到 C# 脚本组件的 Bloom Shader 属性中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/21/GnsWkg5EzRAZoX4.png" width = "70%" height = "70%" alt="图68-  左图：原效果。右图：Bloom 处理后的效果（Iterations 为 4，Blur Spread 为 1.5，Down Sample 为 4，Luminance Threshold 为 0.1）。"/>
</div>


## 运动模糊
运动模糊是真实世界中的摄像机的一种效果。摄像机曝光时如果拍摄场景发生了变化，就会产生模糊的画面。由于在计算机生成的图像中，不存在在曝光这一物理现象，渲染出来的图像往往都是棱角分明，缺少运动模糊。  

运动模糊的实现有多种方法：  
①**累积缓存 accumulation buffer**：混合多张连续图像。当物体快速移动产生多张图像后，我们取它们之间的平均值作为最后的运动模糊图像。这种暴力的方法对性能消耗很大，想要获取多张帧图像意味着我们需要在同一帧里渲染多次场景。  
②**速度缓存 velocity buffer**：这个缓存中存储了各个像素当前的运动速度，然后利用该值来觉得模糊的方向和大小。

本节中，我们将使用类似于累积缓存的方法来实现运动模糊。我们不需要在一帧中把场景渲染多次，但需要保存之前的渲染结果，不断把当前的渲染图像叠加到之前的渲染图像中，从而产生一种运动轨迹的视觉效果。这种方法与原始的利用累计缓存的方法相比性能更好，但模糊效果可能略有影响。

准备工作如下：  
①新建名为 Scene_12_6 的场景，并去掉天空盒子；  
②往场景里放置几个立方体和平面；  
③新建名为 Translating 的 C# 脚本，并拖拽到相机，用于控制相机围绕目标物体旋转；  
④新建名为 MotionBlur 的 C# 脚本，并把脚本拖拽到相机；  
⑤新建名为 Chapter12-MotionBlur 的 Unity Shader。

Translating.cs 的代码如下：  

``` C#
using UnityEngine;

public class Translating : MonoBehaviour
{
    public float speed = 10.0f;
    public Vector3 startPoint = Vector3.zero;
    public Vector3 endPoint = Vector3.zero;
    public Vector3 lookAt = Vector3.zero;
    public bool pingpong = true;

    private Vector3 curEndPoint = Vector3.zero;

    private void Start()
    {
        transform.position = startPoint;
        curEndPoint = endPoint;
    }

    private void Update()
    {
        transform.position = Vector3.Slerp(transform.position, curEndPoint, Time.deltaTime * speed);
        transform.LookAt(lookAt);

        if (pingpong)
        {
            if (Vector3.Distance(transform.position, curEndPoint) < 0.001f)
            {
                curEndPoint = Vector3.Distance(curEndPoint, endPoint) < Vector3.Distance(curEndPoint, startPoint) ? startPoint : endPoint;
            }
        }
    }
}
```

MotionBlur.cs 的代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class MotionBlur : PostEffectsBase {
    public Shader motionBlurShader;
    private Material motionBlurMaterial = null;

    public Material material {  
        get {
            motionBlurMaterial = CheckShaderAndCreateMaterial(motionBlurShader, motionBlurMaterial);
            return motionBlurMaterial;
        }  
    }

    //定义运动模糊在混合图像时使用的模糊参数，并限定范围值，blurAmount 越大，运动拖尾越明显
    [Range(0.0f, 0.9f)]
    public float blurAmount = 0.5f;

    //定义一个RenderTexture类型的变量，并且保存之前图像叠加的结果
    private RenderTexture accumulationTexture;

    //在该脚本不运行时，即调用 OnDisable 函数时，立即销毁 accumulationTexture。保证下一次开始应用运动模糊时重新叠加图像
    void OnDisable() {
        DestroyImmediate(accumulationTexture);
    }

    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            //判断用于混合图像的 accumulationTexture 是否满足条件：不为空且与当前的屏幕分辨率相等
            if (accumulationTexture == null || accumulationTexture.width != src.width || accumulationTexture.height != src.height) {
                DestroyImmediate(accumulationTexture);
                accumulationTexture = new RenderTexture(src.width, src.height, 0);
                // HideAndDontSave 表示该变量不会显示在Hierarchy，也不会保存到场景中
                accumulationTexture.hideFlags = HideFlags.HideAndDontSave;
                Graphics.Blit(src, accumulationTexture);
            }

            //调用 accumulationTexture.MarkRestoreExpected() 来表示需要进行一个渲染纹理的恢复操作，恢复操作指发生在纹理到渲染而该纹理又没有被提前清空或销毁的情况下。在本例中每次调用 OnRenderImage 都需要把当前帧图像和 accumulationTexture 中的图像混合，accumulationTexture 不需要提前清空，因为保存了之前的混合结果
            accumulationTexture.MarkRestoreExpected();

            material.SetFloat("_BlurAmount", 1.0f - blurAmount);

            Graphics.Blit (src, accumulationTexture, material);
            Graphics.Blit (accumulationTexture, dest);
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter12-MotionBlur.shader 的代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 12/Motion Blur" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _BlurAmount ("Blur Amount", Float) = 1.0
    }

    SubShader {
        CGINCLUDE
        
        #include "UnityCG.cginc"
        
        sampler2D _MainTex; 
        fixed _BlurAmount;
        
        struct v2f {
            float4 pos : SV_POSITION;
            half2 uv : TEXCOORD0;
        };

        v2f vert(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            o.uv = v.texcoord;   
            return o;
        }

        //定义两个片元着色器，一个用于更新渲染纹理的 RGB 通道部分，一个用于更新渲染纹理的 A 通道部分。RGB 通道的 shader 对当前图像进行采样，并将其 A 通道设置为 _BlurAmount，以便可以在后面使用它的透明通道进行混合。A 通道直接返回采样结果，该只是为了维护渲染纹理的透明通道，不让其收到混合时使用的透明度值的影响
        fixed4 fragRGB (v2f i) : SV_Target {
            return fixed4(tex2D(_MainTex, i.uv).rgb, _BlurAmount);
        }
        
        half4 fragA (v2f i) : SV_Target {
            return tex2D(_MainTex, i.uv);
        }
        
        ENDCG
        
        ZTest Always Cull Off ZWrite Off

        //更新渲染纹理的 RGB 通道的 Pass，利用 A 通道来混合图像，但不希望该 A 通道写入渲染纹理
        Pass {
            Blend SrcAlpha OneMinusSrcAlpha
            ColorMask RGB
            
            CGPROGRAM
            
            #pragma vertex vert  
            #pragma fragment fragRGB  
            
            ENDCG
        }

        //更新 A 通道的 Pass
        Pass {   
            Blend One Zero
            ColorMask A
                   
            CGPROGRAM  
            
            #pragma vertex vert  
            #pragma fragment fragA
              
            ENDCG
        }
    }
     FallBack Off
}
```

将 Chapter12-MotionBlurShader 文件拖拽到脚本的 Motion Blur Shader 属性上，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/23/nQmI5FfOdJ9g3k7.gif" width = "70%" height = "70%" alt="图69-  运动模糊后的效果。"/>
</div>


# 第十二章 使用深度和法线纹理
在前一章节，学习的屏幕后处理效果都只是在屏幕颜色图像上进行各种操作来实现的。然而，很多时候我们不仅需要当前屏幕的颜色信息，还希望得到深度和法线信息。比如，在进行边缘检测时，直接利用颜色信息会使检测到的边缘信息受物体纹理和光照等外部因素的影响，得到很多不需要的边缘点。一种更好的方法是，我们可以在深度纹理和法线纹理上进行边缘检测，这些图像不会受纹理和光照的影响，而仅仅保存了当前渲染物体的模型信息，通过这样的方式检测出来的边缘更加可靠。

## 获取深度和法线纹理
### 背后的原理
深度纹理实际就是一张渲染纹理，里面存储的像素值不是颜色值，而是一个高精度的深度值。由于被存储在一张纹理中，深度纹理的深度值范围是 \[0, 1\]，而且通常是非线性分布的。

**深度值来自于顶点变换后得到的归一化的设备坐标 Normalized Device Coordinates, NDC 的 z 分量**。一个模型要想最终被绘制在屏幕上，需要把它的顶点从模型空间变换到齐次裁剪坐标系下，通过在顶点着色器中乘以 MVP 矩阵变换得到的。在变换到最后一步，需要使用一个投影矩阵来变换顶点，而透视投影的投影矩阵就是非线性的（正交投影的投影矩阵是线性的）。

> 回忆一下：顶点着色器阶段要把模型空间一步步转换到应用透视裁剪矩阵（或者正交）后的变换结果，合起来即 MVP 矩阵变换。而齐次除法则是底层硬件进行的，得到归一化的设备坐标。OpenGL 中 NDC 的 z 分量范围在 \[-1, 1\] 之间，而在 DirectX 中，z 分量在 \[0, 1\] 之间。

在得到 NDC 之后，深度纹理中的像素值就可以得到了。NDC 的 z 分量范围在 \[-1, 1\] 之间，为了让这些值能够存储到一张图上，需要对其进行映射（d 对应了深度纹理中的像素值）：  

$$ d = 0.5 \cdot z_{NDC} + 0.5$$

--- 

在 Unity 中，深度纹理可以直接来自于真正的深度缓存，也可以是由一个单独的 Pass 渲染而得，这取决于使用的渲染路径和硬件。  
①当使用延迟渲染路径时，因为延迟渲染会把深度信息渲染到 G-buffer 中。  
②当无法直接获取深度缓存时，深度和法线纹理是通过一个渲染。具体实现是：Unity 会使用着色器替换 Shader Replacement 技术选择那些渲染类型（即 SubShader 的 RenderType 标签）为 Opaque 的物体，判断它们使用的渲染队列是否小于等于 2500（内置的 Background、Geometry 和 AlphaTest 渲染队列均在此范围内），如果满足条件，就把它渲染到深度和法线纹理中。因此，要想让物体能够出现在深度和法线纹理中，就**必须在 Shader 中设置正确的 RenderType 标签**。

在 Unity 中，我们可以选择让一个摄像机生成一张深度纹理或是一张深度 + 法线纹理。  
①当只需要一张单独的深度纹理时，Unity 会直接获取深度缓存或是按之前讲到的着色器替换技术，选取需要的不透明物体，并使用它投射阴影时使用的 Pass （即 LightMode 设置为 ShaowCaster 的 Pass）来得到深度纹理。如果 Shader 中不包含这样一个 Pass，那么这个物体就不会出现在深度纹理中（当然，它也不能向其他物体投射阴影）。深度纹理的精度通常是 24 位或 16 位，这取决于使用的深度缓存的精度。  
②如果选择生成一张深度 + 法线纹理，Unity 会创建一张和屏幕分辨率相同、 精度为 32 位（每个通道为 8 位）的纹理，其中观察空间下的法线信息会被编码进纹理的 R 和 G 通道，而深度信息会被编码进 B 和 A 通道。法线信息的获取在延迟渲染中是非常容易得到的，Unity 只需要合并深度和法线缓存即可。而在前向渲染中，默认情况下是不会去创建法线缓存的，因此 Unity 底层使用了一个单独的 Pass 把整个场景再次渲染一遍来完成。这个 Pass 被包含在 Unity 的内置的一个 Unity Shader 中，我们可以在内置的 build_shaders-xxx/DefaultResourcesExtra/Internal-DepthNormalsTexture.shader 文件中找到这个用于渲染深度和法线信息的的 Pass。

### 如何获取
在 Unity 中，首先获取深度纹理需要通过脚本中设置摄像机的 depthTextureMode：

    camera.depthTextureMode = DepthTextureMode.Depth;

一旦设置好了上面的摄像机模式后，我们就可以在 Shader 中通过声明 `_CameraDepthTexture` 变量来访问它。

同理，如果想要获取深度 + 法线纹理，我们需要在代码中这样设置：

    camera.depthTextureMode = DepthTextureMode.DepthNormals;

然后在 Shader 中通过声明 `_CameraDepthNormalsTexture` 来访问它。

我们还可以组合这些模式，让一个摄像机同时产生一张深度和深度 + 法线纹理：  

    camera.depthTextureMode |= DepthTextureMode.Depth;
    camera.depthTextureMode |= DepthTextureMode.DepthNormals;

---

在 Unity 5 中，我们还可以在摄像机的 Camera 组件上看到当前摄像机是否需要渲染深度或深度 + 法线纹理，当在 Shader 中访问到深度纹理 \_CameraDepthTexture 后，我们就可以使用当前像素的纹理坐标对它进行采样。绝大多数情况下，我们直接使用 `tex2D` 函数采样即可，但在某些平台上，我们需要一些特殊处理。Unity 为我们提供了一个统一的宏 `SAMPLE_DEPTH_TEXTURE`，用来处理这些由于平台差异造成的问题。而我们只需要在 Shader 中使用 SAMPLE_DEPTH_TEXTURE 宏对深度纹理进行采样，得到深度值，例如：

    float d = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);

其中，i.uv 是一个 float2 类型的变量，对应了当前像素的纹理坐标。类似的宏还有 `SAMPLE_DEPTH_TEXTURE_PROJ` 和 `SAMPLE_DEPTH_TEXTURE_LOD`。

`SAMPLE_DEPTH_TEXTURE_PROJ` 宏同样接收两个参数 – 深度纹理和一个 float3 或 float4 类型的纹理坐标，它内部使用了 tex2Dproj 这样的函数进行纹理采样，纹理坐标的前两个分量首先会除以最后一个分量，再进行纹理采样。如果提供了第四个分量，还会进行一次比较，通常用于阴影的实现中。第二个参数通常是由顶点着色器输出插值而得的屏幕坐标。例如：  

    float d = SAMPLE_DEPTH_TEXTURE_PROJ(_CameraDepthTetxure, UNITY_PROJ_COORD(i.scrPos));

其中，i.scrPos 是在顶点着色器中通过调用 ComputeScreenPos(o.pos) 得到的屏幕坐标，上述这些宏的定义都可以在 Unity 内置的 HLSLSupport.cingc 文件中找到。

> ComputeScreenPos(o.pos) 得到是一个未进行齐次除法的“假视口坐标”，SAMPLE_DEPTH_TEXTURE_PROJ 宏使用了  tex2Dproj 函数将 i.scrPos 齐次除法后得到真正的视口坐标，再进行采样。而 UNITY_PROJ_COORD 返回一个适合投影纹理读取的纹理坐标，在大多数平台上，它直接返回给定值。

---

之前讲过，通过纹理采样得到的深度值，这些深度值往往是非线性的，这种非线性来自于透视投影使用的裁剪矩阵。然而，在我们的计算过程中通常是需要线性的深度值，也就是说，我们需要把投影后的深度值变换到线性空间下，例如视角空间（观察空间）下的深度值。下面以透视投影为例，推导如何由深度纹理的深度信息倒推顶点变换回到视角空间下的深度值：  

第三章中讲过，当我们使用透视投影的裁剪矩阵 $\,P_{clip}\,$ 对观察空间下的一个顶点进行变换后，裁剪空间下顶点的 z 和 w 分量为：  

$$ z_{clip} = - z_{view} \cfrac {Far + Near}{Far - Near} - \cfrac {2 \cdot Far \cdot Near}{Far - Near} $$
$$ w_{clip} = - z_{view} $$

然后通过齐次除法得到 NDC 下的 z 分量：  

$$ z_{ndc} = \cfrac {z_{clip}} {w_{clip}} = \cfrac {Far + Near}{Far - Near} + \cfrac {2 \cdot Far \cdot Near}{(Far - Near) \cdot z_{view} } $$

在本章最前面提到，深度纹理的深度值是 NDC 的 z 分量映射而得的：  

$$ d = 0.5 \cdot z_{NDC} + 0.5$$

由上面公式可以反向推导出 $\,z_{view}\,$ ：  

$$ z_{view} = \cfrac {1} { \cfrac { Far - Near }{ Far \cdot Near } d - \cfrac { 1 }{ Near } } $$

由于在 Unity 中使用的观察空间中，摄像机正向对应的 z 值均为负值，因此为了得到深度值的正数表示，我们需要对上面的结果取反，最后得到的结果如下：

$$ z_{view}' = \cfrac {1} { \cfrac { Near - Far }{ Near \cdot Far } d + \cfrac { 1 }{ Near } } $$

它的取值范围就是视锥体深度范围，即 \[Near, Far\]。如果我们想得到范围在 \[0, 1\] 之间的深度值，只需要把上面的结果除以 Far 即可。这样，0 就表示该点与摄像机位于同一位置，1 表示该点位于视锥体的远裁剪平面上，结果如下：

$$ z_{01} = \cfrac {1} { \cfrac { Near - Far }{ Near } d + \cfrac { Far }{ Near } } $$

幸运的是，Unity 提供了两个辅助函数来为我们进行上述的计算过程 —— `LinearEyeDepth` 和 `Linear01Depth`。  
①`LinearEyeDepth` 负责把深度纹理的采样结果转换到观察空间下的深度值，也就是上面的 $\,z_{view}'\,$ ；   
②`Linear01Depth` 则返回一个范围在 \[0, 1\] 的线性深度值，也就是上面得到的 $z_{01}$。  
这两个函数内置使用了内置的 `_ZBufferParams` 变量来得到远近裁剪平面的距离。

--- 

如果我们需要获取深度 + 法线纹理，可以直接使用 tex2D 函数对 \_CameraDepthNormalsTexture 进行采样，得到里面存储的深度和法线信息。Unity 提供了辅助函数来为我们对这个采样结果进行解码，从而得到深度值和法线方向。这个函数是 `DecodeDepthNormal`，它在 UnityCG.cginc 被定义：  

    inline void DecodeDepthNormal(float4 enc, out float depth, out float3 normal)
    {
        depth = DecodeFloatRG(enc.zw);
        normal = DecodeViewNormalStereo(enc);
    }

`DecodeDepthNormal` 的第一个参数是对深度 + 法线纹理的采样结果，这个采样结果是 Unity 对深度和法线信息编码后的结果，它的 xy 分量存储的是观察空间下的法线信息，而深度信息被编码进了 zw 分量。通过调用 DecodeDepthNormal 函数对采样结果解码后，我们就可以得到解码后的深度值和法线，这个深度值是范围在 \[0, 1\] 的线性深度值（**这和单独的深度纹理中存储的深度值不同**），得到的法线则是观察空间下的法线方向。同样，我们也可以通过调用 DecodeFloatRG 和 DecodeViewNormalStereo 来解码 + 深度法线纹理中的深度和法线信息。

### 查看深度和法线纹理
可以使用帧调试器查看摄像机生成的深度纹理和深度+法线纹理，图片这里不放出了，在 Camera Render 事件中的 UpdateDepthTexture 和 UpdateDepthNormalsTexture 事件中。

帧调试器看到的深度纹理是非线性空间的深度值，而深度+法线纹理都是由 Unity 编码后的结果。有时线性空间下的深度信息或解码后的法线方向会更加有用，此时可在片元着色器中做转换或解码逻辑：

``` C#
\\使用类似代码来输出线性深度值
float  depth  =  SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture,  i.uv);
float  linearDepth  =  Linear01Depth(depth);
return  fixed4(linearDepth,  linearDepth,  linearDepth,  1.0);

\\或是输出解码后的法线方向
fixed3  normal  =  DecodeViewNormalStereo(tex2D(_CameraDepthNormalsTexture,  i.uv).xy);
return  fixed4(normal  ＊  0.5  +  0.5,  1.0);
```

查看深度纹理时，如果画面几乎是全黑或全白时，可以将相机的远裁剪平面的距离（Unity 默认为1000）调小，使之刚好覆盖场景的所在区域即可。若裁剪平面的距离过大，会导致距离相机较近的物体会被映射到非常小的深度值，导致看起来全黑（场景为封闭区域比较常见）；相反，若场景为开放区域，物体距离相机较远，则会导致画面几乎全白。

## 再谈运动模糊
上一章学习了如何混合多张屏幕图像来模拟运动模糊的效果。而应用更加广泛的技术则是使用**速度映射图**。速度映射纹理中存储每个像素的速度，基于这个速度决定模糊的方向和大小。

**《GPU Gems3》** 在第 27 章(https://developer.nvidia.cn/gpugems/gpugems3/part-iv-image-effects/chapter-27-motion-blur-post-processing-effect) 中介绍了一种生成速度映射图的方法。这种方法利用深度纹理在片元着色器中为每个像素计算其在世界空间下的位置，这是通过使用当前的观察 \* 投影矩阵的逆矩阵对 NDC 下的顶点坐标进行变换得到的。当得到世界空间下的顶点坐标后，我们使用前一帧的观察 \* 投影矩阵得到该位置在前一帧中的 NDC 坐标。我们计算前一帧和当前帧的位置差，生成该像素的速度。优点是可以在一个屏幕后处理步骤中完成整个效果的模拟，但缺点是需要在片元着色器中进行两次矩阵乘法的操作，对性能有所影响。

为了使用深度纹理模拟运动模糊，需要进行如下准备工作：  
①新建名为 Scene_13_2 的场景，并关闭天空盒子；  
②搭建测试运动模糊的场景，放置 3 面墙，4 个立方体；  
③将 Translating.cs 脚本拖拽给摄像机，让其在场景中不断运动；  
④新建名为 MotionBlurWithDepthTexture.cs 脚本，并拖拽给相机；  
⑤新建名为 Chapter13-MotionBlurWithDepthTexture 的 Unity Shader；

MotionBlurWithDepthTexture.cs 脚本的 C# 代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class MotionBlurWithDepthTexture : PostEffectsBase {

    public Shader motionBlurShader;
    private Material motionBlurMaterial = null;

    public Material material {  
        get {
            motionBlurMaterial = CheckShaderAndCreateMaterial(motionBlurShader, motionBlurMaterial);
            return motionBlurMaterial;
        }  
    }

    //本节需要得到摄像机的视角和投影矩阵，定义一个 Camera 变量获取脚本所在的摄像机组件
    private Camera myCamera;
    public Camera camera {        
        get {
            if (myCamera == null) {
                myCamera = GetComponent<Camera>();
            }
            return myCamera;
        }
    }

    //定义运动模糊时模糊图像大小的参数
    [Range(0.0f, 1.0f)]
    public float blurSize = 0.5f;

    //定义一个变量来保存上一帧摄像机的视角 × 投影矩阵
    private Matrix4x4 previousViewProjectionMatrix;
    
    void OnEnable() {
        //为了获取摄像机的深度纹理，在 OnEnable 函数中设置摄像机的状态
        camera.depthTextureMode |= DepthTextureMode.Depth;

        previousViewProjectionMatrix = camera.projectionMatrix * camera.worldToCameraMatrix;
    }
    
    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            material.SetFloat("_BlurSize", blurSize);
            material.SetMatrix("_PreviousViewProjectionMatrix", previousViewProjectionMatrix); //设置矩阵参数并赋给 shader 中对应的参数

            //调用 camera.projectionMatrix 和 camera.worldToCameraMatrix 分别获得当前摄像机的视角矩阵和投影矩阵
            Matrix4x4 currentViewProjectionMatrix = camera.projectionMatrix * camera.worldToCameraMatrix;
            //取逆得到当前帧的视角*投影矩阵的逆矩阵
            Matrix4x4 currentViewProjectionInverseMatrix = currentViewProjectionMatrix.inverse;
            material.SetMatrix("_CurrentViewProjectionInverseMatrix", currentViewProjectionInverseMatrix);
            previousViewProjectionMatrix = currentViewProjectionMatrix;        //将未取逆的结果存储到 previousViewProjectionMatrix 以便在下一帧时传递给材质。

            Graphics.Blit (src, dest, material);        //若有material，则混合；若无则输出原图
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter13-MotionBlurWithDepthTexture 的 Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 13/Motion Blur With Depth Texture" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {} //输入的渲染纹理
        _BlurSize ("Blur Size", Float) = 1.0 //脚本中模糊图像的控制参数
        //我们没有声明 _PreviousViewProjectionMatrix 和 _CurrentViewProjectionInverseMatrix 属性，是因为 Unity 没有提供矩阵的属性，但是仍然可以在 CG 代码块中定义这些矩阵
    }

    SubShader {
        CGINCLUDE
        
        #include "UnityCG.cginc"
        
        sampler2D _MainTex;
        half4 _MainTex_TexelSize;
        sampler2D _CameraDepthTexture;
        float4x4 _CurrentViewProjectionInverseMatrix;
        float4x4 _PreviousViewProjectionMatrix;
        half _BlurSize;
        
        struct v2f {
            float4 pos : SV_POSITION;
            half2 uv : TEXCOORD0;
            half2 uv_depth : TEXCOORD1; //对深度纹理采样的纹理坐标变量
        };
        
        v2f vert(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            o.uv = v.texcoord;
            o.uv_depth = v.texcoord;
            
            #if UNITY_UV_STARTS_AT_TOP
            if (_MainTex_TexelSize.y < 0)
                o.uv_depth.y = 1 - o.uv_depth.y;
            #endif 
            
            return o;
        }
        
        fixed4 frag(v2f i) : SV_Target {
            float d = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv_depth); //对深度纹理进行采样得到该像素的深度缓冲值
            
            float4 H = float4(i.uv.x * 2 - 1, i.uv.y * 2 - 1, d * 2 - 1, 1);    //d 是 NDC 坐标映射而来，若要构建像素的 NDC 坐标 H，需要把这个深度值重新映射回 NDC，使用原映射的反函数，得到范围在 [-1, 1] 的 NDC 坐标。而渲染纹理的 uv 就是范围在 [0, 1] 的视口坐标，NDC 是范围在 [-1, 1] 的视口坐标，也需要映射回去
            float4 D = mul(_CurrentViewProjectionInverseMatrix, H); //逆矩阵回推
            float4 worldPos = D / D.w; //将结果除以 w 分量得到世界空间下的坐标表示 worldPos，这里除是因为 NDC 是被齐次除法后的坐标，需要乘回去，而经过逆矩阵回推的 D 的 w 分量，因为 H 的 w 分量为 1，逆矩阵会使 D 的 w 分量包含齐次除法的信息，而且这个值应该是齐次除法值的倒数，所以是除不是乘（可以尝试推导）

            float4 currentPos = H; //该像素现在的 NDC 坐标
             
            float4 previousPos = mul(_PreviousViewProjectionMatrix, worldPos); //使用前一帧的视角 × 投影矩阵对它进行变换，得到前一帧在 NDC 下的坐标 previousPos 
            previousPos /= previousPos.w; //齐次除法得到前一帧 NDC 的坐标
            
            //使用前一帧和当前帧的 NDC 的视口坐标下的位置差
            float2 velocity = (currentPos.xy - previousPos.xy) / 2.0f;
            
            float2 uv = i.uv;
            float4 c = tex2D(_MainTex, uv);

            uv += velocity * _BlurSize;
            for (int it = 1; it < 3; it++, uv += velocity * _BlurSize) {
                float4 currentColor = tex2D(_MainTex, uv); //使用速度值对像素进行偏移后进行采样
                c += currentColor;
            }
            c /= 3; //取平均值
            
            return fixed4(c.rgb, 1.0);
        }
        
        ENDCG

        //定义模糊所需要的Pass
        Pass {      
            ZTest Always Cull Off ZWrite Off
                    
            CGPROGRAM  
            
            #pragma vertex vert  
            #pragma fragment frag  
              
            ENDCG  
        }
    } 
    FallBack Off
}
```

把该脚本拖拽到摄像机的 MotionBlurWithDepthTexture.cs 脚本中的 Motion Blur Shader 参数中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/25/Jp5ENyLDY2o1TBI.gif" width = "70%" height = "70%" alt="图70-  运用速度映射图的运动模糊后的效果。"/>
</div>

注意：本节实现的运动模糊只适用于场景静止，摄像机运动的情况，因为只考虑了摄像机的运动。如果想要对快速移动的物体产生运动模糊后的效果，就需要更加精确的速度映射图，可以在 Unity 的 ImageEffect 包中找到更多的运动模糊的实现方法。

## 全局雾效
**雾效 Fog**是游戏里经常使用的一种效果。Unity 内置的雾效可以产生基于距离的线性或指数雾效。然而，想要在自己编写的顶点/片元着色器中实现这种雾效，我们需要在 Shader 中添加 `#paragma multi_compile_fog` 指令，同时还需要相关的内置宏，例如 UNITY_FOG_COORDS、UNITY_TRANSFER_FOG 和 UNITY_APPLY_FOG 等。这种方法的缺点在于，我们不仅需要为场景中所有物体添加相关的渲染代码，而且能够实现的效果也非常有限。我们需要对雾效进行一些个性化操作时，例如使用基于高度的雾效等，仅仅使用 Unity 内置的雾效就变得不再可行。

在本节中，我们将会学习一种基于屏幕后处理的全局雾效的实现。使用这种方法，我们不需要更改场景内渲染到物体所使用的 Shader 代码，而仅仅依靠一次屏幕后处理的步骤即可。这种方法的自由性很高，我们可以方便地模拟各种雾效，例如均匀的雾效、基于距离的线性/指数雾效、基于高度的雾效等。

基于屏幕后处理的全局雾效的关键是，根据深度纹理来重建每个像素在世界空间下的位置。我们在模拟运动模糊时已经实现了这个要求，即构建出当前像素的 NDC 坐标，再通过当前摄像机中的视角 * 投影矩阵的逆矩阵来得到世界空间下的像素坐标，但是，这样的实现需要在片元着色器中进行矩阵乘法的操作，而这通常会影响游戏性能。

本节中，我们将会学习一个快速从深度纹理中重构世界坐标的方法。这种方法首先对图像空间下的视锥体射线（从摄像机出发，指向图像上的某点的射线）进行插值，这条射线存储了该像素在世界空间下到摄像机的方向信息。然后，我们把该射线和线性化后的观察空间下的深度值相乘，再加上摄像机的世界位置，就可以得到该像素在世界空间下的位置。当我们得到世界坐标后，就可以轻松的使用各个公式来模拟全局雾效了。

### 重建世界坐标
坐标系中的一个顶点坐标可以通过它相对于另一个顶点坐标的偏移量来求得。重建像素的世界坐标也是基于这种思想。我们只需要知道摄像机在世界空间下的位置，以及世界空间下该像素相对于摄像机的偏移量，把它们相加就可以得到该像素的世界坐标：  

    float4 worldPos = _WorldSpaceCamearaPos + linearDepth * interpolatedRay;

其中，**\_WorldSpaceCamearaPos** 是摄像机在世界空间下的位置，这可以由 Unity 的内置变量直接访问得到。而 linearDepth * interpolatedRay 则可以计算得到该像素相对于摄像机的偏移量，**linearDepth** 是由深度纹理得到的线性深度值，**interpolatedRay** 是由顶点着色器输出并插值后得到的射线，它不仅包含了该像素到摄像机的方向，也包含了距离信息。

**interpolatedRay 来源于对近裁剪平面的 4 个角的某个特定向量的插值**，这 4 个向量包含了它们到摄像机的方向和距离信息，我们可以利用摄像机的近裁剪平面距离、FOV、横纵比计算而得。下图显示了计算时使用的一些辅助向量：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/26/gOjhfcrTqSiNUPw.jpg" width = "50%" height = "50%" alt="图71-  计算 interpolatedRay"/>
</div>

为了方便计算，我们可以先计算两个向量 —— toTop 和 toRight，它们是起点位于近裁剪平面中心、分别指向摄像机正上方和正右方的向量，它们的计算公式如下：  

$$ halfHeight = Near \times tan( \cfrac {FOV} {2} ) $$
$$ toTop = camera.up \times halfHeight $$
$$ toRight = camera.right \times halfHeight \cdot aspect $$

有了这两个辅助向量后，就可以计算 4 个角相对于摄像机的方向：  

$$ TL = camera.forward \cdot Near + toTop - toRight $$
$$ TR = camera.forward \cdot Near + toTop + toRight $$
$$ BL = camera.forward \cdot Near - toTop - toRight $$
$$ BR = camera.forward \cdot Near - toTop + toRight $$

上面求得的 4 个向量不仅包含了方向信息，它们的模对应了 4 个点到摄像机的空间距离。由于我们得到的线性深度值并非是点到摄像机的欧式距离，而是在 z 方向上的距离，不能直接使用深度值和 4 个角的单位方向的乘积来计算它们到相机的偏移量。要把深度值转换成到摄像机的欧式距离，我们以 TL 点为例，如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/26/t9m42ryzPfjTOIB.jpg" width = "40%" height = "40%" alt="图72-  采样得到的深度值并非是点到摄像机的欧式距离"/>
</div>

根据相似三角形原理，TL 所在的射线上，像素的深度值和它到摄像机的实际距离的比等于近裁剪平面的距离和 TL 向量的模的比，即：  

$$ \cfrac {depth}{dist} = \cfrac {Near}{|TL|} $$

即 TL 所在的射线上的像素距离摄像机的欧式距离 dist：  

$$ dist = \cfrac {|TL|}{Near} \times depth $$

为了得到 linearDepth * interpolatedRay 中的 interpolatedRay，需要求得近裁切平面的四个角的向量的类似于  interpolatedRay 的值作插值，以 TL 为例子，即：

$$ linearDepth \times Ray_{TL} = depth \times Ray_{TL} = dist \times \cfrac {TL}{|TL|}  $$
$$ Ray_{TL} = \cfrac {|TL|}{Near} \times depth \times \cfrac {TL}{|TL|} \div depth = \cfrac {TL}{Near}$$

因为近裁切平面的四个角相互对称，令 $\,scale = |TL| / Near\,$，则：  

$$ Ray_{TL} = \cfrac {TL}{|TL|} \times scale , Ray_{TR} = \cfrac {TR}{|TR|} \times scale $$ 
$$ Ray_{BL} = \cfrac {BL}{|BL|} \times scale , Ray_{BR} = \cfrac {BR}{|BR|} \times scale $$ 

屏幕后处理的原理就是使用特定的材质去渲染一个刚好填充整个屏幕的四边形面片，**屏幕后处理所用的模型是一个四边形网格，只包含 4 个顶点**。这个四边形面片的 4 个顶点就对应了近裁剪平面的 4 个角。因此，我们可以把上面的计算结果传递给顶点着色器，顶点着色器根据当前的位置选择它所对应的向量，然后将其输出，经插值后传递给片元着色器得到基于像素的 interpolatedRay，我们就可以直接利用本节一开始提到的公式重建该像素在世界空间下的位置了。

### 雾的计算
在简单的雾效实现中，我们需要计算一个雾效系数 f，作为混合原始颜色和雾的颜色的混合系数：  

    float3 afterFog = f * fogColor + (1 - f) * origColor

这个雾效系数 f 有很多计算方法。在 Unity 内置的雾效实现中，支持三种雾的计算方式 —— 线性 Linear、指数 Exponential 以及指数的平方 Exponential Squared。当给定距离 z 后，f 的计算公式分别如下：  
①Linear：  

$$ f= \cfrac {d_{max} - |z|}{d_{max} - d_{min}},d_{max}和d_{min}分别表示受雾影响的最大最小距离 $$

②Exponential：

$$ f= e^{-d \cdot |z|},d是控制雾的浓度的参数 $$

③Exponential Squared：

$$ f= e^{-(d \cdot |z|)^2},d是控制雾的浓度的参数 $$

在本节中，我们将使用类似线性雾的计算方式，计算基于高度的雾效。具体方法是，当给定一点在世界空间下的高度 y 后，f 的计算公式为：  

$$ f = \cfrac {H_{end} - y}{H_{end} - H_{start}},H_{start}和H_{end} 分别表示受雾影响的起始高度和终止高度 $$

### 实现
准备工作如下：  
①新建名为 Scene_13_3 的场景，并去掉天空盒子；  
②搭建一个雾效场景，构建包含 3 面墙的房间，并放置几个立方体；  
③拖拽一个控制相机不断围绕一个中心运动的脚本 Translating.cs 到相机组件；  
④新建一个名为 FogWithDepthTexture 的 C# 脚本，并拖拽给相机；
⑤新建一个名为 Chapter13-FogWithDepthTexture 的 Unity Shader。

FogWithDepthTexture.cs 的 C# 脚本代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class FogWithDepthTexture : PostEffectsBase {
    public Shader fogShader;
    private Material fogMaterial = null;

    public Material material {  
        get {
            fogMaterial = CheckShaderAndCreateMaterial(fogShader, fogMaterial);
            return fogMaterial;
        }  
    }

    private Camera myCamera;
    public Camera camera {
        get {
            if (myCamera == null) {
                myCamera = GetComponent<Camera>();
            }
            return myCamera;
        }
    }
    
    private Transform myCameraTransform;
    public Transform cameraTransform {
        get {
            if (myCameraTransform == null) {
                myCameraTransform = camera.transform;
            }
            return myCameraTransform;
        }
    }

    [Range(0.0f, 3.0f)]
    public float fogDensity = 1.0f;
    public Color fogColor = Color.white;

    //雾效的起始高度和终止高度
    public float fogStart = 0.0f; 
    public float fogEnd = 2.0f;

    void OnEnable() {
        camera.depthTextureMode |= DepthTextureMode.Depth; //设置模式以在 shader 里获取深度纹理
    }
    
    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            Matrix4x4 frustumCorners = Matrix4x4.identity; //frustum：视锥体，初始化为单位矩阵，把计算近裁剪平面的四个角对应的向量，存储到这个矩阵类型的变量

            float fov = camera.fieldOfView;
            float near = camera.nearClipPlane;
            float aspect = camera.aspect;

            //本节最开始的公式的计算过程如下
            float halfHeight = near * Mathf.Tan(fov * 0.5f * Mathf.Deg2Rad);
            Vector3 toRight = cameraTransform.right * halfHeight * aspect;
            Vector3 toTop = cameraTransform.up * halfHeight;
            Vector3 topLeft = cameraTransform.forward * near + toTop - toRight;
            float scale = topLeft.magnitude / near;
            topLeft.Normalize();
            topLeft *= scale;

            //计算各点的欧式距离
            Vector3 topRight = cameraTransform.forward * near + toRight + toTop;
            topRight.Normalize();
            topRight *= scale;

            Vector3 bottomLeft = cameraTransform.forward * near - toTop - toRight;
            bottomLeft.Normalize();
            bottomLeft *= scale;

            Vector3 bottomRight = cameraTransform.forward * near + toRight - toTop;
            bottomRight.Normalize();
            bottomRight *= scale;

            //按一定顺序把四个方向存储到 frustumCorners 的不同的行中
            frustumCorners.SetRow(0, bottomLeft);
            frustumCorners.SetRow(1, bottomRight);
            frustumCorners.SetRow(2, topRight);
            frustumCorners.SetRow(3, topLeft);

            material.SetMatrix("_FrustumCornersRay", frustumCorners);
            material.SetFloat("_FogDensity", fogDensity);
            material.SetColor("_FogColor", fogColor);
            material.SetFloat("_FogStart", fogStart);
            material.SetFloat("_FogEnd", fogEnd);

            Graphics.Blit (src, dest, material);
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter13-FogWithDepthTexture 的 Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 13/Fog With Depth Texture" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _FogDensity ("Fog Density", Float) = 1.0
        _FogColor ("Fog Color", Color) = (1, 1, 1, 1)
        _FogStart ("Fog Start", Float) = 0.0
        _FogEnd ("Fog End", Float) = 1.0
    }

    SubShader {
        CGINCLUDE
        
        #include "UnityCG.cginc"
        
        float4x4 _FrustumCornersRay;
        
        sampler2D _MainTex;
        half4 _MainTex_TexelSize;
        sampler2D _CameraDepthTexture;
        half _FogDensity;
        fixed4 _FogColor;
        float _FogStart;
        float _FogEnd;
        
        struct v2f {
            float4 pos : SV_POSITION;
            half2 uv : TEXCOORD0;
            half2 uv_depth : TEXCOORD1;
            float4 interpolatedRay : TEXCOORD2;
        };
        
        v2f vert(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            o.uv = v.texcoord;
            o.uv_depth = v.texcoord;
            
            #if UNITY_UV_STARTS_AT_TOP
            if (_MainTex_TexelSize.y < 0)
                o.uv_depth.y = 1 - o.uv_depth.y;
            #endif

            //决定该顶点对应了 4 个角中的哪个角，这里使用判断语句是因为屏幕后处理所用的模型是一个四边形网格，只包含 4 个顶点，因此对性能影响不大
            int index = 0;
            if (v.texcoord.x < 0.5 && v.texcoord.y < 0.5) {
                index = 0;
            } else if (v.texcoord.x > 0.5 && v.texcoord.y < 0.5) {
                index = 1;
            } else if (v.texcoord.x > 0.5 && v.texcoord.y > 0.5) {
                index = 2;
            } else {
                index = 3;
            }

            #if UNITY_UV_STARTS_AT_TOP
            if (_MainTex_TexelSize.y < 0)
                index = 3 - index;
            #endif
            
            o.interpolatedRay = _FrustumCornersRay[index]; //利用索引值来获取 _FrustumCornersRay 中对应的行
              
            return o;
        }
        
        fixed4 frag(v2f i) : SV_Target {
            float linearDepth = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv_depth)); //对深度纹理采样后，使用 LinearEyeDepth 得到观察空间下的线性深度值
            float3 worldPos = _WorldSpaceCameraPos + linearDepth * i.interpolatedRay.xyz;
                        
            float fogDensity = (_FogEnd - worldPos.y) / (_FogEnd - _FogStart); 
            fogDensity = saturate(fogDensity * _FogDensity); //控制雾效系数
            fixed4 finalColor = tex2D(_MainTex, i.uv);
            finalColor.rgb = lerp(finalColor.rgb, _FogColor.rgb, fogDensity);
            return finalColor;
        }
        
        ENDCG
        
        Pass {
            ZTest Always Cull Off ZWrite Off
                     
            CGPROGRAM  
            
            #pragma vertex vert  
            #pragma fragment frag  
              
            ENDCG  
        }
    } 
    FallBack Off
}
```

把 Chapter13-FogWithDepthTexture 拖拽到摄像机的脚本的 Fog Shader 参数中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/26/VpA56mbuRcKTIMg.jpg" width = "70%" height = "70%" alt="图73-  添加全局雾效后的效果"/>
</div>

**本节介绍的使用深度纹理重构像素的世界坐标的方法是非常有用的**。但是需要注意的是，这里的实现是基于摄影机的投影类型是透视投影的前提下。若需要在正交投影的情况下重建世界坐标，需要使用不同的公式，有兴趣可以自己推导。

## 再谈边缘检测
在上一章节，使用的是 **Sobel 算子** 对屏幕图像进行边缘检测，这种直接利用颜色信息进行边缘检测的方法会受纹理、阴影等因素的影响，导致最终的描边并不精确。

在本节中，将利用深度和法线纹理，通过 **Roberts 算子**进行边缘检测，不受纹理和光照影响。Roberts 算子使用的卷积核如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/26/V6PIaFLX7sToSvt.jpg" width = "20%" height = "20%" alt="图74-   Roberts 算子"/>
</div>

Roberts 算子的本质是就是计算左上角和右上角的差值，乘以右上角和左下角的差值，作为评估边缘的依据。在下面的实现中，我们也会按照这样的方式，取对角方向的深度或法线值，比较它们之间的差值，如果超过某个阈值，就认为它们之间存在一条边。

准备工作如下：  
①新建名为 Scene_13_4 的场景，并去掉天空盒；  
②搭建类似上一节全局雾效的场景；   
③将上一节中用到的 Translating.cs 脚本拖拽给相机；  
④新建名为 EdgeDetectNormalsAndDepth 的 C# 脚本，并拖拽给相机；  
⑤新建名为 Chapter13-EdgeDetectNormalAndDepth 的 Unity Shader；  

EdgeDetectNormalsAndDepth.cs 的 C# 脚本代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class EdgeDetectNormalsAndDepth : PostEffectsBase {
    public Shader edgeDetectShader;
    private Material edgeDetectMaterial = null;
    public Material material {  
        get {
            edgeDetectMaterial = CheckShaderAndCreateMaterial(edgeDetectShader, edgeDetectMaterial);
            return edgeDetectMaterial;
        }  
    }

    [Range(0.0f, 1.0f)]
    public float edgesOnly = 0.0f;
    public Color edgeColor = Color.black;
    public Color backgroundColor = Color.white;
    public float sampleDistance = 1.0f; //用于控制对深度+法线纹理采样时，使用的采样距离。从视觉上看，为描边粗细程度

    //sensitivityDepth 和 sensitivityNormals 则影响当邻域的深度值或法线值相差多少时被认为存在一条边界，而如果灵敏度调的很大，那么可能是深度或法线上很小的变化也会成为一条边
    public float sensitivityDepth = 1.0f;
    public float sensitivityNormals = 1.0f;
    
    void OnEnable() {
        GetComponent<Camera>().depthTextureMode |= DepthTextureMode.DepthNormals;
    }

    //ImageEffectOpaque 特性，见第 11 章最前面，目的是为了在不透明的 Pass 执行完毕后立即调用该函数，不对透明物体产生影响。在本例中，不希望对透明物体也被描边
    [ImageEffectOpaque]
    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            material.SetFloat("_EdgeOnly", edgesOnly);
            material.SetColor("_EdgeColor", edgeColor);
            material.SetColor("_BackgroundColor", backgroundColor);
            material.SetFloat("_SampleDistance", sampleDistance);
            material.SetVector("_Sensitivity", new Vector4(sensitivityNormals, sensitivityDepth, 0.0f, 0.0f));

            Graphics.Blit(src, dest, material);
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter13-EdgeDetectNormalAndDepth 的 Shader 代码如下：

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 13/Edge Detection Normals And Depth" {
    Properties {
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _EdgeOnly ("Edge Only", Float) = 1.0
        _EdgeColor ("Edge Color", Color) = (0, 0, 0, 1)
        _BackgroundColor ("Background Color", Color) = (1, 1, 1, 1)
        _SampleDistance ("Sample Distance", Float) = 1.0
        _Sensitivity ("Sensitivity", Vector) = (1, 1, 1, 1)
    }

    SubShader {
        CGINCLUDE
        
        #include "UnityCG.cginc"
        
        sampler2D _MainTex;
        half4 _MainTex_TexelSize;
        fixed _EdgeOnly;
        fixed4 _EdgeColor;
        fixed4 _BackgroundColor;
        float _SampleDistance;
        half4 _Sensitivity;
        
        sampler2D _CameraDepthNormalsTexture;
        
        struct v2f {
            float4 pos : SV_POSITION;
            half2 uv[5]: TEXCOORD0; //定义维数为 5 的纹理坐标数组，第一个坐标存储了屏幕颜色图像的采样纹理，剩下四个存储使用 Roberts 算子时需要采样的纹理坐标
        };
          
        v2f vert(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            half2 uv = v.texcoord;
            o.uv[0] = uv;
            
            #if UNITY_UV_STARTS_AT_TOP //解决平台差异性
            if (_MainTex_TexelSize.y < 0)
                uv.y = 1 - uv.y;
            #endif

            //计算罗伯特算子，使用 _SampleDistance 控制采样距离
            o.uv[1] = uv + _MainTex_TexelSize.xy * half2(1,1) * _SampleDistance;
            o.uv[2] = uv + _MainTex_TexelSize.xy * half2(-1,-1) * _SampleDistance;
            o.uv[3] = uv + _MainTex_TexelSize.xy * half2(-1,1) * _SampleDistance;
            o.uv[4] = uv + _MainTex_TexelSize.xy * half2(1,-1) * _SampleDistance;
            return o;
        }
        
        half CheckSame(half4 center, half4 sample) {
            //对输入的两个点的采样后的深度和法线信息进行处理。注意，这里没有解码得到真正的法线值，而是直接使用了 xy 分量。是因为我们只需要比较两个采样值的差异，不需要知道它们真正的法线
            half2 centerNormal = center.xy;        
            float centerDepth = DecodeFloatRG(center.zw);        
            half2 sampleNormal = sample.xy;
            float sampleDepth = DecodeFloatRG(sample.zw);

            //两个采样点的对应值相减并取绝对值在乘以灵敏度参数，再把差异值的每个分量相加和一个阈值比较，若小于阈值则返回 1，说明差异不明显，否则则返回 0
            half2 diffNormal = abs(centerNormal - sampleNormal) * _Sensitivity.x;
            int isSameNormal = (diffNormal.x + diffNormal.y) < 0.1;

            //用偏差是否达到深度的十分之一来判定深度值是否相近
            float diffDepth = abs(centerDepth - sampleDepth) * _Sensitivity.y;
            int isSameDepth = diffDepth < 0.1 * centerDepth;
            
            // return:
            // 1 - if normals and depth are similar enough
            // 0 - otherwise
            return isSameNormal * isSameDepth ? 1.0 : 0.0;
        }
        
        fixed4 fragRobertsCrossDepthAndNormal(v2f i) : SV_Target {
            //使用 4 个纹理坐标对深度+法线纹理进行法线采样
            half4 sample1 = tex2D(_CameraDepthNormalsTexture, i.uv[1]);
            half4 sample2 = tex2D(_CameraDepthNormalsTexture, i.uv[2]);
            half4 sample3 = tex2D(_CameraDepthNormalsTexture, i.uv[3]);
            half4 sample4 = tex2D(_CameraDepthNormalsTexture, i.uv[4]);
            
            half edge = 1.0;
            edge *= CheckSame(sample1, sample2); //调用 checksame 计算对角线上两个纹理值的差值，若返回 0 则两点间存在一条边界，反之则为 1
            edge *= CheckSame(sample3, sample4);
            
            fixed4 withEdgeColor = lerp(_EdgeColor, tex2D(_MainTex, i.uv[0]), edge);
            fixed4 onlyEdgeColor = lerp(_EdgeColor, _BackgroundColor, edge);
            return lerp(withEdgeColor, onlyEdgeColor, _EdgeOnly);
        }
        
        ENDCG

        Pass { 
            ZTest Always Cull Off ZWrite Off
            
            CGPROGRAM      
            
            #pragma vertex vert  
            #pragma fragment fragRobertsCrossDepthAndNormal
            
            ENDCG  
        }
    } 
    FallBack Off
}
```

将 Chapter13-EdgeDetectNormalAndDepth 拖入摄像机脚本的 Edge Detect Shader 参数中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/27/5DIAmgKqwtXaZhV.png" width = "60%" height = "60%" alt="图75-   在深度和法线纹理上进行更健壮的边缘检测。上图：在原图上描边的效果（Edges only 为 0）。右图：只显示描边的效果（Edges only 为 1）"/>
</div>

本节实现的描边效果是基于整个屏幕空间进行的，场景内所有物体都会被添加描边效果。但有时，我们希望只对特定的物体进行描边，这时需要 Unity 提供的 Graphics.DrawMesh 或 Graphics.DrawMeshNow 函数，把需要描边的物体再次渲染一次（在所有不透明物体渲染完毕之后），再使用本节提到的边缘检测算法计算深度或法线纹理中每个像素的梯度值，判断它们是否小于阈值。若是，则在 Shader 中使用 clip() 函数将该像素剔除掉，从而显示出原来的颜色。

## 扩展阅读
在本章中，我们只使用了深度和法线纹理，但实际上我们可以在 Unity 中创建任何需要的缓存纹理。这可以通过使用 Unity 的着色器替换 shader replacement 功能，即调用 Camera.RenderWithShader(shader, replacementTag) 函数，把整个场景再次渲染一遍来得到，而在很多时候，这实际上也是 Unity 创建深度和法线纹理时使用的方法。

深度和法线纹理在屏幕特效的实现中扮演很重要的角色，很多特殊的屏幕效果都可以基于深度和法线纹理去实现，原作者给出了 Unity 在 2011 年的 SIGGRAPH（计算机图形学的顶级会议）上做的一个关于使用深度纹理实现各种特效的演讲，感兴趣可前往阅读：https://blog.unity.com/community/special-effects-with-depth-talk-at-siggraph


# 第十三章 非真实感渲染
一些游戏渲染是以**照相写实主义 photorealism** 作为主要目标，但也有一些游戏使用了**非真实感渲染 Non-Photorealistic Rendering, NPR** 的方法来渲染游戏画面。非真实感渲染的一个主要目标是，使用一些渲染方法使得画面达到和某些特殊的绘画风格相似的效果，例如卡通、水彩风格等。

## 卡通风格的渲染
要实现卡通渲染有很多方法，其中之一就是使用**基于色调的着色技术 tone-based shading**。在实现中，我们往往会使用漫反射系数对一张一维纹理进行采样，以控制漫反射的色调，也就是第六章实现的渐变纹理。

卡通的高光效果和之前学习的光照不同，模型的高光往往是一块块分界明显的纯色区域。

除了光照模型不同外，卡通风格通常还需要在物体边缘部分绘制轮廓。在上一章节，使用的是屏幕后处理技术对屏幕图像进行描边。在本节中，将介绍基于模型的描边方法，这种方法的实现更加简单，而且大多数情况下效果不错。

### 渲染轮廓线
在实时渲染中，轮廓线的渲染应用广泛。在《Real Time Rendering, third edition》中，作者将绘制模型轮廓的方法分为了 5 种：  
①基于观察角度和表面法线的轮廓线渲染：使用视角方向和表面法线的点乘结果来得到轮廓线的信息。这种方法简单快捷，可以在一个 Pass 中就得到渲染结果，但局限性很大，很多模型渲染出来的描边效果都不尽人意。  
②过程式几何轮廓渲染：使用两个 Pass 渲染。第一个 Pass 渲染背面的面片，并使用某些技术让它的轮廓可见；第二个 Pass 再正常渲染正面的面片。这种方法的优点在于快速有效，并且适用于绝大多数表面平滑的模型，但它的缺点是不适用于类似于立方体这样的平整的模型。  
③基于图像处理的轮廓线渲染：我们在 11、12 章介绍的边缘检测的方法就属于这个类别。这种方法的优点在于，可以适用于任何种类的模型。但它也有自身的局限所在，一些深度和法线变化很小的轮廓无法被检测出来，例如桌子上的纸张。  
④基于轮廓边检测的轮廓线渲染：上面提到的各种方法，一个最大的问题是，无法控制轮廓线的风格渲染。对于一些情况，我们希望可以渲染出独特风格的轮廓线，例如水墨风格等。为此，我们希望可以检测出精确的轮廓边，然后直接渲染它们。检测一条边是否是轮廓边的公式也很简单，我们只需要检查和这条边相邻的两个三角面片是否满足以下条件：

$$ (n_0 \cdot v > 0) \neq (n_1 \cdot v > 0)$$

其中，$\,n_0\,$ 和 $\,n_1\,$ 分别表示两个相邻三角面片的法向，v 是从视角到该边上任意顶点的方向。上述公式的本质在于检查两个相邻的三角面片是否一个朝正面、一个朝背面。我们可以在几何着色器 Geometry Shader 的帮助下实现上面的检测过程。当然，这种方法也有缺点，除了实现相对复杂外，它还会有动画连贯性的问题。也就是说，由于是逐帧单独提取轮廓，所以在帧与帧之间会出现跳跃性。  
⑤最后一个种类就是混合了上述几种渲染方法。例如，首先找到精确的轮廓边，把模型和轮廓边渲染到纹理中，再使用图像处理的方法识别出轮廓线，并在图像空间下进行风格化渲染。

--- 

在本节中，我们将会在 Unity 中使用**过程式几何轮廓线渲染**的方法来对模型进行轮廓描边。我们将使用两个 Pass 渲染模型：在第一个 Pass 中，我们会使用轮廓线颜色渲染整个背面的面片，并在视角(观察)空间下把模型顶点沿着法线方向向外扩张一段距离，一次来让背部轮廓线可见。代码如下：  

    viewPos = viewPos + viewNormal * _Outline;

但是，如果直接使用顶点法线进行扩展，对于一些内凹的模型，就可能发生背面面片遮挡正面面片的情况。为了尽可能防止出现这样的情况，在扩张背面顶点之前，我们首先对顶点法线的 z 分量进行处理，使它们等于一个定值，然后把法线归一化再对顶点进行扩张。这样的好处在于，扩展后的背面更加扁平化，从而降低了遮挡正面面片的可能性。代码如下：  

    viewNormal.z = -0.5;
    viewNormal = normalize(viewNormal);
    viewPos = viewPos + viewNormal * _Outline;

### 添加高光
卡通风格中的高光是模型上一块块分界明显的纯色区域。因此不能在使用之前学习的光照模型，回顾一下之前实现的 Blinn-Phong 模型，我们使用法线点乘光照方向以及视角方向和的一半，再和另一个参数进行指数操作得到高光反射系数，代码如下：  

    float spec = pow(max(0, dot(normal, halfDir)), _Gloss);

对于卡通渲染需要的高光反射光照模型，我们同样需要计算 normal 和 halfDir 的点乘结果，但不同的是，我们把该值和一个阈值进行比较，如果小于该阈值，则高光反射系数为 0，否则返回 1：  

    float spec = dot(worldNormal, worldHalfDir);
    spec = step(threshold, spec);

在上面的代码中，我们使用 Cg 的 `step` 函数来实现和阈值比较的目的。step 函数接受两个参数，第一个参数是参考值，第二个参数是待比较的数值。如果第二个参数大于等于第一个参数，则返回 1，否则返回 0。

但是，这种粗暴的判断方法会在高光区域的边界造成锯齿，因为高光区域的边缘不是平滑渐变的，而是从 0 突破到 1。要想对其进行抗锯齿处理，我们可以在边界很小的一块区域内，进行平滑处理，代码如下：  

    float spec = dot(worldNormal, worldHalfDir);
    spec = lerp(0, 1, smoothstep(-w, w, spec - threshold))

使用了 Cg 的 `smoothstep` 函数。其中，w 是一个很小的值，当 spec - threshold 小于 -w 时，返回 0，大于 w 时，返回 1，否则在 0 到 1 之间进行插值。这样的效果是，我们可以在 \[-w, w\] 区间内，即高光区域的边界处，得到一个从 0 到 1 平滑变化的 spec 值，从而实现抗锯齿的目的。尽管我们可以把 w 设为一个很小的定值，但在本例中，我们选择使用领域像素之间的近似导数值，这可以通过 CG 的 `fwidth` 函数来得到。

>     ddx(v) = 该像素点右边的值 - 该像素点的值  
>     ddy(v) = 该像素点下面的值 - 该像素点的值  
>     fwidth（v） = abs(ddx(v)) + abs(ddy(v))  //邻域像素之间的近似导数值  
> 当代 GPU 在像素化的时候一般是以 2 x 2 像素为基本单位，那么在这个 2 x 2 像素块当中，右侧的像素对应的 fragment 的 x 坐标减去左侧的像素对应的 fragment 的 x 坐标就是ddx；下侧像素对应的 fragment 的坐标 y 减去上侧像素对应的 fragment 的坐标 y 就是 ddy，ddx 和 ddy 代表了相邻两个像素在设备坐标系当中的距离。

### 实现
准备工作如下：  
①新建名为 Scene_14_1 的场景，并去掉天空盒；  
②往场景中拖入一个 Suzanne 模型；  
③新建名为 ToonShadingMat 的材质，并赋给上一步的模型；  
④新建名为 Chapter14-ToonShading 的 Unity Shader，并赋给上一步的材质。

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 14/Toon Shading" {
    Properties {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _MainTex ("Main Tex", 2D) = "white" {}
        _Ramp ("Ramp Texture", 2D) = "white" {}
        _Outline ("Outline", Range(0, 1)) = 0.1 //控制轮廓线宽度
        _OutlineColor ("Outline Color", Color) = (0, 0, 0, 1) 
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _SpecularScale ("Specular Scale", Range(0, 0.1)) = 0.01 //控制高光反射的阈值
    }
    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        Pass {
            NAME "OUTLINE" //定义名称，方便其他 shader 调用
            
            Cull Front //把正面的三角面片剔除掉，只渲染背面
            
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "UnityCG.cginc"
            
            float _Outline;
            fixed4 _OutlineColor;
            
            struct a2v {        
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            }; 
                
            struct v2f {        
                float4 pos : SV_POSITION;
            };
            
            v2f vert (a2v v) {
                v2f o;
                //先变换到观察空间
                float4 pos = mul(UNITY_MATRIX_MV, v.vertex); 
                float3 normal = mul((float3x3)UNITY_MATRIX_IT_MV, v.normal);
                //观察空间是右手坐标系，所以是减
                normal.z = -0.5;
                pos = pos + float4(normalize(normal), 0) * _Outline;
                o.pos = mul(UNITY_MATRIX_P, pos); //变换到裁剪空间中
                return o;
            }
            
            float4 frag(v2f i) : SV_Target {
                return float4(_OutlineColor.rgb, 1); //渲染着整个背面即可       
            }
            
            ENDCG
        }
        
        Pass {
            Tags { "LightMode"="ForwardBase" } //前向渲染标签，光照模型需要 Unity 提供光照等信息，从而要对pass进行相应的设置
            
            Cull Back
        
            CGPROGRAM
        
            #pragma vertex vert
            #pragma fragment frag
            
            #pragma multi_compile_fwdbase
        
            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            #include "UnityShaderVariables.cginc"
            
            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _Ramp;
            fixed4 _Specular;
            fixed _SpecularScale;
        
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
                float4 tangent : TANGENT;
            }; 
        
            struct v2f {
                float4 pos : POSITION;
                float2 uv : TEXCOORD0;
                float3 worldNormal : TEXCOORD1;        
                float3 worldPos : TEXCOORD2;    
                SHADOW_COORDS(3) //阴影相关宏
            };
            
            v2f vert (a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos( v.vertex);
                o.uv = TRANSFORM_TEX (v.texcoord, _MainTex);
                o.worldNormal  = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                TRANSFER_SHADOW(o); //阴影相关宏
                return o;
            }
            
            float4 frag(v2f i) : SV_Target { 
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 worldHalfDir = normalize(worldLightDir + worldViewDir);
                
                fixed4 c = tex2D (_MainTex, i.uv);
                fixed3 albedo = c.rgb * _Color.rgb;
                
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos); //利用内置宏计算当前世界坐标下的阴影值

                //半兰伯特漫反射系数和阴影值相乘得到最终的漫反射系数
                fixed diff =  dot(worldNormal, worldLightDir);
                diff = (diff * 0.5 + 0.5) * atten;
                
                fixed3 diffuse = _LightColor0.rgb * albedo * tex2D(_Ramp, float2(diff, diff)).rgb;
                
                fixed spec = dot(worldNormal, worldHalfDir);
                fixed w = fwidth(spec) * 2.0; //抗锯齿操作
                fixed3 specular = _Specular.rgb * lerp(0, 1, smoothstep(-w, w, spec + _SpecularScale - 1)) * step(0.0001, _SpecularScale); //0.0001是为了在 _SpecularScale 为 0 时完全消除高光反射的光照
                
                return fixed4(ambient + diffuse + specular, 1.0); //环境光、漫反射和高光反射相加的结果
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```

效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/28/vxXMywPiJB6pWNK.jpg" width = "70%" height = "70%" alt="图76- 卡通风格的渲染效果"/>
</div>

本节实现的卡通渲染光照模型是一种非常简单的实现，要获取更出色的卡通效果，需自行查阅更多资料。


## 素描风格的渲染
另一个非常流行的非真实渲染时素描风格的渲染。微软研究院的 Praun 等人在 2001 年的 SIGGRAPH 上发表了一篇非常著名的论文。在这篇文章中，他们使用了提前生成的素描纹理来实现实时的素描风格渲染，这些纹理组成了一个**色调艺术映射 Tonal Art Map，TAM**，如下图所示：

<div  align="center">  
<img src="https://s2.loli.net/2023/12/28/8ChEtLKZicqxYJ2.jpg" width = "70%" height = "70%" alt="图77- 一个 TAM 的例子（来源：Praun E, et al. Real-time hatching）"/>
</div>

上图中，从左到右纹理中的笔触逐渐增多，用于模拟不同光照下的漫反射效果，从上到下对应了每张纹理的**多级渐远纹理 mipmaps**。这些纹理的生成并不是简单的对上一层纹理进行降采样，而是需要保持笔触之间的间隔，以便真实模拟素描的效果。

本节将会实现简化版的论文中提出的算法，我们不考虑多级渐远纹理的生成，而直接使用 6 张纹理进行渲染。在渲染阶段，我们首先在顶点着色阶段计算逐顶点的光照，根据光照结果来决定 6 张纹理的混合权重，并传递给片元着色器。然后，在片元着色器中根据这些权重来混合 6 张纹理的采样结果。

准备工作如下：  
①新建名为 Scene_14_2 的场景，并去掉天空盒；  
②拖拽 TeddyBear 模型到场景。可以往场景中拖入一张纸张图像作为背景；  
③新建名为 HatchingMat 的材质，并赋给上一步的模型；  
④新建名为 Chapter14-Hatching 的 Unity Shader，并赋给上一步创建的材质。

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 14/Hatching" {
    Properties {
        _Color ("Color Tint", Color) = (1, 1, 1, 1)
        _TileFactor ("Tile Factor", Float) = 1 //纹理的平铺系数，越大，素描线条越密
        _Outline ("Outline", Range(0, 1)) = 0.1 //描边的粗细
        //渲染时的6张素描纹理，其线条密度依次增加
        _Hatch0 ("Hatch 0", 2D) = "white" {}
        _Hatch1 ("Hatch 1", 2D) = "white" {}
        _Hatch2 ("Hatch 2", 2D) = "white" {}
        _Hatch3 ("Hatch 3", 2D) = "white" {}
        _Hatch4 ("Hatch 4", 2D) = "white" {}
        _Hatch5 ("Hatch 5", 2D) = "white" {}
    }
    
    SubShader
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        UsePass "Unity Shaders Book/Chapter 14/Toon Shading/OUTLINE" //使用卡通渲染的 pass 进行渲染轮廓，别忘了 Pass 名字要全部转为大写
        
        Pass {
            Tags { "LightMode"="ForwardBase" }
            
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag 
            
            #pragma multi_compile_fwdbase
            
            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            #include "UnityShaderVariables.cginc"
            
            fixed4 _Color;
            float _TileFactor;
            sampler2D _Hatch0;
            sampler2D _Hatch1;
            sampler2D _Hatch2;
            sampler2D _Hatch3;
            sampler2D _Hatch4;
            sampler2D _Hatch5;
            
            struct a2v {
                float4 vertex : POSITION;
                float4 tangent : TANGENT; 
                float3 normal : NORMAL; 
                float2 texcoord : TEXCOORD0; 
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
                //因为声明了 6 张纹理，从而需要 6 个混合权重，把其存储在两个 fixed3 类型的变量中
                fixed3 hatchWeights0 : TEXCOORD1;
                fixed3 hatchWeights1 : TEXCOORD2;
                float3 worldPos : TEXCOORD3; //为添加阴影，从而需要声明 worldPos 变量
                SHADOW_COORDS(4) //使用 SHADOW_COORDS 宏声明阴影纹理的采样坐标
            };
            
            v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = v.texcoord.xy * _TileFactor; //使用 _TileFactor 得到纹理采样坐标
                
                fixed3 worldLightDir = normalize(WorldSpaceLightDir(v.vertex));
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                fixed diff = max(0, dot(worldLightDir, worldNormal)); //漫反射系数

                o.hatchWeights0 = fixed3(0, 0, 0); //把权重值初始化定位0
                o.hatchWeights1 = fixed3(0, 0, 0);

                //把 diff 缩放到 [0,7] 得到 hatchFactor，通过 hatchFactor 来计算其所处的自区间来计算对应的纹理混合权重
                float hatchFactor = diff * 7.0;
                
                if (hatchFactor > 6.0) {
                    // Pure white, do nothing
                } else if (hatchFactor > 5.0) {
                    o.hatchWeights0.x = hatchFactor - 5.0;
                } else if (hatchFactor > 4.0) {
                    o.hatchWeights0.x = hatchFactor - 4.0;
                    o.hatchWeights0.y = 1.0 - o.hatchWeights0.x;
                } else if (hatchFactor > 3.0) {
                    o.hatchWeights0.y = hatchFactor - 3.0;
                    o.hatchWeights0.z = 1.0 - o.hatchWeights0.y;
                } else if (hatchFactor > 2.0) {
                    o.hatchWeights0.z = hatchFactor - 2.0;
                    o.hatchWeights1.x = 1.0 - o.hatchWeights0.z;
                } else if (hatchFactor > 1.0) {
                    o.hatchWeights1.x = hatchFactor - 1.0;
                    o.hatchWeights1.y = 1.0 - o.hatchWeights1.x;
                } else {
                    o.hatchWeights1.y = hatchFactor;
                    o.hatchWeights1.z = 1.0 - o.hatchWeights1.y;
                }
                
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                TRANSFER_SHADOW(o); //计算阴影纹理的采样坐标
                return o; 
            }
            
            fixed4 frag(v2f i) : SV_Target {
                //混合权重，对每张纹理进行采样，并和他们对应的权重值相乘得到每张纹理的采样颜色
                fixed4 hatchTex0 = tex2D(_Hatch0, i.uv) * i.hatchWeights0.x;
                fixed4 hatchTex1 = tex2D(_Hatch1, i.uv) * i.hatchWeights0.y;
                fixed4 hatchTex2 = tex2D(_Hatch2, i.uv) * i.hatchWeights0.z;
                fixed4 hatchTex3 = tex2D(_Hatch3, i.uv) * i.hatchWeights1.x;
                fixed4 hatchTex4 = tex2D(_Hatch4, i.uv) * i.hatchWeights1.y;
                fixed4 hatchTex5 = tex2D(_Hatch5, i.uv) * i.hatchWeights1.z;

                //计算纯白在渲染中的贡献度，主要是为了光照最亮的部分是纯白色，即素描的留白部分
                fixed4 whiteColor = fixed4(1, 1, 1, 1) * (1 - i.hatchWeights0.x - i.hatchWeights0.y - i.hatchWeights0.z - i.hatchWeights1.x - i.hatchWeights1.y - i.hatchWeights1.z);
                
                fixed4 hatchColor = hatchTex0 + hatchTex1 + hatchTex2 + hatchTex3 + hatchTex4 + hatchTex5 + whiteColor;
                
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos); //得到阴影值
                                
                return fixed4(hatchColor.rgb * _Color.rgb * atten, 1.0);
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```

效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/28/PRnjh2JEz5oZC3A.jpg" width = "70%" height = "70%" alt="图78- 素描风格的渲染效果"/>
</div>


# 第十四章 使用噪声
## 消融效果
**消融 dissolve** 效果常见于游戏中的角色死亡、地图烧毁等效果。在这些效果中，消融往往从不同的区域开始，并向看似随机的方向扩张，最后整个物体都将消失不见。在本节中，我们将在 Unity 中实现这种效果。

消融效果的原理：概括来说就是噪声纹理+透明度测试。我们使用噪声纹理采样的结果和某个控制消融程度的阈值比较，如果小于阈值，就使用 clip 函数把它对应的像素裁剪掉，这些部分就对应了被“烧毁”的区域。而镂空区域边缘的烧焦效果则是将两种颜色混合，再用 pow 函数处理后，与原纹理颜色混合后的的结果。

准备工作如下：  
①新建名为 Scene_15_1 的场景，并去掉天空盒；  
②往场景中放置一个立方体；  
③新建名为 DissolveMat 的材质，并赋给上一步创建的立方体；  
④新建名为 Chapter15-Dissolve 的 Unity Shader，并赋给上一步创建的材质。

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 15/Dissolve" {
    Properties {
        _BurnAmount ("Burn Amount", Range(0.0, 1.0)) = 0.0 //控制燃烧的效果
        _LineWidth("Burn Line Width", Range(0.0, 0.2)) = 0.1 //控制燃烧烧焦效果时的线宽
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _BumpMap ("Normal Map", 2D) = "bump" {}
        _BurnFirstColor("Burn First Color", Color) = (1, 0, 0, 1) //燃烧边界的第一种颜色
        _BurnSecondColor("Burn Second Color", Color) = (1, 0, 0, 1) //燃烧边界的渐变的第二种颜色
        _BurnMap("Burn Map", 2D) = "white"{} //对应的噪声纹理
    }
    SubShader {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        
        Pass {
            Tags { "LightMode"="ForwardBase" }

            Cull Off
            
            CGPROGRAM
            
            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            
            #pragma multi_compile_fwdbase
            
            #pragma vertex vert
            #pragma fragment frag

            fixed _BurnAmount;
            fixed _LineWidth;
            sampler2D _MainTex;
            sampler2D _BumpMap;
            fixed4 _BurnFirstColor;
            fixed4 _BurnSecondColor;
            sampler2D _BurnMap;
            float4 _MainTex_ST;
            float4 _BumpMap_ST;
            float4 _BurnMap_ST;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                float4 texcoord : TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float2 uvMainTex : TEXCOORD0;
                float2 uvBumpMap : TEXCOORD1;
                float2 uvBurnMap : TEXCOORD2;
                float3 lightDir : TEXCOORD3;
                float3 worldPos : TEXCOORD4;
                SHADOW_COORDS(5) //阴影纹理
            };
            
            v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uvMainTex = TRANSFORM_TEX(v.texcoord, _MainTex);
                o.uvBumpMap = TRANSFORM_TEX(v.texcoord, _BumpMap);
                o.uvBurnMap = TRANSFORM_TEX(v.texcoord, _BurnMap);

                //把光源信息从模型空间变换到切线空间
                TANGENT_SPACE_ROTATION;
                o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
                  
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                TRANSFER_SHADOW(o); //得到阴影信息
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                fixed3 burn = tex2D(_BurnMap, i.uvBurnMap).rgb;

                //将采样结果和用于控制消融效果的属性 _BurnAmount 相减并传递给 clip 函数，如果结果小于 0 则该像素将会被剔除，从而不会显示到屏幕上，而如果通过了测试，则将进行正常的光照效果
                clip(burn.r - _BurnAmount);

                //如果通过了测试，则进行正常的光照计算
                float3 tangentLightDir = normalize(i.lightDir);
                fixed3 tangentNormal = UnpackNormal(tex2D(_BumpMap, i.uvBumpMap));
                fixed3 albedo = tex2D(_MainTex, i.uvMainTex).rgb;
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
                fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));

                //计算燃烧颜色 burncolor，在 _LineWidth 范围内模拟烧焦的颜色变化，使用 smoothstep 函数计算混合系数 t，当 t 为 1 时，表明该像素位于消融的边界，当 t 为 0 时，表明该像素为正常的模型颜色
                fixed t = 1 - smoothstep(0.0, _LineWidth, burn.r - _BurnAmount);
                fixed3 burnColor = lerp(_BurnFirstColor, _BurnSecondColor, t); //混合两种火焰颜色
                burnColor = pow(burnColor, 5); //pow 让其更加接近烧焦效果
                
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);        
                fixed3 finalColor = lerp(ambient + diffuse * atten, burnColor, t * step(0.0001, _BurnAmount)); //使用 t 来混合正常的光照颜色（环境光+漫反射）和烧焦颜色，而 step 是保证 _BurnAmount 为 0 时不显示任何消融效果
                
                return fixed4(finalColor, 1);
            }
            ENDCG
        }

        //自定义投射阴影的 pass，让阴影能够配合透明度测试产生正确的效果
        Pass {
            Tags { "LightMode" = "ShadowCaster" }
            
            CGPROGRAM
            
            #pragma vertex vert
            #pragma fragment frag
            
            #pragma multi_compile_shadowcaster
            
            #include "UnityCG.cginc"
            
            fixed _BurnAmount;
            sampler2D _BurnMap;
            float4 _BurnMap_ST;
            
            struct v2f {
                V2F_SHADOW_CASTER; //得到定义阴影投射所需要定义的变量
                float2 uvBurnMap : TEXCOORD1;
            };
            
            v2f vert(appdata_base v) {
                v2f o;
                TRANSFER_SHADOW_CASTER_NORMALOFFSET(o) //该宏用于填充 V2F_SHADOW_CASTER 背后声明的一些变量
                o.uvBurnMap = TRANSFORM_TEX(v.texcoord, _BurnMap);
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                fixed3 burn = tex2D(_BurnMap, i.uvBurnMap).rgb;
                clip(burn.r - _BurnAmount); //利用噪声纹理的采样结果 uvBurnMap 来剔除片元
                SHADOW_CASTER_FRAGMENT(i) //完成阴影投射部分，把结果输出到深度图和阴影映射纹理中
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```

将噪声纹理拖拽到材质的 Burn Map 属性上，再调整材质的 Burn Amount 属性，就可以看到消融的效果了。可以自己实现一个辅助脚本用于控制材质的 Burn Amount 属性，或者使用 Shader 动画来实现动画效果。效果如下：  

<table><tr>
<td><img src='https://s2.loli.net/2023/12/29/i8PvuV4ZhpKW2EX.jpg' width="330" alt="图79- 消融效果使用的噪声纹理"></td>
<td><img src='https://s2.loli.net/2023/12/29/DPnZNSjpFLer56R.gif' width="600" alt="图80- 箱子的消融效果"></td>
</tr></table>

## 水波效果
在模拟水面的过程中，我们往往也会使用噪声纹理。此时，噪声纹理通常会用作一个高度图，以不断修改水面的法线方向。为了模拟水不断流动的效果，我们会使用和时间相关的变量来对噪声纹理进行采样，当得到法线信息后，再进行正常的反射 + 折射计算，得到最后的水面波动效果。

本节中，我们将使用一个由噪声纹理得到的法线贴图，实现一个包含菲涅尔反射的水面效果。和第 9 章实现的透明玻璃类似。首先使用一张立方体纹理 Cubemap 作为环境纹理，模拟反射。为了模拟折射效果，我们使用 GrabPass 来获取当前屏幕的渲染纹理，并使用切线空间下的法线方向对像素的屏幕坐标进行偏移，再使用该坐标对渲染纹理进行屏幕采样，从而模拟近似的折射效果。

和之前不同，水波的法线纹理是由一张噪声纹理生成而得。除此之外，我们没有使用一个定值来混合反射和折射颜色，而是使用之前菲涅尔系数来动态决定混合系数。我们使用的公式计算菲涅尔系数：  

$$ fresnel = pow(1 - max(0, v \cdot n),4) $$

其中，v 和 n 分别对应了视角方向和法线方向。它们之间的夹角越小，fresnel 值越小，反射越弱，折射越强。菲涅尔系数还经常会用于边缘光照的计算中。

---

准备工作如下：  
①新建名为 Scene_15_2 的场景，并去掉天空盒；  
②搭建水波效果的测试场景，构建一个由 6 面墙围成的封闭房间，房间中放置一个平面来模拟水面；  
③新建名为 WaterWaveMat 的材质，并赋给上一步的平面（水面）；  
④新建名为 Chapter15-WaterWave 的 Unity Shader，并赋给上一步的材质；  
⑤使用第 9 章 1.2节中实现的创建立方体纹理的脚本创建一个立方体纹理，用于得到本场景适用的环境纹理。在 Project 创建一张名为 Wave_Cubemap 的立方体纹理（右键 -> Create -> Legacy -> Cubemap）；点击菜单栏 GameObject -> Render into CubeMap。  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 15/Water Wave" {
    Properties {
        _Color ("Main Color", Color) = (0, 0.15, 0.115, 1) //控制水面颜色
        _MainTex ("Base (RGB)", 2D) = "white" {} //水面波纹材质纹理
        _WaveMap ("Wave Map", 2D) = "bump" {} //由噪声纹理生成的法线纹理
        _Cubemap ("Environment Cubemap", Cube) = "_Skybox" {}
        _WaveXSpeed ("Wave Horizontal Speed", Range(-0.1, 0.1)) = 0.01 //法线纹理在 x 方向上的平移速度
        _WaveYSpeed ("Wave Vertical Speed", Range(-0.1, 0.1)) = 0.01 //法线纹理在 y 方向上的平移速度
        _Distortion ("Distortion", Range(0, 100)) = 10 //模拟折射时图像的扭曲程度
    }

    SubShader {
        Tags { "Queue"="Transparent" "RenderType"="Opaque" } //Queue 设置成 Transparent 可以确保该物体渲染时，其他所有不透明物体都已经被渲染到屏幕上，否则可能无法正确得到“透过水面看到的图像”；而设置为 RenderType 则是为了在使用着色器替换时，该物体可以在需要时被正确渲染，见第 12 章开头
        
        GrabPass { "_RefractionTex" } //GrabPass 定义了一个抓去屏幕图像的 Pass，在该 Pass 中定义一个字符串，该字符串内部的名称决定了抓去得到的屏幕图像将会被存入哪个纹理中
        
        Pass {
            Tags { "LightMode"="ForwardBase" } 
            
            CGPROGRAM
            
            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            
            #pragma multi_compile_fwdbase
            
            #pragma vertex vert
            #pragma fragment frag

            fixed4 _Color;
            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _WaveMap;
            float4 _WaveMap_ST;
            samplerCUBE _Cubemap;
            fixed _WaveXSpeed;
            fixed _WaveYSpeed;
            float _Distortion;    
            sampler2D _RefractionTex;
            float4 _RefractionTex_TexelSize;
            
            struct a2v {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT; 
                float4 texcoord : TEXCOORD0;
            };
            
            struct v2f {
                float4 pos : SV_POSITION;
                float4 scrPos : TEXCOORD0;
                float4 uv : TEXCOORD1;
                float4 TtoW0 : TEXCOORD2;  
                float4 TtoW1 : TEXCOORD3;  
                float4 TtoW2 : TEXCOORD4; 
            };
            
            v2f vert(a2v v) {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.scrPos = ComputeGrabScreenPos(o.pos); //通过调用 ComputeGrabScreenPos 来得到对应被抓取屏幕图像的采样坐标，它的主要代码和 ComputeScreenPos 类似，最大的不同是针对平台差异问题
                
                o.uv.xy = TRANSFORM_TEX(v.texcoord, _MainTex); 
                o.uv.zw = TRANSFORM_TEX(v.texcoord, _WaveMap);
                
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;  
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);  
                fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);  
                fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w; 

                o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x); 
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y); 
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z); 
                
                return o;
            }
            
            fixed4 frag(v2f i) : SV_Target {
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                //通过 _Time.y 变量和 _WaveXSpeed、_WaveYSpeed 计算法线纹理当前偏移量
                float2 speed = _Time.y * float2(_WaveXSpeed, _WaveYSpeed);

                //利用偏移量对法线纹理进行两次采样从而模拟两层交叉的水面波动的效果
                fixed3 bump1 = UnpackNormal(tex2D(_WaveMap, i.uv.zw + speed)).rgb;
                fixed3 bump2 = UnpackNormal(tex2D(_WaveMap, i.uv.zw - speed)).rgb;
                fixed3 bump = normalize(bump1 + bump2); //两次结果相加并归一化得到切线空间下的法线方向

                //利用 _Distortion 和 _RefractionTex_TexelSize 来对屏幕图像的采样坐标进行偏移，模拟折射效果，offset 越大，水面扭曲程度越大
                float2 offset = bump.xy * _Distortion * _RefractionTex_TexelSize.xy;

                //选择切线空间下的法线方向来进行偏移，因为该空间下的法线可以反应顶点局部空间下的法线方向，在计算偏移后的屏幕坐标，需要把偏移量和屏幕坐标的 z 分量相乘，从而模拟深度变大、折射程度越大的效果
                i.scrPos.xy = offset * i.scrPos.z + i.scrPos.xy;

                //对 scrPos 进行透视除法，再使用该坐标对抓取的屏幕图像 _RefractionTex 进行采样
                fixed3 refrCol = tex2D( _RefractionTex, i.scrPos.xy/i.scrPos.w).rgb;
                
                //把法线方向从切线空间变换到世界空间下
                bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));
                fixed4 texColor = tex2D(_MainTex, i.uv.xy + speed);
                fixed3 reflDir = reflect(-viewDir, bump); //根据视角方向和法线方向得到反射方向
                fixed3 reflCol = texCUBE(_Cubemap, reflDir).rgb * texColor.rgb * _Color.rgb; //使用Cubemap进行采样并把结果和主纹理颜色相乘后得到反射颜色
                
                fixed fresnel = pow(1 - saturate(dot(viewDir, bump)), 4); //计算菲涅尔系数
                fixed3 finalColor = reflCol * fresnel + refrCol * (1 - fresnel); //混合折射和反射颜色，并作为最终的输出颜色
                return fixed4(finalColor, 1);
            }
            ENDCG
        }
    }
    // Do not cast shadow
    FallBack Off
}
```

水面使用的噪声纹理是本书资源的 Water_Noise.png，只不过我们需要的是法线纹理，可以通过在它的纹理面板把纹理类型设置为 Normal Map，同时选中 Create from grayscale 来把灰度图生成法线纹理。将生成的法线纹理拖拽到 Wave Map 上，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/29/SJ8wQrMc2nfT5Zp.gif" width = "70%" height = "70%" alt="图81- 包含菲涅耳反射的水面波动效果。视角方向和水面法线的夹角越大，反射效果越强"/>
</div>

## 再谈全局雾效
第十二章讲过如何使用深度纹理来实现一种基于屏幕后处理的全局雾效，效果是基于高度的均匀雾效，即在同一个高度上，雾的浓度是相同的。然而一些时候我们希望可以模拟一种不均匀的雾效，同时让雾不断飘动，使雾看起来更飘渺。而这就可以通过一张噪声纹理来实现。

本节的实现大部分和第十二章完全一样，只是增加了噪声相关参数和属性，准备工作如下：  
①新建名为 Scene_15_3 的场景，并去掉天空盒；  
②搭建雾效测试场景，3 面墙的房间，放置几个立方体；  
③新建名为 FogWithNoise 的 C# 脚本，并拖拽给场景中的相机；  
④新建名为 Chapter15-FogWithNoise 的 Unity Shader。

FogWithNoise.cs 的 C# 脚本代码如下：  

``` C#
using UnityEngine;
using System.Collections;

public class FogWithNoise : PostEffectsBase {
    public Shader fogShader;
    private Material fogMaterial = null;
    public Material material {  
        get {
            fogMaterial = CheckShaderAndCreateMaterial(fogShader, fogMaterial);
            return fogMaterial;
        }  
    }
    
    private Camera myCamera;
    public Camera camera {
        get {
            if (myCamera == null) {
                myCamera = GetComponent<Camera>();
            }
            return myCamera;
        }
    }

    private Transform myCameraTransform;
    public Transform cameraTransform {
        get {
            if (myCameraTransform == null) {
                myCameraTransform = camera.transform;
            }
            return myCameraTransform;
        }
    }

    [Range(0.1f, 3.0f)]
    public float fogDensity = 1.0f;
    public Color fogColor = Color.white;
    public float fogStart = 0.0f;
    public float fogEnd = 2.0f;

    public Texture noiseTexture;

    [Range(-0.5f, 0.5f)]
    public float fogXSpeed = 0.1f;
    [Range(-0.5f, 0.5f)]
    public float fogYSpeed = 0.1f;
    [Range(0.0f, 3.0f)]
    public float noiseAmount = 1.0f; //控制噪声的程度，当 noiseAmount 为 0 时，得到一个均匀的雾效

    void OnEnable() {
        GetComponent<Camera>().depthTextureMode |= DepthTextureMode.Depth;
    }
        
    void OnRenderImage (RenderTexture src, RenderTexture dest) {
        if (material != null) {
            //下面计算原理详见 12.3 章节
            Matrix4x4 frustumCorners = Matrix4x4.identity;

            float fov = camera.fieldOfView;
            float near = camera.nearClipPlane;
            float aspect = camera.aspect;
            
            float halfHeight = near * Mathf.Tan(fov * 0.5f * Mathf.Deg2Rad);
            Vector3 toRight = cameraTransform.right * halfHeight * aspect;
            Vector3 toTop = cameraTransform.up * halfHeight;
            
            Vector3 topLeft = cameraTransform.forward * near + toTop - toRight;
            float scale = topLeft.magnitude / near;
            
            topLeft.Normalize();
            topLeft *= scale;
            
            Vector3 topRight = cameraTransform.forward * near + toRight + toTop;
            topRight.Normalize();
            topRight *= scale;
            
            Vector3 bottomLeft = cameraTransform.forward * near - toTop - toRight;
            bottomLeft.Normalize();
            bottomLeft *= scale;
            
            Vector3 bottomRight = cameraTransform.forward * near + toRight - toTop;
            bottomRight.Normalize();
            bottomRight *= scale;
            
            frustumCorners.SetRow(0, bottomLeft);
            frustumCorners.SetRow(1, bottomRight);
            frustumCorners.SetRow(2, topRight);
            frustumCorners.SetRow(3, topLeft);
            
            material.SetMatrix("_FrustumCornersRay", frustumCorners);

            material.SetFloat("_FogDensity", fogDensity);
            material.SetColor("_FogColor", fogColor);
            material.SetFloat("_FogStart", fogStart);
            material.SetFloat("_FogEnd", fogEnd);

            material.SetTexture("_NoiseTex", noiseTexture);
            material.SetFloat("_FogXSpeed", fogXSpeed);
            material.SetFloat("_FogYSpeed", fogYSpeed);
            material.SetFloat("_NoiseAmount", noiseAmount);

            Graphics.Blit (src, dest, material);
        } else {
            Graphics.Blit(src, dest);
        }
    }
}
```

Chapter15-FogWithNoise 的  Shader 代码如下：  

``` C C for Graphics
Shader "Unity Shaders Book/Chapter 15/Fog With Noise" {
    Properties {
    //声明属性
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _FogDensity ("Fog Density", Float) = 1.0
        _FogColor ("Fog Color", Color) = (1, 1, 1, 1)
        _FogStart ("Fog Start", Float) = 0.0
        _FogEnd ("Fog End", Float) = 1.0
        _NoiseTex ("Noise Texture", 2D) = "white" {}
        _FogXSpeed ("Fog Horizontal Speed", Float) = 0.1
        _FogYSpeed ("Fog Vertical Speed", Float) = 0.1
        _NoiseAmount ("Noise Amount", Float) = 1
    }
    SubShader {
        CGINCLUDE
        
        #include "UnityCG.cginc"
        
        float4x4 _FrustumCornersRay; //Unity 没有提供矩阵的属性，所以没在 Properties 中声明
        
        sampler2D _MainTex;
        half4 _MainTex_TexelSize;
        sampler2D _CameraDepthTexture;
        half _FogDensity;
        fixed4 _FogColor;
        float _FogStart;
        float _FogEnd;
        sampler2D _NoiseTex;
        half _FogXSpeed;
        half _FogYSpeed;
        half _NoiseAmount;
        
        struct v2f {
            float4 pos : SV_POSITION;
            float2 uv : TEXCOORD0;
            float2 uv_depth : TEXCOORD1;
            float4 interpolatedRay : TEXCOORD2;        
        };
        
        v2f vert(appdata_img v) {
            v2f o;
            o.pos = UnityObjectToClipPos(v.vertex);
            o.uv = v.texcoord;
            o.uv_depth = v.texcoord;

            #if UNITY_UV_STARTS_AT_TOP //适配不同平台
            if (_MainTex_TexelSize.y < 0)
                o.uv_depth.y = 1 - o.uv_depth.y;
            #endif
            
            int index = 0;
            if (v.texcoord.x < 0.5 && v.texcoord.y < 0.5) {
                index = 0;
            } else if (v.texcoord.x > 0.5 && v.texcoord.y < 0.5) {
                index = 1;
            } else if (v.texcoord.x > 0.5 && v.texcoord.y > 0.5) {
                index = 2;
            } else {
                index = 3;
            }

            #if UNITY_UV_STARTS_AT_TOP
            if (_MainTex_TexelSize.y < 0)
                index = 3 - index;
            #endif
            
            o.interpolatedRay = _FrustumCornersRay[index];
            return o;
        }
        
        fixed4 frag(v2f i) : SV_Target {
            float linearDepth = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv_depth)); //根据深度纹理构建该像素在世界空间中的位置
            float3 worldPos = _WorldSpaceCameraPos + linearDepth * i.interpolatedRay.xyz;    
            
            float2 speed = _Time.y * float2(_FogXSpeed, _FogYSpeed); //计算雾的偏移量
            float noise = (tex2D(_NoiseTex, i.uv + speed).r - 0.5) * _NoiseAmount; //对噪声纹理进行采样从而得到噪声值
                    
            float fogDensity = (_FogEnd - worldPos.y) / (_FogEnd - _FogStart);         
            fogDensity = saturate(fogDensity * _FogDensity * (1 + noise)); //计算雾效混合系数
            
            fixed4 finalColor = tex2D(_MainTex, i.uv);
            finalColor.rgb = lerp(finalColor.rgb, _FogColor.rgb, fogDensity);
            return finalColor;
        }
        
        ENDCG
        
        Pass {              
            CGPROGRAM  
            
            #pragma vertex vert  
            #pragma fragment frag  
              
            ENDCG
        }
    } 
    FallBack Off
}
```

将 Chapter15-FogWithNoise 拖拽到摄像机的脚本中的 Fog Shader 属性中，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2023/12/30/YK8PdcguWLUmRfQ.gif" width = "70%" height = "70%" alt="图82- ：使用噪声纹理后的非均匀雾效"/>
</div>

## 扩展阅读
噪声纹理由计算机利用某些算法生成的，可以被认为是一种程序纹理 Procedure Texture。Perlin 纹理、Worley 纹理是比较常使用的噪声纹理类型。Perlin 噪声可以用于生成更自然的噪声纹理；Worley 噪声则通常用于模拟诸如石头、水、纸张等多孔噪声。有兴趣自行查阅资料。