---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第七章，Shader(二) - 
description: "unity3d 高级编程 主程 shader Property shaderlab vertex fragment"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---


	SubShader模块
	
SubShader在ShaderLab中可以有很多个，每个SubShader都可以针对一种设备。当Unity去展示一个模型时，会从第一个SubShader开始寻找是否匹配该设备，如果不匹配则选择下一个SubShader，直到找到能够匹配该设备为止。
格式：
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
SubShader中定义了Tag标签，Pass共同有用的状态(Common State)。Pass代表了一个渲染管道，每次通过Pass进行渲染都像是通过一次完整渲染管道。
当一个SubShader被设备选中进行渲染时，定义的每个Pass都会渲染该物体一次，如果有很多灯光相互作用的话可能不止一次。通过Pass渲染一次会承担相当大的代价，所以我们在编写Shader Pass时需要小心，尽量减少Pass的使用，将Pass数量最小化。建议只有在我们无法使用单个Pass达到效果时，才启用多个Pass进行Shader编程。
所有在SubShader块中的渲染状态(Render Statement)都可以在Pass中进行特殊化，而定义在SubShader中的渲染状态则是所有Pass公有的渲染状态，同时Pass可以定义自己的个性化的渲染状态。
比如以下例子：
SubShader{
Lighting Off
Pass{
        Cull Front
    }
Pass{
Cull Back
    }
}
Fallback “Diffuse”
例子中，SubShader设置了一个共同的渲染状态(Common State)就是把灯光影响关掉，灯光不再影响着色，第一个Pass中使用了独立的渲染状态是，裁切模型前半部分，第二个Pass中也使用了独立的渲染状态，裁切模型模型后半部分，然后他们共同拥有Common State是关闭灯光效果，所以第一个Pass就有了两个状态，一个是关闭灯光效果，第二个是裁切前半部分模型，而第二个Pass的也有两个状态，一个是关闭灯光效果，另一个是裁切后半部分模型。
SubShader最后还有一个Fallback标识，代表了当所有SubShader都对设备无效时，则选择什么类型的Shader作为替代。而当你认为无法替代，或者无需替代的时候可以关闭Fallback的功能。在上面的例子中，Fallback “Diffuse”表示了，当所有SubShader都无法起作用的时候漫反射Diffuse替代当前Shader。

SubShader本身并不复杂，但里有几个重要的模块，下面我们要详细讲一讲Pass块的格式和属性内容。

Pass模块

前面提到过每个Pass块都会对这个物体模型进行一次渲染，Pass块中也有自己的格式，如下：
Pass
{
	[Name and Tags]
	[Render State]
}
Pass可以定义自己的名字，和Tag标签。Pass可以通过Tag标签来变更渲染引擎里的设置。Pass里的这个Tag和SubShader上的Tag是同一种标签，SubShader起到了共享标签的作用，而Pass里的Tag则是独立的个性化的。

Tag标签详解
Tag的格式很简单，就如同KeyValue一样，每个标签都有一个Key，对应一个Value，Key之间用空格隔开。
Tags { “TagKey1” = “Value1” “TagKey2” = “Value2” }

那么有哪些Tag可以供我们使用呢？

先介绍SubShader和Pass共用的Tag。

Queue 标签
Queue是渲染顺序的标签。可以用这个标签决定模型在绘制时所用的顺序。
Shader会在渲染前决定物体属于哪个渲染顺序，从而确保所有透明或半透明物体的绘制顺都排在不透明物体的后面。

例子，Tags { "Queue" = "Transparent" }

如下为渲染顺序的定义选择。
Background：这种类型的顺序会排在所有物体的前面。
Geometry(默认)：大多数物体都使用这种方式。不透明物体使用这个类型。
AlphaTest：使用Alpha Test的物体使用这个类型的渲染顺序。它被排在不透明物体的渲染后面。
Transparent：排在Geometry和AlphaTest后渲染。所有alpha混合的物体都应该选择这个类型（因为Shader不会入depth buffer）。
Overlay：这个类型是最后渲染的，比前面所有类型都要置后。

在实际的运用过程中，Render Queue是可以在这几个类型有中间状态的。比如在Geometry之后渲染，但在Transparent之前。这种方式在透明和半透明的物体中特别适用，因为透明和半透明物体没有深度缓存深度缓存，所以他们之间的渲染顺序是混乱的，当两个半透明物体放在前后位置，进行渲染时，摄像机看到的是一个比较混乱的排序方式，因为他们没有深度缓存数据。所以在半透明物体的运用中我们时常人工指定某些物体的排序大小，比如，
Tags { "Queue" = "Transparent+1" }
Tags { "Queue" = "Transparent+2" }
后者比前者在渲染绘制时排在前面，因为+1表达了渲染顺序往前靠一位，+2比+1更往前靠，数字越大渲染时越排在前面。在Shader标签里，就会提前告诉着色器这个物体的渲染顺序，这样在渲染中就不会因为没有深度缓存数据而导致前后渲染的混乱。

渲染顺序到2500(“Geometry+500”)是非透明物体和可优化绘制顺序（自动排序）物体最佳的性能位置。而超过2500是半透明物体和手动排序物体的区间范围。天空盒(Skybox)被排在非透明物体和半透明物体中间。

RenderType标签
RenderType标签是自由识别类标签，他没有自己的标签类型选择，可以为任意字符串。
RenderType的主要作用是被识别与替代。在Shader替换效果中比较常用，比如场景中有些类型的物体需要在某个时刻替换某种Shader，并且不是真正的替换Shader，而是只针对某个Camera摄像机，而其他的Camera摄像机保持原样。这时就需要用标识RenderType标签，再用Camera.RenderWithShader或Camera.SetReplacementShader进行替换，因为函数中需要找出某种类型的Shader进而才可以进行替换，所以RenderType就是那个标识记号。
Camera.RenderWithShader或Camera.SetReplacementShader渲染的规则是，没有RenderType标记的全部替换并渲染，有标记的但跟给定的标记一样的才渲染，否则不渲染。
Unity内部Shader已经全部标记了RenderType，可以在它的Shader源码内参考。

DisableBatching标签
DisableBatching可以将该物体上的Draw Call Batching打开或关闭。为什么要关闭呢，因为在一些Shader中需要对顶点进行更改达到Mesh变化的目的，而Draw Call Batching会对所有的可合并的模型进行合并后放入场景空间，所以这些对Mesh操作的Shader就不起作用了，需要关闭Draw Call Batching才能重新得到该Shader要的效果。
DisableBatching有三种类型可以选择。
第一种是True，标志着Draw Call Batching 功能在这个物体上将被关闭。
Tags { "DisableBatching" = "True" }
第二种是False，标志着Draw Call Batching 功能在这个物体上不关闭。False是默认值，也就是说如果不去设置它时，Shader模式是不关闭Draw Call Batching的。
Tags { "DisableBatching" = "False" }
第三种是LODFading，这个比较特殊，只在LOD fading设置被开启时关闭这个物体上的Draw Call Batching功能。一般常用在树上，会动且有LOD功能。
Tags { "DisableBatching" = "LODFading" }

这里简单介绍下Draw Call Batching功能，后面章节中会有详细的介绍。
每个物体都有一个材质球，每个材质球都有一个Shader，每个Shader都起码有 一个Pass渲染管线，所以每个物体在渲染时，引擎会抓取模型的顶点，网格，颜色，贴图，UV，法线等数据进行一次渲染操作。那么如果每个物体都单独走一次渲染管线，几百几千甚至几万个物体一起渲染时，就会因为渲染引擎工作量过大CPU和GPU负荷过高而导致画面渲染次数减少，也就是帧数下降，因为每一帧一个画面。帧数下降导致画面有卡顿感，不流畅。所以要使用Batching批处理进行优化，于是就有了Draw Call Batching。它是怎么优化的呢？把所拥有有相同材质球的模型放在一起走一次渲染管线，不同的材质球对应的模型不能进行Draw Call Batching。这样大大加强了处理速度，原来一个一次的处理方式，变成了一批一次，达到了优化的目的，绘制画面的次数上去了，每秒的帧数（FPS）也上就去了。
Draw Call Batching分两种，一种是动态的，一种是静态的。
要进行动态的Batching批处理要求有点高，比如它要求模型必须少于900个顶点甚至更少，又比如它要求Batching批处理对象的Scale大小是一模一样的否则不予处理，再比如必须使用同一种材质球而且是同一个实例，不同材质球或者同一材质球而不同实例也不行，还有如果有烘培的模型，它的烘培图必须是同一张，甚至Shader中不能使用多个Pass等等苛刻的条件。所以想要做动态批处理还是比较困难的。
静态Batching就比较好弄了，只有在编辑器的右上角勾选静态标志就可以了，不过一旦勾上静态标志，物体就再也不能动不能旋转和缩放了。
当然Draw Call Batching本身有很多的限制，但对优化渲染管线还是有一定好处的，如果想要加大加强加快优化的力度，还有其他很多方法，各个章节中也有介绍优化方案，特别是针对Draw Call Batching的替代方案，在模型章节中有很深入的讲解。

ForceNoShadowCasting标签
ForceNoShadowCasting是针对是否强制关闭阴影投射的。当ForceNoShadowCasting为True时，物体将不会去绘制阴影部分，着色器也不会把时间浪费在阴影的绘制上了。默认为False的，默认状态下，物体会去绘制阴影部分的渲染图。
Tags { "ForceNoShadowCasting" = "True" }

IgnoreProjector标签
IgnoreProjector针对的是投射阴影的开关功能。当IgnoreProjector为True时，着色器将关闭Projector投射阴影的绘制功能，这个物体将不会绘制投射阴影部分的渲染。默认状态下是False，默认状态下，物体是能响应Projector投射的阴影绘制的。
Tags { "ForceNoShadowCasting" = "True" }

CanUseSpriteAtlas标签
CanUseSpriteAtlas是针对图集里的元素的，它是UGUI中常使用。当CanUseSpriteAtlas为False时，将不会对打进图集里的贴图进行绘制。默认为True。
Tags { "CanUseSpriteAtlas" = "False" }


PreviewType标签
PreviewType是展示材质球预览形状的默认选项的。编辑器中Material材质球展示预览效果时，默认是球形，通过这个标签，可以设置为Plane面板形状，也可以设置为Skybox天空盒形状。
Tags { "PreviewType" = "Plane" }

以上是SubShader和Pass共享的Tag标签，下面我们介绍一些只有在Pass里使用的Tag标签。

LightMode 标签
可以在Pass里设置一个光照模型，主要是针对光的相互作用。
值类型选择：
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

PassFlags 标签
OnlyDirectional：当使用ForwardBase的Pass标签时，这个标签会只让主方向光和环境光数据通过进入Shader。这意味着不重要的灯光数据，不会被通过进入顶点光或球面谐波Shader变量。
RequireOptions 标签
RequrieOptions可以做到，一个Pass申明当有某种外部条件发生的时候才被渲染。
SoftVegetation：当软植物(Soft Vegetation)在Quality Settings中被设置时，才渲染这个Pass。

Render State Command渲染状态指令详解

在Pass和SubShader里都可以设置各种渲染状态来达到渲染的效果。比如alpha的混合或者前后的裁切，还有深度的测试等等。

 

我们来介绍下Pass中的渲染状态Command指令
1.	Cull，前后裁切指令。
Cull是种优化的指令，可以让远离屏幕的部分不渲染。所有的物体都有前后两面，比如一个Box箱子你永远都看 不到屏幕另一侧的部分，所以可以对Box箱子后半部分进行裁切不渲染。
参数 Cull Back，裁切后半部分，保留前半部分。
 
参数Cull Front，裁切前半部分，保留后半部分。
 
参数Cull Off，关闭裁切功能。
 
Cull默认是裁切后半部分的，这样可以达到优化的效果，如果你没有对它进行设置的话。其实裁切后半部分和完全不裁切，在有深度缓存数据时是显示的效果是完全一样的，只是一个做了优化另一个没做而已。

2.	ZTest，设置深度缓存测试模式。
ZTest是在渲染绘制中对前后深度进行测试的模式，默认情况下以LEqual小于等于其他物体时测试为不通过，于是就不渲染那部分。其实就是测试前面的物体是否挡住了该物体部分，如果挡住了就不渲染。这个默认尝试就是LEqual参数。
其他参数还包括
ZTest Less | Greater | LEqual | GEqual | Equal | NotEqual | Always
参数 小于|大于|小于|小于等于|大于等于|等于|不等于|关闭测试
意义完全可以由LEqual进行类推，
Less就是该物体上的部分深度在其他物体后面时不渲染。
 
Greater就是该物体上的部分深度在其他物体部分前面时不渲染，也就是被覆盖的显示，没被覆盖的反而不显示。
 
Equal就是该物体上的部分深度与其他一致时才渲染，否则不渲染。
 
NotEqual就是该物体上的部分深度与其他物体不一致时才渲染，否则不渲染。
 

3.	ZWrite，设置深度缓存写入开关。
Zwrite可以控制物体渲染的像素是否写入depth buffer深度缓存数据中。如果没有depth buffer深度缓存数据，物体将无法辨识前后关系也就无法实现阻挡物体不绘制的功能。这在大多数不透明物体中都是需要depth buffer深度缓存数据来识别前后遮挡的，但在半透明和透明物体中就不需要depth buffer，因为半透明和透明物体没有depth buffer深度缓存数据，所以没有所谓的写入一说，也就是通常在半透明和透明物体的Shader中，ZWrite是关闭的。
 
 
如上两个图，图1为ZWrite On时的效果，黄色Box遮挡住了后面的蓝色Box因为有depth buffer深度缓存的写入，前后关系有依据。图2为ZWrite Off的效果，后面蓝色的Box被提到了黄色Box的前面显示，这是因为黄色Box的depth buffer没有写入，在渲染时，面片前后关系没有了依据，导致了错乱。

4.	Offset，设置深度缓存的偏移。
Offset允许你设置物体在深度缓冲中的偏移量，这相当于让物体在屏幕显示中提前或者推后显示位置，但实际世界中的位置不变。
Offset有两个参数，Factor偏移缩放大小和Units偏移量。Factor相当于缓存数据偏移放大的倍数，Units相当于直接偏移的量。
如果从公式上理解，可以理解为Buffer = (x or y)*Factor + Units。
举例 Offset 3 , 5，可以理解为偏移放大3倍，并且再偏移5个单位。
注意，偏移负方向才是离屏幕越近的方向。所以Offset 0 , -1时，两个物体重合部分肯定是拥有Offset 0 , -1的提前显示。如果两个物体前后差的比较远，那么想把后面的物体提到前面来显示，那么就要加大偏移量了，比如 Offset 0 , - 1000 或者 Offset -100, -1等。

如下图举例说明Offset的作用：
对白色的做处理Offset 0 , 0 -------两者重叠，时而显示普通的有贴图的立方体，时而显示白色的立方体。
 


对白色的做处理Offset 0 , -1------两者重叠部分完全被白色覆盖,因为白色立方体做了偏移量的处理，深度缓存中的坐标离屏幕更近。
 

对白色的做处理Offset 0 , 0 ---- 现在把两个立方体的距离拉开，正常情况下按透视的距离进行显示。
 

对白色的做处理Offset 0 , -1000 ----现在加重白色立方体的偏移量，深度缓存数据向屏幕偏移，这时很多有贴图的立方体部分被覆盖，因为偏移量加大，提前量就加大了，但绿色立方体其他部分的深度数据还是比偏移后的大，所以那个部分还是无法覆盖。
 

对白色的做处理Offset -100 , -1000------再次加大偏移量，这次加入了缩放比例的偏移，偏移的更多，使得白色立方体全部覆盖绿色立方体。
 

注意以上图中两个立方体的世界坐标一直都没有更改。同理，如果要让物体向后推也是同样方法，负数变正就可以。

5.	Blend，设置颜色混合的方式。
 
Blend主要作用是颜色混合，颜色混合是在所有渲染完成图像都成型后进行的，这时像素都已经被绘制好了，并且屏幕上也已经有成像的图形，当还没有完全完成，因为是物体各自画各自的已经画完了放入了缓存中，但alpha还没有发挥出效果，这时就需要混合来做补充，这之前的屏幕上所有的图像都是实体的，没有半透明和透明部分。总之Blend最重要的作用就是让alpha起到半透明的作用，当然也可以做到颜色加强，改变，混合，削弱等作用。
 图1
 图2
图1为不做Blend的效果，没有发挥alpha的半透明和透明的作用。图2为做了Blend的效果，贴图上的alpha发挥了应该有的透明效果。
使用的Shader很简单，图1只注释了Blend那一行，图2没有注释任何代码:
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
上述例子Blend用源图中的alpha进行操作将透明通道起作用。下面我们来看看Blend的使用方法以及他的参数配置。
1.	Blend Off，关闭混合，默认是关闭颜色混合的。

2.	Blend SrcFactor DstFactor，当前的模型产生的图像上的颜色与ScrFactor(Source Factor)相乘，屏幕上的已经存在的颜色与DstFactor(Destination Factor)相乘，两者的结果相加，混合的结果放入屏幕。这里的ScrFactor(Source Factor)和DstFactor(Destination Factor)是固定的系数，ScrFactor中的Source 意味着本物体，代表本物体颜色进行的计算，Destination意味着Blend前已经在屏幕上绘制的颜色。这些系数可以为如下之：
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

混合说的通俗点就是把某一像素位置上原来的颜色和将要画上去的颜色，通过某种方式混在一起，从而实现特殊的效果。新图片颜色被称作“源颜色”，而屏幕上已存在的图片颜色则被称作“目标颜色”。

3.	BlendOp Op，此命令可以改变两个颜色结果相加的操作。颜色混合的最后一步是相加，通过这个命令可以改变最后一步的符号，比如相减，取最小值，取最大值等。

下面是可以使用的操作符号：

Add，相加，默认的操作符号。
Sub，相减，目标结果 – 源结果。
RevSub，相减，源结果 – 目标结果。
Min，取最小值，目标结果和源结果中取最小值。
Max，取最大值，目标结果和源结果中取最大值。
还有一些只能在特定显卡下使用的操作符，比如LogicalClear，LogicalSet等等只能在DX11.1及以上的版本中使用。

4.	BlendOp OpColor,  OpAlpha，此命令与上面一样也是改变操作符号的，但此命令可以把颜色和Alpha的操作符号分开来改变。OpColor参数代表目标结果和源结果的颜色操作符号，OpAlpha参数代表目标结果和源结果的Alpha操作符号。

5.	当使用Multiple render target(MRT)多管线目标渲染时，还可以针对不同的管线目标进行操作，但仅限于固定级别显卡以上的设备，比如DX11/12, GLCore, Metal, PS4等。

下面是可操作的命令格式，N为目标管线索引(0-7)：

Blend N SrcFactor DstFactor
Blend N SrcFactor DstFactor, SrcFactorA DstFactorA
BlendOp N Op
BlendOp N OpColor, OpAlpha

6.	AlphaToMask On/Off，此命令为开关Alpha To Coverage(A2C)。Alpha To Coverage(A2C)是一种经由流水线完成的“Alpha Test”。在使用了多重采样(Multi-sample)的场合下，经由检测当前需要绘制的fragment的alpha值来决定该fragment在对应像素上的sample覆盖率。启用AlphaToMask的条件是必须在multisample anti-aliasing(MSAA，在QualitySetting中设置)开启的情况下，才有效。

Alpha To Coverage(A2C)是对Alpha Test和Alpha Blend的两种透明绘制方法的折中改进方法。Alpha Test过于粗糙，“非0即1”的强硬手段导致边缘有强烈的锯齿感，这种阈值控制的剔除法在渲染上始终有很多不足。Alpha Blend(文中的Blend命令)则可以根据物体本身的纹理上的颜色和透明度柔和的将Alpha混合显示在屏幕上并没有锯齿感，但这种机制不是对每个物件并行处理的，对当前DrawCall产生作用时必然会考虑的是当前“画板上已经画了些什么”即所谓的“已经绘制在屏幕上的图像，即目标结果”。这样最后的结果就与深度信息无关了，混合的结果跟绘制的顺序有关(前文中Queue标签的绘制顺序)。所以说对于Alpha Blend，必须让绘制满足从后往前的顺序执行，而无法做场景中深度的排序。

在介绍常用的Blend混合方法之前，我想有必要介绍下颜色的操作符，即乘法，加法，减法，最小值，最大值的意义。
1.	颜色乘法，可以视为颜色（或者说图像）的叠加。
为什么乘法颜色节点能够让颜色叠加。我们把两个 RGBA值转换成为0~1之间的范围就可以明白了。比如白色（1,1,1,1），灰色（0.5,0.5,0.5,1），两个颜色相乘得到（0.5,0.5,0.5,1） 还是灰色，也就是说在白色的底板上，画上灰色的颜色，颜色就叠加了，这就是颜色的叠加方法。颜色相乘的公式是 Color a, Color b, a*b = new new Color(a.r * b.r, a.g * b.g, a.b * b.b, a.a * b.a); 从此公式可以看出，因为所有数字都小于等于1，所以相乘只会让颜色更加深，也就是更加接近黑色，不断的叠加各种各样的颜色，最后都会无限接近黑色(0,0,0,1)或者无限接近黑色透明(0,0,0,0)。

2.	颜色加法，可以视为颜色（或者说图像）加白(或者说亮)。
为什么说加法可以视为颜色的加白操作。因为颜色相加后，不能超过最大值1（或者另一种表达方式的数字255）,超过部分都会被截断。加法的公式是：Color a, Color b, a+b = new Color(a.r + b.r, a.g + b.g, a.b + b.b, a.a + b.a); 从此公式可以看出，因为所有的数字都小于等于1，所以r，g，b，a，随着不断加各种颜色，最终会到达最大值1。也就是不透明白色(1,1,1,1)。所以颜色无论加什么颜色都会更加接近白色，也就得到我们的结论，颜色加法可以视为颜色（或者说图像）加白(或者说亮)。

3.	颜色减法，可以视为颜色(或者说图像)的色差。
如果两个一样的颜色比如红色，相减，就变成了(0,0,0)黑色，而两个相差比较大的颜色相减，结果就会接近白色(1,1,1)。颜色的相减公式为：Color a, Color b, a-b = new Color(a.r - b.r, a.g - b.g, a.b - b.b, a.a - b.a); 相减的结果大于等于0小于等于1，一直做减法操作的话，就会无限接近黑色(0,0,0)或者透明黑色即无色(0,0,0,0)。因此可以说，颜色相减，可以视为颜色（或者说图像）的色差。

4.	颜色最大值，可以视为颜色（或者说图像）的最白(或者说亮)状态。
因为RGB的所有颜色都做了最大值操作，无论哪个颜色都会更加接近白色(1,1,1)，所以两个颜色或者图像做最大值操作就是取其最白(或者说亮)的状态。

5.	颜色最小值，可以视为颜色（或者说图像）的最暗状态。
与最大值相同因为RGB的所有颜色都做了最小值操作，无论哪个颜色都会更加接近黑色(0,0,0)，所以两个颜色或者图像做最小值操作就是取其最暗(黑)的状态。

有了这些颜色操作的常识我们就可以更加深入的了解Blend混合方法了。
常用的Blen混合方法：https://blog.csdn.net/baicaishisan/article/details/79072127
1.	透明度混合Blend SrcAlpha OneMinusSrcAlpha，即常用半透明物体的混合方式。
 
这时最常用的半透明混合了，首先要保证半透明绘制的顺序比实体的要后面，所以Queue标签是必要的Tags {"Queue" = "Transparent"}。Queue标签告诉着色器此物体是透半透明物体排序。Blend SrcAlpha OneMinusSrcAlpha解释下，以上图为例，油桶是带此Shader的混合目标。当绘制油桶时，后面的实体BOX已经绘制好并且放入屏幕里了，所以ScrAlpha与油桶渲染完的图像相乘，部分区域Alpha为0即相乘后为无(颜色)，这时正好另一部分由OneMinusSrcAlpha(也就是1-ScrAlpha)为1即相乘后原色不变，两个颜色相加后就相当于油桶的透明部分叠加后面实体Box的画面，于是就形成了上面的这幅画面。反过来也是一样，当ScrAlpha为1时，源图像为不透明状态，则两个颜色在相加前最终变成了，源图像颜色+无颜色=源图像颜色，于是就有了上图中油桶覆盖实体Box的图像部分。

2.	加白加亮叠加混合 Blend One One，即在原有的颜色上叠加屏幕颜色更加白或亮。
 
第一参数One代表本物体的颜色。第二个参数代表屏幕上的颜色。两种颜色相加则，导致形成的图像更加亮白。这样我们就看到了一个图像加亮加白的图像了。

3.	保留原图色彩Blend One Zero，即只显示自身的图像色彩不加任何其他效果。
 
本物体颜色，加上，零，就是本物体颜色。这是最简单的混合了。

4.	自我叠加（加深）混合Blend SrcColor Zero，即源图像与源图像自我叠加。
 
与上面相比，加深了本物体的颜色。第一个参数本物体的颜色与本物体的颜色相乘，加深了颜色，第二个参数为零，使得屏幕颜色不被使用。所以形成的图像为颜色加色的图像。

5.	目标源叠加（正片叠底）混合Blend DstColor  SrcColor，即把目标图像和源图像叠加显示。
 
第一个参数，本物体颜色与屏幕颜色相乘，颜色叠加。第二个参数，屏幕颜色与本问题颜色相乘，颜色叠加。两种颜色相加，加亮加白。这个混合效果就如同两张图像颜色叠加后的效果。

6.	软叠加混合Blend DstColor  Zero，即把目标源图像叠加到前面来。
 
与前面的叠加混合效果相似，这个只做一次叠加，并不做颜色相加操作，使得图像看起来在叠加部分并没有那么亮白的突出。因为第二个参数为零，表示后面的屏幕颜色与零相乘即为零。

7.	差值混合BlendOp Sub，Blend One One，即注重黑白通道的差值。
 
在这个混合中使用了混合操作改变，从默认的加法改成了减法，使得两个颜色从加法变为了减法，不再是变白变亮的操作，而是反其道成为了色差的操作。

6.	ColorMask，设置颜色遮罩过滤。

ColorMask可以进行颜色的遮罩功能，这相当于在渲染时把某种颜色剔除一样。
ColorMask可供选择参数 ColorMask RGB | A | 0 | R,G,B,A的任意组合。

当ColorMask 0 时将关闭所有的颜色通道，相当于没有颜色渲染就没有成型的图像了，实际上你是看不见它的，只是在Editor下可以点选出来看到轮廓。
 

当ColorMask R|G|B|A时我们来看看效果，第一张是RGBA参数时的图，即为全通道颜色渲染的模型图。
       
 
第2张是只开放R(红色)通道其他通道被拒绝，第3张是只开放G(绿色)通道，第4张是只开放B(蓝色)通道，第5张是只开放A (alpha通道)。
从上图的例子中可以充分的明白了，ColorMask对颜色通道的操作。
ColorMasks虽然也可以在Multiple render target(MRT)下做一些额外的操作，比如ColorMask RGB 3后面数字3对RGB通道中进行数字3的遮罩功能，但MRT需要显卡的支持，比如PC显卡中大多数在2006年后支持MRT从GeForce 8XXX，Radeon X2400，Intel G45开始。在手机上能支持MRT并不多，而且对MRT的功能也限制了很多，基本可以说无法使用的状态。

接下来我们重点讲讲Fixed-function部分的功能



3.	有哪些常用的画面风格。
卡通风格，次世代风格，素风格，漫反射风格
https://www.zhihu.com/question/22530411

4.	有哪些固定方法编写图形渲染算法
(https://blog.uwa4d.com/archives/USparkle_Shader.html，https://blog.uwa4d.com/archives/usparkle_cartoonshading.html)
http://www.unity.5helpyou.com/2992.html

5.	实战练习shader编写。
6.	优化
*高光和反射时对镜头做优化
*类型：漫反射，镜面反射，自发光(高光)，法线贴图，凹凸贴图，环境闭塞贴图(光照的吸收情况)，细节图，贴花图，描边，透明（硬裁切Cutout，透明Transparent，虚化Fade），裁切，lightmap，顶点偏移
*算法：加，减，乘，除，点积，叉积，反射，归一化，转换，投影，N幂，对数，指数，alpha混合，屏幕点，世界点，相对物体点，顶点色，顶点等。
*清除，合并多余的编译指令，采样UV定义，变量
*shader提前解析，在初次加载时进行解析
*少用if
*少用高级计算函数如SIN
*memory access越少越好
*计算量越少越好，变量越少越好，可取的资源越少越好
*Unity3D中配备了强大的阴影和材料的语言工具称为ShaderLab，以程式语言来看，它类似于CgFX和Direct3D的效果框架语法，它描述了材质所必须要的一切咨询，而不仅仅局限于平面顶点/像素着色。
Rendering Paths是Unity3D中一个重要的概念，中文翻译就是“渲染通道”。它可以很大程度上影响光线和阴影的渲染效果，但具体要依赖于具体的游戏内容和硬件设备，以及平台。Unity3D中有三种渲染通道类型，从高到低分别为：Deferred Lighting，Forward Rendering，Vertex Lit。如果平台或设备显卡不能支持高级别的通道类型，Unity3D会自动选择稍微低一些的类型。
三种类型的细节比较，详情看参考手册。
渲染通道与shader的关系:
Deferred Lighting通道类型不关心有多少个光源会影响它，每个物体一般都会绘制两次；类似地，Vertex Lit 只绘制一次。所以对于这两种类型来说，shader对表现效果的改变大多在于多重纹理方面。
Forward 通道类型的表现效果要取决于shader和场景中的光源。它有两种基本的计算方式Vertex-Lit 和 Pixel-Lit。可以翻译为逐顶点渲染法和逐像素渲染法吧应该，对应着D3D中的顶点着色和像素着色过程。
Vertex-Lit 用于对网格模型表面顶点进行光照计算，一次性将所有光源的影响都计算在内，所以无论场景中有多少个光源，这种方式绘制的物体只绘制一次。
Pixel-Lit 会计算每个像素上面最终的光照，因此一个物体必须先呗绘制一次来获得环境光和主方向光的光照信息，再绘制一次来获得其他每个额外的光源信息。应用Pixel-Lit的物体的大小也会影响绘制的效率，越大的物体，绘制越慢。
Vertex-Lit 的开销大于Pixel-Lit，但是Pixel-Lit可以提供很多非常好的效果。