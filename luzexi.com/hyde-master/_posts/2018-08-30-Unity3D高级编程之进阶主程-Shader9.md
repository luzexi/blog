---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第七章，Shader(九) -  Cg
description: "unity3d 高级编程 主程 shader pipe 渲染管道 subshader shaderlab vertex fragment pass"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

Unity3D中实用的shader编程实际上是HLSL的变体，也可以说事Cg(C for Graphics)的变体，因为Cg 和 HLSL 在实际使用中是一样的。

Cg语言规范是和Microsoft的High Level Shading Language兼容的。Cg着色器遵循Microsoft的最新的D3DX Effects格式标准集，也完全兼容Microsoft的HLSL。

在当前市场中多种多样不同的设备环境下，要匹配尽量多的设备就得使用 DX9-style HLSL。

Unity3D编写的Shader会被编译成各种不同的语言以适应不同的平台：


向量运算的意义：

加，减，乘，除，dot点积，cross叉积，project映射投射，Reflect反射。

加法：

		公式为 (a,b,c) + (a1,b1,c1) = (a+a1,b+b1,c+c1)

		从坐标点角度看，可以认为是对坐标点做偏移操作。

		从颜色角度看，可以认为是对颜色加亮操作。

		从向量角度看，可以认为是两个向量组成的菱形求其对角边，或者说两个头尾连接的向量组成的三角形求其斜边。

减法：

		公式为 (a,b,c) - (a1,b1,c1) = (a-a1,b-b1,c-c1)

		从坐标角度看，可以认为是对坐标做偏移操作，也可以认为是求两个点之间的向量。

		从颜色角度看，可以认为两个颜色的反差度，也可以认为是变暗操作。

		从向量角度看，可以认为后面一个向量反向后再求对角边，也就是同一个点出发的两个向量组成的三角形的斜边。

乘法：

		公式为 (a1,b1,c1)x(a2,b2,c3) = (a1*b1,a1*b2,c1*c2)

		从坐标点角度看，可以认为a1,b1,c1缩放了a2,b2,c2。

		从颜色角度看，可以认为是颜色叠加。

		从向量角度看，无意义。

dot点积：

		向量意义为，向量a与向量b的夹角，cos的值与a，b的线段长度的乘积。

		也可以判断，a，b两个向量是否垂直，如果等于0为垂直，

		或者判断是否为反方向，如果小于0为反方向。

cross叉积：

		公式, cross( (x,y,z) , (x1,y1,z1) ) = (y1*z2 - z1*y2, z1*x2 - x1*z2, x1*y2 - y1*x2)

		向量意义为，求得a，b向量形成的平面的垂直向量。

		或者大小为以a、b为两边的平行四边形的面积，即为|a|*|b|*sinθ

Shader中if和for的效率问题以及使用策略，if在不同的显卡下效率不同。
https://www.sohu.com/a/219886389_667928
https://blog.csdn.net/lou09020207yes/article/details/76033977

局部变量使用if几乎没有影响。
https://blog.csdn.net/xiaxl/article/details/72530119

一个pass，一个drawcall。证明
https://blog.csdn.net/lyh916/article/details/45725499

Unity3d中Shader的一些关于矩阵变换的基本信息
https://blog.csdn.net/yutyliu/article/details/56013807

Unity3d中Shader的一些常用方法
https://blog.csdn.net/yutyliu/article/details/56278452

三维数学基础（一）坐标系、向量、矩阵
https://blog.csdn.net/iosevanhuang/article/details/9052165

向量点积(Dot Product),向量叉积(Cross Product)
http://www.waitingfy.com/archives/320

[CG编程] 基本光照模型的实现与拓展以及常见光照模型解析
https://www.cnblogs.com/QG-whz/p/5189831.html

LOD和渲染队列----渲染通道通用指令(一)
https://www.cnblogs.com/HangZhe/p/7228746.html

混合模式、Alpha测试、深度测试、通道遮罩、面剔除的使用----渲染通道通用指令(二)
https://www.cnblogs.com/HangZhe/p/7231026.html

GrabPass截屏的使用和Shader的组织优化
https://www.cnblogs.com/HangZhe/p/7233065.html

HLSL内置函数
http://www.cppblog.com/lai3d/archive/2008/10/23/64889.html

3.	有哪些常用的画面风格。
卡通风格，次世代风格，素风格，漫反射风格
https://www.zhihu.com/question/22530411

次世代风格，由4张贴图组成
主图，法线贴图，高光贴图，自发光贴图

https://blog.csdn.net/baidu_26153715/article/details/44873667
漫反射，高光贴图，法线贴图，Cube Map 实现

卡通风格shader制作方式
https://blog.csdn.net/u011047171/article/details/46723645

饱和度是指色彩的鲜艳程度，也称色彩的纯度。饱和度取决于该色中含色成分和消色成分（灰色）的比例。含色成分越大，饱和度越大；消色成分越大，饱和度越小。纯的颜色都是高度饱和的，如鲜红，鲜绿。混杂上白色，灰色或其他色调的颜色，是不饱和的颜色，如绛紫，粉红，黄褐等。完全不饱和的颜色根本没有色调，如黑白之间的各种灰色

https://www.cnblogs.com/softcat/p/6296136.html
shader 内置time

4.	有哪些固定方法编写图形渲染算法
(https://blog.uwa4d.com/archives/USparkle_Shader.html，https://blog.uwa4d.com/archives/usparkle_cartoonshading.html)
http://www.unity.5helpyou.com/2992.html

Shadow Map 原理和改进
https://blog.csdn.net/ronintao/article/details/51649664
https://www.cnblogs.com/lijiajia/p/7231605.html
https://blog.csdn.net/me_badman/article/details/61916613

动态阴影
https://blog.uwa4d.com/archives/sparkle_shadow.html

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