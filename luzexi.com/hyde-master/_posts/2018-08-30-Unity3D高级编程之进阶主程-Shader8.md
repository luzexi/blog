---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第七章，Shader(八) - Suferface
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

###### Surface Shaders 是Unity3D创建的，用来简化手动编写Shader的一种方式。Surface 会帮我们生成HLSL，但不是种语言，我们依然要关注HLSL如何编写。

Surface shader 和 vertex fragment shader 一样需要书写 CGPROGRAM..ENDCG，并且写入#pragma surface 来表明使用Surface，不同的是：Surface 它不能有Pass，所有的Surface编码都写在SubShader中，然后再由shader编译器编译成多个Pass。

###### #pragma surface 的格式如下：

		#pragma surface surfaceFunction lightModel [可选择参数]

参数说明：

1，surfaceFunction，是一个有输入输出参数的函数，其格式为

		void surf (Input IN, inout SurfaceOutput o)

Surface通过这个函数来修改渲染细节，下面的程序中必须有这个名字和格式的函数。


2，lightModel，选择使用哪种光照模型。在内置的光照模型中，有以物理为基础的 Standard 和 StandardSpecular，和无物理基础的Lambert (diffuse漫反射) and BlinnPhong (specular镜面反射)。还可以定义自己的光照模型。其中内置的光照模型：

		Standard 光照模型使用 SurfaceOutputStandard 作为输出结构并且匹配了Unity3D内置的Standard Shader(金属流)。

		StandardSpecular 光照模型使用 SurfaceOutputStandardSpecular 作为输出结构并且匹配了Unity3D内置的Standard Shader(镜面反射)。

		Lambert 和 BlinnPhong 光照模型不以物理为基础，不过使用它们可以在低端设备上运行得更快。


3，可选择参数。

Transparency 和 alpha testing

		 Transparency 和 alpha testing 是控制alpha和alphaTest也就是alpha裁切 的参数。透明物体可以有两种，一种是传统的alpha混合，另一种是更加物理化的“premultiplied blending”左自乘混合，它能够在半透明上保留适度的镜面反射。
		 is controlled by alpha and alphatest directives. Transparency can typically be of two kinds: traditional alpha blending (used for fading objects out) or more physically plausible “premultiplied blending” (which allows semitransparent surfaces to retain proper specular reflections). Enabling semitransparency makes the generated surface shader code contain blending commands; whereas enabling alpha cutout will do a fragment discard in the generated pixel shader, based on the given variable.

alpha or alpha:auto - Will pick fade-transparency (same as alpha:fade) for simple lighting functions, and premultiplied transparency (same as alpha:premul) for physically based lighting functions.
alpha:blend - Enable alpha blending.
alpha:fade - Enable traditional fade-transparency.
alpha:premul - Enable premultiplied alpha transparency.
alphatest:VariableName - Enable alpha cutout transparency. Cutoff value is in a float variable with VariableName. You’ll likely also want to use addshadow directive to generate proper shadow caster pass.
keepalpha - By default opaque surface shaders write 1.0 (white) into alpha channel, no matter what’s output in the Alpha of output struct or what’s returned by the lighting function. Using this option allows keeping lighting function’s alpha value even for opaque surface shaders.
decal:add - Additive decal shader (e.g. terrain AddPass). This is meant for objects that lie atop of other surfaces, and use additive blending. See Surface Shader Examples
decal:blend - Semitransparent decal shader. This is meant for objects that lie atop of other surfaces, and use alpha blending. See Surface Shader Examples
Custom modifier functions can be used to alter or compute incoming vertex data, or to alter final computed fragment color.

vertex:VertexFunction - Custom vertex modification function. This function is invoked at start of generated vertex shader, and can modify or compute per-vertex data. See Surface Shader Examples.
finalcolor:ColorFunction - Custom final color modification function. See Surface Shader Examples.
finalgbuffer:ColorFunction - Custom deferred path for altering gbuffer content.
finalprepass:ColorFunction - Custom prepass base path.
Shadows and Tessellation - additional directives can be given to control how shadows and tessellation is handled.

addshadow - Generate a shadow caster pass. Commonly used with custom vertex modification, so that shadow casting also gets any procedural vertex animation. Often shaders don’t need any special shadows handling, as they can just use shadow caster pass from their fallback.
fullforwardshadows - Support all light shadow types in Forward rendering path. By default shaders only support shadows from one directional light in forward rendering (to save on internal shader variant count). If you need point or spot light shadows in forward rendering, use this directive.
tessellate:TessFunction - use DX11 GPU tessellation; the function computes tessellation factors. See Surface Shader Tessellation for details.
Code generation options - by default generated surface shader code tries to handle all possible lighting/shadowing/lightmap scenarios. However in some cases you know you won’t need some of them, and it is possible to adjust generated code to skip them. This can result in smaller shaders that are faster to load.

exclude_path:deferred, exclude_path:forward, exclude_path:prepass - Do not generate passes for given rendering path (Deferred Shading, Forward and Legacy Deferred respectively).
noshadow - Disables all shadow receiving support in this shader.
noambient - Do not apply any ambient lighting or light probes.
novertexlights - Do not apply any light probes or per-vertex lights in Forward rendering.
nolightmap - Disables all lightmapping support in this shader.
nodynlightmap - Disables runtime dynamic global illumination support in this shader.
nodirlightmap - Disables directional lightmaps support in this shader.
nofog - Disables all built-in Fog support.
nometa - Does not generate a “meta” pass (that’s used by lightmapping & dynamic global illumination to extract surface information).
noforwardadd - Disables Forward rendering additive pass. This makes the shader support one full directional light, with all other lights computed per-vertex/SH. Makes shaders smaller as well.
nolppv - Disables Light Probe Proxy Volume support in this shader.
noshadowmask - Disables Shadowmask support for this shader (both Shadowmask and Distance Shadowmask).
Miscellaneous options

softvegetation - Makes the surface shader only be rendered when Soft Vegetation is on.
interpolateview - Compute view direction in the vertex shader and interpolate it; instead of computing it in the pixel shader. This can make the pixel shader faster, but uses up one more texture interpolator.
halfasview - Pass half-direction vector into the lighting function instead of view-direction. Half-direction will be computed and normalized per vertex. This is faster, but not entirely correct.
approxview - Removed in Unity 5.0. Use interpolateview instead.
dualforward - Use dual lightmaps in forward rendering path.
To see what exactly is different from using different options above, it can be helpful to use “Show Generated Code” button in the Shader Inspector.
