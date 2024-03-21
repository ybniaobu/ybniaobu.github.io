---
title: 基于星穹铁道的卡通渲染
date: 2024-03-20 13:25:53
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
  - NPR
top_img: /images/black.jpg
cover: https://s2.loli.net/2024/03/20/w9by4joTmI1McSC.jpg
mathjax: true
description: 本文以米哈游的崩坏星穹铁道的卡通人物渲染来切入 NPR (Non-Photorealistic Rendering) ，从而对卡通渲染有个初步的了解，并加深对渲染的理解，为做出自己的风格化渲染打下基础。
---

> 本文主要借鉴了 b 站 up 主给我柠檬养乐多你会跟我玩吗的视频，bv 号：BV1CN411C7qx。

# 模型处理
## Blender 模型预处理
米哈游的模型都公布在模之屋 (https://www.aplaybox.com/) ，由于公布的模型都是 MMD 软件的 pmx 格式，我们需要使用 Blender 软件的插件 MMD_Tools (https://github.com/UuuNyaa/blender_mmd_tools) 转换为 FBX 格式，方便导入到 Unity。笔者使用的 blender 版本为 3.6.8 LTS，当前 MMD_Tools 插件只支持到 3.6 LTS。

具体的模型处理操作这里不进行图文说明了，依据本文开头说的视频中描述，步骤大致如下：  
①导入模型后，先把 joints 和 rigidbodies 删除；  
②进入编辑模式，将除了脸以外的使用同一张贴图的材质进行合并，脸不合并是因为我们只需要对脸进行描边，而不需要对眼睛、眉毛、牙齿舌头嘴巴进行描边。（视频里最后剩下了以下材质：上衣、下衣、头发、牙舌口、目、眉毛、眼睛1、脸、表情）；  
③把带内侧二字的材质删除，该材质使用的模型顶点区域是和外侧共用的，而且视频中使用的布洛妮娅的模型的裙内侧贴图是纯蓝色，可以直接在 Shader 中渲染背面时将基础颜色设置为蓝色（我觉得对于内侧贴图有图画的可以保留，不重复描边即可<font color=Red>【待实验】</font>）；  
④为了在 Unity 中减少渲染的纹理消耗，将形态键分离（模型中自带一些表情相关**形态键 Shape Keys**，Unity 中叫**混合形状 BlendShapes**，也有叫**变形目标 Morph Target** 的），选择跟脸相关材质对应的模型顶点（视频中选择了脸、眉毛、牙舌口、目、眼睛1、表情），并进行分离，也就是脸对应一个 mesh，身体对应一个 mesh，然后删除身体的 mesh 里的所有的形态键，并删除脸 mesh 和身体 mesh 中不属于该 mesh 的多余的材质；

## 导出 FBX
随后导出 FBX 格式，在 Blender 的导出设置中 **Apply Scalings** 选择 **FBX Units Scale** 可以解决导入 Unity 时缩放为 100 的问题。**Forward** 选择导出物体在 Blender 中的我们想要的面朝方向，**Up** 选择导出物体在 Blender 中我们想要的向上的方向，并在 Unity FBX 导入界面勾选 **Bake Axis Conversion**，这样可以解决旋转出错的问题。比如 Blender 的猴头，默认是面朝 Blender 的 -Y 轴，向上方向为 Z 轴，故导出时 Forward 选择 -Y Forward，Up 选择 Z Up，勾选 Bake Axis Conversion 后，清除物体的方向，就可以发现猴头在 Unity 面朝 Z 方向，上方为 Y 方向，符合 Unity 默认方向。

## FBX 替换材质
FBX 文件在 Unity 中右边箭头展开，可以直接把材质复制出来，其中表情材质替换为 URP 的 Unlit Shader，并赋予表情贴图，其他材质替换为我们的 Shader，见后面的 Shader 准备小节。然后把新的这些材质一个个替换进 FBX 文件。然后给所有材质选择好 Material area 属性，并把所有贴图赋予好。

# 贴图
## 贴图预处理
> 有些贴图模之屋的文件中并不包含，比如光照贴图、渐变纹理以及面部的 SDF 贴图（**Signed Distance Field** 建议去额外了解该项技术），需要自己去获取。我贴图是在 https://github.com/umaichanuwu/GenshinLinks 中下载的。

①在 Unity 导入贴图后，首先对所有贴图关闭压缩，即 Compression 选择 None；②所有的光照贴图 LightMap 关闭 **sRGB（Color Texture）**，原因见下面伽马校正；③ Ramp 贴图的 Mip Maps 关掉，Wrap Mode 改为 Clamp，Filter Mode 改为 Point (no filter)；④丝袜贴图和脸部 SDF 贴图也关闭 sRGB（Color Texture）。

## 伽马校正
在 Unity 中，若选择了**线性空间**渲染，对于 sRGB 方式存储的贴图（比如**存储颜色的贴图**），需要勾选 sRGB（Color Texture），这样这些经过 Gamma 0.45 处理的贴图在 Shader 的计算输入时（采样时）会自动进行一个 Gamma 2.2 处理变换到线性空间，这样 Shader 在线性空间中计算会更加准确，Unity URP 在渲染的最后会调用一个 Pass 再进行一次**伽马校正**（**伽马编码**），即 Gamma 0.45，这样显示器输出 Gamma 2.2 平衡后颜色才会正确。但是对于**法线贴图**、**光照贴图**等这些直接按线性方式存储的贴图，需要取消勾选 sRGB（Color Texture），这样 Unity 就不会自动**去除伽马校正 Remove Gamma Correction**，即 Gamma 2.2 处理，保证在 Shader 的计算中是线性的，同样 Unity 会自动进行伽马校正，从而正确地在显示器显示。如下图：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/20/B8XVCEP1TpvLNjc.png" width = "70%" height = "70%" alt="图1 - 线性空间"/>
</div>

除了纹理，在线性空间模式下，ShaderLab 的颜色属性也会被认为是 sRGB 颜色，会自动进行 Remove Gamma Correction。但是对于材质的 Float 或 Vector 属性，不会自动进行色彩空间转换，因此对于一些跟色彩相关的 Float 或 Vector 属性，比如金属度，需要把它们指定到 sRGB 空间，此时可以添加 `[Gamma]` 特性。另外要注意，Alpha 通道是不受 Gamma 校正的影响的。

## 贴图作用分析
根据视频中的布洛妮娅的贴图为例：  
**①**首先 4 张**颜色贴图**，分别为上衣、下衣、头发和脸：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/21/VDvatGcKzIuMJn6.png" width = "100%" height = "100%" alt="图2 - 上衣、下衣和头发的颜色贴图"/>
</div>

这 3 张贴图的 RGB 通道就是颜色信息，A 通道都为 1。有些角色的 A 通道可能会存储一些自发光区域，也就是自发光 mask，比如布洛妮娅的脸部的颜色贴图的 A 通道就包含了眼睛的自发光区域，如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/21/TdAjsLyuMk52KUV.png" width = "67%" height = "67%" alt="图3 - 脸部颜色贴图的 RGB 和 Alpha 通道"/>
</div>

**②**接下来 3 张**光照贴图 LightMap**，分别为上衣、下衣和头发：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/21/wtqnl5RJEW63Lo9.png" width = "100%" height = "100%" alt="图4 - 上衣、下衣和头发的 LightMap"/>
</div>

- **R 通道**：环境光遮罩 AO；  
- **G 通道**：阴影遮罩 ShadowMask，黑色区域的地方永远处于阴影中；  
- **B 通道**：高光强度遮罩 SpecularIntensityMask，控制高光范围和强度；  
- **A 通道**：Ramp 贴图索引信息，根据颜色数值对 Ramp 贴图采样。

具体作用详见代码部分。

**③** 1 张**脸部的魔法贴图 FaceMap**：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/21/7rbjRqh6M9Jyivt.png" width = "67%" height = "67%" alt="图5 - FaceMap"/>
</div>

- **R 通道**：阴影阈值控制，详见代码；  
- **G 通道**：非 SDF 阴影区域的面部阴影，比如舌头、眼睛等等；  
- **B 通道**：鼻子描边，仔细看贴图中央，有根细细的白条；  
- **A 通道**：SDF 面部阴影。

具体作用详见代码部分。

**④** 2 张**丝袜贴图 Stockings**，分为手部和腿部。A 通道无信息，不使用：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/21/QJTNz2YjdlFt1fh.png" width = "100%" height = "100%" alt="图6 - Stockings"/>
</div>

- **R 通道**：腿部皮肤区域控制，看看腿部颜色贴图就懂了；  
- **G 通道**：丝袜花纹；  
- **B 通道**：丝袜纹理；

具体作用详见代码部分。

**⑤** 4 张**Ramp 贴图**，包含身体的冷暖 Ramp 以及头发的冷暖 Ramp，图片不在这里展示了，反正也看不清楚，建议在 PS 放大至 2000% 查看。身体的冷暖 Ramp 贴图包含 256 × 16 个色阶；头发的冷暖 Ramp 包含 256 × 2 个色阶，具体作用详见代码部分。


# Shader 准备
先准备好包含材质属性以及一些基础 Pass 的 shader，比如 ShadowCaster、DepthOnly 和 DepthNormals Pass，这些 Pass 直接使用 URP 内置的代码。然后新建一个包含主要渲染工作的 Pass，我命名为了 ToonForward，引入自己的 ToonInput.hlsl 和 ToonForwardPass.hlsl 文件。

视频中的代码并不支持 SRP Batcher，因为使用了 URP 内置的 ShadowCaster、DepthOnly 以及 DepthNormals Pass，导致了 CBuffer 不统一。若需要 SRP Batcher 友好，可以自己更改上述 Pass，这些 Pass 都很简单，但在本篇文章中就不对视频的 Shader 做出修改了。

## 主 Shader 文件
我把 Shader 文件命名为了 StarRailToon.shader，该 Shader 的材质属性不在这里做详细介绍，见使用时的代码说明。Shader 代码如下：  

``` C
shader "Custom/StarRailToon"
{
    Properties
    {
        [KeywordEnum(None,Face,Hair,UpperBody,LowerBody)] _Area ("Material Area", Float) = 0
        [HideInInspector] _HeadForward ("", Vector) = (0, 0, 1)
        [HideInInspector] _HeadRight ("", Vector) = (1, 0, 0)
        
        [Header(Base Color)]
        [HideInInspector] _BaseMap ("", 2D) = "white" {}
        [NoScaleOffset] _FaceColorMap ("Face color map (Default white)", 2D) = "white" {}
        [NoScaleOffset] _HairColorMap ("Hair color map (Default white)", 2D) = "white" {}
        [NoScaleOffset] _UpperBodyColorMap ("UpperBody color map (Default white)", 2D) = "white" {}
        [NoScaleOffset] _LowerBodyColorMap ("LowerBody color map (Default white)", 2D) = "white" {}
        _FrontFaceTintColor ("Front face tint color (Default white)", Color) = (1,1,1)
        _BackFaceTintColor ("Back face tint color (Default white)", Color) = (1,1,1)
        _Alpha ("Alpha (Default 1)", Range(0, 1)) = 1
        _AlphaClip ("Alpha clip (Default 0.333)", Range(0, 1)) = 0.333
        
        [Header(Light Map)]
        [NoScaleOffset] _HairLightMap ("Hair light map (Default black)", 2D) = "black" {}
        [NoScaleOffset] _UpperBodyLightMap ("UpperBody light map (Default black)", 2D) = "black" {}
        [NoScaleOffset] _LowerBodyLightMap ("LowerBody light map (Default black)", 2D) = "black" {}
        
        [Header(Ramp Map)]
        [NoScaleOffset] _HairCoolRamp ("Hair cool ramp (Default white)", 2D) = "white" {}
        [NoScaleOffset] _HairWarmRamp ("Hair warm ramp (Default white)", 2D) = "white" {}
        [NoScaleOffset] _BodyCoolRamp ("Body cool ramp (Default white)", 2D) = "white" {}
        [NoScaleOffset] _BodyWarmRamp ("Body warm ramp (Default white)", 2D) = "white" {}
        
        [Header(Indirect Lighting)]
        _IndirectLightFlattenNormal ("Indirect light flatten normal (Default 0)", Range(0, 1)) = 0
        _IndirectLightUsage ("Indirect light usage (Default 0.5)", Range(0, 1)) = 0.5
        _IndirectLightOcclusionUsage ("Indirect light occlusion usage (Default 0.5)", Range(0, 1)) = 0.5
        _IndirectLightMixBaseColor ("Indirect light mix base color (Default 1)", Range(0, 1)) = 1
        
        [Header(Main Lighting)]
        _MainLightColorUsage ("Main light color usage (Default 1)", Range(0, 1)) = 1
        _ShadowThresholdCenter ("Shadow threshold center (Default 0)", Range(-1, 1)) = 0
        _ShadowThresholdSoftness ("Shadow threshold softness (Default 0.1)", Range(0, 1)) = 0.1
        _ShadowRampOffset ("Shadow ramp offset (Default 0.75)", Range(0, 1)) = 0.75
        
        [Header(Face)]
        [NoScaleOffset] _FaceMap ("Face map (Default black)", 2D) = "black" {}
        _FaceShadowOffset ("Face shadow offset (Default -0.01)", Range(-1, 1)) = -0.01
        _FaceShadowTransitionSoftness ("Face shadow transition softness (Default 0.05)", Range(0, 1)) = 0.05
        
        [Header(Specular)]
        _SpecularExponent ("Specular exponent (Default 50)", Range(1, 128)) = 50
        _SpecularKsNonMetal ("Specular Ks non-metal (Default 0.04)", Range(0, 1)) = 0.04
        _SpecularKsMetal ("Specular Ks metal (Default 1)", Range(0, 1)) = 1
        _SpecularBrightness ("Specular brightness (Default 1)", Range(0, 10)) = 1
        
        [Header(Stockings)]
        [NoScaleOffset] _UpperBodyStockings ("Upper body stockings (Default black)", 2D) = "black" {}
        [NoScaleOffset] _LowerBodyStockings ("Lower body stockings (Default black)", 2D) = "black" {}
        _StockingsDarkColor ("Stockings dark color (Default black)", Color) = (0, 0, 0)
        [HDR] _StockingsLightColor ("Stockings light color (Default 1.8, 1.48299, 0.856821)", Color) = (1.8, 1.48299, 0.856821)
        [HDR] _StockingsTransitionColor ("Stockings transition color (Default 0.360381, 0.242986, 0.358131)", Color) = (0.360381, 0.242986, 0.358131)
        _StockingsTransitionThreshold ("Stockings transition threshold (Default 0.58)", Range(0, 1)) = 0.58
        _StockingsTransitionPower ("Stockings transition power (Default 1)", Range(0.1, 50)) = 1
        _StockingsTransitionHardness ("Stockings transition hardness (Default 0.4)", Range(0, 1)) = 0.4
        _StockingsTextureUsage ("Stockings texture usage (Default 0.1)", Range(0, 1)) = 0.1
        
        [Header(Rim Lighting)]
        _RimLightWidth ("Rim light width (Default 1)", Range(0, 10)) = 1
        _RimLightThreshold ("Rim light threshold (Default 0.05)", Range(-1, 1)) = 0.05
        _RimLightFadeout ("Rim light fadeout (Default 1)", Range(0.01, 1)) = 1
        [HDR] _RimLightTintColor ("Rim light tint color (Default white)", Color) = (1, 1, 1)
        _RimLightBrightness ("Rim light brightness (Default 1)", Range(0, 10)) = 1
        _RimLightMixAlbedo ("Rim light mix albedo (Default 0.9)", Range(0, 1)) = 0.9
        
        [Header(Emission)]
        [Toggle(_EMISSION_ON)] _UseEmission ("Use emission (Default No)", Float) = 0
        _EmissionMixBaseColor ("Emission mix base color (Default 1)", Range(0, 1)) = 1
        [HDR] _EmissionTintColor ("Emission tint color (Default white)", Color) = (1, 1, 1)
        _EmissionIntensity ("Emission intensity (Default 1)", Range(0, 100)) = 1
        
        [Header(Outline)]
        [Toggle(_OUTLINE_ON)] _UseOutline ("Use outline (Default Yes)", Float) = 1
        [Toggle(_OUTLINE_VERTEX_COLOR_SMOOTH_NORMAL)] _OutlineUseVertexColorSmoothNormal ("Use vertex color smooth normal (Default No)", Float) = 0
        _OutlineWidth ("Outline width (Default 1)", Range(0, 10)) = 1
        _OutlineGamma ("Outline gamma (Default 16)", Range(1, 255)) = 16
        
        [Header(Surface Options)]
        [Enum(UnityEngine.Rendering.CullMode)] _Cull ("Cull (Default back)", Float) = 2
        [Enum(UnityEngine.Rendering.BlendMode)] _SrcBlend ("Src blend mode (Default One)", Float) = 1
        [Enum(UnityEngine.Rendering.BlendMode)] _DstBlend ("Dst blend mode (Default Zero)", Float) = 0
        [Enum(UnityEngine.Rendering.BlendOp)] _BlendOp ("Blend operation (Default Add)", Float) = 0
        [Enum(Off, 0, On, 1)] _ZWrite ("ZWrite (Default On)", float) = 1
        _StencilRef("Stencil reference (Default 0)", Range(0, 255)) = 0
        [Enum(UnityEngine.Rendering.CompareFunction)]_StencilComp ("Stencil comparison (Default disabled)", Float) = 0
        [Enum(UnityEngine.Rendering.StencilOp)] _StencilPassOp ("Stencil pass operation (Default keep)", Float) = 0
        [Enum(UnityEngine.Rendering.StencilOp)] _StencilFailOp ("Stencil fail operation (Default keep)", Float) = 0
        [Enum(UnityEngine.Rendering.StencilOp)] _StencilZFailOp ("Stencil ZFail operation (Default keep)", Float) = 0
        
        [Header(Draw Overlay)]
        [Toggle(_DRAW_OVERLAY_ON)] _UseDrawOverlay ("Use draw overlay (Default No)", Float) = 0
        [Enum(UnityEngine.Rendering.BlendMode)] _SrcBlendModeOverlay ("Overlay pass src blend mode (Default One)", Float) = 1
        [Enum(UnityEngine.Rendering.BlendMode)] _DstBlendModeOverlay ("Overlay pass dst blend mode (Default Zero)", Float) = 0
        [Enum(UnityEngine.Rendering.BlendOp)] _BlendOpOverlay ("Overlay pass blend operation (Default Add)", Float) = 0
        _StencilRefOverlay ("Overlay pass stencil reference (Default 0)", Range(0, 255)) = 0
        [Enum(UnityEngine.Rendering.CompareFunction)] _StencilCompOverlay ("Overlay pass stencil comparison (Default disabled)", Float) = 0
    }

    SubShader
    {
        Tags 
        {
            "RenderPipeline" = "UniversalPipeline" 
            "IgnoreProjector" = "True"
        }
        
        LOD 100
        
        HLSLINCLUDE
        
        #pragma shader_feature_local _AREA_FACE
        #pragma shader_feature_local _AREA_HAIR
        #pragma shader_feature_local _AREA_UPPERBODY
        #pragma shader_feature_local _AREA_LOWERBODY
        #pragma shader_feature_local _EMISSION_ON
        #pragma shader_feature_local _OUTLINE_ON
        #pragma shader_feature_local _OUTLINE_VERTEX_COLOR_SMOOTH_NORMAL
        #pragma shader_feature_local _DRAW_OVERLAY_ON
        
        ENDHLSL
        
        Pass
        {
            Name "ToonForward"
            
            Tags { "LightMode" = "UniversalForward" }
            
            Cull [_Cull]
            Stencil
            { 
                Ref [_StencilRef]
                Comp [_StencilComp]
                Pass [_StencilPassOp]
                Fail [_StencilFailOp]
                ZFail [_StencilZFailOp]
            }
            Blend [_SrcBlend] [_DstBlend]
            BlendOp [_BlendOp]
            ZWrite [_ZWrite]
            
            HLSLPROGRAM
            #pragma vertex ToonForwardVert
            #pragma fragment ToonForwardFrag

            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
            #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
            #pragma multi_compile_fragment _ _ADDITIONAL_LIGHT_SHADOWS
            #pragma multi_compile_fragment _ _SHADOWS_SOFT _SHADOWS_SOFT_LOW _SHADOWS_SOFT_MEDIUM _SHADOWS_SOFT_HIGH
            
            #include "Assets/ShaderLibrary/ToonInput.hlsl"
            #include "Assets/ShaderLibrary/ToonForwardPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "ShadowCaster"
            Tags { "LightMode" = "ShadowCaster" }
            
            ZWrite [_ZWrite]
            ZTest LEqual
            ColorMask 0
            Cull [_Cull]
            
            HLSLPROGRAM
            #pragma target 4.5
            
            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

            // -------------------------------------
            // Universal Pipeline keywords

            // -------------------------------------
            // Unity defined keywords
            #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE

            // This is used during shadow map generation to differentiate between directional and punctual light shadows, as they use different formulas to apply Normal Bias
            #pragma multi_compile_vertex _ _CASTING_PUNCTUAL_LIGHT_SHADOW
            
            #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
            ENDHLSL
        }

        Pass
        {
            Name "DepthOnly"
            Tags { "LightMode" = "DepthOnly" }
            
            ZWrite [_ZWrite]
            ColorMask 0
            Cull [_Cull]
            
            HLSLPROGRAM
            #pragma target 4.5
            
            #pragma vertex DepthOnlyVertex
            #pragma fragment DepthOnlyFragment

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

            // -------------------------------------
            // Unity defined keywords
            #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"
            
            #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/DepthOnlyPass.hlsl"
            ENDHLSL
        }
        
        Pass
        {
            Name "DepthNormals"
            Tags { "LightMode" = "DepthNormals" }
            
            ZWrite [_ZWrite]
            Cull [_Cull]

            HLSLPROGRAM
            #pragma target 4.5
            
            #pragma vertex DepthNormalsVertex
            #pragma fragment DepthNormalsFragment

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local _NORMALMAP
            #pragma shader_feature_local _PARALLAXMAP
            #pragma shader_feature_local _ _DETAIL_MULX2 _DETAIL_SCALED
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

            // -------------------------------------
            // Unity defined keywords
            #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE

            // -------------------------------------
            // Universal Pipeline keywords
            #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/RenderingLayers.hlsl"

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"
            
            #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/LitDepthNormalsPass.hlsl"
            ENDHLSL
        }
    }
}
```

## ToonInput.hlsl

## ToonForwardPass.hlsl
