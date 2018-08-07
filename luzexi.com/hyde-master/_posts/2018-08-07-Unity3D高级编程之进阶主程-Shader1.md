---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(一) - 常用的Shader
description: "unity3d 高级编程 主程 shader 漫反射 镜面反射 自发光 球面反射"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

###	常用的Shader

===

1， Standard Shader是Unity5中新加入的一个Shader，它被设计的目的是为了替代很多旧版本里表现“真实世界”的Shader，比如石头，木头，玻璃，塑料，金属等材质。

它被设置为Unity3D的默认Shader，当创建一个新的材质球时，你不需要去一个很长的列表里在这么多样式中挑选Shader，你直接可以使用Standard Shader，因为它包含了他们大部分的功能比如反射，凹凸贴图，透贴等。

Standard Shader支持了大部分的功能，并且这些功能可以被自由的打开和关闭。

Standard Shader还包括了一个高级的灯光模式叫Physically Based Shading。

Physically Based Shading(缩写PBS)类似于在材质和等之间相互作用来模拟真实世界灯光反射效果，它在需要渲染真实世界的场景里非常适用。

2，	Diffuse漫反射。

最简单的Shader之一，它使用 Lambertian 漫反射灯光模式计算灯光，亮度只依赖于灯光和表面的角度，并且不随着摄像头的移动而变化。

漫反射主要用于粗糙表面的物体，比如植物，墙体，衣服等，其表面看起来是平滑的，但用放大镜仔细观察，就会看到其表面是凹凸不平的，所以平行的光线照射过去后，被这些粗糙的表面反射向不同的方向，就像是散漫无规则的扫射一样。

![镜面反射](/assets/book/7/shader1.png)

当物体被使用漫反射的Shader后，他的亮度会显得比较散漫和暗淡，因为反射光线并不会集中摄入摄像机，而是散漫开来。所以使用Diffuse大大减弱了光照的强度，使得光在物体上体现的并不是很强烈。

人眼之所以能看清物体的全貌，主要是靠漫反射光在眼内的成像。如是全部单向反射的物体表面，不但看不清物体的外貌，还会引起某一方向上的眩光干扰现象。人们依靠漫反射现象才能从不同方向看到物体。在环境光学中，常把无光泽的饰面材料近似地看作均匀漫反射表面。

3，	Specular镜面反射。

镜面反射是指反射面比较光滑，当平行入射的光线射到这个反射面时，仍会平行地向一个方向反射出来，这种反射就属于镜面反射。

镜面反射其反射波的方向与反射平面的法线夹角（反射角），与入射波方向与该反射平面法线的夹角（入射角）相等，且入射波、反射波，及平面法线同处于一个平面内。

摄影时应避免镜面反射光线进入摄影机镜头，由于镜面反射光线极强，在像片上将形成一片白色亮点，影响地物本身在像片上的显现。

镜面反射遵循光的反射定律。反射所成像的性质是正立的，等大的，位于物体异侧的虚像。镜面反射入射的光线是平行光线，反射到光滑的镜面，又以平行光线出去。

镜面反射和Diffuse一样使用了 Lambertian 光线计算模式，并且在此基础上加入了一个取决于观察者的高亮反射，我们称之为 Blinn-Phong 光线模式。

Blinn-Phong 有一个高亮镜面反射它取决于表面角度，光线角度，和观察角度。高亮点实际上只是用实时适应的方式去模拟模糊映像的光源。高亮点的模糊级别可以通过控制亮点滑块来调整。

另外 Specular Shader 中，主贴图的 alpha 通道扮演了镜面反射图的角色（也可以称为光泽图），alpha通道描述了物体的哪些区域有更强烈的反射效果，哪些区域反射效果比较暗淡。

alpha为0时（即黑色的区域，把alpha单独提出来成为一副图的话就是黑色的部分），区域内没有反射效果，alpha为1即白色的区域，区域内反射效果为最强烈。

如果你想在不同区域内有不同的反射效果，这种类型的光泽图对你来说很有用。比如生锈的金属的高亮反射比较弱，光滑的金属则有强烈的高亮点反射。再比如口红的镜面反射比皮肤大，而皮肤的镜面反射又比衣服大。

所以我们可以制作不同区域的光泽图来表现不同区域的高亮反射。前面说在Specular Shader中的主图的alpha已经扮演了这个光泽图的功能，所以我们只要用alpha来表现不同区域的高亮反射效果就可以了。

![镜面反射](/assets/book/7/shader2.png)

镜面反射和漫反射，反射角度和入射角相等，都遵循光的反射定律，唯一的区别，就是镜面反射的反射面比较平，因而光束比较统一而且反射方向比较一致，漫反射的反射平面高低不平导致反射光的光束也杂乱无章。两者都符合反射定律。

4，	Vertex-Lit  顶点渲染。

Vertex-Lit 是所有常用Shader里最最简单的一个Shader。

无论有多少灯光，Vertex-Lit只接受一个灯光，在渲染时只做顶点计算的光效果。

它是顶点光计算方式，不会渲染像素级别的效果，例如点光线纹理，法线贴图，阴影等。

它只会关注模型的形状，如果你想要参入其他因素的，就必须使用其他类型的Shader做替代，比如Pixel-Lit像素渲染。

Vertex-Lit就像是最朴素的渲染者，没有反射光的互相影响作用，只做顶点的光计算并且只接受一个光。

5，	Transparent 半透明。

Transparent 类型的Shader被用在全透明和半透明的物体上，因为可以透过物体看到物体背后的东西，所以具有特殊性，例如玻璃瓶，玻璃窗，水，眼镜，酒，水晶等。

Transparent 的Shader主图上的alpha通道可以决定哪些区域是透明的，透明到什么程度。用它可以制作出效果丰富的玻璃类型的物体，或者显示器表面，或者科幻的效果。

主图的alpha通道里，如果alpha为0（黑色区域）时会表现为是全透明，否则alpha为1（白色区域）时会表现为不透明。如果你的主图没有alpha通道，物体就相当于是个完全的不透明物体。

半透明物体的排序问题。

使用Transparent的物体在游戏中常常会有一些诡异的问题，这是由于传统显卡程序导致的透明物体的排序问题。

例如当我们透过两扇透明的玻璃窗看东西的时候，会发现这两扇玻璃窗有点问题，似乎后面的玻璃在前面但窗又被挡在后面。

这个问题直到现在还一直都存在而且并没有什么完美的方案解决它，我们只能通过编写Shader的规则来告诉显卡，哪扇窗户在前面，哪扇窗户在后面才能让处理器正确的显示玻璃前后这个半透明物体的排序情况。

为此，我们在使用半透明物体的时候，就应该明白不能过度使用半透明物体，过多的半透明物体将导致很多排序问题。如果设计中不可避免，就应该提醒美术设计师注意半透明物体的排序，让他们在遇到问题前有所准备。

![镜面反射](/assets/book/7/shader3.png)

用裁切 Cutout 做半透明。

用裁切Cutout做半透明，是半透明物体的另一种特殊形式，对Cutout来说只渲染两种情况，物体上的某部位要么完全透明要么完全不透明。

它使用在一些某些不需要半透明但又需要全透明的物体上，这些物体上的部位通常都是要么完全透明，要么完全不透明，比如说栏杆，铁环，有很多洞的盒子等。

![镜面反射](/assets/book/7/shader4.png)

Cutout本身就是为了节省模型细节的替代物，大多数使用Cutout作为Shader的物体都可以用重新制作模型细节并且挖空裁切模型的做法来替代。

制作这样的细节非常耗费美术资源，也增加了巨量的三角面数和顶点数，导致增加内存和CPU的压力，于是用Cutout来替代挖空和裁切能更好的优化内存和CPU，也同样节省了不少3D的工作量。

{% include advertisement_content.html %}

6，	Self-Illuminated 自发光。

这里称述下自发光的意思，很多人对自发光有很多误解，比如自己会产生光源影响周围环境，自己会产生光晕效果等都是错误的。

自发光的意义很简单，不受光源影响，自己能够独自呈现高亮的效果，并且不影响周围环境。

![镜面反射](/assets/book/7/shader5.png)

Self-Illuminated 自发光允许我们自己定义模型的哪些部位是亮的哪些部位是暗的。

这个Shader能够通过自发光贴图的alpha通道对物体某部分的光亮进行调整，让某部分的亮度看起来很亮或者很暗，就像是自己有发光体一样。

其实Self-Illuminated通过alpha值在调整亮度使得不需要任何的光源都能展示足够的亮度，当没有光照着模型的时候模型本身也具备了类似光源照射的足够亮度，而当有光源照射时，任何顶点光或者像素光都只是简单的对 Self-Illuminated 的光量进行简单的相加。

所以无论光源有没有照射到自发光模型，模型都会根据自发光贴图进行自发光，并且加上灯光照射的亮度。

Self-illuminate 常常用在模型某个部位需要自己加亮的物体上。比如，部分墙上的贴图能用自发光来模拟灯光的，天花板的LED小灯，汽车仪表盘，大型建筑上的广告牌，或者高楼大厦上每个窗户的灯光等。

但凡能够自己发光的，并且不影响其他物体的光线的，都可以使用自发光Self-Illuminate来替代灯光。

由于点光源或者聚光灯都是非常消耗GPU和CPU的，一旦大量使用就会对性能造成成吨的伤害(CPU消耗)导致画面延迟卡顿。

平行光，点光源和聚光灯对周围物体的影响也是很难控制的，一旦大量使用就很难调控某个特殊物体或部位的亮度。

使用Self-Illuminate很好的解决了这些问题，不仅节省了大量因为点光源和聚光灯导致的性能开销，而且对物体局部光亮的把控更精准到位。

7，	Reflective 球面反射环境模拟。

Reflective 球面反射Shader使用一个Cubemap来对周围的环境进行反射模拟。

我们可以通过改变主贴图上的alpha通道来对镜面反射度进行调整，比如玻璃，油水，铂金类质地的物体的镜面反射度高，所以主图的alpha就相对大，而金属类，液体类或者显示屏表面镜面反射度小，所以主图的alpha就可以相对小。

Reflective 主要是模拟球面反射后在镜面上的出现的环境，常用在汽车玻璃，建筑玻璃镜子，金属制品的镜面反射等。

Reflective 渲染时需要一个模拟周围环境的Cubemap，Cubemap是由6张图拼成的一个定义周围环境的立方图。周围环境是如何展示的都由Cubemap来定义，它有顶，底，左，右，前，后，6个面。

Reflective 关键点在Cubemap，这里简单说下制作Cubemap的几种方式：

1，	使用贴图制作 Cubemap。最方便的制作Cubemap的方式就是通过设置图片的类型将一张贴图设置为Cubemap来让Unity自己识别和拆分。

如图为 Cubemap 资源：

![镜面反射](/assets/book/7/shader6.png)

那么Unity是如何识别上下左右前后的6个面的呢？如下图：

![镜面反射](/assets/book/7/shader7.png)
 
Unity支持这4种布局的图片，只要你按照这四种布局的一种来布置图片的顺序就能得到正确的Cubemap图。

2，	通过设置Cubemap资源中的图来制作Cubemap。

先创建一个Cubemap资源，对Cubemap里的属性进行设置，前后左右上下的6张图，以及大小，是否使用MipMap等。如下图：

![镜面反射](/assets/book/7/shader8.png)

3，	使用Unity的API来创建Cube图。

Camera.RenderToCubemap函数可以记录在场景的某个点上记录6个面的图，从而我们可以通过Camera.RenderToCubemap创建的6张环境图来导出自己需要Cubemap图。

代码例子：

{% highlight c# %}

public class RenderCubemapWizard : ScriptableWizard
{
    public Transform renderFromPosition;
    public Cubemap cubemap;

    void OnWizardUpdate()
    {
        string helpString = "Select transform to render from and cubemap to render into";
        bool isValid = (renderFromPosition != null) && (cubemap != null);
    }

    void OnWizardCreate()
    {
        // create temporary camera for rendering
        GameObject go = new GameObject("CubemapCamera");
        go.AddComponent<Camera>();
        // place it on the object
        go.transform.position = renderFromPosition.position;
        go.transform.rotation = Quaternion.identity;
        // render into cubemap
        go.GetComponent<Camera>().RenderToCubemap(cubemap);

        // destroy temporary camera
        DestroyImmediate(go);
    }

    [MenuItem("GameObject/Render into Cubemap")]
    static void RenderCubemap()
    {
        ScriptableWizard.DisplayWizard<RenderCubemapWizard>(
            "Render cubemap", "Render!");
    }
}

{% endhighlight %}


