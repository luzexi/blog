---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第七章，Shader(五) - Pass(二)
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### Render State Command渲染状态指令详解

Render State Command 是渲染状态的指令，它在Pass和SubShader都有效，放在Pass就只在当前Pass里有效，放在SubShader中则是多个Pass的共同状态。

不过不是所有的Render State都可以成为共同拥有的渲染状态的，有些Render State只在Pass中有效，而另一些则可以成为共同渲染状态。

===

下面我们来介绍下Pass中的渲染状态Command指令：

### Cull，前后裁切指令。

Cull是种优化的指令，可以让远离屏幕的部分不渲染。所有的物体都有前后两面，比如一个Box箱子你永远都看 不到屏幕另一侧的部分，所以可以对Box箱子后半部分进行裁切不渲染。

可设置参数如下：

![cull](/assets/book/7/shader20.png)

        上图为参数 Cull Back，裁切白色Box的后半部分，保留前半部分。

![cull](/assets/book/7/shader21.png)

        上图为参数 Cull Front，裁切白色Box的前半部分，保留后半部分，展示的后半部部分与后面的箱子相交。

![cull](/assets/book/7/shader22.png)

        参数 Cull Off，关闭裁切功能，与Cull Back展示的效果一样，但其实多渲染了后半部分的网格模型。

Cull默认是裁切后半部分的，这样可以达到优化的效果，如果你没有对它进行设置的话。其实裁切后半部分和完全不裁切，在有深度缓存数据时是显示的效果是完全一样的，只是一个做了优化另一个没做而已。

### ZTest，设置深度缓存测试模式。

ZTest的功能是在两个物体相交时渲染绘制中对前后深度进行测试，默认情况下以LEqual小于等于其他物体时测试为不通过，当不通过时就不渲染那部分模型。

LEqual参数，其实就是测试前面的物体是否挡住了该物体部分，如果挡住了就不渲染的意思，它也是ZTest的默认参数。

其他参数还包括

    ZTest Less | Greater | LEqual | GEqual | Equal | NotEqual | Always

    参数 小于 | 大于 | 小于 | 小于等于 | 大于等于 | 等于 | 不等于 | 关闭测试

参数解释：

![ztest](/assets/book/7/shader23.png)

    上图中为参数 Less 该物体上的部分深度在其他物体后面时不渲染。

![ztest](/assets/book/7/shader24.png)

    上图中为参数 Greater 该物体上的部分深度在其他物体部分前面时不渲染，也就是被覆盖的显示，没被覆盖的反而不显示。

![ztest](/assets/book/7/shader25.png)

    上图中为参数 Equal 该物体上的部分深度与其他一致时才渲染，否则不渲染。

![ztest](/assets/book/7/shader26.png)

    上图中为参数 NotEqual 该物体上的部分深度与其他物体不一致时才渲染，否则不渲染。

### ZWrite，设置深度缓存写入开关。

Zwrite可以控制物体渲染的像素是否写入depth buffer深度缓存数据中。

如果没有depth buffer深度缓存数据，物体将无法辨识前后关系也就无法实现阻挡物体不绘制的功能。

这在大多数不透明物体中都是需要depth buffer深度缓存数据来识别前后遮挡的，但在半透明和透明物体中就不需要depth buffer。

半透明和透明物体没有depth buffer深度缓存数据，所以没有所谓的写入一说，也就是通常在半透明和透明物体的Shader中，ZWrite是关闭的。

![zwrite](/assets/book/7/shader27.png)

![zwrite](/assets/book/7/shader28.png)

如上两个图，图1为ZWrite On时的效果，黄色Box遮挡住了后面的蓝色Box因为有depth buffer深度缓存的写入，前后关系有依据。

图2为ZWrite Off的效果，后面蓝色的Box被提到了黄色Box的前面显示，这是因为黄色Box的depth buffer没有写入，在渲染时，面片前后关系没有了依据，导致了错乱。

{% include advertisement_content.html %}

### Offset，设置深度缓存的偏移。

Offset允许你设置物体在深度缓冲中的偏移量，这相当于让物体在屏幕显示中提前或者推后显示位置，但实际世界中的位置不变。

Offset有两个参数，Factor偏移缩放大小和Units偏移量。Factor相当于缓存数据偏移放大的倍数，Units相当于直接偏移的量。

如果从公式上理解，可以理解为Buffer = (x or y)*Factor + Units。

举例 Offset 3 , 5，可以理解为偏移放大3倍，并且再偏移5个单位。

注意，偏移负方向才是离屏幕越近的方向。所以Offset 0 , -1时，两个物体重合部分肯定是拥有Offset 0 , -1的提前显示。

如果两个物体前后差的比较远，那么想把后面的物体提到前面来显示，那么就要加大偏移量了，比如 Offset 0 , - 1000 或者 Offset -100, -1等。

下面举例说明Offset的作用：

对白色的做处理Offset 0 , 0 -------两者重叠，时而显示普通的有贴图的立方体，时而显示白色的立方体，如下图所示：
 
![offset](/assets/book/7/shader29.png)

对白色的做处理Offset 0 , -1------两者重叠部分完全被白色覆盖,因为白色立方体做了偏移量的处理，深度缓存中的坐标离屏幕更近，如下图所示：

![offset](/assets/book/7/shader30.png)

对白色的做处理Offset 0 , 0 ---- 现在把两个立方体的距离拉开，正常情况下按透视的距离进行显示，如下图所示：

![offset](/assets/book/7/shader31.png) 

对白色的做处理Offset 0 , -1000 ---- 现在加重白色立方体的偏移量，深度缓存数据向屏幕偏移，这时很多有贴图的立方体部分被覆盖，因为偏移量加大，提前量就加大了，但绿色立方体其他部分的深度数据还是比偏移后的大，所以那个部分还是无法覆盖，如下图所示：

![offset](/assets/book/7/shader32.png)

对白色的做处理Offset -100 , -1000------再次加大偏移量，这次加入了缩放比例的偏移，偏移的更多，使得白色立方体全部覆盖绿色立方体，如下图所示：
 
![offset](/assets/book/7/shader33.png)

注意以上所有图中两个立方体的世界坐标一直都没有更改。如果要让物体向后推也是同样方法，负数变正就可以，所有的偏移都是渲染上的偏移，与物体实际坐标无关。
