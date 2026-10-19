---
title: Clear coat & Anisotropic 材质
date: 2026-10-09 18:59:14
categories: 
  - [游戏开发, 图形]
  - [游戏开发, Unity]
tags:
  - 图形
  - 游戏开发
  - Unity
  - PBR
  - Shader
top_img: /images/black.jpg
cover: https://files.seeusercontent.com/2026/09/04/1Kto/Clearcoat-Anisotropic.gif
mathjax: true
description: XXXXXXXXXXXXXXXXXXXXXXXXXX
---

> 之前在 [Custom Better PBR in Unity](https://ybniaobu.github.io/2024/10/22/2024-10-22-BetterPBR1/) 中实现了 BRDF 的标准/基础材质模型，主要用于模拟**各向同性 Isotropic** 的**电介质 Dielectric**（非金属 non-metallic）或**导体 Conductor**（金属 metallic）材质表面。本篇文章在此基础上继续补充**清漆/透明涂层 Clear coat** 和**各向异性 Anisotropic** 材质。
>
> 本文主要参考了 Filament 的 [Physically Based Rendering](https://google.github.io/filament/Filament.md.html)，Khronos 的 glTF 2.0 材质扩展：[KHR_materials_clearcoat](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_clearcoat) 与 [KHR_materials_anisotropy](https://github.com/KhronosGroup/glTF/tree/main/extensions/2.0/Khronos/KHR_materials_anisotropy) 。

# Clear coat
## BRDF
标准材质模型适合描述由单一层构成的各向同性表面，但**多层材质 Multi-layer Material** 其实相当常见，尤其是在**基础层 Base Substrate** 之上再覆盖一层薄薄半透明层的材质，在现实中的例子有诸如车漆、易拉罐、漆面木材、亚克力等。这种清漆层是一层**电解质 Dielectric** 材料，它能透射光线而不散射，但可能会吸收光线。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/10/26/xSe2/ClearcoatAnisotropic01_CoatLayer.png" width = "50%" height = "50%" alt="图1 - Clear Coat Layer"/>
</div>

清漆层可以作为标准材质模型的一种扩展来模拟，即在原模型/基础层上再叠加一个高光波瓣，同样使用 Cook-Torrance 微表面 BRDF，即：  

$$ f_r(l, v) = \cfrac {D(h)F(l, h)G(l, v)} {4(v \cdot n)(l \cdot n)} $$

由于清漆层通常固定为低粗糙度的各向同性电介质，除了选用标准材质模型的 DFG 项外，我们还可以选用一个计算代价更低的 V 项来替换 Smith-GGX 可见性项，但不会明显牺牲视觉质量，即 Kelemen 提出的一个遮蔽阴影函数，公式和代码如下：  

$$ V(l,h) = \cfrac {1}{4(l \cdot h)^2} $$

    float V_Kelemen(float LoH)
    {
        return 0.25 / (LoH * LoH);
    }

> 这个近似可见性项的提出主要是为了快速计算以及便于进行重要性采样，它出自 Csaba Kelemen 与 László Szirmay-Kalos 在 Eurographics 2001 的文章 A Microfacet Based Coupled Specular-Matte BRDF Model with Importance Sampling 中提出的模型。同时，这个近似只有当表面趋近完美镜面，此时微表面法线 $\,h\,$ 与宏观法线 $\,n\,$ 高度接近，即 $\,l \cdot h \approx n \cdot l \approx n \cdot v\,$ 时，代替 Smith-GGX 可见性项的误差才小。反过来，粗糙度一旦升高，$\,h\,$ 与 $\,n\,$ 偏离越大，该近似就会带来可见误差并破坏能量守恒。由于清漆层通常是低粗糙度的，故可以套用此近似项。

总的来说，Clear Coat BRDF 是一个 Cook-Torrance 镜面反射微表面 BRDF，其中法线分布函数 D 选用 GGX、菲涅尔项 F 选用 Schlick 近似式、可见性项 V 选用 Kelemen 近似。当然可见性项选用 Smith-GGX 完全也是可以的，只是要多付出一些计算量以换取更符合物理模型的效果。在大多数低粗糙度清漆层的场景下，两者的视觉结果几乎没有差别。本文两种 V 项都会介绍，下面将使用 Kelemen 可见性项的 Coat 模型称为<u>近似模型</u>，使用 Smith-GGX 可见性项的模型称为<u>完整模型</u>。

> 这里所说的“完整模型”只是为了行文方便而采用的称呼，严格来说它并不完整，毕竟仍是实时渲染下的近似。实时渲染中的 Clear Coat 模型有许多 trick 可以自由取舍，如果你不满意我在完整模型里采用的做法，完全可以自行添加或修改，不过我认为它目前的效果已经足够，而且近似模型在多数情况下也基本够用。

## 材质参数
清漆材质模型包含此前为标准材质模型定义的全部参数，并在此基础上额外增加了以下的参数：  

| 参数 | 可选性 | 含义 |
| :---- | :---- | :---- |
| ClearCoat | 必需 | 清漆层的强度，取值 0 到 1 |
| ClearCoatRoughness | 必需 | 清漆层粗糙程度，取值 0 到 1 |
| ClearCoatColor | 可选 | 清漆层自身的颜色，用于模拟有色清漆。它作为透射/吸收色，会衰减穿过清漆层的光线 |
| ClearCoatIOR | 可选 | 清漆层与空气界面的折射率（Index Of Refraction），默认 1.5，对应垂直入射反射率 F0 = 0.04 |
| ClearCoatNormal | 可选 | 清漆层专用的切线空间法线贴图，用于单独刻画清漆层自身的表面细节，与基础材质的法线相互独立 |

* 近似模型只使用两个必需的参数，即清漆强度 ClearCoat 和清漆层粗糙度 ClearCoatRoughness。另外三个可选参数在完整模型中使用；  
* 如果没有提供清漆层法线贴图 ClearCoatNormalTexture，那么清漆层法线视同基础层法线。我个人认为清漆层法线贴图的必要性非常低，因为这样艺术家就要先为基础层制作一次高模，并烘焙基础层法线，再修改基础层高模，为其添加清漆层，然后烘焙清漆层法线，非常麻烦，更不要说清漆层对模型表面的影响微乎其微。但它适用于油漆的皱皮效果或者天然带有纹路的漆料；  
* 现代油漆/涂料的主要成分诸如丙烯酸树脂、醇酸树脂、环氧树脂和聚氨酯的折射率 IOR 基本都在 1.5 左右，漆面木材使用的天然大漆/生漆的主要成分漆酚的折射率也大致等于 1.5，由此可以推算出 F0 = 0.04。故近似模型固定使用 0.04 的菲涅尔反射率 F0，而完整模型提供 IOR 以允许艺术家进行更精细的控制，由于绝大多数材质的折射率不会超过 4，故可以钳制在 1 - 4 的范围内；  

$$ f_0(1.5) = \cfrac {(1.5 - 1)^2}{(1.5 + 1)^2} = 0.04 $$

* 清漆层颜色 ClearCoatColor 是一个非常特殊的参数，跟基础层的 Albedo 表达的菲涅尔反射率不同，它表达的是垂直入射的**透射率 Transmittance**，用于模拟清漆层的光线**吸收 Absorption** 效应，材质的观察颜色也由其部分决定。需要注意的是，清漆层的反射是无颜色的。

## 近似模型
近似模型忽略了光线在清漆层和基础层之间的许多影响，具有以下假设：  
1. 清漆层无限薄，且不存在折射；  
2. 清漆层和基础层的折射率互不影响，菲涅尔项可以独立计算；  
3. 层与层之间不存在散射现象；  

另外为了材质的能量守恒，需要使用 Clear Coat BRDF 的 Fresnel 项 $\,F_c\,$ 来考虑能量损失（不使用 ClearCoatColor），故 BRDF 公式可以写为：  

$$ f(l,v) = f_d(l,v)(1 - F_c) + f_r(l,v)(1 - F_c) + f_c(l,v) $$

其中 $\,f_d\,$、$\,f_r\,$、$\,f_c\,$ 分别为 Diffuse BRDF、Specular BRDF 和 Clear Coat BRDF。

### 直接光部分
直接光部分的逻辑上面都讲过了，直接上伪代码如下：  

    float3 BRDF(...)
    {
        ...
        // compute Fd and Fr from standard model

        float coatAlphaRoughness = coatRoughness * coatRoughness;

        // clear coat BRDF
        float  Dc = D_GGX(NoH, coatAlphaRoughness);
        float  Vc = V_Kelemen(LoH);
        float  Fc = F_Schlick(0.04, VoH) * clearCoat; // clear coat strength
        float Frc = Dc * Vc;

        return lerp(Fd + Fr, Frc, Fc); // account for energy loss in the base layer
    }


### IBL 部分
IBL 部分也进行了简化，直接使用宏观菲涅耳项（基于 $\,n \cdot v\,$ 而非 $\,v \cdot h\,$ 或 $\,l \cdot h\,$）乘以采样立方体贴图得到的预过滤环境贴图颜色，也就是不使用 Spilt Sum Approximation 的老 IBL 近似方案，在文章 [IBL 基于图像的光照（二）](https://ybniaobu.github.io/2024/08/16/2024-08-16-IBL_Basics2/) 开头也提到过。至于能量守恒，也是使用宏观菲涅耳项，伪代码如下：  

    float3 evaluateSpecularIBL(float3 r, float coatRoughness)
    {
        float lod = 8.0 * coatRoughness; // This assumes 9 mip levels
        return decodeEnvironmentMap(textureCubeLod(environmentMap, r, lod));
    }

    float3 evaluateIBL(...)
    {
        float Fc = F_Schlick(0.04, NoV) * clearCoat;
        iblDiffuse *= 1.0 - Fc;
        iblSpecular *= 1.0 - Fc;
        iblSpecular += evaluateSpecularIBL(r, coatRoughness) * specularOcclusion * Fc;
    }

下图是我在 Unity 自己管线中实现的清漆近似模型效果，其中红蓝球的金属度都为 1，粗糙度都为 0.6。红球从左到右对应 ClearCoat 参数从 0 到 1。蓝球 ClearCoat 参数为 1，从左到右对应 ClearCoatRoughness 参数从 0 到 1：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/10/27/F4vy/ClearcoatAnisotropic02_ClearCoat.png" width = "60%" height = "60%" alt="图2 - Clear Coat 近似模型效果"/>
</div>

可以看到当 ClearCoatRoughness 较高时，由于使用的是宏观菲涅耳，在掠射角附近趋近 1，此时会被不带颜色的清漆层所主导，故在球体边缘呈现白色，这也是近似模型在视觉上的主要瑕疵，但在低粗糙度的情形下与完整模型的效果是非常接近的。

### Emission 部分
glTF 2.0 材质扩展中有提到，由于清漆层是在发光层之上的，所以会变暗，可以让清漆层的宏观菲涅尔项去减弱 emission 的亮度：  

    coatEmission = emission * (1 - clearCoat * clearCoatFresnel)

我个人感觉完全没有这个必要性，自己也尝试了一下，对于发光物体，肉眼基本看不出区别，特别是在有 Bloom 后处理的时候，所以我并没有采用。

## 完整模型
完整模型的假设与近似模型大致相同，二者的主要区别在于：完整模型对清漆层使用与基础层相同的 Specular BRDF，并额外考虑清漆层对光线的吸收，由此引入了 ClearCoatColor 参数。在考虑清漆层光线吸收的同时，还额外考虑了一个物理效应：即随视角变化，光线在清漆层中的传播距离变长，被吸收得更多，从而导致的变暗现象，具体详见 View-Dependent Absorption 小节。由上所述，一个能量守恒的完整模型可大致建模为：  

$$ f(l, v) = f_c(l, v) + T_c(l, v)(1 - E_c(v)) f_{sub}(l, v) $$

其中 $\,f_c\,$、$\,f_{sub}\,$ 分别为清漆层 BRDF 和基础 (Substrate) 层 BRDF；$\,T_c\,$ 为清漆层体积透射率 Transmittance，也就是 ClearCoatColor；$\,E_c(v)\,$ 为清漆层反射率，这部分我在代码中并没有考虑进去，尝试过使用宏观菲涅尔，但肉眼几乎看不到区别，故舍弃。如果考虑清漆强度参数 $\,\omega_c\,$，上述公式还可以写为：  

$$ f(l, v) = \omega_c f_c(l, v) + lerp(1,T_c(1 - E_c),\omega_c) f_{sub}(l, v) $$

### View-Dependent Absorption
对于有吸收效应的清漆层来说，由于在介质中光程的变长，会呈现出掠射角下的变暗效应。当然入射光和出射光都有该效应，这种效应可以通过透射率参数被建模为（这部分内容参考了 [OpenPBR Surface Specification](https://academysoftwarefoundation.github.io/OpenPBR/index.html#model/coat/view-dependentabsorption) 的相关内容）：  

$$ T_c^{\textstyle 1 / \mu_i^t + 1 / \mu_o^t} $$

其中 $\,\mu_i^t\,$ 和 $\,\mu_o^t\,$ 分别为入射出射光线折射进清漆层的余弦值，可以通过折射率 $\,\eta_c\,$ 和入射出射角度的余弦值 $\,\mu_i\,$ 与 $\,\mu_o\,$ 计算出来：  

$$ \mu_i^t = \sqrt{1 - \cfrac{1 - \mu_i^2}{\eta_c^2} } $$

注意，OpenPBR 中将 ClearCoatColor 定义为了透射率的平方 $\,T_c^2\,$，正好是上述公式在垂直入射时的结果。因为考虑入射光线在实时渲染中不太方便，特别是 IBL 部分，另外为了直接光与 IBL 统一，我对上述公式做出了简化，即只考虑出射方向，也就是视角方向 $\,v\,$，并根据清漆强度在 1 与 ClearCoatColor 之间进行插值，代码如下：  

    float3 ClearCoatAbsorption(float coat, float coatIOR, float coatNoV, float3 coatColor)
    {
        float coatNeta = 1.0 / coatIOR;
        float cosNT = sqrt(1.0 - coatNeta * coatNeta * (1.0 - coatNoV * coatNoV));
        return lerp(1.0, pow(coatColor, 1.0 / cosNT), coat);
    }

### 直接光部分
注意一下，若使用 ClearCoatNormal 贴图的话，clear coat BRDF 中使用到的跟法线 N 有关的参数不要错误使用基础层的法线计算了。其它也没什么好说的了，直接上伪代码：  

    float3 BRDF(...)
    {
        ...
        // compute Fd and Fr from standard model

        float coatAlphaRoughness = coatRoughness * coatRoughness;
        float coatF0 = IORToF0(coatIOR);
        float3 coatAbsorption = ClearCoatAbsorption(coat, coatIOR, coatNoV, coatColor);

        // clear coat BRDF
        float  Dc = D_GGX(coatNoH, coatAlphaRoughness);
        float  Vc = V_SmithGGXCorrelated(coatNoV, coatNoL, coatAlphaRoughness);
        float  Fc = F_Schlick(coatF0, VoH);
        float Frc = Dc * Vc * Fc;

        return (Fd + Fr) * coatAbsorption + Frc * coat;
    }

### IBL 部分
IBL 部分改为使用 Spilt Sum Approximation 的方案，就是采用预积分贴图时，需要采用两次，基础层和清漆层各一次，毕竟粗糙度不一致，伪代码如下：  

    float2 prefilteredDFG_LUT(float roughness, float NoV) 
    {
        return texture2DLod(dfgLut, float2(NoV, roughness), 0.0).rg;
    }

    float3 evaluateIBL(...)
    {
        iblDiffuse *= coatAbsorption;
        iblSpecular *= coatAbsorption;

        float3 coatR = reflect(-v, coatN);
        float coatNoV = max(dot(coatN, v), 1e-4);
        float3 prefiltered = evaluateSpecularIBL(coatR, coatRoughness);
        float2 env = prefilteredDFG_LUT(coatRoughness, coatNoV);
        float3 specularColor = coatF0 * env.x + (1.0 - coatF0) * env.y;

        iblSpecular += prefiltered * specularColor * specularOcclusion * coat;
    }

最终在 Unity 中效果如下，其中红蓝绿球的金属度都为 1，粗糙度都为 0.6。值得注意的是，与近似模型不同，红蓝绿球的 BaseColor 都为白色，只是 ClearCoatColor 都设置为了对应的颜色，故第一排的效果逻辑不一致。第一排红球从左到右对应 ClearCoat 参数从 0 到 1，ClearCoatRoughness 参数都为 0，ClearCoatIOR 为默认 1.5；第二排蓝球 ClearCoat 参数都为 1，从左到右对应 ClearCoatRoughness 参数从 0 到 1，ClearCoatIOR 为默认 1.5；第三排绿球 ClearCoat 参数都为 1，ClearCoatRoughness 参数都为 0，从左到右 ClearCoatIOR 从 1 到 3.5，每个增加 0.5。

<div align="center">  
<img src="https://files.seeusercontent.com/2026/10/27/Syd6/ClearcoatAnisotropic03_ClearCoat.png" width = "60%" height = "60%" alt="图3 - Clear Coat 完整模型效果"/>
</div>

我还与 Blender 的 EEVEE 渲染器做了对比，由于光照单位不一致，只能大致调整，但效果可以说很接近了（色调映射使用 ACES 1.3 和 Unity 统一）：  

<div align="center">  
<img src="https://files.seeusercontent.com/2026/10/28/0bgM/ClearcoatAnisotropic04_BlenderCl.png" width = "60%" height = "60%" alt="图4 - Blender EEVEE 渲染器的 Clear Coat 效果"/>
</div>

### 其它可选择项
OpenPBR 中还提到清漆材质的另外三种物理现象，可以考虑添加以获取更 ground-truth 的效果（这里就简单说明一下，有兴趣可以自行查看 OpenPBR 的文档）：  
1. 由于清漆层内部多次反射，导致光线多次照射到下层材料以及更多的吸收，因此材质的观察颜色会变深。具体表现为随着清漆层 IOR 的增加，导致内部反射增加，材质的表面颜色会越来越深。但这种效应并不美术友好，因为艺术家往往希望材质观察颜色匹配上输入的颜色，所以 OpenPBR 采用了 ClearCoatDarkening 参数来进行控制。我也觉得这种效应可以通过调暗 ClearCoatColor 颜色来近似模拟，故没有采用；  
2. 粗糙清漆层的存在会增加基础层波瓣的表现粗糙度，这种效应可以通过修改基础层粗糙度来模拟。我感觉必要性不大，也有美术不友好的问题，且清漆层通常都是低粗糙度的，该效应基本可以忽略；  
3. 全内反射效应 Total Internal Reflection (TIR)，这玩意初中就应该学过：当光线从较高折射率的介质进入到较低折射率的介质时，如果入射角大于某一临界角时，折射光线会消失，导致所有的入射光线被反射。当清漆层 IOR 高于基础层 IOR 时，就有可能出现这效应，从而导致基础层的反射增加，没有光线折射进基础层。不过要注意，在平滑界面的近似下，从外部入射的光线实际上不可能发生 TIR，故我的模型中不考虑。

## 渲染路径问题
前向管线下，近似模型和完整模型的实现都比较直接，均无特殊处理，这里主要讨论它们在延迟管线下的问题和处理方式。
* 首先是 ClearCoatNormal 贴图的问题。在延迟管线下，如果要为清漆层保存一份独立法线，几乎必须为它额外开一个 GBuffer 渲染纹理，代价太大，我认为不值得。因此建议延迟管线下不使用专用贴图，或者对需要它的材质在延迟管线中改走前向渲染；  
* 对于近似模型，可以在延迟光照之后额外用一个 Alpha 混合的叠加 Pass 来绘制清漆层。基础模型照常由延迟光照 Pass（全屏 Shader）计算并写入颜色缓冲，随后以 `ZTest Equal`、`ZWrite Off` 再绘制一遍带有清漆层的几何体（前向）或者使用 stencil 做全屏绘制（从 GBuffer 取 clearCoat / coatRoughness，当然如果这样直接和基础材质一起走一次全屏绘制就行，完全没必要分开）。该 Pass 只求清漆层的镜面波瓣 `Dc * Vc`（不含菲涅耳权重），并把 `clearCoat * F_Schlick(0.04, NoV)` 存入 alpha，混合方式设为 `Blend SrcAlpha OneMinusSrcAlpha, Zero One`，此时颜色缓冲的结果为 `base * (1 - Fc) + coat * Fc`。由于混合权重只能取标量，这里的 `Fc` 只能基于 `NoV` 的宏观菲涅耳项计算，无法像直接光那样使用 `VoH`，因此直接光部分的效果会略有折扣；  
* 完整模型则基本只建议在延迟管线中走前向渲染。一方面它除了 ClearCoatNormal 还需要 ClearCoatColor（RGB）与 ClearCoatIOR，GBuffer 根本塞不下。另一方面，它的基础层衰减是基于 coatAbsorption 的三个通道，无法像近似模型那样用一个叠加 Pass 复原，从而做到部分延迟部分前向，因此只能整体前向求值。

# Anisotropic

