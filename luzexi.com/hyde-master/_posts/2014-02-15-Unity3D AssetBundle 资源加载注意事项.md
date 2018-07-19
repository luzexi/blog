---
layout: post
status: publish
published: true
title: Unity3D AssetBundle 资源加载注意事项
description: "unity3d assetbundle 加载 说明"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 253
wordpress_url: http://www.luzexi.com/?p=253
date: !binary |-
  MjAxNC0wMi0xNSAxNDo0Nzo1NyArMDgwMA==
date_gmt: !binary |-
  MjAxNC0wMi0xNSAwNjo0Nzo1NyArMDgwMA==
tags:
- Unity3D
- 游戏通用模块
- 前端技术
---
Unity3D AssetBundle 资源加载注意事项.U3d资源打包以及加载出过很多问题，在这里把所有该注意的东西列出来分享下。

1.由于Unity3D的自身的问题，在资源打包时会遗漏一些引用的资源，导致在游戏加载 AssetBundle 时没有问题，但在把资源实例化时出现资源找不到，或者资源缺少引用资源等问题。

解决办法： 在打包获取资源时，分3个步骤做：1.创建空的临时的预置体Prefab 2.把要打包的资源实例化进场景 3.把实例化后的物体装进刚创建的空的预置体Prefab 4.最后再用该预置体打包成AssetBundle（注意多资源打包最好用BuildAssetBundleExplicitAssetNames接口，原先使用BuildAssetBundle出现过很多问题）

2.WWW资源url加载AssetBundle时会遇到版本资源更新的问题。U3D自4.0后就不再有删除某缓存资源的接口，所以资源更新时遇到麻烦。

解决办法： 

<del>1.客户端记录由服务器发过来的MD5码，倘若与本地存储的MD5码不同则增加自动增加一个版本号。这个办法方便但会积累很多的旧的无用的资源占用手机，导致用户有可能因为游戏占用太多硬盘资源而删除游戏。 2.客户端记录由服务器发过来的CRC校验码，用CRC校验码来判定当前资源是否需要重新加载，调用接口为static function LoadFromCacheOrDownload(url: string, version: int, crc: uint = 0): WWW; 这个办法可以解决旧资源删除问题，但最主要的问题是资源在打包后由于U3D的资源CRC校验码并不是普通的校验码，所以只能根据他的接口来判定，4.2以及4.2以前都没有获取CRC校验码的接口，所以每次打包资源后，都需要校对校验码。如何校对CRC校验码请自己去搜索资料。</del>

2015-09-28: 我这里做一下修正，其实这个问题就是assetbundle版本更新解决方案。答案可以在这里找到[Unity3D之如何将包大小减少到极致](/unity3d/游戏架构/前端技术/2014/06/06/Unity3D之如何将包大小减少到极致.html) 

{% include advertisement_content.html %}

3.如何知道哪些资源包需要加载以及加载地址和版本号。

<del>解决办法： 1.使用XML，将所有需要加载的资源信息写入XML文件。每次游戏开始，先读取该文件。 2.由服务器的逻辑服务器提供需要加载的资源信息。</del>

在那1年后的我写了一个关于资源更新的解决方案[https://github.com/luzexi/version_update](https://github.com/luzexi/version_update) 是一个用php写的脚本，将资源进行记录，每次生成版本，对比资源是否有被修改，或删除，或增加，有兴趣可以看看。

转载请注明出处：http://www.luzexi.com
