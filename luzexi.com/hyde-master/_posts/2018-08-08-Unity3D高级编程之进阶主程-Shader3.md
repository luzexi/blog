---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(二) - 渲染管道
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment 光栅化 片元 图元"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### SubShader模块

SubShader相当于子Shader的意思，SubShader在Shader中可以有很多个，每个SubShader都可以针对一种设备，但不是所有SubShader都会被采用。当Unity去展示一个模型时，会从第一个SubShader开始寻找是否匹配当前的设备，如果不匹配则与下一个SubShader继续匹配，直到找到能够匹配该设备为止。如果所有的SubShader都没有匹配上，则会使用 Fallback 标签的Shader。

===

格式如下：

{% highlight c#%}

SubShader
{
	[Tags]
	[CommonState]
	[Pass1] Pass { [Name and Tags] [RenderSetup] }
    [Pass2] Pass { [Name and Tags] [RenderSetup] }
    [Pass3] Pass { [Name and Tags] [RenderSetup] }
    […]
}
Fallback “name”  or Fallback Off

{% endhighlight %}

SubShader中可以定义了Tag标签，以及与Pass共同拥有的状态(Common State)。

Pass代表了一个渲染管道，每次通过Pass进行渲染都是通过一次完整渲染管道。

当一个SubShader被设备选中进行渲染时，定义的每个Pass都会渲染该物体一次，如果有很多灯光相互作用的话可能不止一次。通过Pass渲染一次会承担相当大的代价，所以我们在编写Shader Pass时需要小心，尽量减少Pass的使用，尽量将Pass数量最小化。

建议只有在我们无法使用单个Pass达到效果时，才启用多个Pass进行Shader编程。

所有在SubShader块中的渲染状态都可以在Pass中进行特殊化，也就是说定义在SubShader中的渲染状态是所有Pass公有的渲染状态，同时Pass可以定义自己的个性化的渲染状态。

比如以下例子：

{% highlight c# %}

SubShader
{
    Lighting Off
    Pass
    {
        Cull Front
    }
    Pass
    {
        Cull Back
    }
}
Fallback “Diffuse”

{% endhighlight %}

上述例子中 SubShader 设置了一个共同的渲染状态(Common State)就是把灯光影响关掉，灯光不再影响着色，第一个Pass中使用了独立的渲染状态是，裁切模型前半部分，第二个Pass中也使用了独立的渲染状态，裁切模型模型后半部分，然后他们共同拥有Common State是关闭灯光效果，所以第一个Pass就有了两个状态，一个是关闭灯光效果，第二个是裁切前半部分模型，而第二个Pass的也有两个状态，一个是关闭灯光效果，另一个是裁切后半部分模型。

SubShader最后还有一个Fallback标识，代表了当所有SubShader都对设备无效时，则选择备用的Shader作为替代。而当你认为无法替代，或者无需替代的时候可以关闭Fallback的功能 Facllbck off 。

在上面的例子中，Fallback “Diffuse”表示了，当所有SubShader都无法起作用的时候漫反射Diffuse替代当前Shader。

SubShader本身并不复杂，但里有几个重要的模块，以及这些模块中的标签和用途可能会稍微繁琐一些，下面我们要详细讲一讲Pass块的格式和属性内容。

### 渲染管道

提到渲染管道，不得不提到渲染管线流程，一整个渲染管线流程分八个阶段。

###### 阶段1， 指定几何对象

这个阶段引擎在选择某些几何对象作为渲染目标。

###### 阶段2，顶点处理

主要工作就是“变换三维顶点坐标”和“光照计算”。

顶点变换或坐标系变换包括：

        Object space，模型坐标；

        World space，世界坐标；
        
        Eye space，视点坐标（视锥裁剪）；
        
        Clip and Project space，裁剪空间坐标（也叫屏幕坐标，进行投影、剪裁、映射的过程）。

Eye space坐标转换到Clip and Project space坐标的过程其实就是一个投影、剪裁、映射的过程。

除了坐标转换和光照计算，第2阶段还做了纹理坐标变换(纹理矩阵)，材质状态纹理坐标生成。

这个阶段所接收到的数据则是每个顶点的属性特征，输出则是变换后的顶点数据。

这个阶段也是Shader里，vertex fragment着色器编程中 vertex 顶点转换函数的处理阶段，vetex顶点函数就在这个阶段编写顶点的变化。

{% include advertisement_content.html %}

###### 阶段3，图元装配

在顶点处理之后,顶点的全部属性都已经被确定 在这个阶段顶点将会根据应用程序送往的图元规则.

###### 阶段4，图元处理(裁剪 消隐)

将图元与用户定义的裁剪平面和模型投影矩阵所建立的视景比较，裁剪且丢弃位于视景和裁剪平面外部的图元，不在予以处理。若是采用透视投影，那么将会对每个顶点的x,y,z坐标分别乘以透视矩阵。

接着由视口变换将顶点坐标变换至窗口坐标，最后执行消隐操作，把那些被遮挡的部分进行消除。

###### 阶段5，栅格化

三维模型经过了顶点变换已经投影到了二维平面上，而且被装配成图元了，但是屏幕是一个一个细细小小的像素格子组成的，对于一个图片，电脑是怎么知道应该把它搞到哪几个像素上去显示的呢？这就是光栅化。说白了就是，把想看见的图片，搞成像素能显示的。光栅化工作告诉了显卡，哪个格子是你这个图元应该画上的，哪个不是。

传递过来的图元数据，在此将会被分解成更小的单元并对应帧缓冲区的各个像素，这些单元被称之为片元。一个片元可能包含窗口大小，深度，颜色，纹理坐标等属性。

片元的属性是图元上顶点数据等经过插值而确定的，比如颜色，深度等。

###### 阶段6，片元处理

这个阶段首先对片元上纹理，通过纹理坐标取得纹理内存中相对应的颜色。

再进行雾化，通过片元距离当前视点位置修改颜色。

最后颜色汇总，将纹理,主定义的颜色,雾化的颜色,次颜色光照阶段计算的颜色汇总一起。

这个阶段就是vertex fragment着色器编程中 fragment 片元转换函数的处理阶段，fragment 片元函数通过这个阶段的编程对片元进行修改。

###### 阶段7，像素操作（Pixel Operation）

所有的片元的测试都在这里进行，包括：

        像素所有权测试->裁剪测试->alpha测试->模板测试->深度测试->混合->抖动显示->逻辑操作

###### 阶段8，帧缓冲区（Frame Buffer）

执行帧缓冲的写入等操作，写入帧缓冲区，等待最后产生了显示出来的像素。

渲染管线的整个过程如下图所示：

![render pipe](/assets/book/7/shader11.png)

上图中 vertex框就是可编程部分的 vertex 函数，它可以修改顶点部分的数据来表现个性化的效果，fragment框也是一样，是 fragment 函数的调用点，修改通过这个fragment函数可以修改片元数据达到需要的效果。

Unity3D在渲染管线各阶段中所用到的功能，和我们前面所描述的阶段有很相似的地方，如下图所示：

![render pipe](/assets/book/7/shader12.png)

上图所要表达的功能和阶段，正是我们要在Shader章节中讲解和说明的部分。

 
 参考资料

        1，渲染管线流程 作者：不详

        2，【图形学】渲染管道 作者：阵雷
