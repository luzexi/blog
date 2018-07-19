---
layout: post
status: publish
published: true
title: Unity3D--Texture图片空间和内存占用分析
description: "unity3d texture 优化 分析"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 308
wordpress_url: http://www.luzexi.com/?p=308
date: !binary |-
  MjAxNC0wNS0yMSAxMTowNToyMSArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wNS0yMSAwMzowNToyMSArMDgwMA==
tags:
- Unity3D
- 前端技术
categories:
- unity3d
- 前端技术
---
Texture图片空间和内存占用分析。由于U3D并没有很好的诠释对于图片的处理方式，所以很多人一直对于图集的大小和内存的占用情况都不了解。在此对于U3D的图片问题做一个实际数据的分析。此前的项目都会存在这样或者那样的打包后包大小与内存占用情况的问题，所以这次所以彻彻底底得分析下U3D对于Texture的处理方式。程序里的内存优化请参考[《unity3d优化之路》](/unity3d/游戏架构/前端技术/2014/02/22/Unity3d优化之路.html)。减少U3D包大小请参考[《unity3d之如何将包大小减少到极致》](/unity3d/游戏架构/前端技术/2014/06/06/Unity3D之如何将包大小减少到极致.html)。

我打包多种类型的项目，空项目和10张放在Resources文件夹中的图为比较案例。以下是比较数据。

IPHONE：

1.空项目----空间占用量42.3MB----IPA大小10MB

2.10张1200*520无压缩Texure 单张图占用量2.8MB----空间占用量70.2MB----IPA大小22.9MB

3.10张1200*520压缩成1024*1024PVRTC4 单张图占用量0.5MB----空间占用量47.3MB----IPA大小13.2MB

4. 10张1024*1024无压缩Texture 单张图占用量4MB----空间占用量82.3MB----IPA大小14.6MB

5.10张1024*1024压缩为PVRTC4格式 单张图占用量0.5MB----空间占用量47.3MB----IPA大小11.6MB
 
宗上数据总结：

一、2的N次方大小的图片会得到引擎更大的支持，包括压缩比率，内存消耗，打包压缩大小，而且支持的力度非常大。

二、减小图片的占用大小和内存方式有:图片大小变化(Maxsize),色彩位数变化(16位，32位)，压缩(PVRC)。

三、U3D对于图片的格式是自己生成的，而并不是你给他什么格式，他就用什么格式，一张1024*1024图在无压缩格式下，它会被U3D以无压缩文件形式存放，也就是说U3D里的Texture Preview里显示的占用大小**MB不只是内存占用大小，还是空间占用大小。如下图所示：
<img class="alignnone size-full wp-image-310" src="/assets/uploads/2014/05/QQ截图20140521102030.png" alt="QQ截图20140521102030" width="626" height="726" />

U3D的内部机制为自动生成图片类型来替换我们给的图片，在图片的压缩方式上需要进行谨慎的选择。

{% include advertisement_content.html %}

压缩格式在U3D的[Component Reference](http://docs.unity3d.com/Documentation/Components/class-Texture2D.html)里有介绍我就不再详细介绍，只介绍几个重点的:

RGBA32格式为无压缩最保真格式，但也是最浪费内存和空间的格式。Automatic Turecolor和它一个意思。

RGBA16格式为无压缩16位格式，比32位节省一半的空间和内存。Automatic 16bits和它一个意思。

RGBA Compressed PVRTC 4bits格式为PVRTC图片格式，它相当于把图片更改了压缩方式新生成了一个图片来替换原来的我们给的图片格式(比如我们给的是PNG格式)。

注意：U3D所有图片的压缩格式都会以另一种方式来存储，不会以你给的方式来存储，只有你指定了某种格式，它才会转成你要的格式。而且压缩格式在Android里并不一定有效，因为Android的机型多，GPU的渲染方式也不一样，有的是Nvidia，有的是PowerVR，最最好的在安卓机子上启用RGBA16方式，因为这个是适应所有机型的，并且比32位占用量少一半，但也需要因项目而异，只是推荐使用的格式，可以多用。
 
转载请注明出自：http://www.luzexi.com
