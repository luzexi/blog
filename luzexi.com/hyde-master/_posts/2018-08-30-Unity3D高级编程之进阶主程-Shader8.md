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

###### 1，surfaceFunction，是一个有输入输出参数的函数，其格式为

		void surf (Input IN, inout SurfaceOutput o)

Surface通过这个函数来修改渲染细节，下面的程序中必须有这个名字和格式的函数。


###### 2，lightModel，选择使用哪种光照模型。在内置的光照模型中，有以物理为基础的 Standard 和 StandardSpecular，和无物理基础的Lambert (diffuse漫反射) and BlinnPhong (specular镜面反射)。还可以定义自己的光照模型。其中内置的光照模型：

		Standard 光照模型使用 SurfaceOutputStandard 作为输出结构并且匹配了Unity3D内置的Standard Shader(金属流)。

		StandardSpecular 光照模型使用 SurfaceOutputStandardSpecular 作为输出结构并且匹配了Unity3D内置的Standard Shader(镜面反射)。

		Lambert 和 BlinnPhong 光照模型不以物理为基础，不过使用它们可以在低端设备上运行得更快。


###### 3，可选择参数。

###### 透明物体 和 alpha testing 功能参数

透明物体 和 alpha testing 类型的参数可以控制alpha和alphaTest也就是alpha混合和alpha裁切 的功能。

透明物体分为两种，一种是传统的alpha混合，另一种是更加物理化的“premultiplied blending”左自乘混合，它能够在半透明上保留适度的镜面反射。

启用半透明使得surface shader包含了alpha混合功能。而启用alpha cutout 将根据alpha值在生成像素时中裁切不符合的部分片段。

透明物体和alpha testing的功能指令如下：

		alpha 或者 alpha:auto，选择渐进透明也就是 alpha:fade 作为简单光照函数，并且选择左自乘透明方式也就是 alpha:premul 作为物理基础的光照函数。

		alpha:blend，开启alpha混合。

		alpha:fade，开启传统的渐进透明函数。

		alpha:premul，开启左自乘alpha透明度。

		alphatest:VariableName，开启alpha cutout裁切功能。将根据跟定的值裁切不符合的alpha片段。裁切后你可以通过 addshadow 指令生成阴影。

		keepalpha，保持alpha值，让那些不透明的物体也有alpha通道为1.0的值。这个操作使得灯光函数在不透明物体上也起到alpha的作用。

		decal:add，开启叠加贴花功能，让贴花贴图使用叠加混合。

		decal:blend，开启半透明贴花功能，让贴花贴图使用alpha混合。

###### 

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
