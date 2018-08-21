---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(八) - Surface
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

###### Surface Shaders 是Unity3D创建的，用来简化手动编写Shader的一种方式。

###### Surface 能帮我们生成HLSL，不过Surface并不是种语言，可以用开关和指令集来形容Surface，并且我们依然要关注HLSL如何编写。

===

Surface shader 和 vertex fragment shader 一样需要书写 CGPROGRAM..ENDCG，并且写入#pragma surface 来表明使用Surface，不同的是：Surface 它不能有Pass，所有的Surface编码都写在SubShader中，然后再由shader编译器编译成多个Pass。

###### #pragma surface 的格式如下：

		#pragma surface surfaceFunction lightModel [可选择参数]

参数说明：

###### 1，surfaceFunction，是一个有输入输出参数的函数，其格式为

		void surf (Input IN, inout SurfaceOutput o)

Surface通过这个函数来修改渲染细节，Shader中必须写有这个名字和格式的函数。

###### 2，lightModel，选择光照模型。在内置的光照模型中，有以物理为基础的 Standard 和 StandardSpecular，和无物理基础的Lambert (diffuse漫反射) and BlinnPhong (specular镜面反射)。除此之外，还可以定义自己的光照模型。

其中内置的光照模型：

		Standard 光照模型使用 SurfaceOutputStandard 作为输出结构并且匹配了Unity3D内置的Standard Shader(金属流)。

		StandardSpecular 光照模型使用 SurfaceOutputStandardSpecular 作为输出结构并且匹配了Unity3D内置的Standard Shader(镜面反射)。

		Lambert 和 BlinnPhong 光照模型则不以物理为基础，使用它们时可以在低端设备上运行得更快。

SurfaceOutputStandard 和 SurfaceOutputStandardSpecular 在 UnityPBSLighting.cginc 中有定义。

Lambert 和 BlinnPhong 在 Lighting.cginc 中有定义，分别是 UnityLambertLight 和 UnityBlinnPhongLight。

###### 3，可选择指令。

###### 透明物体 和 alpha testing 功能指令

透明物体 和 alpha testing 类型的参数可以控制alpha和alphaTest也就是alpha混合和alpha裁切 的功能。

透明物体分为两种，一种是传统的alpha混合，另一种是更加物理化的“premultiplied blending”左自乘混合，它能够在半透明上保留适度的镜面反射。

启用半透明使得surface shader包含了alpha混合功能。而启用alpha cutout 将根据alpha值在生成像素时中裁切掉不符合的片段。

透明物体和alpha testing的功能指令如下：

		alpha 或者 alpha:auto，指令意思为，选择渐进透明也就是 alpha:fade 作为简单光照函数，并且选择左自乘透明方式也就是 alpha:premul 作为物理基础的光照函数。

		alpha:blend，开启alpha混合。

		alpha:fade，开启传统的渐进透明函数。

		alpha:premul，开启左自乘alpha透明度。

		alphatest:VariableName，开启alpha cutout裁切功能。将根据跟定的值裁切不符合的alpha片段。裁切后可以通过 addshadow 指令生成阴影。

		keepalpha，保持alpha值，让那些不透明的物体也有alpha通道为1.0的值。这个操作使得灯光函数在不透明物体上也起到alpha的作用。

		decal:add，开启叠加贴花功能，让贴花贴图使用叠加混合。

		decal:blend，开启半透明贴花功能，让贴花贴图使用alpha混合。

###### 自定义函数，自定义函数可以计算和修改顶点数据，或者计算和修改最终的片元颜色。

vertex:VertexFunctionName，自定义修改顶点函数。这个函数被调用来计算和修改模型的每个顶点的数据。

		函数的格式如下：

		void VertexFunctionName(inout appdata_full v, out Input o)

		函数中必须带有两个类型的参数，inout appdata_full 和 out Input。

其中 appdata_full 在Unity内置的UnityCG.cginc文件中，结构为如下：

{% highlight c# %}

struct appdata_full {
    float4 vertex : POSITION;
    float4 tangent : TANGENT;
    float3 normal : NORMAL;
    float4 texcoord : TEXCOORD0;
    float4 texcoord1 : TEXCOORD1;
    float4 texcoord2 : TEXCOORD2;
    float4 texcoord3 : TEXCOORD3;
    fixed4 color : COLOR;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

{% endhighlight %}

其中 Input 的结构为如下：

		Input结构里当使用某张纹理的uv时，可以使用“uv”加变量名来提取该纹理坐标，或者“uv2”加变量名使用第二纹理uv。（uv2一般用于lightmap）

		float3 viewDir - 视图方向（view direction）。

		float4 with COLOR semantic - 每个顶点插值后的颜色。

		float4 screenPos - 屏幕空间中的位置。

		float3 worldPos - 世界空间中的位置。

		float3 worldRefl - 世界空间中的反射向量。 如果surface 
		shader没有赋值o.Normal，将会包含世界反射向量。

		float3 worldNormal - 世界空间中的法线向量。如果surface 
		shader没有赋值o.Normal，将会包含世界法向量。

		float3 worldRefl; INTERNAL_DATA - 世界空间中的反射向量。如果surface 
		shader没有赋值o.Normal，将会包含这个参数。为了获得逐像素法线贴图的反射向量，请使用WorldReflectionVector 
		(IN, o.Normal)。参见例子： Reflect-Bumped shader。

		float3 worldNormal; INTERNAL_DATA -世界空间中的法线向量。如果surface 
		shader没有赋值o.Normal，将会包含世界法向量。为了获得逐像素法线贴图的法向量，请使用WorldNormalVector 
		(IN, o.Normal)。

finalcolor:ColorFunctionName，设置自定义函数修改最终颜色。

		函数格式如下：

		void ColorFunctionName(Input IN, SurfaceOutput o, inout fixed4 color)

		函数中必须带有Input IN, SurfaceOutput o, inout fixed4 color，这三个参数。

finalgbuffer:ColorFunctionName，自定义修改延迟路径gbuffer内容。

finalprepass:ColorFunctionName，自定义修改旧有的延迟渲染前的路径。

###### 阴影和网格细化指令

阴影和网格细化这两种指令，可以用来控制处理阴影渲染和网格细分。

		addshadow，生成一个阴影渲染管线。addshadow 经常会与顶点修改函数一起使用，让阴影有一定的动画效果。通常shader不需要任何特殊的方法处理阴影，他们可以用shader caster作为后备方案。

		fullforwardshadows，支持所有正向渲染路径下的灯光阴影。默认情况下shader在正向渲染中只支持1个方向光的阴影渲染。如果你需要在正向渲染中对点光源进行阴影渲染，就得使用这个指令。

		tessellate:TessFunctionName，使用DX11 GPU 网格细化。自定义函数中计算了三角形的边和里面的细分因素。

		网格细分化功能，需要使用Shader Model 4.6 target进行编译，所以只能在 DX11/12, OpenGL Core and PS4/XB1 上使用。

		TessFunctionName的格式为：

		float4 TessFunctionName(appdata v0, appdata v1, appdata v2)

###### 代码生成操作指令

		exclude_path:deferred, exclude_path:forward, exclude_path:prepass，不生成指定的渲染路径。

		三个指令分别代表延迟渲染, 正向渲染 和 旧式延迟渲染。

		noshadow，关闭所有阴影渲染支持。

		noambient，不使用环境光或者光探测。

		novertexlights，在正向渲染中不使用任何光探测或顶点光照。

		nolightmap，关闭所有烘培光照图。

		nodynlightmap，关闭实时动态全局自发光的支持。

		nodirlightmap，关闭方向光的支持。

		nofog，关闭内置迷雾效果。

		nometa，不生成meta渲染管线。

		noforwardadd，关闭正向渲染增加的渲染管线。

		nolppv，关闭 Light Probe Proxy Volume。

		noshadowmask，关闭 Shadowmask。


###### 其他指令

		softvegetation，Suface shader只在 Soft Vegetation 开启时渲染。

		interpolateview，对视向上的做插值操作，替换像素shader里的计算。

		halfasview，替换观察向量为一半方向向量传给光照函数。

		dualforward，在正向渲染中使用双重光照图。


### 自定义光照模型

光照模型是用来计算光照的，包括，光照影响，阴影，漫反射，镜面反射，环境光等。

Unity3D内置的光照模型有 Lambert (漫反射光照) 和 BlinnPhong (镜面反射光照) 两种，这两种光照模型被写在Unity3D内置文件 Lighting.cginc 中，可以前往查看，window上的地址为(unity install path)/Data/CGIncludes/Lighting.cginc，Mac上的地址为 /Applications/Unity/Unity.app/Contents/CGIncludes/Lighting.cginc。

除了这两种光照模型，我们还可以定义自己的光照模型。

所有自定义的光照模型函数都必须以 Lighting 作为开头定义在Shader文件的 Surface块中。函数定义可以如下：

		half4 Lighting<Name> (SurfaceOutput s, UnityGI gi); 这种方式使用正向渲染，且没有视向的依赖。

		half4 Lighting<Name> (SurfaceOutput s, half3 viewDir, UnityGI gi); 这种方式使用正向渲染，且依赖视向。

		half4 Lighting<Name>_Deferred (SurfaceOutput s, UnityGI gi, out half4 outDiffuseOcclusion, out half4 outSpecSmoothness, out half4 outNormal); 这种方式使用的是光照延迟渲染。

		half4 Lighting<Name>_PrePass (SurfaceOutput s, half4 light); 这种方式使用的是旧的光照延迟渲染。

自定义GI模型：

		half4 Lighting<Name>_GI (SurfaceOutput s, UnityGIInput data, inout UnityGI gi);

		如果你想使用内置的GI模型，有 DecodeLightmap 和 ShadeSHPerPixel 两种。他们都写在 UnityGlobalIllumination.cginc 的 UnityGI_Base 里。

### Tessellation 网格细分

在模型渲染时很容易有锯齿感，Tessellation 网格细分可以改善这种锯齿感，Surface支持Tessellation 网格细分的功能，但只在DirectX 11 / OpenGL Core GPU上有效。

Tessellation 功能指令为

		tessellate:FunctionName 这个方法计算三角形的边和里面的细分因素。

		它是在 vertex:FunctionName 之后才被调用的，所以每个顶点都会被替换映射。

		Surface 可以选择使用 phong tessellation 平滑模型表面，而不用替换顶点映射。

tessellate 有如下限制：

		1，只计算三角形，没有方形，没有等值线细分。

		2，当tessellation被使用时，Shader 自动被编译成 Shader Model 4.6 target，意味着这个Shader只在 DX11/12, OpenGL Core and PS4/XB1 下工作。

tessellate Function 的格式如下：

		float4 tessellateFunction(appdata v0, appdata v1, appdata v2);

开启 phong tessellation 功能，需要加入指令：

		tessphong:VariableName

		VariableName 为Phong变量强度。

参考资料：

		Unity3D：Writing Surface Shaders
