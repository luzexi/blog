---
layout: post
status: publish
published: true
title: Unity3D-重新编译Mono加密DLL
description: "unity3d mono 加密dll"
author:
  display_name: 陆泽西
  login: luzexi
  email: jesse_luzexi@163.com
  url: http://www.luzexi.com
author_login: luzexi
author_email: jesse_luzexi@163.com
author_url: http://www.luzexi.com
wordpress_id: 565
wordpress_url: http://www.luzexi.com/?p=565
date: !binary |-
  MjAxNS0wNC0xMSAxODo1ODozNiArMDgwMA==
date_gmt: !binary |-
  MjAxNS0wNC0xMSAxMDo1ODozNiArMDgwMA==
categories:
- Unity3D
- 游戏架构
- 前端技术
tags:
- Unity3D
- 加密
---
Unity3D-重新编译Mono加密DLL。安卓应用总是让人头疼，游戏遭到破解与反编译是研发的人最不愿意看到的。自己的辛苦劳动成果被人随意窃取与利用，对这些咬牙切齿的痛恨。所以我们需要加强自身的反破解技术力量。不过这世上没有破解不了的东西，道高一尺魔高一丈，我们做的只是让破解更加困难而已。让那些破解的人付出点代价才能得到他们想要的，如果他们觉得代价太高，看不清前面的道路，他们就有可能放弃，然后我们的目的达到了。

游戏本身加密方式有很多，对apk加壳，防止apk二次打包等。对这些android的加密与破解技术看过比较好的文章参考：

[《Android安全及病毒分析》](http://blog.csdn.net/column/details/security-android.html) ，其中[《Android APK加壳技术方案【2】》](http://blog.csdn.net/androidsecurity/article/details/8809542) 最为经典。而本篇文章我们主要来说说针对Unity3D的加密。

闲扯就到这里，我们开始说正事：

Unity3D所有客户端的代码都会以dll文件形式存下来，当游戏应用被开启时c#vm(也就是mono的虚拟机)会去加载所有dll，从而开始运行真正的程序画面了。而破解的很大一部分都是通过解压apk后拿到主逻辑dll，对dll进行反编译，然后修改后重新编译，再放入apk重新签名打包。所以我们需要针对dll进行加密，以防止他们反编译dll。

加密一个dll文件非常容易，无论你用什么算法都行，但是在哪解密呢？答案是libmono.so。libmono.so是mono的核心程序，它承载了加载解析dll和虚拟机运行的功能。所以说libmono.so是关键，我们需要修改mono内核程序并重新编译它。

下面将开始mono的编译过程，别看步骤写得简单明了，其实我花了起码一个多星期的思考，尝试，失败，再思考，再尝试，再失败.....总结其中原因一方面也是自己的愚钝的资质，另一方面是unity mono和mono并不一样，unity mono缺少编译文档并且还混合着原mono的编译文档，导致误判了很多：

1.首先不要认为unity mono 与 原生态mono一样。可以编译mono就可以同样步骤编译unity mono。我在这里尝试了很久，使用configure进行编译，尝试使用不同的编译参数，进行编译，最后发现unity mono使用的是ndk-9下的linux-4.8编译器，所有参数都是根据这个编译器所设定的。

2.unity mono 地址：[https://github.com/Unity-Technologies/mono](https://github.com/Unity-Technologies/mono) 你需要从这里下载unity mono。

3.mono需要autoconf automake libtool pkg-config这些工具。你最好还是去下载安装了。你可以用brew安装。brew install autoconf automake libtool pkg-config。

4.我一开始使用mac x86_64进行编译，折腾了很久然后建了个linux-x86_64虚拟机来编译，然后又折腾了很久，又建了个linux-i386来重新编译mono，因为我一直认为交叉编译需要加些不同的编译参数和变量。在linux-i386首次编译成功后又开始转化到mac上，进行交叉编译也一样成功，最后发现其实是我没找对路子。这路子就是unity 的mono-build-tool：[https://github.com/Unity-Technologies/monobuildtools](https://github.com/Unity-Technologies/monobuildtools) 它已经在unity mono的项目里了，在mono的external/buildscripts下。

5.buildscripts下的build_runtime_android.sh是编译安卓平台的关键。它是unity制作的一个自动编译 mono 流程的脚本。你需要将这个脚本copy到mono根目录下再执行。

6.脚本里写些内容，如果你懒得看，我帮你稍微解释下。它会去检查你当前的ANDROID_NDK_ROOT环境变量是否是指向ndk-9，所以你需要去下ndk-9版本，放到机子上，然后编辑环境变量ANDROID_NDK_ROOT指向它，如果你没有它会通过perl模块lwp-download去下载ndk-9，但是你必须要要有这个perl模块才行，我劝你还是老老实实自己去下吧。ndk版本下载地址参考这里：[《android-sdk-ndk-studio-下载列表和构建说明》](/前端技术/2015/04/06/Android-SDK-NDK-Studio-下载列表和构建说明.html)。如果是linux下编译环境变量设定参考这里：[《linux环境变量简介》](/后端技术/2015/04/07/linux环境变量简介.html)。然后呢，它会用git去clone一个编译时用到的库，这个也是unity自己改编过的一个库，地址为：[https://github.com/Unity-Technologies/krait-signal-handler](https://github.com/Unity-Technologies/krait-signal-handler) ，这个库有个坑说下：perl脚本build.pl头部有个命令是#!/usr/bin/env perl -w，这个在部分机子上并不兼容，如果你有错误停在这里这个文件上，你可以将env去除再尝试手动perl build.pl 运行构建一遍没问题再重新编译，原因参考：[http://abloz.com/2011/01/13/why-use-usr-bin-env.html](http://abloz.com/2011/01/13/why-use-usr-bin-env.html) 。最后就先make clean && make distclean 清除前面编译的内容，然后进行预编译configure，参数都在脚本里设置好了，你不需要关心了。预编译后就开始make编译了。

7.执行build_runtime_android.sh后terminal基本都是刷屏的节奏。刷刷刷的编译输出，你根本来不及看清到底做到哪了做了些什么内容。而config.log这个文件记录所有的编译输出，包括哪行错误了，哪行通过了。调试基本也考这个log文件，如果关键部位错误它会停止，然后你就可以针对性的查了。这里提醒一点，编译时它很多地方都是在检测编译器是否正常，因为它要确认编译器对错误的编译内容是否能够检测到，所以很多错误内容只是测试内容-你需要省略掉。

8.如果编译成功，那就恭喜你了。windows下我没有测试过，有可能会增加不少坑，我建议还是用linux或者mac编译吧，因为我搜集资料的时候不少人对windows下编译mono都抱怨不少。那么我们开始迈入下一个坑吧:)

下面介绍加解密DLL部分：

加密算法自己选我不多说了，但我这里要引用一篇同样介绍mono的dll加密的文章，我觉得也写得满不错的，但是文章描述不够详尽。我这篇文章弥补了他的不足，将细节补充得更加细致。你大可以两篇文章加起来参考。[http://www.unitymanual.com/home.php?mod=space&uid=7672&do=blog&id=1440](http://www.unitymanual.com/home.php?mod=space&uid=7672&do=blog&id=1440) 不知道地址是不是原作者的，如果不是我再更换吧。

{% include advertisement_content.html %}

1.首先找到dll解密入口。mono下/mono/metadata/image.c里mono_image_open_from_data_with_name是关键方法，参数中的data是dll传入的数据。你要做的就是将它解密后传给datac，这个方法程序你必须看下，因为你要了解下解密程序放在哪才合适。

2.大部分dll都会通过mono_image_open_from_data_with_name这个方法进行加载，但不是所有dll，例如mscorlib.dll和System.Core.dll就不会，可能还有其他dll，我并不确定还有哪些。所以你还是得辨别下哪些dll会通过这个方法，这样你才能确定哪个dll可以加密。如何判断data属于哪个dll呢，参数name就是data的路径名，name打印出来后就像:/data/app/com.xx.xx.apk/assets/bin/Data/Managed/xxx.dll 这样。

3.打印调试。你可以使用g_message例如：g_warning("dll name: %s \n", name); 其他的打印调试你可以查看源码中的它写的代码。很容易找到，查关键字LOG吧。

4.改完后重新编译mono，找到libmono.so(find . -name libmono.so)，完成编译后libmono.so的平台有好几个，你可以根据自己的平台来选。有人拷贝这些mono重新编译过的文件去覆盖了unity编辑器的原来mono文件，这样也可行。但我选择在打包android时再从外部复制libmono.so，这样就可以绕过编译器重新编译后无法读取无加密dll的麻烦，可以少做一层无意义的编辑器状态下的加解密工作。

5.mono解密部分就到这里了。其他部分的关键就是你的加解密程序了，是否能够加密和解密都是ok的并且都是不改变size。你需要的参数有data和data_len，mono_image_open_from_data_with_name方法里面都有。

6.为了安全起见我使用c来编写加密程序，因为我认为c#和c的编译器对于变量内存的存储机制不一样，怕引起不必要的麻烦。
这里要非常感谢一个人，全程都在提供帮助：炽乐@宗树

转载请注明出处：http://www.luzexi.com
