---
layout: post
status: publish
published: true
title: Unity3D之slua集成第三方库
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 602
wordpress_url: http://www.luzexi.com/?p=602
date: !binary |-
  MjAxNS0wOS0yNiAxNDo0NjoyNiArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wOS0yNiAwNjo0NjoyNiArMDgwMA==
categories:
- Unity3D
- 游戏通用模块
- lua
tags:
- Unity3D
- Unity3D架构
- 客户端架构
- 游戏架构
- lua
---

Unity3D中使用lua最近越来越火，我比较中意slua的思路与代码质量。因为先前的项目对slua做了几个第三方库的封装，所有在空出来的时间就对slua做了fork加入了一些大家都比较常用的第三方库。

[源码](https://github.com/luzexi/slua-3rd-lib)

这面是源码的地址，如果喜欢可以star或watch，我会一直更新以方便大家。

至今集成的第三方库罗列一下:

1.pbc ([https://github.com/cloudwu/pbc](https://github.com/cloudwu/pbc)) 是云风用c写的google protocol buffers。

2.lpeg ([http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html](http://www.inf.puc-rio.br/~roberto/lpeg/lpeg.html)) 是正则表达式的解析与匹配库.

3.lua-cjson ([http://www.kyne.com.au/~mark/software/lua-cjson.php](http://www.kyne.com.au/~mark/software/lua-cjson.php)) 是一个支持json的库，他的效率非常惊人。

4.lua-socket ([http://w3.impa.br/~diego/software/luasocket/home.html](http://w3.impa.br/~diego/software/luasocket/home.html)) 是一个用c写的socket的库，里面封装了TCP和UDP。提供了网络连接与传输的API，你可以用它在lua里实现网络层。

5.sproto ([https://github.com/cloudwu/sproto](https://github.com/cloudwu/sproto)) 这也是云风写的一个关于网络协议，他类似与google protocol buffer，不同的是他在google protocol buffer基础上对协议的格式和内容做了修改，使得解析与构造的效率非常高。

6.sqlite ([https://github.com/LuaDist/lsqlite3](https://github.com/LuaDist/lsqlite3)) 是一个以文件形式存在的轻量级数据库，他被很多软件用来做本地数据存储。他的api轻便好用，广受程序员欢迎。

源码中已将所有第三方库构建进slua里，并完成了android(x86,armv7),ios,mac,windows(x86,x64)这几个平台的测试。

如果你需要加入自己的第三方库或者说你希望去除一些你用不到的第三方库，你可以在源码的build里修改我写的自动build的批处理程序。

1.make_ios.sh 构建ios平台的批处理程序。需要在mac下运行。

2.make_osx.sh 构建osx平台的批处理程序。需要在mac下运行。

3.make_android.sh 构建android平台的批处理程序。需要在mac或linux下运行。

4.make-windows-32.cmd 构建windows x86 dll。需要在windows下运行。

5.make-windows-64.cmd 构建windows x64 dll。需要在windows下运行。

这里说明一些构建时需要注意的事情。ios构建时需要注意脚本里的xcode地址。如果你不是xcode7.0。可能需要设置一下脚本里的路径。android的构建脚本也是一样，需要配置你机子上正确的ndk路径。windows下编译dll需要用的MingGw，我在源码中用git的submodule的方式安了一个，你需要运行一下submodule的更新命令git submodule update --init。MingGw很大300mb，你要做好心理准备，或者你直接去这个地址下载一个[https://github.com/luzexi/MinGW](https://github.com/luzexi/MinGW)
