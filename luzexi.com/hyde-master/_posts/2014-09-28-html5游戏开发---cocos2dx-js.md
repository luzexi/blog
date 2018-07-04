---
layout: post
status: publish
published: true
title: html5游戏开发---cocos2dx-js
description: "html5 游戏 开发 cocos2d"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 412
wordpress_url: http://www.luzexi.com/?p=412
date: !binary |-
  MjAxNC0wOS0yOCAxMjowNToyMyArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wOS0yOCAwNDowNToyMyArMDgwMA==
categories:
- 前端技术
- 后端技术
tags:
- cocos2d
- html5
---
html5游戏开发---cocos2dx-js。在神经猫发热后，我开始关注html5游戏。所以前段时间做了2个html5小游戏当试水，用的引擎是cocos2dx-js。在这里我想做回忆下，html5的制作过程和需要注意的地方，也阐述下当下html5游戏渠道的问题。

我先接触了egret引擎，它是国内html5优秀的游戏引擎，采用typescripts做主语言。在我用了几天发现，它的主语言并非js所以html5底层的东西会被下意识的屏蔽掉，何不用js的引擎来写html5呢，即能快速开发又能随时接触html5底层。所以我使用了cocos2dx-js。

关于cocos2dx-js的教程就不在这里写了，接口跟cocos2dx-c++的差不多。

js写html5需要注意的几点：

1.html5游戏最需要快速加载，js代码不能分成很多个一个个加载，减慢了加载速度也容易流失用户。所以在写玩html5时需要对所有js代码进行压缩合并。google的closure compiler解决了这个问题，cocos2dx-js也将这个功能融入进里面。下面是步骤：

在cocos2dx-js-v3.0里使用cocos compile -p web -m release来对html5的js代码进行压缩。

在cocos2dx-html5(与cocos2dx-js-v3.0是两个版本，cocos2dx-js-v3.0比较好用)里打包html5需要用到ant和build.xml。打包主要内容为将所有js文件打包成同一个文件，目的是减少加载文件数量，加快加载速度。打包命令：cd到项目文件夹下。敲入:ant或者ant -buildfile build.xml。build.xml中囊括了要打包的js文件。其中指定了js打包编译工具complie.jar是google closure compiler专门压缩js文件大小工具。

关于微信分享:

微信api已经在微信的浏览器里已经注入了它的api。

微信api的机制其实就是事件机制，比如当你点击分享时，调用某个你制定的方法显示你想显示的文字和图片。

微信api你可以参考 [https://github.com/zxlie/WeixinApi](https://github.com/zxlie/WeixinApi)

主要用到的有分享朋友圈，分享给朋友，分享后回调。你可以首先看这几个重点。

{% include advertisement_content.html %}

关于html5游戏渠道:

1.现在html5游戏的渠道很少，而且现在html5游戏基本呈现在微信浏览器中，所以微信的公众平台是渠道之一，而且似乎占比比较大。

2.在国外html5游戏在电视上的占比也开始大了，我不知道国内的情况，未来国内是否有可能在电视上呈现html5游戏。

3.html5游戏的app形式，这个还是很有潜力的，因为渠道太少倒逼html5游戏发展成轻型app。也就是说html5的app就只是个连接地址，但在手机上呈现的是一个图标，打开后以浏览器形式访问。

最后大家可以看看这个小游戏，请使用手机二维码扫描。

<img class="alignnone size-full wp-image-413" src="/assets/uploads/2014/09/qrcode.png" alt="qrcode" width="280" height="280" />

源码地址：[https://github.com/hangzhou-JIMI/MemoryFruits](https://github.com/hangzhou-JIMI/MemoryFruits)

转载请注明出处：http://www.luzexi.com
