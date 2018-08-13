---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(二) - Property
description: "unity3d 高级编程 主程 shader Property shaderlab vertex fragment"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### 编写Shader的几种方式

着色器部分的编写是客户端的精髓，如果说架构表现了逻辑整合能力，算法展现了数学能力，那么着色器编写展现了表现能力，在游戏项目中三者缺一不可，特别是作为主程或者说资深的U3D人员来说是极其重要的。

===

Unity3D使用了自己创造的语言命名为ShaderLab来编写图形Shader。在ShaderLab里模拟了类似CgFX和Direct3D Effects(.FX)的语言，从而可以编写绝大部分的图形渲染材质。

ShaderLab编写的Shader属性会在Unity的材质球编辑器中显示，ShaderLab可以实现基于多种目标硬件的能力，并且每个ShaderLab完全能够编写图形硬件渲染状态，以及使用vertex和fragment编程。Shader的编写是基于高级Cg/HLSL编程语言基础上的。

下面几节中，我们会详细的讲解，如何用ShaderLab去写Unity Shader，包括Surface Shader, fixed-function Shader,vertex和fragment的Shader编程。

Shader编程需要一些基本的OpenGL或Direct3D渲染状态的知识，还有一些HLSL,Cg,GLSL或者Metal Shader编程语言的知识。如果并没有掌握也没有关系，我们将以通俗易懂的语言解释大部分的图形渲染知识。

编写Unity Shader的几种形式：

###### 1，fixed-function，这种形式的Shader使用了Unity自己编制的比较简单的图形语言来编写Shader。

编写完毕后在导入和编译时，Unity内部会将这个fixed-function形式的代码，转化成vertex和fragment编程形式。

因为fixed-function形式编写起来比较简单易懂，所以一些简单的Shader可以用fixed-function来编写，可以节省很多学习底层图形学的时间也增加了不少效率。缺点是，功能并不是很全，复杂一点的或者说个性化点的Shader就需要用其他的方式实现。

###### 2，Surface shaders 是Unity建立的一种新的Shader编程方式，目的是让Shader编程更加简单。

在Surface Shader里没有任何的自定义语言，贴图，或者变量。它会生成所有你需要手写的代码，让你专注于编写渲染部分的规则，其中一个特色是Surface Shader不需要写Pass，它会自动帮你编译成为多个你需要的多个Pass。

Surface Shader编译器最终会将所有代码转换成vertex-fragment类型的Shader。

###### 3，vertex and fragment shaders是ShaderLab Shader包含的几种不同的编写方式中最最底层的编码方式，它使用的HLSL语言编程，我们称之为底层硬件着色器着色编程。

使用vertex-fragment方式只是ShaderLab的理念中的其中一部分，fixed-function和suface都是其理念的一部分，并不一定说要使用vertex-fragment底层的才好，其他方式也同样能帮助你节省很多时间。

比如你想对灯光作用进行着色编程，你完全可以使用Surface Shader去实现你的想法，如果只想对贴图上色和混合，你使用fixed-function更为简单省事，如果你想对顶点颜色，或对顶点坐标进行更改，或者片元的着色需要更改等一些更复杂的图形操作，那你就需要使用vertex-fragment Shader进行编程。

针对不同的复杂度，不同的项目，不同领域的着色渲染，我们完全可以选择不同的方式来编写Shader。

无论你用哪种方式编写Shader，其实都是在ShaderLab语言结构下的，前面说过ShaderLab是Unity指定的组织Shader的官方编程语言，所以在Unity编写Shader首先要学会如何使用ShaderLab。

了解了基本的ShaderLab语言结构和语法，再去学习surface和vertex-fragment就会相对比较容易。而fixed-function是完全基于ShaderLab语法结构编写的，所以fixed-function形式的写法完全可以参照ShaderLab的使用方法来编写。


### ShaderLab语法与参数详解

我们需要着重介绍下ShaderLab的格式，Unity3D中所有的Shader编程方式都是基于ShaderLab的，因此ShaderLab语法是基础。

###### Shader的开头

“Shader” 是Shader文件的根命令。每个文件必须定义一个且只有一个Shade命令，它可以定义一个Shader的名字，是一个Shader的开头。

在“Shader”里面你才可以表达有哪些属性，有哪些渲染模式被开启和关闭，有多少条渲染管线等等那些你需要的定义以及变化。

格式：

        Shader “name”
        {
        	[Properties属性]
        	Subshaders
        	[Fallback]
        	[CustomEditor]
        }

定义了一个Shader后，Shader的Name就会出现在Unity编辑器的材质球视窗的Shader列表里，可供你选择。Shader中定义的属性，供Shader计算使用，并且这些属性会显示在材质球编辑器的视窗中。

{% include advertisement_content.html %}

###### Properties属性

Shader中可以自己定义一些属性用来在后面的渲染计算中使用。比如数字，颜色，方向，坐标，贴图。

Properties格式：

        Properties
        {
        	PropertyName1 (“display name名字”，type类型) = value默认值
        	PropertyName2 (“display name名字”，type类型) = value默认值
        	PropertyName3 (“display name名字”，type类型) = value默认值
        	…
        }

Properties属性格式需要中括号框起来，大括号内都是属性的详细说明。

其中PropertyName和 display name是不一样的，Property Name是属性的真正名字，可以用来通过材质球才程序中更改和赋值，或者在后面的渲染计算中使用。而display name 只是在编辑器中显示该属性名的名字，没有其他用途。

下面举例来说明各种类型的书写方式。

数字

        Name属性名 (“display name显示名”, Range(最小值,最大值)) = 默认值

        上述属性块定义了一个数字，起始有一个默认值，该值可以在最大值和最小值范围内，不得超出最大和最小值，定义后可以在材质球的编辑器中滑动的来选择值的大小。

        Name属性名 (“display name显示名”, Float) = 默认值

        上述属性块定义了一个浮点数数值，具有一个默认值，同样定义后，可以在材质球的编辑器中输入浮点数来重新赋值。

        Name属性名 (“display name显示名”, Int) = 默认值

        上述属性块定义了一个整数数值，具有一个默认值，同样定义后，可以在材质球的编辑器中输入整数来重新赋值。

颜色

        Name属性名 (“display name显示名”, Color) = (R,G,B,A)

        上述属性块定义了一个颜色，具有一个RGBA(红，绿，黑，alpha)的默认值，定义后可以在材质球的编辑器中选择一个颜色来重新赋值。

方向和坐标

        Name属性名 (“display name显示名”, Vector) = (0,0,0,0)
        
        上述属性块定义了一个方向或者坐标，具有一个默认值，定义后可以在材质球的编辑器中填写4个值来重新赋值。

贴图

        Name属性名 (“display name显示名”, 2D) = “white” {}
        
        上述属性块定义了一个2D贴图，具有一个纯白色的默认贴图，定义后可以在材质球的编辑器中选择新的贴图来重新赋值。

        Name属性名 (“display name显示名”, Cube) = {}
        
        上述属性块定义了一个Cube镜面盒子，空的默认值，定义后可以在材质球的编辑器中选择新的Cube镜面盒子来重新赋值。

        Name属性名 (“display name显示名”, 3D) = “white” {}
        
        上述属性块定义了一个3D贴图，具有一个纯白色的默认贴图，定义后可以在材质球的编辑器中选择新的3D贴图来重新赋值。这个3D Texture不是普通的3D模型的贴图，而是图像后处理形成的3D颜色校正效果贴图。3D Texture由脚本或者着色器产生，而非手绘制造的。可参考Unity API中的Texture3D类。

综合上面的表述，我们可以制作一个全属性Properties例子。

{% highlight java %}

Properties
{
    _Scale ("Scale", Range (0.02,0.15)) = 0.07
    _PahLength("Path Length", Float) = 1.5
    _OrderNumber("Order Number", Int) = 53
    _RefrColor ("Refraction color", Color) = (0.34, 0.85, 0.92, 1) 
    _Direction ("Direction", Vector) = (2, 4.5, 1.2, 3.3) 
    _RefractionTex ("Environment Refraction", 2D) = "white" {}
    _RefractionMap ("Refraction Map", Cube) = “black” {}
}

{% endhighlight %}

这个全属性例子包含了上面所有属性，在编辑器上展示的属性如下。

![shader全属性](/assets/book/7/shader9.png)

注意，这里为什么属性前都要加一个”_”，这是个习惯性写法，没有任何强制性，只是大部分程序员在书写Shader属性变量的习惯，逐渐形成了一个传统。

如果硬要套个说法，那就只能是”_”代表内部变量。就像我们书写代码变量时会有一些小习惯和规则一样，没有其他影响。

###### Property属性默认值

每个Property属性都有一个默认值，默认值是必须填写的，否则Shader编译器不会通过编译。这里我们说明一下所有属性的默认值。

        1，Number类型。Range，默认值必须在最大和最小值之间，比如Range(0,1)时值为默认值必须为0到1之间。Float，默认值可以为任意浮点数。Int，默认值可以为任意整数，且仅仅只能是整数，不能是浮点数。

        2，Color颜色类型。Color，默认值为4个浮点数，按照RGBA的顺序，第一个代表红色，第二个代表绿色，第三个代表蓝色，第四个代表透明度alpha。

        3，Vector方向坐标类型。Vector，默认值为4个浮点数，按照xyzw的顺序，第一个代表X坐标，第二个代表Y坐标，第三个代表Z坐标，第四个代表角度，深度等其他意义。Vector四个坐标完全可以代表任意值，比如前两个代表2D的X和Y，后面两个代表宽度和长度，再比如，前两个代表uv，后两个代表宽度和长度，或者比如前三个表示x，y，z，最后一个代表透明度alpha。

        4，2D Texture贴图类型。2D Texture，默认值有一个默认的颜色，”white”代表(RGBA:1,1,1,1)白色，”black”代表(RGBA:0,0,0,0)黑色，”gray”代表(RGBA:0.5,0.5,0.5,0.5)灰色，”bump”代表(RGBA:0.5,0.5,1,0.5)灰黄色，”red”代表(RGBA:1,0,0,0)红色。

        5，Cube，3D Texture，2DArray非2D贴图类型。默认值为空，当材质球编辑器中没有设置他们的实例时，他们使用灰色(RGBA:0.5,0.5,0.5,0.5)做默认值。

每个属性值的值在Material材质上都会被序列化后写入Material材质球的Meta文件。

Shader里实际上可以有很多参数，比如矩阵，数组等，但这些都需要在Shader实时运行中设置和获取的变量，不能成为Property的一部分，因为他们不是Property属性的一部分，也不会被记录存储在Mate文件里。

在Mate文件里的属性，我们不但可以通过Unity编辑器视窗中来赋值和变更，也可以通过脚本程序来赋值和改变，例如Material.SetFloat可以用来设置Shader中Float类型的属性，Material.SetColor用来设置Color类型的属性，Material. SetTexture可以用来设置 Texture类型的属性。

没有在Property中的属性类型或变量也同样可以使用Material.SetXXX(“PropertyName”,value)来设置，比如我们可以通过Material. SetMatrix设置矩阵类型在Shader中的变量，虽然矩阵没有在属性块中，我们照样可以设置，只要它在Shader程序变量中声明过就可以。

下面代码就是一个很好的例子：

{% highlight c# %}

public class ExampleClass : MonoBehaviour
{
    public float rotateSpeed = 30f;
    public void Update ()
    {
        Quaternion rot = Quaternion.Euler (0, 0, Time.time * rotateSpeed);
        Matrix4x4 m = Matrix4x4.TRS (Vector3.zero, rot, Vector3.one);
        GetComponent<Renderer>().material.SetMatrix ("_TextureRotation", m);
    }
}

///// Shader部分
Shader "RotatingTexture"
{
    Properties
    {
        _MainTex ("Base (RGB)", 2D) = "white" {}
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 pos : SV_POSITION;
            };

            float4x4 _TextureRotation;

            v2f vert (float4 pos : POSITION, float2 uv : TEXCOORD0)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(pos);
                o.uv = mul(_TextureRotation, float4(uv,0,1)).xy;
                return o;
            }

            sampler2D _MainTex;
            fixed4 frag (v2f i) : SV_Target
            {
                return tex2D(_MainTex, i.uv);
            }
            ENDCG
        }
    }
}

{% endhighlight %}

上述代码中，_TextureRotation 并不是Property属性块中的变量，SetMatrix也同样可以通过材质球实例在程序运行时进行设置。

这就告诉我们，程序运行时设置Shader中的变量，并不是一定要在Property属性块中声明的。

其他比如获取Material中的变量可以通过Material.GetXXX(“PropertyName”)来获得，比如Material. GetTexture，Material.GetColor，Material.GetFloat等等。关于Material的很多功能可以通过查看Material的API来了解，这里不再详细介绍。

###### 属性的Attributes特性和Drawers绘制工具

在书写属性时可以加入一些特性来让属性的用途更加明显，属性的特性有如下：

        [HideInInspector] 在视窗中隐藏。

        [NoScaleOffse] 不显示贴图的tiling/offset属性。

        [Normal] 表明这张贴图是张法线贴图。

        [HDR] 表明这贴图是张高动态范围贴图。

        [Gamma] 表明这个Float或者Vector属性是个RGB类型的颜色，可能会需要转换成以符合的格式使用。

        [PerRendererData] 表明这张贴图每帧都会被MaterialPropertyBlock重新设置。

书写方式就是在属性前加入这些特性，如下：

{% highlight c# %}

Properties
{
    [HideInInspector] _Scale ("Scale", Range (0.02,0.15)) = 0.07
    _PahLength("Path Length", Float) = 1.5
    _OrderNumber("Order Number", Int) = 53
    [Gamma]  _RefrColor ("Refraction color", Color) = (0.34, 0.85, 0.92, 1) 
    _Direction ("Direction", Vector) = (2, 4.5, 1.2, 3.3) 
    [NoScaleOffse]  _RefractionTex ("Environment Refraction", 2D) = "white" {}
    _RefractionMap ("Refraction Map", Cube) = “black” {}
}

{% endhighlight %}

Drawers绘制特性列举：

        Drawer和Attribute的区别就是Drawer主要关注点在工具的绘制上，而不是对属性的限制。

        [Toggle] 表明此数字为开关类型，只有0，或1的选择。
        
        [Enum] 表明此变量只能从几个选项中选择。
        
        [KeywordEnum] 表明只能在选项中选择关键字，这个特性通常用在开关Shader的部分功能。
        
        [PowerSlider] 显示一个非线性的属性选择范围。
        
        [IntRange] 显示一个数字范围，这个数字范围内必须是整数。
        
        [Space] 在视窗中的属性前创建一个垂直的空隙。
        
        [Header] 在视窗的属性前展示一个文字说明。

例如下面的属性绘制特性

{% highlight c# %}

Properties
{
        [PowerSlider] _Scale ("Scale", Range (0.02,0.15)) = 0.07
        [Space(100)] _PahLength("Path Length", Float) = 1.5
        [Toggle] _OrderNumber("Order Number", Int) = 53
        [Enum(small,1,big,2,huge,10)] _Size ("size", Float) = 1
        _Direction ("Direction", Vector) = (2, 4.5, 1.2, 3.3) 
        [Header(ok head)] _RefractionTex ("Environment Refraction", 2D) = "white" {}
        _RefractionMap ("Refraction Map", Cube) = "black" {}
}

{% endhighlight %}

绘制结果截图

![Drawer绘制](/assets/book/7/shader10.png) 

