---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(四) - Pass
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass tag"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

===

{% include unity3d_book_copyright.html %}

### Pass

Pass块的格式，如下：

{% highlight c# %}

Pass
{
    [Name and Tags]
    [Render State]
}

{% endhighlight %}

Pass可以定义自己的名字，和Tag标签，以及渲染状态Render State。

Pass通过Tag标签来变更渲染引擎里的设置。Pass里的这个Tag和SubShader上的Tag是同一种标签，SubShader起到了共享标签的作用，而Pass里的Tag则是独立的个性化的。

### Tag标签详解

Tag的格式很简单，就同Key-Value一样，每个标签都有一个Key，对应一个Value，Key之间用空格隔开。

        Tags { “TagKey1” = “Value1” “TagKey2” = “Value2” }

那么究竟有哪些Tag可以供我们使用？

我们先介绍SubShader和Pass共用的Tag。

###### Queue 标签Tag

Queue是渲染顺序的标签。可以用这个标签决定模型在绘制时所用的顺序。

Shader会在渲染前决定物体属于哪个渲染顺序，从而确保所有透明或半透明物体的绘制顺都排在不透明物体的后面。

        例子，Tags { "Queue" = "Transparent" }

如下为渲染顺序的定义选择。

        Background：这种类型的顺序会排在所有物体的前面。

        Geometry(默认)：大多数物体都使用这种方式。所有不透明物体使用这个类型。

        AlphaTest：使用Alpha Test的物体使用这个类型的渲染顺序。它被排在不透明物体的渲染后面。

        Transparent：排在Geometry和AlphaTest后渲染。所有alpha混合的物体都应该选择这个类型（因为Shader不会入depth buffer）。

        Overlay：这个类型是最后渲染的，比前面所有类型都要置后。

在实际的运用过程中，Render Queue是可以有自己的排序状态的，不同的Shader同一种排序类型可以有不同的前后关系。

比如在两个物体都使用Geometry类型的渲染顺序，第一个物体使用 Geometry+1 比其他的 Geometry 类型物体更加滞后渲染，第二个物体使用 Geometry+2 写法，比第一个物体还要靠后渲染，目的有可能是需要在其他物体渲染完毕后再对周围环境做出反应。

这种方式在透明和半透明的物体中特别常用，因为透明和半透明物体没有深度缓存测试，所以他们之间的渲染顺序是混乱的，当两个半透明物体放在前后位置进行渲染时，摄像机看到的是一个比较混乱的排序方式。因此在半透明物体的运用中我们时常人工指定某些物体的排序大小，比如如下：

        Tags { "Queue" = "Transparent+1" }
        
        Tags { "Queue" = "Transparent+2" }

后者比前者在渲染绘制时排在后面，先绘制了+1的半透明物体后，再绘制+2的半透明物体，结果是+2物体会覆盖+1物体而显示得靠前。

例子中“+1”表达了渲染顺序往后靠一位，+2比+1更往前后，数字越大渲染时越排在后面。

“Queue”标签在Shader里会提前告诉着色器这个物体的渲染顺序，这样在渲染中就不会因为没有深度缓存数据而导致前后渲染的混乱。

渲染顺序到2500(“Geometry+500”)是非透明物体和可优化绘制顺序（自动排序）物体最佳的性能位置。

超过2500是半透明物体和手动排序物体的区间范围。天空盒(Skybox)被排在非透明物体和半透明物体中间。

###### RenderType 标签Tag

RenderType 标签是自由识别类标签，他没有自己的标签类型选择，可以为任意字符串。

RenderType的主要作用是被识别与替代。在Shader替换效果中比较常用，比如场景中有些类型的物体需要在某个时刻替换某种Shader，并且不是真正的替换Shader，而是只针对某个Camera摄像机，其他的Camera摄像机保持原样。这时就需要用到RenderType标签，再用Camera.RenderWithShader或Camera.SetReplacementShader进行替换，因为函数中需要找出某种类型的Shader进而才可以进行替换，所以RenderType就是那个标识记号。

Camera.RenderWithShader或Camera.SetReplacementShader渲染的规则是，没有RenderType标记的全部替换并渲染，有标记的但跟给定的标记一样的才渲染，否则不渲染。

Unity内部Shader已经全部标记了RenderType，可以在它的Shader源码内参考。

###### DisableBatching标签

DisableBatching可以将该物体上的Draw Call Batching打开或关闭。

为什么要关闭呢，因为在一些Shader中需要对顶点进行更改达到Mesh变化的目的，而Draw Call Batching会对所有的可合并的模型进行合并后放入场景空间，所以这些对Mesh操作的Shader就不起作用了，需要关闭Draw Call Batching才能重新得到该Shader要的效果。

DisableBatching有三种类型可以选择。

第一种是True，标志着Draw Call Batching 功能在这个物体上将被关闭。

        Tags { "DisableBatching" = "True" }

第二种是False，标志着Draw Call Batching 功能在这个物体上不关闭。False是默认值，也就是说如果不去设置它时，Shader模式是不关闭Draw Call Batching的。

        Tags { "DisableBatching" = "False" }

第三种是LODFading，这个比较特殊，只在LOD fading设置被开启时关闭这个物体上的Draw Call Batching功能。一般常用在树上，会动且有LOD功能。

        Tags { "DisableBatching" = "LODFading" }

###### 这里简单介绍下Draw Call Batching功能，后面章节中会有详细的介绍。

每个物体都有一个材质球，每个材质球都有一个Shader，每个Shader都起码有 一个Pass渲染管线，每个物体在渲染时，引擎会抓取模型的顶点，网格，颜色，贴图，UV，法线等数据进行一次渲染操作。

那么如果每个物体都单独走一次渲染管线，几百几千甚至几万个物体一起渲染时，就会因为渲染引擎工作量过大CPU和GPU负荷过高而导致画面渲染次数减少，也就是帧数下降，因为每一帧一个画面。帧数下降导致画面有卡顿感，不流畅。所以要使用Batching批处理进行优化，于是就有了Draw Call Batching。

它是怎么优化的呢？把所拥有有相同材质球的模型放在一起走一次渲染管线，不同的材质球对应的模型不能进行Draw Call Batching。

这样大大加强了处理速度，原来一个一次的处理方式，变成了一批一次，达到了优化的目的，绘制画面的次数上去了，每秒的帧数（FPS）也上就去了。

Draw Call Batching分两种，一种是动态的，一种是静态的。

要进行动态的Batching批处理要求有点高，比如它要求模型必须少于900个顶点甚至更少，又比如它要求Batching批处理对象的Scale大小是一模一样的否则不予处理，再比如必须使用同一种材质球而且是同一个实例，不同材质球或者同一材质球而不同实例也不行，还有如果有烘培的模型，它的烘培图必须是同一张，甚至Shader中不能使用多个Pass等等苛刻的条件。所以想要做动态批处理还是比较困难的。

静态Batching就比较好弄了，只有在编辑器的右上角勾选静态标志就可以了，不过一旦勾上静态标志，物体就再也不能动不能旋转和缩放了。

当然Draw Call Batching本身有很多的限制，但对优化渲染管线还是有一定好处的，如果想要加大加强加快优化的力度，还有其他很多方法，每个部分的优化方案都写进了各个章节中，特别是针对Draw Call Batching的替代方案，在模型章节中有很深入的讲解。

###### ForceNoShadowCasting 标签Tag

ForceNoShadowCasting是针对是否强制关闭阴影投射功能的。

当ForceNoShadowCasting为True时，物体将不会去绘制阴影部分，着色器也不会把时间浪费在阴影的绘制上了。默认为False的，默认状态下，物体会去绘制阴影部分的渲染图。

        Tags { "ForceNoShadowCasting" = "True" }

###### IgnoreProjector 标签Tag

IgnoreProjector针对的是投射阴影的开关功能。

当IgnoreProjector为True时，着色器将关闭Projector投射阴影的绘制功能，这个物体将不会绘制投射阴影部分的渲染。默认状态下是False，默认状态下，物体是能响应Projector投射的阴影绘制的。

        Tags { "ForceNoShadowCasting" = "True" }

###### CanUseSpriteAtlas 标签Tag

CanUseSpriteAtlas是针对图集里的元素的，它是UGUI中常使用。当CanUseSpriteAtlas为False时，将不会对打进图集里的贴图进行绘制。默认为True。

        Tags { "CanUseSpriteAtlas" = "False" }

###### PreviewType 标签Tag

PreviewType是展示材质球预览形状的默认选项的。编辑器中Material材质球展示预览效果时，默认是球形，通过这个标签，可以设置为Plane面板形状，也可以设置为Skybox天空盒形状。

        Tags { "PreviewType" = "Plane" }

{% include advertisement_content.html %}

### 以上是SubShader和Pass共享的Tag标签，下面我们介绍一些只有在Pass里使用的Tag标签。

###### LightMode 标签Tag

通过设置LightMode可以在Pass里设置一个光照模型，主要是针对光的相互作用。

值类型选择范围有：

        Always：一直都渲染，不应用光照作用。

        ForwardBase： 使用正向渲染。环境光，主方向光，vertex/SH lights 和 烘培贴图功能被开启。

        ForwardAdd：使用正向渲染。在ForwardBase的基础上，添加像素光的应用，每个Pass（渲染管线）一个灯管。

        Deferred：使用延迟渲染。着色使用g-buffer。

        ShadowCaster：渲染物体depth到阴影图(Shadowmap)或者一张深度贴图(depth texture)。

        MontionVectors：计算每个物体的运动矢量。

        PrepassBase：使用旧的延迟光照。渲染法线和镜面反射。

        PrepassFinal：使用旧的延迟光照。用合并贴图，光照和发散光，来渲染最终颜色。

        Vertex：当物体不使用烘培贴图时，使用旧的顶点灯光渲染模式。所有顶点光都被开启。

        VertexLMRGBM：当物体使用烘培贴图时，使用旧的顶点灯光渲染模式。PC和Console平台上烘培贴图是RGBM编码格式。

        VertexLM：当物体使用烘培贴图时，使用久的顶点灯光渲染模式。在手机平台上烘培贴图是double-LDR编码。

###### PassFlags 标签Tag

        OnlyDirectional：当使用ForwardBase的Pass标签时，这个标签会只让主方向光和环境光数据通过进入Shader。这意味着不重要的灯光数据，不会被通过进入顶点光或球面谐波Shader变量。

###### RequireOptions 标签Tag

RequrieOptions可以做到，一个Pass申明当有某种外部条件发生的时候才被渲染。

        SoftVegetation：当软植物(Soft Vegetation)在Quality Settings中被设置时，才渲染这个Pass。

