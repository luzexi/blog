---
layout: post
status: publish
published: true
title: Unity3D之AssetBundle资源加载封装
description: "unity3d assetbundle 加载封装"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 284
wordpress_url: http://www.luzexi.com/?p=284
date: !binary |-
  MjAxNC0wNC0xNiAyMTowNzo1MyArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wNC0xNiAxMzowNzo1MyArMDgwMA==
tags:
- Unity3D
- 游戏通用模块
- 前端技术
---
Unity3D之AssetBundle资源加载封装。在《临兵斗者三国志》中我使用了U3D的AssetBundle资源动态加载机制，原因是某些画面资源太多，IO阻塞过慢会造成游戏奔溃。在开发过程中，遇到点问题:

1.当资源更改变化时，如何能快速得反应到开发中。

解决方案:

我使用宏定义UNITY_EDITOR来判断是否是开发编辑状态。当处于开发编辑状态时，自动读取指定目录下U3D本身资源，而不使用AssetBundle。这样就达到了当prefb变化时能快速反应到开发编辑中。而当不是处于编辑状态时，则正常使用异步加载读取AssetBundle。这个方式唯一的毛病就是，必须让所有U3D程序员都非常清除明白，如果写错，编辑模式下会没问题，发布后会出问题，所以需要检查。

2.当不同资源之间有重复的资源时如何将AssetBundle空间占有量最小化。

解决方案:

GUI资源之间有特别多的重复的问题，挑出几个重复得特别厉害的，比如ICON图集，公用图集。在打包期间把他们设为共享资源，并在加载时首先加载共享资源，这样既节省了AssetBundle空间占有量，也节省了内存。这个方式的毛病是当你将资源更改要打包某个资源时，需要将所有与共享有关的资源重新打包一遍。

3.如何应对自动释放资源问题。

{% include advertisement_content.html %}

解决方案:

在游戏中有指定资源释放和自动释放所有AssetBundle资源以销毁内存(这里不是指销毁U3D内存，而是AssetBundle内存，U3D内存管理分图片内存，AssetBundle内存，编译程序)。销毁指定资源就按正常来没有争议。销毁所有资源就要有点措施了，因为有些资源是不能被销毁的，因为它们是共享资源，需要全程跟着游戏走，所以当自动销毁所有资源时，将共享资源排除在外。并且在销毁后调用Resources.UnloadUnusedAssets();和GC.Collect();

4.打包AssetBundle方式。

解决方案:

打包AssetBundle方式有几种:1.单资源打包，也就是说一个.prefb或Texture打一个包。2.多个资源打包，将某些资源都打成一个AssetBundle，节省了几个资源包之间的共享资源也减小了多个AssetBundle引起的空间扩大问题。但并不是说所有项目都是多个资源打成一个AssetBundle是好的。《临兵斗者三国志》就是一大部分使用单一打包，而共享资源使用多个资源打成一个AssetBundle的方式。

最后奉上本人对AssetBundle封装的源码。[https://github.com/luzexi/Unity3DGameResource](https://github.com/luzexi/Unity3DGameResource)

你也可以去我的[github](https://github.com/luzexi)上查看找我做的一些源码插件，如果喜欢的话可以star或者fllow。
 
转发请注明出自：http://www.luzexi.com
