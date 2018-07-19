---
layout: post
status: publish
published: true
title: Unity3D-HTTP网络层封装
description: "unity3d web http 网络层 封装"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 248
wordpress_url: http://www.luzexi.com/?p=248
date: !binary |-
  MjAxNC0wMi0xNSAxNTo1OTo1NyArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wMi0xNSAwNzo1OTo1NyArMDgwMA==
tags:
- Unity3D
- 游戏架构
- 前端技术
categories:
- Unity3D
- 游戏架构
- 前端技术
---
Unity3D-HTTP网络层封装。短连接的C#封装在这里做些分享。我把网络层封装成DLL在项目中使用，所以在设计时要将接口封装的很好。 我又对HTTP部分进行了改造不再使用DDL封装了（所以去除文章部分暂时用横线做标记），所有源码都以git的submodule形式作为项目的模块。源码会再文章的最后给出来。这次改装主要是针对程序员是否能快速理解，快速上手的方面来进行的。随着对HTTP网络层不断的理解加深，我将给出更加全面的HTTP源码模块，我还对本HTTP模块写了测试案例，你们可以看测试案例，进行使用。谢谢各位的关注，再次感谢不断得支持。

一.首先说下网络层需要的一些接口，在游戏里需要用到的网络层都具有大部分的共同点，无论是TCP长连接还是HTTP短连接有部分区别，HTTP的网络事件要相对少一点。

HTTP接口基本为：数据发送Send接口，网络错误事件，数据包回调执行句柄。HTTP网络层的数据包预发送接口被我去除，因为我回想，这个功能完全可以用模块外部的程序代替，而且可以针对每个界面来写，这样更加清晰，而且主要是这个功能障碍了程序员对程序的理解。

二.说明下网络层中几个类的用途和作用。 HTTPSession类，是整个网络层的主类，承载了发送，接受，事件相应的事务。

HTTPPacketRequest类，网络请求数据包基础类。每个请求数据包的最基础的数据，包含一个m_strAction地址变量，这个是因为每个HTTP请求的地址不同而设的变量，每个请求地址前缀都相同比如：http://luzexi.com/ ，而后缀有可能变化，比如author/regist/。所以这个变量要在你创建的时候在构建函数里进行设置。

HTTPPacketAck类，网络回调数据包基础类。每个回调数据包的基础数据，每个数据包都含有几个基础的变量，我现在写的是code错误码 和 desc错误描述，你也可以改成其他你想要的，如果你理解整这段程序。总之，继承这个回调数据类的类中可以放置本次回调的数据，有数据回调时，系统将自动解析成这个类。

HTTPSession类，会话类是网络层的主类，有发送接口和网络主地址，还有一个报错接口。是整个HTTP网络层的接入口。

HttpDummySession类，我还做了虚拟会话，主要是给HTTP和单机切换用，有些游戏想做网络和单机一起兼顾，可以用HTTP虚拟类来实现，不需要改原来写好的HTTP请求，只要你对加个请求句柄处理就可以了。不过一般人不会用到。

{% include advertisement_content.html %}

三.网络层最重要的是能够快速，方便使用，能够适应变化多端的需求改变。

每个请求类（HTTPPacketRequest子类）和回调类（HTTPPacketAck子类）都是自动生成参数和自动映射回调类实例的，但千万记住，回调过来的数据和HTTPPacketAck子类里的变量名必须一致。

四.关于HTTP的Head数据部分

HEAD部分我已经写好了，但是一个统一的HEAD变量，在session类中你可以找到。因为head部分都是统一的参数变量，动的比较少，所以才这么做。在游戏项目里，可以用HEAD部分的数据也可以不用，因为完全可以用数据包的形式代替。

最后奉上源码：[Unity3D-HTTP](https://github.com/luzexi/Unity3DNetwork-http)

测试案例：[Unity3D-HTTP-Test](https://github.com/luzexi/Unity3DNetwork-http-Test)

转载请注明出处:http://www.luzexi.com
