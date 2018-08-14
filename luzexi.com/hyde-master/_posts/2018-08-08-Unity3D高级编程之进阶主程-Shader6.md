---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(六) - Pass(三)Blend
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### Blend，设置颜色混合的方式。

Blend 主要作用是颜色混合，颜色混合是在所有渲染完成图像都成型后进行的，这时像素都已经被绘制好了，并且屏幕上也已经有成像的图形，但还没有完全完成，因为是物体各自画各自的已经画完了放入了缓存中，但alpha还没有发挥出效果，这时就需要混合来做补充，这之前的屏幕上所有的图像都是实体的，没有半透明和透明部分。总之Blend最重要的作用就是让alpha起到半透明的作用，当然也可以做到颜色加强，改变，混合，削弱等作用。

===

![blend](/assets/book/7/shader34.png)
 
        图1

![blend](/assets/book/7/shader34.png)

        图2

图1为没启用Blend的效果，没有发挥alpha的半透明和透明的作用。

图2为启用了Blend的效果，贴图上的alpha发挥了应该有的透明效果。

Blend在Shader上的格式，图1只是注释掉了Blend那一行，图2没有注释任何代码:

{% highlight c# %}

Shader "Test/Blend" {
    Properties {
        _MainTex ("Base (RGB) Trans (A)", 2D) = "white" {}
    }
    SubShader
    {
        Pass
        {
            Blend SrcAlpha OneMinusSrcAlpha 

            SetTexture [_MainTex] {
                    Combine Primary * Texture
            }
        }
    }
}

{% endhighlight %}

上述例子将Alpha混合进颜色中。

下面我们来看看Blend的使用方法以及它的参数配置。

###### 1，Blend Off，关闭混合，默认是关闭颜色混合的。

###### 2，Blend SrcFactor DstFactor。

当前物体产生的图像上的颜色与ScrFactor(Source Factor)参数相乘，屏幕上的已经存在的颜色与DstFactor(Destination Factor)参数相乘，两者的结果相加的结果放入帧缓存。

这里的ScrFactor(Source Factor)和DstFactor(Destination Factor)是固定的系数，ScrFactor中的Source 意味着本物体，代表本物体颜色相关，Destination意味着Blend前已经在屏幕上绘制的颜色相关。

这些系数说明如下：

    One，表示完整的颜色，不加修改，无论是本物体的颜色还是屏幕上的颜色。

    Zero，表示数据零，本物体和屏幕上的颜色都不要。

    SrcColor，表示本物体的颜色。

    SrcAlpha，表示本物体的Alpha。

    DstColor，表示屏幕上的颜色。

    DstAlpha，表示屏幕上的Alpha。

    OneMinusSrcColor,表示1-Source Color，即反转本物体上的Color。

    OneMinusSrcAlpha,表示1-Source Alpha，即反转本物体上的Alpha。

    OneMinusDstColor,表示1-Destination Color，即反转屏幕上的Color。

    OneMinusDstAlpha,表示1-Destination Alpha，即反转屏幕上的Alpha。

假设：颜色信息的四个分量（红Red，绿Green，蓝Blue，透明度Alpha）

        （1）“源颜色”  ：（Rs, Gs, Bs, As）

        （2）“目标颜色”：（Rd, Gd, Bd, Ad）

        （3）“源因子”  ：（Sr, Sg, Sb, Sa）

        （4）“目标因子”：（Dr, Dg, Db, Da）

那么混合产生的新颜色可以表示为：（Rs*Sr + Rd*Dr , Gs*Sg + Gd*Dg , Bs*Sb + Bd*Db , As*Sa + Ad*Da）

如果颜色的某一分量超过了1.0，则它会被自动截取为1.0，不需要考虑越界的问题。

混合说的通俗点就是把某一像素位置上原来的颜色和将要画上去的颜色，通过某种公式计算得出另一种颜色，从而实现特殊的效果。

源图片颜色被称作“源颜色”，而屏幕上已存在的图片颜色则被称作“目标颜色”。

###### 3，BlendOp Op，此命令可以改变两个颜色结果相加的操作。

颜色混合的最后一步是相加，通过这个命令可以改变最后一步的符号，比如相减，取最小值，取最大值等。

具体可用的操作符如下：

    Add，相加，默认的操作符号。

    Sub，相减，目标结果 – 源结果。

    RevSub，相减，源结果 – 目标结果。

    Min，取最小值，目标结果和源结果中取最小值。

    Max，取最大值，目标结果和源结果中取最大值。

还有一些只能在特定显卡下使用的操作符，比如LogicalClear，LogicalSet等等只能在DX11.1及以上的版本中使用。

###### 4，BlendOp OpColor,  OpAlpha，此命令与上面一样也是改变操作符号的，但此命令可以把颜色和Alpha的操作符号分开来改变。

OpColor参数代表目标结果和源结果的颜色操作符号，OpAlpha参数代表目标结果和源结果的Alpha操作符号。

可用的参数与BlendOp Op一样，只是拆分了Color和Alpha。

###### 5，当使用Multiple render target(MRT)多管线目标渲染时，还可以针对不同的管线目标进行操作，但仅限于固定级别显卡以上的设备，比如DX11/12, GLCore, Metal, PS4等。

下面是可操作的命令格式，N为目标管线索引(0-7)：

        Blend N SrcFactor DstFactor

        Blend N SrcFactor DstFactor, SrcFactorA DstFactorA

        BlendOp N Op

        BlendOp N OpColor, OpAlpha

###### 6，AlphaToMask On/Off，此命令为开关Alpha To Coverage(A2C)。

Alpha To Coverage(A2C)是一种经由流水线完成的“Alpha Test”。

在使用了多重采样(Multi-sample)的场合下，经由检测当前需要绘制的fragment的alpha值来决定该fragment在对应像素上的sample覆盖率。

启用AlphaToMask的条件是必须在multisample anti-aliasing(MSAA，在QualitySetting中设置)开启的情况下才有效。

Alpha To Coverage(A2C)是对Alpha Test和Alpha Blend的两种透明绘制方法的折中改进方法。

Alpha Test过于粗糙，“非0即1”的强硬手段导致边缘有强烈的锯齿感，这种阈值控制的剔除法在渲染上始终有很多不足。

Alpha Blend(文中的Blend命令)则可以根据物体本身的纹理上的颜色和透明度柔和的将Alpha混合显示在屏幕上并没有锯齿感，但这种机制不是对每个物件并行处理的，对当前DrawCall产生作用时必然会考虑的是当前“画板上已经画了些什么”即所谓的“已经绘制在屏幕上的图像，即目标结果”。这样最后的结果就与深度信息无关了，混合的结果跟绘制的顺序有关(前文中Queue标签的绘制顺序)。

所以说对于Alpha Blend，必须让绘制满足从后往前的顺序执行，而无法做场景中深度的排序。

Alpha To Coverage(A2C)中和了Alpha Test和Alpha Blend的方法，让半透明既不那么粗糙，也有深度数据来判断前后的关系。

### 在介绍常用的Blend混合方法之前，我想有必要介绍下颜色的操作符，即颜色的乘法，加法，减法，最小值，最大值的表现结果。

###### 1，颜色乘法，可以视为颜色（或者说图像）的叠加。

为什么乘法颜色节点能够让颜色叠加。

我们把两个RGBA值转换成为0-1之间的范围就可以明白了。

比如白色（1,1,1,1），灰色（0.5,0.5,0.5,1），两个颜色相乘得到（0.5,0.5,0.5,1） 还是灰色，也就是说在白色的底板上，画上灰色的颜色，颜色就叠加了，这就是颜色的叠加方法。

颜色相乘的公式是 Color a, Color b, a*b = new Color(a.r * b.r, a.g * b.g, a.b * b.b, a.a * b.a); 从此公式可以看出，因为所有数字都小于等于1，所以相乘只会让颜色更加深，也就是更加接近黑色，不断的叠加各种各样的颜色，最后都会无限接近黑色(0,0,0,1)或者无限接近黑色透明(0,0,0,0)。

###### 2，颜色加法，可以视为颜色（或者说图像）加白(或者说亮)。

为什么说加法可以视为颜色的加白操作。因为颜色相加后，不能超过最大值1（或者另一种表达方式的数字255）,超过部分都会被截断。

加法的公式是：Color a, Color b, a+b = new Color(a.r + b.r, a.g + b.g, a.b + b.b, a.a + b.a); 从此公式可以看出，所有的数字都小于等于1，所以r，g，b，a，随着不断加各种颜色，最终会到达最大值1。也就是不透明白色(1,1,1,1)。

所以颜色无论加什么颜色都会更加接近白色，也就得到我们的结论，颜色加法可以视为颜色（或者说图像）加白(或者说亮)。

###### 3，颜色减法，可以视为颜色(或者说图像)的色差。

如果两个一样的颜色比如红色，相减，就变成了(0,0,0)黑色，而两个相差比较大的颜色相减，结果就会无限接近白色(1,1,1)。

颜色的相减公式为：Color a, Color b, a-b = new Color(a.r - b.r, a.g - b.g, a.b - b.b, a.a - b.a); 相减的结果大于等于0小于等于1，一直做减法操作的话，就会无限接近黑色(0,0,0)或者透明黑色即无色(0,0,0,0)。

因此我们可以认为，颜色相减，可以视为颜色（或者说图像）的色差。

###### 4，颜色最大值，可以视为颜色（或者说图像）的最白(或者说亮)状态。

因为RGB的所有颜色都做了最大值操作，无论哪个颜色都会更加接近白色(1,1,1)，所以两个颜色或者图像做最大值操作就是取其最白(或者说亮)的状态。

###### 5，颜色最小值，可以视为颜色（或者说图像）的最暗状态。

与最大值相同因为RGB的所有颜色都做了最小值操作，无论哪个颜色都会更加接近黑色(0,0,0)，所以两个颜色或者图像做最小值操作就是取其最暗(黑)的状态。

{% include advertisement_content.html %}

### 有了这些颜色操作的常识我们就可以更加深入的了解Blend混合方法了。

### 常用的Blen混合方法：

###### 1， 透明度混合Blend SrcAlpha OneMinusSrcAlpha，即常用半透明物体的混合方式。
 
这时最常用的半透明混合了，首先要保证半透明绘制的顺序比实体的要后面，所以Queue标签是必要的Tags {"Queue" = "Transparent"}。Queue标签告诉着色器此物体是透半透明物体排序。

![blend](/assets/book/7/shader36.png)

Blend SrcAlpha OneMinusSrcAlpha 我们来解释下，以上图为例，图中油桶是带此Shader的混合目标。

当绘制油桶时，后面的实体BOX已经绘制好并且放入屏幕里了，所以ScrAlpha与油桶渲染完的图像相乘，部分区域Alpha为0即相乘后为无(颜色)，这时正好另一部分由OneMinusSrcAlpha(也就是1-ScrAlpha)为1即相乘后原色不变，两个颜色相加后就相当于油桶的透明部分叠加后面实体Box的画面，于是就形成了上面的这幅画面。

反过来也是一样，当ScrAlpha为1时，源图像为不透明状态，则两个颜色在相加前最终变成了，源图像颜色+无颜色=源图像颜色，于是就有了上图中油桶覆盖实体Box的图像部分。

###### 2，加白加亮叠加混合 Blend One One，即在原有的颜色上叠加屏幕颜色更加白或亮。
 
![blend](/assets/book/7/shader37.png)

第一参数One代表本物体的颜色。第二个参数代表屏幕上的颜色。两种颜色相加则，导致形成的图像更加亮白。这样我们就看到了一个图像加亮加白的图像了。

###### 3，保留原图色彩Blend One Zero，即只显示自身的图像色彩不加任何其他效果。

![blend](/assets/book/7/shader38.png)

本物体颜色，加上，零，就是本物体颜色。这是最简单的混合了。

###### 4，自我叠加（加深）混合Blend SrcColor Zero，即源图像与源图像自我叠加。

![blend](/assets/book/7/shader39.png)

与上面相比，加深了本物体的颜色。第一个参数本物体的颜色与本物体的颜色相乘，加深了颜色，第二个参数为零，使得屏幕颜色不被使用。所以形成的图像为颜色加色的图像。

###### 5，目标源叠加（正片叠底）混合Blend DstColor  SrcColor，即把目标图像和源图像叠加显示。

![blend](/assets/book/7/shader40.png)

第一个参数，本物体颜色与屏幕颜色相乘，颜色叠加。第二个参数，屏幕颜色与本问题颜色相乘，颜色叠加。两种颜色相加，加亮加白。这个混合效果就如同两张图像颜色叠加后的效果。

###### 6，软叠加混合Blend DstColor  Zero，即把目标源图像叠加到前面来。

![blend](/assets/book/7/shader41.png)
 
与前面的叠加混合效果相似，这个只做一次叠加，并不做颜色相加操作，使得图像看起来在叠加部分并没有那么亮白的突出。因为第二个参数为零，表示后面的屏幕颜色与零相乘即为零。

###### 7，差值混合BlendOp Sub，Blend One One，即注重黑白通道的差值。

![blend](/assets/book/7/shader42.png)

在这个混合中使用了混合操作改变，从默认的加法改成了减法，使得两个颜色从加法变为了减法，不再是变白变亮的操作，而是反其道成为了色差的操作。

### ColorMask，设置颜色遮罩过滤。

ColorMask可以进行颜色的遮罩功能，这相当于在渲染时把某种颜色剔除一样。

ColorMask可供选择参数 ColorMask RGB | A | 0 | R,G,B,A的任意组合。

当ColorMask 0 时将关闭所有的颜色通道，相当于没有颜色渲染就没有成型的图像了，实际上你是看不见它的，只是在Editor下可以点选出来看到轮廓。如下图所示：

![blend](/assets/book/7/shader43.png)

当ColorMask R|G|B|A时我们来看看效果，第一张是RGBA参数时的图，即为全通道颜色渲染的模型图。如下图所示：

![blend](/assets/book/7/shader44.png)

![blend](/assets/book/7/shader45.png)

![blend](/assets/book/7/shader46.png)

![blend](/assets/book/7/shader47.png)

![blend](/assets/book/7/shader48.png)

第2张是只开放R(红色)通道其他通道被拒绝，第3张是只开放G(绿色)通道，第4张是只开放B(蓝色)通道，第5张是只开放A (alpha通道)。

从上图的例子中可以充分的明白了，ColorMask对颜色通道的操作。

ColorMasks虽然也可以在Multiple render target(MRT)下做一些额外的操作，比如ColorMask RGB 3 后面数字3对RGB通道中进行数字3的遮罩功能，但MRT需要显卡的支持，比如PC显卡中大多数在2006年后支持MRT从GeForce 8XXX，Radeon X2400，Intel G45开始。

在手机上能支持MRT并不多，而且对MRT的功能也限制了很多，基本可以说在手机平台上无法使用。
