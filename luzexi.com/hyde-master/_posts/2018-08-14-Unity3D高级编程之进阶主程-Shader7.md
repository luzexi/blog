---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第七章，Shader(七) - Fixed Function
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### Fixed-function 是Unity3D官方自己编写的一套API，它与Surface shaders，及vertex and fragment program是区分开来的。Fixed-function，Suface program 和 vertex and fragment programs 三种编写Shader的方法不能同时使用，任何一种方式被编写后，其他两种都将无效。

===

Fixed-function 相对于Suface 和 Vertex and fragment 来说简单很多，它是由一些简单的开关的函数组成的。我们可以用 Fixed-function 来做一些简单的灯光，贴图的变化。

下面我们来了解一下这些开关和函数的语法。

###### Pass的第一个指令如果是Fixed-function的命令，则认为是Pass开启Fixed-function program。

顶点和灯光的颜色计算在模型渲染中被最先执行，它们在顶点转换上操作，并且计算的基础颜色在贴图使用前就被应用到了顶点上。

###### Material块

Material标签，可以用来定义材质球的各种光属性的颜色。格式如下：

		Material
		{
		    Diffuse [_Color]
		    Ambient [_Color]
		    Shininess [_Shininess]
		    Specular [_SpecColor]
		    Emission [_Emission]
		}

参数如下：

Diffuse color，漫反射颜色，这是物体的基础颜色。

Ambient color，环境光颜色，被光照到后反应出来的颜色。

Specular color，镜面反射的高光颜色。

Shininess number，高光圈的大小，也就是尖锐度，数值范围在0-1之间。为0时高光圈很大覆盖整个物体，为1时高光圈很小，几乎缩小成一个点。

Emission color，自身发出的颜色，当没有光照到的时候。

整个光照颜色的公式为：

	color = Ambient * 设置中的Ambient强度 + (灯光颜色 * Diffuse + 灯光颜色 * Specular) + Emission

其中灯光颜色，当多个灯光照射时会叠加多次。

###### Color颜色

Color颜色设置不透明物体的颜色。格式如下：

		Color color

###### Lighting 开关

Lighting 是一个灯光的开关，Material块, ColorMaterial 和 SeparateSpecular都受到它的影响，如果Lighting 关掉的话，它们都将失效。

格式如下：

		Lighting On | Off

###### SeparateSpecular 镜面反射开关

SeparateSpecular 选项开启后会在最后一个pass后加上一个镜面反射光的pass，这使得镜面放射光不受到贴图的影响。

格式如下：

		SeparateSpecular On | Off

###### ColorMaterial Material块中的颜色

ColorMaterial可以在每个顶点上替换Material中的颜色，其中AmbientAndDiffuse 替换Material块中的 Ambient 和 Diffuse 的颜色，Emission 替换Material块中 Emission 的颜色。

		ColorMaterial AmbientAndDiffuse | Emission


###### SetTexture 设置贴图

Fixed function 的贴图功能可以替换旧有的合并效果。我们可以写很多个 SetTexture 命令在 Pass里，所有的贴图都会依次被应用，就像绘制程序里的层级一样。

另外，SetTexture 命令必须放在Pass的最后部分。

SetTexture 格式为：

		SetTexture [TextureName] {Texture Block}

其中 TextureName 为贴图的变量名，{Texture Block} 里面的有两种操作，一种是 combine 为主要操作命令，另一个是 constantColor 命令用来设置一个常量颜色。

combine 命令在SetTexture里最为重要，它的格式可以为：

		combine src1 * src2，src1 与 src2 相乘，颜色叠加。

		combine src1 + src2，src1 与 src2 相加，颜色加白加量。

		combine src1 - src2，src1 减去 src2，获得色差。

		combine src1 lerp (src2) src3，使用src2的alpha值计算src1到src3的插值。

		注意src2的alpha为1时结果为src1，src2的alpha为0时结果为src3。

		combine src1 * src2 + src3，src1 与 src2的alpha相乘，然后加上src3。

###### 上面所有的 src 属性可以是texture，或者previous，或者constant，或者primary。这几个变量分别是如下意思：

		texture 是贴图变量TextureName贴图上的颜色。

		previous 是前面计算结果的颜色。

		primary 是顶点经过灯光计算后的颜色。也就是前面Material块中diffuse, ambient and specular colors与灯光的计算结果。

		constant 是常量颜色，可以使用 ConstantColor 在 combine 前设置。

###### src变量后面可以跟的参数有

		Double 和 Quad 参数。src后面可以跟随double和quad，分别表示2x，4x的亮度，相当于乘2，乘4的效果。

		lerp 插值，所有src变量都可以使用 lerp 做插值操作。

		alpha 透明通道，src后面跟着alpha参数的话，就表示只使用src的alpha通道。

###### 合并时对Color和Alpha分开处理

有时我们需要对颜色和透明通道分开处理，可以在SetTextue时在颜色处理后面加逗号表示分开处理color和alpha。

具体处理格式为：

		SetTexture [_MainTex] { combine previous * texture, previous + texture }

其中 previous * texture 为颜色处理部分，previous + texture 为alpha处理部分。

###### Fixed function 所有功能的完整示例

{% highlight c# %}

Shader "VertexLit"
{
    Properties
    {
        _Color ("Main Color", Color) = (1,1,1,0)
        _SpecColor ("Spec Color", Color) = (1,1,1,1)
        _Emission ("Emmisive Color", Color) = (0,0,0,0)
        _Shininess ("Shininess", Range (0.01, 1)) = 0.7
        _MainTex ("Base (RGB)", 2D) = "white" {}
        _ConstantColor ("Main Color", Color) = (1,1,1,0)
    }
    SubShader
    {
        Pass
        {
            Material
            {
                Diffuse [_Color]
                Ambient [_Color]
                Shininess [_Shininess]
                Specular [_SpecColor]
                Emission [_Emission]
            }
            Lighting On
            SeparateSpecular On
            SetTexture [_MainTex]
            {
            	constantColor [_ConstantColor]
                Combine texture * primary DOUBLE, texture * primary
            }
            SetTexture [_MainTex]
            {
            	ConstantColor (1,1,1,1)
                combine constant lerp(texture) previous
            }
            SetTexture [_MainTex]
            {
                combine previous * texture
            }
        }
    }
}
{% endhighlight %}

{% include advertisement_content.html %}

###### Alpha Test 透明裁切

Alpha Test 是像素即将要输出到屏幕前的最后一步处理，它通过比较像素的alpha值来决定是否展示。

格式如下：

		AlphaTest Off | On

		AlphaTest 操作符号 AlphaValue

其中AlphaValue可以是0-1之间的任何浮点数，也可以是给定的变量。

操作符号可以为：

		Greater	只有alpha值 大于 给出的AlphaValue的像素才会被渲染。

		GEqual	只有alpha值 大于等于 给出的AlphaValue的像素才会被渲染。

		Less	只有alpha值 小于 给出的AlphaValue的像素才会被渲染。

		LEqual	只有alpha值 小于等于 给出的AlphaValue的像素才会被渲染。

		Equal	只有alpha值 等于 给出的AlphaValue的像素才会被渲染。

		NotEqual	只有alpha值 不等于 给出的AlphaValue的像素才会被渲染。

		Always	渲染所有像素，相当于AlphaTest Off，关闭了AlphaTest的功能。

		Never	不渲染任何像素。

###### AlphaTest 根据Alpha值进行像素的裁切，而不是走alpha混合的路，所以没有深度数据问题，在渲染时不会造成前后渲染排序错乱问题。

在使用AlphaTest 制作树和枝叶时，由于AlphaTest裁切过于生硬导致画面上有锯齿感，我们可以采用AlphaTest和Alpha Blend混合使用的方式让画面更加平滑。如下代码：

{% highlight c# %}

Shader "Vegetation" {
    Properties {
        _Color ("Main Color", Color) = (.5, .5, .5, .5)
        _MainTex ("Base (RGB) Alpha (A)", 2D) = "white" {}
        _Cutoff ("Base Alpha cutoff", Range (0,.9)) = .5
    }
    SubShader {
        // Set up basic lighting
        Material {
            Diffuse [_Color]
            Ambient [_Color]
        }
        Lighting On

        // Render both front and back facing polygons.
        Cull Off

        // first pass:
        // render any pixels that are more than [_Cutoff] opaque
        Pass {
            AlphaTest Greater [_Cutoff]
            SetTexture [_MainTex] {
                combine texture * primary, texture
            }
        }

        // Second pass:
        // render in the semitransparent details.
        Pass {
            // Dont write to the depth buffer
            ZWrite off
            // Don't write pixels we have already written.
            ZTest Less
            // Only render pixels less or equal to the value
            AlphaTest LEqual [_Cutoff]

            // Set up alpha blending
            Blend SrcAlpha OneMinusSrcAlpha

            SetTexture [_MainTex] {
                combine texture * primary, texture
            }
        }
    }
}

{% endhighlight %}

上述代码中两个Pass来渲染树和枝叶，第一Pass对Alpha进行硬裁切，第二个Pass将前面其余没有被裁切的部分进行Alpha混合，这样就能让画面更加平滑，虽然alpha混合会导致渲染排序问题，但主要枝干能够正常排序，剩下的边缘部分只起到平滑作用，牺牲一点排序错位让画面更加柔和。

###### Fog 迷雾颜色

Fog 命令可以设置雾颜色，根据与摄像机的不同距离，可以叠加不同的颜色。

格式如下：

		Fog {Fog Commands}

Fog Commands 有：

		Mode Off | Global | Linear | Exp | Exp2

		Mode 可以设置计算公式，Off为关闭，Global为全局，Linear为线性插值计算颜色，Exp为指数公式计算颜色，Exp2为以2 为底的指数公式计算。

		Color ColorValue

		Color 用来设置迷雾颜色。

		Density FloatValue

		Density 用来设置迷雾密度。

		Range FloatValue, FloatValue

		Range 用来设置迷雾的显示范围。近圈距离，外圈距离，他们之间的用线性的方式展示迷雾。

###### Fog命令不会与 Vertex Fragment program 和 Surface program 冲突，它是独立于他们之外的功能。

###### 在没有设置Fog的默认情况下，Fog会根据 Lighting Window(Window > Lighting > Settings) 里的设置进行渲染。
