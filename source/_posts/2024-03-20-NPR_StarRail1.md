---
title: 基于星穹铁道的卡通渲染（一）
date: 2024-03-20 13:25:53
categories: 
  - [unity, unity shader]
tags:
  - 游戏开发
  - unity
  - 图形学
  - NPR
top_img: /images/black.jpg
cover: https://s2.loli.net/2024/03/26/dZTwsApi59CSUal.gif
description: 本篇主要内容为模型、贴图预处理以及 Toon Shader 的光照。本文以米哈游的崩坏星穹铁道的卡通人物渲染来切入 NPR (Non-Photorealistic Rendering) ，从而对卡通渲染有个初步的了解，并加深对渲染的理解，为做出自己的风格化渲染打下基础。
---

> 本文主要借鉴了 b 站 up 主给我柠檬养乐多你会跟我玩吗的视频，bv 号：BV1CN411C7qx。

# 模型处理
## Blender 模型预处理
米哈游的模型都公布在模之屋 (https://www.aplaybox.com/) ，由于公布的模型都是 MMD 软件的 pmx 格式，我们需要使用 Blender 软件的插件 MMD_Tools (https://github.com/UuuNyaa/blender_mmd_tools) 转换为 FBX 格式，方便导入到 Unity。笔者使用的 blender 版本为 3.6.8 LTS，当前 MMD_Tools 插件只支持到 3.6 LTS。

具体的模型处理操作这里不进行图文说明了，依据本文开头说的视频中描述，步骤大致如下：  
①导入模型后，先把 joints 和 rigidbodies 删除；  
②进入编辑模式，将除了脸以外的使用同一张贴图的材质进行合并，脸不合并是因为我们只需要对脸进行描边，而不需要对眼睛、眉毛、牙齿舌头嘴巴进行描边。（视频里最后剩下了以下材质：上衣、下衣、头发、牙舌口、目、眉毛、眼睛1、脸、表情）；  
③把带内侧二字的材质删除，该材质使用的模型顶点区域是和外侧共用的，而且视频中使用的布洛妮娅的模型的裙内侧贴图是纯蓝色，可以直接在 Shader 中渲染背面时将基础颜色设置为蓝色（我觉得对于内侧贴图有图画的可以保留，不重复描边即可）；  
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

- **R 通道**：环境光遮罩 AO，静态阴影区域；  
- **G 通道**：阴影阈值图，动态阴影区域，阴影阈值控制阴影何时产生，黑色区域的地方永远处于阴影中；  
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
- **G 通道**：丝袜花纹，越黑越不容易看到皮肤；  
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
该文件声明 Shader 变量，视频里在 CBuffer 里使用了 #if 指令，但官方不建议在 CBuffer 里使用 #if 或 #ifdef ，因为这样 SRP batcher 就会不兼容，可以把 CBuffer 里的 #if 都去掉，但本文章展示中就不去除了。代码如下：  

``` C
#ifndef CUSTOM_TOON_INPUT_INCLUDED
#define CUSTOM_TOON_INPUT_INCLUDED

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl"

CBUFFER_START(UnityPerMaterial)
float3 _HeadForward;
float3 _HeadRight;
float4 _BaseMap_ST;

float3 _FrontFaceTintColor;
float3 _BackFaceTintColor;
float _Alpha;
float _AlphaClip;

float _IndirectLightFlattenNormal;
float _IndirectLightUsage;
float _IndirectLightOcclusionUsage;
float _IndirectLightMixBaseColor;

float _MainLightColorUsage;
float _ShadowThresholdCenter;
float _ShadowThresholdSoftness;
float _ShadowRampOffset;

#if _AREA_FACE
    float _FaceShadowOffset;
    float _FaceShadowTransitionSoftness;
#endif

#if _AREA_HAIR || _AREA_UPPERBODY || _AREA_LOWERBODY
    float _SpecularExponent;
    float _SpecularKsNonMetal;
    float _SpecularKsMetal;
    float _SpecularBrightness;
#endif

#if _AREA_UPPERBODY || _AREA_LOWERBODY
    float3 _StockingsDarkColor;
    float3 _StockingsLightColor;
    float3 _StockingsTransitionColor;
    float _StockingsTransitionThreshold;
    float _StockingsTransitionPower;
    float _StockingsTransitionHardness;
    float _StockingsTextureUsage;
#endif

float _RimLightWidth;
float _RimLightThreshold;
float _RimLightFadeout;
float3 _RimLightTintColor;
float _RimLightBrightness;
float _RimLightMixAlbedo;

#if _EMISSION_ON
    float _EmissionMixBaseColor;
    float3 _EmissionTintColor;
    float _EmissionIntensity;
#endif

#if _OUTLINE_ON
    float _OutlineWidth;
    float _OutlineGamma;
#endif

CBUFFER_END

TEXTURE2D(_BaseMap);
SAMPLER(sampler_BaseMap);

#if _AREA_FACE
    TEXTURE2D(_FaceColorMap);
    SAMPLER(sampler_FaceColorMap);
#elif _AREA_HAIR
    TEXTURE2D(_HairColorMap);
    SAMPLER(sampler_HairColorMap);
#elif _AREA_UPPERBODY
    TEXTURE2D(_UpperBodyColorMap);
    SAMPLER(sampler_UpperBodyColorMap);
#elif _AREA_LOWERBODY
    TEXTURE2D(_LowerBodyColorMap);
    SAMPLER(sampler_LowerBodyColorMap);
#endif

#if _AREA_HAIR
    TEXTURE2D(_HairLightMap);
    SAMPLER(sampler_HairLightMap);
#elif _AREA_UPPERBODY
    TEXTURE2D(_UpperBodyLightMap);
    SAMPLER(sampler_UpperBodyLightMap);
#elif _AREA_LOWERBODY
    TEXTURE2D(_LowerBodyLightMap);
    SAMPLER(sampler_LowerBodyLightMap);
#endif

#if _AREA_HAIR
    TEXTURE2D(_HairCoolRamp);
    SAMPLER(sampler_HairCoolRamp);
    TEXTURE2D(_HairWarmRamp);
    SAMPLER(sampler_HairWarmRamp);
#elif _AREA_FACE || _AREA_UPPERBODY || _AREA_LOWERBODY
    TEXTURE2D(_BodyCoolRamp);
    SAMPLER(sampler_BodyCoolRamp);
    TEXTURE2D(_BodyWarmRamp);
    SAMPLER(sampler_BodyWarmRamp);
#endif

#if _AREA_FACE
    TEXTURE2D(_FaceMap);
    SAMPLER(sampler_FaceMap);
#endif

#if _AREA_UPPERBODY
    TEXTURE2D(_UpperBodyStockings);
    SAMPLER(sampler_UpperBodyStockings);
#elif _AREA_LOWERBODY
    TEXTURE2D(_LowerBodyStockings);
    SAMPLER(sampler_LowerBodyStockings);
#endif

#endif
```

## ToonForwardPass.hlsl
准备好输出输入结构，把顶点着色器和片元着色器的基础代码先填写好：

``` C
#ifndef CUSTOM_TOON_FORWARD_PASS_INCLUDED
#define CUSTOM_TOON_FORWARD_PASS_INCLUDED

struct Attributes
{
    float3 positionOS   : POSITION;
    float3 normalOS      : NORMAL;
    float4 tangentOS     : TANGENT;
    float2 uv           : TEXCOORD0;
};

struct Varyings
{
    float2 uv                       : TEXCOORD0;
    float4 positionWSAndFogFactor   : TEXCOORD1; //w: vertex fog factor
    float3 normalWS                 : TEXCOORD2;
    float3 viewDirectionWS          : TEXCOORD3;
    float3 SH                       : TEXCOORD4;
    float4 positionCS               : SV_POSITION;
};

Varyings ToonForwardVert(Attributes input)
{
    Varyings output;

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS);
    VertexNormalInputs vertexNormalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

    output.uv = TRANSFORM_TEX(input.uv, _BaseMap);
    output.positionCS = vertexInput.positionCS;

    return output;
}

float4 ToonForwardFrag(Varyings input, bool isFrontFace : SV_IsFrontFace) : SV_TARGET
{
    float3 baseColor = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, input.uv).rgb;
    
    float3 albedo = 0;

    float4 color = float4(albedo, 1);
    return color;
}

#endif
```


# 正式编写 Shader（光照）
## 颜色贴图区分部位
首先根据 Keyword 区分不同部位使用不同的贴图：

``` C
float4 areaMap = 0;

#if _AREA_FACE
    areaMap = SAMPLE_TEXTURE2D(_FaceColorMap, sampler_FaceColorMap, input.uv);
#elif _AREA_HAIR
    areaMap = SAMPLE_TEXTURE2D(_HairColorMap, sampler_HairColorMap, input.uv);
#elif _AREA_UPPERBODY
    areaMap = SAMPLE_TEXTURE2D(_UpperBodyColorMap, sampler_UpperBodyColorMap, input.uv);
#elif _AREA_LOWERBODY
    areaMap = SAMPLE_TEXTURE2D(_LowerBodyColorMap, sampler_LowerBodyColorMap, input.uv);
#endif

baseColor = areaMap.rgb;
```

此时模型的眼睛和裙子内侧都没有渲染出来，因为我们没有增加对应的基础颜色，所以我们需要乘上基础颜色 tint color。根据渲染的正反面，使用 `lerp()` 函数充当 if 语句的功能，片元着色器函数的参数 `SV_IsFrontFace` 是 HLSL 的内置参数，判断当前 fragment 是正面还是反面：

    baseColor *= lerp(_BackFaceTintColor, _FrontFaceTintColor, isFrontFace);

然后把眼睛 1 （目透明）材质的 Front face tint color 材质参数调整为黑色；把下衣材质的 Surface Options 中的 Cull 改为 Off，也就是关闭背面剔除，同时 Back face tint color 材质参数调整为蓝色（#1643A3）。

此时眼睛 1 是全黑色，我们希望得到透明效果，更改代码如下：  

    float alpha = _Alpha;
    float4 color = float4(albedo, alpha);

然后眼睛 1 材质的 Alpha 参数改为 0.66，渲染队列改为 Transparent，即 3000；Surface Options 中 Blend Mode 改为 SrcAlpha 和 OneMinusSrcAlpha。这下就可以看到正确的眼睛了。

## 光照图准备
还是根据 Keyword 区分不同部位使用不同的 lightMap 和 faceMap：  

``` C
float4 lightMap = 0;

#if _AREA_HAIR
    lightMap = SAMPLE_TEXTURE2D(_HairLightMap, sampler_HairLightMap, input.uv);
#elif _AREA_UPPERBODY
    lightMap = SAMPLE_TEXTURE2D(_UpperBodyLightMap, sampler_UpperBodyLightMap, input.uv);
#elif _AREA_LOWERBODY
    lightMap = SAMPLE_TEXTURE2D(_LowerBodyLightMap, sampler_LowerBodyLightMap, input.uv);
#endif

float4 faceMap = 0;
    
#if _AREA_FACE
    faceMap = SAMPLE_TEXTURE2D(_FaceMap, sampler_FaceMap, input.uv);
#endif
```

## 准备光照需要的参数
在顶点着色器内增加如下代码，获取模型世界坐标、法线方向世界坐标、视角方向世界坐标：  

``` C
output.positionWSAndFogFactor = float4(vertexInput.positionWS, ComputeFogFactor(vertexInput.positionCS.z));
output.normalWS = vertexNormalInput.normalWS;
output.viewDirectionWS = unity_OrthoParams.w == 0 ? GetCameraPositionWS() - vertexInput.positionWS : GetWorldToViewMatrix()[2].xyz;
```

URP 自带雾效支持，可以在 Window -> Rendering -> Lighting 修改雾效相关设置。若要让 Shader 支持这些雾效，就需要计算雾效因子，最后和输出颜色混合，这里不详细展开，详见百度。

在 viewDirectionWS 的计算中，`unity_OrthoParams` 为 Unity 内置的全局变量，在 UnityInput.hlsl 中定义，它的 w 值为 1 时相机是正交投影 orthographic projection，为 0 是透视投影 perspective projection。透视投影时视角方向为物体指向摄像机的方向，正如代码所示。而正交投影时视角方向就是相机前方的反方向（指向相机），取世界到观察空间变换矩阵第 3 行为该方向，数学逻辑如下：<font color="#727bff">观察空间是右手坐标系，所以在观察空间中，摄像机的前方为 -z 轴方向；而世界空间到观察空间的变换矩阵就是观察空间（对于摄像机来说就是它自己的模型空间）到世界空间的变换矩阵的逆矩阵，所以我们要得到世界空间下的 viewDirection，就需要把它从观察空间转换到世界空间，也就是需要世界空间到观察空间的变换矩阵 WorldToViewMatrix 的逆矩阵。因为我们无需关心缩放和平移，只需要关心旋转，而旋转矩阵是正交矩阵，也就是说 WorldToViewMatrix 的前三行前三列的转置矩阵就是它的逆矩阵。我们将观察空间指向摄像机的向量，即 (0, 0, 1)，乘上 WorldToViewMatrix 的转置矩阵，就相对于取 WorldToViewMatrix 的第三行，得到了 viewDirection 的世界坐标。</font>

> 视频中的 viewDirectionWS 的获取的方式其实 Unity URP 已经用一个函数帮我们封装好了，我们直接使用 `GetWorldSpaceViewDir(positionWS)` 函数即可。

接着在片元着色器获取阴影坐标和主光源信息，并对需要使用的向量统一进行归一化处理：

``` C
float3 positionWS = input.positionWSAndFogFactor.xyz;
float4 shadowCoord = TransformWorldToShadowCoord(positionWS);
Light mainLight = GetMainLight(shadowCoord);
float3 lightDirectionWS = normalize(mainLight.direction);

float3 normalWS = normalize(input.normalWS);
float3 viewDirectionWS = normalize(input.viewDirectionWS);
```

## 环境光（间接光）
视频中对环境光模拟使用**球谐函数 Spherical harmonics**，**SH**，球谐函数从数学公式上来理解有些难度，反正我是看不懂。我们只需要知道该函数想表达什么，对环境光的模拟有什么帮助就行了。

球谐函数是傅里叶级数的高维类比，由一组表示球体表面的基函数构成。对于任意的周期函数，傅里叶基函数的线性组合都能够用来对函数进行拟合，类似的，对于任意球面函数，球谐函数的线性组合可以用来对球面函数进行拟合。和傅里叶基函数类似，基函数（阶数）越多，还原准确度越高。

<div  align="center">  
<img src="https://s2.loli.net/2024/03/22/1iZpQvR79XHohcS.png" width = "70%" height = "70%" alt="图7 - 球谐函数"/>
</div>

**球谐光照**是基于预计算辐射度传输理论实现的一种实时渲染技术。预计算辐射度传输技术能够实时重现在区域面光源照射下的全局照明效果。这种技术通过在运行前对场景中光线的相互作用进行预计算，计算每个场景中每个物体表面点的光照信息，然后用球谐函数对这些预计算的光照信息数据进行编码，在运行时读取数据进行解码，重现光照效果。

对于空间上的一点，受到的光照在各个方向上是不同的，也即各向异性，所以空间上一点如果要完全还原光照情况，那就需要记录周围球面上所有方向的光照。如果环境光照可以用简单函数表示，那直接求点周围球面上的积分就可以了。但是通常光照不会那么简单，并且用函数表示光照也不方便，所以经常用的方法是使用环境光贴图，比如 cubemap。假如我想知道这个点各个方向的光照情况，那么就必须在 cubemap 对应的各个方向进行采样。对于一个大的场景来说，每个位置点的环境光都有可能不同，如果把每个点的环境光贴图储存起来，并且每次获取光照都从相应的贴图里面采样，可想而知这样的方法是非常昂贵的。

利用球谐函数就可以很好的解决这个问题，由于球谐基函数阶数是无限的，所以只能取前面几组基来近似，一般在光照中大都取 3 阶，也即 9 个球谐系数，其中每组系数需要 3 个参数，总结 27 个参数。Unity 将这些光照信息编码为 unity_SHAr、unity_SHAg、unity_SHAb、unity_SHBr、unity_SHBg、unity_SHBb、unity_SHC 等参数，然后采样的时候，通过一定的算法对参数进行解码。对球谐函数采样可以在顶点着色器、片元着色器中进行或者混合模式（L0L1 在 fragment 里完成，vertex 中完成 L2）。其实 Unity URP 都把这些封装好了，我们直接使用就行。

> 这里不对球谐函数源码进行分析了，有空可以额外了解。以后应该会写关于 URP 源码的文章。

视频中选择在顶点着色器中进行采样，我们直接使用 URP 内置的 `SampleSH(normalWS)` 函数即可，采样得到的就是颜色。在顶点着色器添加如下代码：

    output.SH = SampleSH(lerp(vertexNormalInput.normalWS, float3(0, 0, 0), _IndirectLightFlattenNormal));

`_IndirectLightFlattenNormal` 参数默认为 0，可以控制它把法线压短来降低高阶项的影响。该参数为 0 时，即法线较强时，模型具有较多细节；该参数为 1 时，即法线为 0 时，反映在模型上就是纯色。因为我们是卡通渲染不需要这么多法线细节，所以所有的材质中将该参数都改为 1。

之后在片元着色器中计算环境光照，使用 `_IndirectLightUsage` 控制环境光使用量，增加的代码如下：

    float3 indirectLightColor = input.SH.rgb * _IndirectLightUsage;
    ...
    albedo += indirectLightColor;

## 使用光照图进行环境光遮罩
LightMap 的 R 通道包含了 AO 细节，即一些**静态阴影**，不受光照变化而变化的阴影。我们使用 Keyword 来区分上衣、下衣和头发以及脸部，毕竟脸部使用的是 FaceMap。  
①对于上衣、下衣和头发，将 indirectLightColor 乘上 LightMap 的 R 通道，并使用  `_IndirectLightOcclusionUsage` 参数来控制 R 通道的使用量，该值默认为 0.5；  
②对于脸部，相对复杂点，我们同时看脸部颜色贴图和 FaceMap 的 R 通道，R 通道的上面的白色长方形区域对应牙舌口的区域，黑色区块就是 SDF 区域遮罩，下面是眼睛睫毛眉毛。而 FaceMap 的 G 通道只有下面那块是亮的，也就是眼睛睫毛眉毛区域，而牙舌口的区域是黑色的，因为牙舌口常处于阴影中。利用这些信息做脸部 AO，我们要把去掉 SDF 动态阴影的部分，所以使用 `step()` 函数，当 faceMap.r > 0.5 返回 0（灰色部分也是大于 0.5 的，rgb 大概在 160，160，160），这时候使用 faceMap.g 中的阴影信息；当 faceMap.r <= 0.5 时，step 函数返回 1，也就是黑色动态 SDF 区域，lerp 函数直接返回 1，不使用 faceMap 的 G 通道。同时再套一个 lerp() 函数，同样使用 `_IndirectLightOcclusionUsage` 参数来控制阴影程度，该值默认为 0.5。

最后使用 `_IndirectLightMixBaseColor` 参数控制与 baseColor，也就是颜色贴图，的混合程度，该参数默认值为 1。

``` C
#if _AREA_HAIR || _AREA_UPPERBODY || _AREA_LOWERBODY
indirectLightColor *= lerp(1, lightMap.r, _IndirectLightOcclusionUsage);
#else
indirectLightColor *= lerp(1, lerp(faceMap.g, 1, step(faceMap.r, 0.5)), _IndirectLightOcclusionUsage);
#endif

indirectLightColor *= lerp(1, baseColor, _IndirectLightMixBaseColor);
```

静态阴影效果如下图（三月七头发光照图 R 通道无信息）：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/23/Yv9zRS4wlo2iGCa.jpg" width = "100%" height = "100%" alt="图8 - 静态阴影"/>
</div>

整体间接光照效果如下图：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/23/SAReVMj17r4wgc3.jpg" width = "100%" height = "100%" alt="图9 - 间接光照"/>
</div>

## 主光源（直接光）
首先使用兰伯特来控制阴影，添加如下代码：  

``` C
float mainLightShadow = 1;

#if _AREA_HAIR || _AREA_UPPERBODY || _AREA_LOWERBODY
{
    float NdotL = dot(normalWS, lightDirectionWS);
    // mainLightShadow = step(0, NdotL); //常用的 NPR 硬阴影
}
#elif _AREA_FACE
#endif
```

代码注释部分是常用的卡通风格的硬阴影，即把兰伯特**二值化**，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/23/ElvA8a2TOILnWPy.jpg" width = "100%" height = "100%" alt="图10 - 兰伯特二值化的硬阴影"/>
</div>

此时的阴影的细节肯定是不够的，就需要用到 LightMap 的 G 通道了，该通道代表了阴影的阈值，用这个阈值跟重映射后的 NdotL 进行比较，从而判断阴影区域。先将 NdotL 重映射到 0 到 1：

    float remappedNdotL = NdotL * 0.5 + 0.5;

LightMap 的 G 通道的灰度值越大，即该区域越容易被照亮，所以我们使用 `mainLightShadow = step(lightMap.g, remappedNdotL);` 就不太对，因为这样 lightMap.g 值越大，返回 0 的可能就越大，就意味着该区域属于阴影的可能越大，和我们的设想相反。所以使用如下代码：  

    mainLightShadow = step(1 - lightMap.g, remappedNdotL);

这样 lightMap.g 越大，返回 1 的可能就越大，该区域就越容易被照亮，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/23/5kvWSq7oNZfH8uE.jpg" width = "100%" height = "100%" alt="图11 - 使用光照图的动态阴影"/>
</div>

可以看到阴影细节多出来了非常多，可以转动太阳光，就可以看到动态的阴影变化了。但是用 `step()` 函数的结果阴影边缘会很硬，可以改用 `smoothstep()` 函数。

> **smoothstep(min, max, x)** 函数：如果 x 在 (min，max) 范围内，就返回介于 (0，1) 之间的平滑的**埃尔米特 Hermite 插值** ，它从开始逐渐加速，在接近结束时减慢速度。

同时使用 `_ShadowThresholdCenter` 和 `_ShadowThresholdSoftness` 参数来控制，修改后的代码如下：  

    mainLightShadow = smoothstep(1 - lightMap.g + _ShadowThresholdCenter - _ShadowThresholdSoftness, 1 - lightMap.g + _ShadowThresholdCenter + _ShadowThresholdSoftness, remappedNdotL);

看代码就知道，`_ShadowThresholdCenter` 控制的是开始产生阴影区域的位置，默认为 0；`_ShadowThresholdSoftness` 控制的是阴影边缘羽化的程度（宽度），默认为 0.1。可以自己改变参数看看效果，默认值的效果如下：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/23/jrT7nYVkucyCHWF.jpg" width = "100%" height = "100%" alt="图12 - 使用 smoothstep 后的“软”阴影效果"/>
</div>

将阴影值再乘上 LightMap 的 R 通道，也就是静态阴影区域：  

    mainLightShadow *= lightMap.r;

---

接下来是脸部阴影，也就是使用 FaceMap 的 A 通道，即 **SDF 有向距离场**图（SDF 可以看作一种矢量的渲染方式，这门技术建议额外去了解学习，这里就只讲如何使用了）。使用这张图很简单，就是用光照和角色方向的夹角跟贴图的值比较，从而产生阴影。注意对于计算面部阴影，是根据面部的朝向和光照的方向的差值去产生阴影，而不是面部的法线方向。所以我们需要一个头部的面朝方向，这个值用 C# 脚本去获取。首先在 Shader 先拿到头部的前向量和右向量，然后叉乘出上向量，注意左手坐标系下的叉乘顺序，增加的代码如下（在 `#elif _AREA_FACE` 下增加），别忘了归一化：  

    float3 headForward = normalize(_HeadForward);
    float3 headRight = normalize(_HeadRight);
    float3 headUp = cross(headForward, headRight);

因为对于计算面部阴影，y 轴的值是不需要的，我们只需要看 x、z 两个轴就可以了。所以我们要把光向量投影到头坐标系下的水平面（前右组成的平面），从而方便和头部方向计算。首先计算 lightDirectionWS 和 headUp 的 cos 值，因为这两个都归一化过，所以无需再次归一化，直接点乘。然后乘上垂直于水平面的上向量，得到光向量在水平面垂直方向上的投影，用光向量减去该值，得到光向量投影在水平面上指向光源的向量，最后归一化：

    float3 fixedLightDirectionWS = normalize(lightDirectionWS - dot(lightDirectionWS, headUp) * headUp);

然后用该光向量点乘右向量，这样光照到右脸为 1 到 0，照至左脸为 0 到 -1。使用 `sign()` 函数，这样光照到右脸就为 1，光照到左脸为 -1。但是 SDF 贴图右脸是黑的，左脸是白的，与我们方向相反，再取个负值。把这些都乘到 uv 的 u 上。这样光照从右脸到左脸，uv 就会水平翻转。然后根据该 uv 对 FaceMap 进行采样，代码如下：  

    float2 sdfUV = float2(sign(dot(fixedLightDirectionWS, headRight)), 1) * input.uv * float2(-1, 1);
    float sdfValue = SAMPLE_TEXTURE2D(_FaceMap, sampler_FaceMap, sdfUV).a;

效果如下，可以尝试改变光线看看：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/23/PB4DOTIg1tbJRNe.jpg" width = "100%" height = "100%" alt="图13 - SDF 左脸右脸阴影翻转"/>
</div>

上面没有解决光照到背面的问题，并且动态效果只有一个左脸和右脸阴影翻转，接下来我们需要通过一个阈值来控制光照的前后问题，以及让阴影变化更加合理自然。将光向量点乘前向量，在重映射到 0 至 1，再反向一下，这样照正面是 0，照背面是 1。我们希望光越照在正前方该阈值越低，即越容易被点亮。使用 `step()` 函数来跟上面算出来的 sdfValue 进行比较，sdfValue 比阈值大就会被点亮。增加代码如下：  

    float sdfThreshold = 1 - (dot(fixedLightDirectionWS, headForward) * 0.5 + 0.5);
    float sdf = step(sdfThreshold, sdfValue);

将 sdf 输出就可以看到一个合理的面部 SDF 阴影了，如下图：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/25/f4vVYC6D29ixoW3.jpg" width = "100%" height = "100%" alt="图14 - SDF 脸部硬阴影"/>
</div>

但是将光照方向设置到模型的正背面，可以看到面部仍然被点亮了，这是因为此时光向量和右向量成 90 度，故 sdfValue = 0；光向量和前向量成 180 度，点乘为 1，故 sdfThreshold = 0。因为 step(a, b) 比较是 b >= a 时为 1，此时相等，正好为 1，故有此错误。所以我们需要对 sdfValue 稍微偏移一点来避免此问题，需要减去个某个值，故我们对 sdfValue 加上 `_FaceShadowOffset`（默认值为 -0.01）：

    sdfValue += _FaceShadowOffset;

此时得到的面部阴影仍然比较硬，同理，把 `step()` 函数改为 `smoothstep()` 函数，并用 `_FaceShadowTransitionSoftness` 参数进行控制，改动后代码如下：  

    float sdf = smoothstep(sdfThreshold - _FaceShadowTransitionSoftness, sdfThreshold + _FaceShadowTransitionSoftness, sdfValue);

最后把 SDF 遮罩外的五官的阴影也显示出来，即使用 FaceMap 的 G 通道，并使用 R 通道进行阈值控制，原理之前使用光照图进行环境光遮罩中讲过，忘了可以回去看，就是当 R 通道小于 0.5，使用 SDF；大于 0.5，使用 FaceMap 的 G 通道，即 AO 部分。代码如下：  

    mainLightShadow = lerp(faceMap.g, sdf, step(faceMap.r, 0.5));

图片就不放出来了，就是多了点眼睛以及口腔内部的阴影细节。接下来需要使用 C# 脚本将头向量传递给 Shader，否则模型旋转阴影不会发生变化：

``` C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HeadVectorGenerator : MonoBehaviour
{
    public Transform HeadBoneTransform;
    public Transform HeadForwardTransform;
    public Transform HeadRightTransform;
    public string ShaderName = "Custom/StarRailToon"; //名字自己修改

    private Renderer[] allRenderers;

    private int HeadForwardID = Shader.PropertyToID("_HeadForward");
    private int HeadRightID = Shader.PropertyToID("_HeadRight");
    
#if UNITY_EDITOR
    private void OnValidate() //OnValidate 方法，在 Unity 加载脚本或检查器中的值更改时调用，而 #if UNITY_EDITOR 可以确保代码只在 Unity 编辑器中编译和执行，而不会在构建的游戏中执行。
    {
        LateUpdate();
    }
#endif

    private void LateUpdate()
    {
        if (allRenderers == null)
        {
            allRenderers = GetComponentsInChildren<Renderer>(true);
        }

        for (int i = 0; i &lt; allRenderers.Length; i++)
        {
            Renderer r = allRenderers[i];

            foreach (Material mat in r.sharedMaterials)
            {
                if (mat.shader.name == ShaderName)
                {
                    mat.SetVector(HeadForwardID, HeadForwardTransform.position - HeadBoneTransform.position);
                    mat.SetVector(HeadRightID, HeadRightTransform.position - HeadBoneTransform.position);
                }
            }
        }
    }
}
```

接着在模型的头骨上创建 3 个空物体分别为，HeadCenter、HeadRight、HeadForward，分别设置好其对应的位置。然后将这三个空物体赋予上面的脚本。这样子模型旋转可以实时更新阴影了，但是 Scene 窗口里不会变化，需要到 Game 窗口中运行游戏查看。

## 阴影渐变
首先查看身体的 Ramp 图，一张冷色一张暖色，都是 256 × 16 行，但是 16 行中每两行是一样的，所以实际上为 8 行，每行间隔 1/16 = 0.0625；头发的 Ramp 图，也是一张冷色一张暖色，虽然是 256 × 2 行，但是两行颜色都一样。用 LightMap 的 alpha 通道的灰度值来决定使用 Ramp 图的哪一个行，头发的 LightMap 的 alpha 通道是黑的是因为头发的 Ramp 图只有一行。

我们需要把灰度值转换为 Ramp 图的行序号，也就是 UV 的 V 部分。首先上衣和下衣的 LightMap 的 Alpha 通道一共有 8 个突变：分别为 5，37，69，101，133，165，197，228。都除以 255 分别为 0.02，0.15，0.27，0.4，0.52，0.65，0.77，0.89。将这些值 `round((Alpha + 0.0425)/0.0625)` 就可以得到 1，3，5，7，9，11，13，15 行。把这些数字减去 1 除以 2 就可以得到行序号 0 ~ 7 了。但是视频里说，偶数行的采样没问题，但是奇数行会有问题，需要转变 4 行，也就是原来的第 1 行对应第 5 行，第 3 行对应第 7 行，第 5 行对应第 1 行，第 7 行对应第 3 行，这就是为什么 `rawIndex + 4 < 8 ? rawIndex + 4 : rawIndex + 4 - 8`。同时使用 `fmod()` 函数来判断奇偶性，代码如下：

``` C
int rampRowIndex = 0;
int rampRowNum = 1;

#if _AREA_HAIR
{
    rampRowIndex = 0; //行序号
    rampRowNum = 1; //行总数
}
#elif _AREA_UPPERBODY || _AREA_LOWERBODY
{
    int rawIndex = (round((lightMap.a + 0.0425)/0.0625) - 1)/2;
    rampRowIndex = lerp(rawIndex, rawIndex + 4 < 8 ? rawIndex + 4 : rawIndex + 4 - 8, fmod(rawIndex, 2));
    rampRowNum = 8;
}
#elif _AREA_FACE
{
    rampRowIndex = 0; //脸也使用身体 Ramp 的第 1 行
    rampRowNum = 8;
}
#endif
```

接下来是 UV 的 U 部分，因为使用渐变纹理就是使用阴影值对其进行采样，LightMap 的 alpha 通道决定行号，而阴影值决定列号，阴影值越大，即阴影值越接近 1，采样值在右侧越亮；阴影值越小，越紧接 0，采样值在左侧越暗。观察 Ramp 图，实际上我们需要用到的是 Ramp 图的后面的部分，所以使用 `_ShadowRampOffset` 重映射一下，该值默认值为 0.75。新增代码如下：  

    float rampUVx = mainLightShadow * (1 - _ShadowRampOffset) + _ShadowRampOffset;
    float rampUVy = (2 * rampRowIndex + 1) * (1.0 / (rampRowNum * 2));
    float2 rampUV = float2(rampUVx, rampUVy);

mainLightShadow 的值范围为 (0, 1)，_ShadowRampOffset 为 0.75，即 rampUVx 被重映射到了 (0.75, 1)。rampUVy 则是把 8 行重新映射回 16 行。这样就构建好了 Ramp 图采样所需要的 UV 坐标。

之后就是对 Ramp 图进行采样，新增代码如下：  

    float3 coolRamp = 1;
    float3 warmRamp = 1;

    #if _AREA_HAIR
        coolRamp = SAMPLE_TEXTURE2D(_HairCoolRamp, sampler_HairCoolRamp, rampUV).rgb;
        warmRamp = SAMPLE_TEXTURE2D(_HairWarmRamp, sampler_HairWarmRamp, rampUV).rgb;
    #elif _AREA_FACE || _AREA_UPPERBODY || _AREA_LOWERBODY
        coolRamp = SAMPLE_TEXTURE2D(_BodyCoolRamp, sampler_BodyCoolRamp, rampUV).rgb;
        warmRamp = SAMPLE_TEXTURE2D(_BodyWarmRamp, sampler_BodyWarmRamp, rampUV).rgb;
    #endif

接下来就是判断什么时候使用暖色，什么时候使用冷色。光照朝上方为黑夜，也就是光向量的 y 值为负数（注意：光向量是指向光源的，所以反过来！！）；光照朝下方为白天，也就是光向量的 y 值为正数。白天用暖色，黑夜用冷色。因为光向量被归一化过，y 值范围为 (-1, 1)，需要将其重映射至 (0, 1) 方便使用 lerp() 函数，新增代码如下：

    float isDay = lightDirectionWS.y * 0.5 + 0.5;
    float3 rampColor = lerp(coolRamp, warmRamp, isDay);

将 rampColor 输出，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/25/bCjUiyzcfQnZhTo.jpg" width = "100%" height = "100%" alt="图15 - 阴影渐变"/>
</div>

最后把主光源的颜色和颜色贴图颜色混合上去，并使用 `_MainLightColorUsage` 参数来控制主光源的影响，该值默认为 1。增加的代码如下：  

    float3 mainLightColor = lerp(desaturation(mainLight.color), mainLight.color, _MainLightColorUsage);
    mainLightColor *= baseColor * rampColor;

`desaturation()` 函数是自己写的去掉饱和度的函数，怎么去饱和度的逻辑，详见《Unity Shader入门精要》读书笔记（四）。代码如下：  

    float3 desaturation(float3 color)
    {
        float3 grayXfer = float3(0.3, 0.59, 0.11); //这个值不同标准可能不太一样
        float grayf = dot(color, grayXfer);
        return float3(grayf, grayf, grayf);
    }

主光源（直接光）效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/25/2Wh8lHZXeVm6fPq.jpg" width = "100%" height = "100%" alt="图16 - 直接光"/>
</div>

将直接光和环境光混合，代码和效果如下：  

    float3 albedo = 0;
    albedo += indirectLightColor;
    albedo += mainLightColor;

<div  align="center">  
<img src="https://s2.loli.net/2024/03/25/pTb1mt7rLWzSwnv.jpg" width = "100%" height = "100%" alt="图17 - 直接光和间接光"/>
</div>

## 高光
只有头发和身体有高光，首先使用 Blinn-Phong 模型，`_SpecularExponent` 参数控制高光指数，默认值为 50，新增代码如下：  

``` C
float3 specularColor = 0;
    
#if _AREA_HAIR || _AREA_UPPERBODY || _AREA_LOWERBODY
{
    float3 halfVectorWS = normalize(viewDirectionWS + lightDirectionWS);
    float NdotH = dot(normalWS, halfVectorWS);
    float blinnPhong = pow(saturate(NdotH), _SpecularExponent);
}
```

LightMap 的 B 通道控制着高光的阈值，该区域越亮越会产生高光。观察 LightMap 的 B 通道，可知皮肤和布料衣服没有高光，皮料衣服和衣服上的金属有高光，金属的高光强度差异很大，同时强度也会大于皮料衣服。卡通渲染一般都会限制高光产生的区域。为了区分金属和非金属的高光，我们计算了 nonMetalSpecular 和 metalSpecular：  
①为了防止高光过于油腻（可以试试 blinnPhong * lightMap.b 直接输出颜色），nonMetalSpecular 使用 `step()` 函数对高光进行二值化。lightMap.b 越亮越会产生高光，所以把 blinnPhong 反向，使用 1.04 是为了偏移一点，可以防止 blinnPhong 为 1，lightMap.b 为 0 时，光照图黑色区域也产生高光。同时这个 0.04 的偏移值越大，NdotH 就需要越大才能产生高光。`_SpecularKsNonMetal` 控制的则是非金属的反射率，非金属反射率固定为 0.04。代码如下：  

> 参数中的 Ks 指镜面反射系数。离线渲染中，通常可以用 kd,ks,kt (分别代表物体的漫反射系数，镜面反射系数，透射系数) 来简单地描述一个物体的基本材质。

    float nonMetalSpecular = step(1.04 - blinnPhong, lightMap.b) * _SpecularKsNonMetal;

② metalSpecular 就简单点，因为不同金属反射率差异很大，阈值图直接用作金属反射率，并使用 `_SpecularKsMetal` 控制，该值默认为 1：

    float metalSpecular = blinnPhong * lightMap.b * _SpecularKsMetal;

接下来就是区分金属和非金属区域了，使用 LightMap 的 A 通道可以区分，其中 0.52 是金属对应的部位，因此可以据此提取出来金属，这里取 a 通道和 0.52 的差值的绝对值，使用 0.1 作为插值范围，也就是 a 通道指在 0.42 - 0.62 之间被映射到了 0 - 1 之间，越靠近 0.52 金属度越大。新增如下代码：  

    float metallic = 0;

    #if _AREA_UPPERBODY || _AREA_LOWERBODY
        metallic = saturate((abs(lightMap.a - 0.52) - 0.1) / (0 - 0.1));
    #endif

> 这里我不知道为什么，和视频中效果不太一样，我的效果并不太好，采样出来的 metalic 锯齿感很强烈，所以我先把 0.1 改为了 0.0625，还使用了 step() 函数：`metallic = step(0.01, metallic);` 把金属区域的值都重映射到了 1。

之后，根据区分后的金属或非金属区域，使用 `lerp()` 函数进行区分，因为金属会吸收光的颜色并反射自身颜色，故金属高光乘上 baseColor。再乘上主光源颜色和高光亮度控制参数 `_SpecularBrightness`，该值默认值为 1：  

    specularColor = lerp(nonMetalSpecular, metalSpecular * baseColor, metallic);
    specularColor *= mainLight.color;
    specularColor *= _SpecularBrightness;

将 `_SpecularExponent` 参数改为 10，`_SpecularBrightness` 参数改为 10 就可以看见明细的高光了，效果如下：  

<div  align="center">  
<img src="https://s2.loli.net/2024/03/25/HypAQ9dWMSBcsDL.jpg" width = "100%" height = "100%" alt="图18 - 高光"/>
</div>

将高光加入 albedo：

    albedo += specularColor;

这样间接光 + 直接光 + 高光的效果如下：

<div  align="center">  
<img src="https://s2.loli.net/2024/03/25/GTjboau6gsPhilM.jpg" width = "100%" height = "100%" alt="图18 - 间接光直接光高光，可爱捏~"/>
</div>

欲知后事如何，请看下篇分解。