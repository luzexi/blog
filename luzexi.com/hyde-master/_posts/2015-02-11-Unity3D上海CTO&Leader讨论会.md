---
layout: post
status: publish
published: true
title: Unity3D上海CTO&Leader讨论会
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 486
wordpress_url: http://www.luzexi.com/?p=486
date: !binary |-
  MjAxNS0wMi0xMSAyMDoxNToyNCArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wMi0xMSAxMjoxNToyNCArMDgwMA==
categories:
- Unity3D
- 生活
tags:
- Unity3D
---
今天去了趟unity上海分部的CTO&Leader讨论会。记录下会议内容很多无聊的东西我都略过不写了，报告下我们比较关心的，或者将来会遇到的困难。
1.现在unity3d 4.6.2 打包64位的ios app 不靠谱，bug有400多个，不建议升级。4月1号前说是会出个稳定版，但讨论中透露，稳定版中也会由许多未知因素。IL2P打成c++的方式有众多问题和困难。

2.他们说了个热更新的解决方案，但只限安卓。我听了下，原理就是把所有程序都打成一个dll，用加载资源的方式加载，然后再实时编译。这个要求代码不能附着在prefab上。然后我说了关于使用lua更新机制，虽然现在在u3d里仍不是非常成熟，但困难是永远都有的，各位可以值得一试，不要等到别人用熟了你才开始，那已经慢了好大一步了。

3.问了关于内存释放的问题，他们阐述说resources.unloadunusedassets并不是根据引用计数来释放内存，而是根据世界树中的实例检测，而System.GC.Collect()是根据引用计数销毁的，所以可以选择两个一起使用，也就是我们现在的方式。

4.关于内存，他们在阐述他们提供的一个性能测试服务中，提到由于ngui底层是mesh重构，而每次重构都会积累内存消耗，所以ngui的内存问题是比较严重，他们暗示项目使用unity自己的ugui。

5.关于混淆，他们只有一般的混淆方案。网上可以找到。没什么特别的自己方案。混淆的注意点是，混淆时安卓和ios等平台对接的接口不要进行混淆，在平台调用时会找不到相应接口。unity技术团队说他们可以提供混淆服务，但我觉得并不困难。

6.unity5 将与unity4完全不同，互不兼容，也就是像cocosx 2与cocosx 3一样，拆分两个版本维护。IL2P将会逐渐替代mono的打包方式。mono也很难升级，因为高版本的授权费用很高，一般的用户承受不了。

7.以下是一些unity5比较有用的改善。没兴趣关注的就不提了。

*场景中的shader可以自动合并成一个shader，减低drawcall。但我怎么觉得这里面有好多空噱头。
*assetbundle打包可以进行增量更新，无变化的assetbundle会被自动识别，加快了批量打assebbundle的速度。
*更多得采用多线程处理，加快unity应用的速度。
*支持webGL平台的开发。在支持webGL浏览器里不需要unity插件。
*unity5升级为64位，编辑器可使用更多内存。

8.最后他们提供说一个真机性能测试的服务，可以提供60页的性能测试报表，其实我觉得没什么用。性能卡点自己找起来比较准确点。
<img class="alignnone size-full wp-image-487" src="/assets/uploads/2015/02/IMG_0972.jpg" alt="IMG_0972" width="1763" height="885" />

转载注明出处 http://www.luzexi.com
