---
layout: post
status: publish
published: true
title: SSR 安装配置，以及 Mac 下使用 P-s-i-phon
description: "mac V.P.N P-s-i-phon SSR 安装配置"
excerpt_separator: ===
tags:
- 后端技术
- 其他技术
---

###### 文内所有敏感词语都做了处理，请自行脑补。

最近国内封锁 V.P.N，阿里云这个畜生把我的服务器给封了访问外网的权限。原本布置在香港服务器上SSR，直接给废了。

于是我疯狂得找渠道翻过这道墙，毕竟程序员需要向全世界学习的啊。把V.P.N封了，让程序员怎么活。

一开始我想把服务器迁到国外去，但是国外的网络和国内连的真的不好，一个ssh卡的要死，速度还是慢。

何况我的个人博客在国内，以后还想在国内做些自己的idea，所以需要服务器连接速度在国内更快一些，就放弃了搬出去的念头，转而专心搞如何躲猫猫。

这里来讲讲我的心路历程和经验，SSR 安装配置，以及 Mac 下使用 P-s-i-phon的经验和说明。

===

一开始用的是 SSR，这里顺便描述下 SSR 在Linux 上的安装和配置。

1， 需要Linux服务器安装 Python，最好是Python3。

2， 再从网上下一份SSR源码，使用 manyuser分支 [github地址](https://github.com/shadowsocksr-backup/shadowsocksr/tree/manyuser) [ssr备用网盘连接](https://pan.baidu.com/s/1_oc9aKwNF8Kp2UaDdAdUWA)

3， 解压后，将 shadowsocksr-manyuser/debian/config.json 考一份到指定的目录，比如 shadowsocksr-manyuser 文件夹。

4， 更改config.json，配置，密码，加密方式，协议，混淆方式等。

		经过我的调查推荐使用，aes-128-ctr 作为加密方式，auth_sha1_v4 作为协议方式， tls1.2_ticket_auth 作为混淆方式

		其他还需要你填写，端口，密码。

{% highlight json %}
{
    "server":"0.0.0.0",
    "port_password":
    {
        "端口":"密码"
    },
    "local_address": "127.0.0.1",
    "local_port":3331,
    "timeout":300,
    "fast_open": false,
    "workers": 1,
    "protocol": "auth_sha1_v4",
    "protocol_param": "",
    "method":"aes-128-ctr",
    "obfs": "tls1.2_ticket_auth",
    "obfs_param": "www.bing.com"
}

{% endhighlight %}

5， 完成配置后，可以运行了。记得把服务器的端口在安全组里开起来，否则外网访问不了端口。

		python shadowsocksr-manyuser/shadowsocks/server.py -c config.json

这是调试模式，所有的log日志都打印在终端上，你可以用客户端去访问下试试。

想启用后台模式可以使用以下命令：

		python shadowsocksr-manyuser/shadowsocks/server.py -c config.json -d start

关闭命令为：

		python shadowsocksr-manyuser/shadowsocks/server.py -d stop

6，客户端安装。下载客户端软件。[mac版-网盘](https://pan.baidu.com/s/1QckyHag4QK8X7AP74Yci5Q) [win版-网盘](https://pan.baidu.com/s/1AnZ6lKawhAbHqXBS9_evFg)

然后新建服务器，填上你的服务器地址，端口，密码，加密方式，协议方式，混淆方式等。如图所示

![服务器参数](/assets/uploads/2018/08/vpn1.png)

7， 规则配置，规则让你在访问国内外的网站时使用不同的网络，国内使用直连，国外使用SSR。

配置的地址可以参考 [https://github.com/ACL4SSR/ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)

客户端，服务器到这里全都完成了，开始ssr试试吧。

{% include advertisement_content.html %}

- - -

### 下面我讲讲 P-s-i-phon 在 Mac 上的使用。

P-s-i-phon 是一个开源的项目，致力于无国家网络的组织，其背后的力量到底是什么不得而知，我只知道他允许你免费使用V.P.N。

不过它会搜集你的上网资料哦，具体用在什么地方，会用来做什么就不得而知。所以我建议不要使用的太过于频繁，或者不用使用在敏感位置，适度使用我觉得没什么问题。

P-s-i-phon 它有win版本，ios版本(海外的appstore)，android版本，就是没有Mac版本。

但幸好它是开源的，可以用它的核心代码在Mac上布置一下，然后用代理的方式用它来访问’外网‘。

###### 开门见山写具体步骤：

1， psiphon是golang编写的，所以需要go语言。下载并安装go语言 [go语言mac版](https://pan.baidu.com/s/1Xra482GlKBgfiPpu5WKpYw) 

2， 从GitHub上下载psiphon源码

		git clone https://github.com/Psiphon-Labs/psiphon-tunnel-core

		或 (P-s-i-phon-网盘)(https://pan.baidu.com/s/14vQZfLwHP3nlV5uUeGjUgA)

3， 编译P-s-i-phon，因为在Mac下会有编译bug，所以我们需要修改一个行文件。

		修改make.bash文件里

		BUILDDATE=$(date --iso-8601=seconds)

		改为

		BUILDDATE=$(date +%Y-%m-%dT%H:%M:%S%z)

		然后执行

		./make.bash osx

4， 编译时，它需要git一些组件，大概10分钟左右的时间。编译完成后，进入当前目录的bin/darwin/文件夹。

新建一个文件，命名为 config.json ，然后将一下配置放入文件内。

{% highlight json %}

{
"LocalHttpProxyPort":8081,
"LocalSocksProxyPort":1081,
"PropagationChannelId":"FFFFFFFFFFFFFFFF",
"RemoteServerListDownloadFilename":"remote_server_list",
"RemoteServerListSignaturePublicKey":"MIICIDANBgkqhkiG9w0BAQEFAAOCAg0AMIICCAKCAgEAt7Ls+/39r+T6zNW7GiVpJfzq/xvL9SBH5rIFnk0RXYEYavax3WS6HOD35eTAqn8AniOwiH+DOkvgSKF2caqk/y1dfq47Pdymtwzp9ikpB1C5OfAysXzBiwVJlCdajBKvBZDerV1cMvRzCKvKwRmvDmHgphQQ7WfXIGbRbmmk6opMBh3roE42KcotLFtqp0RRwLtcBRNtCdsrVsjiI1Lqz/lH+T61sGjSjQ3CHMuZYSQJZo/KrvzgQXpkaCTdbObxHqb6/+i1qaVOfEsvjoiyzTxJADvSytVtcTjijhPEV6XskJVHE1Zgl+7rATr/pDQkw6DPCNBS1+Y6fy7GstZALQXwEDN/qhQI9kWkHijT8ns+i1vGg00Mk/6J75arLhqcodWsdeG/M/moWgqQAnlZAGVtJI1OgeF5fsPpXu4kctOfuZlGjVZXQNW34aOzm8r8S0eVZitPlbhcPiR4gT/aSMz/wd8lZlzZYsje/Jr8u/YtlwjjreZrGRmG8KMOzukV3lLmMppXFMvl4bxv6YFEmIuTsOhbLTwFgh7KYNjodLj/LsqRVfwz31PgWQFTEPICV7GCvgVlPRxnofqKSjgTWI4mxDhBpVcATvaoBl1L/6WLbFvBsoAUBItWwctO2xalKxF5szhGm8lccoc5MZr8kfE0uxMgsxz4er68iCID+rsCAQM=",
"RemoteServerListUrl":"https://s3.amazonaws.com//psiphon/web/mjr4-p23r-puwl/server_list_compressed",
"SponsorId":"FFFFFFFFFFFFFFFF",
"UseIndistinguishableTLS":true
}

{% endhighlight %}

这里意思是http代理监听端口为8081，socks5监听端口为1081，可以自行修改。

然后就可以启动 P-s-i-phon 了，启动命令如下：

		./psiphon-tunnel-core-x86_64 -config ./config.json

5， 安装客户端代理软件，我谷歌浏览器，所以使用 Proxy SwitchyOmega 做代理。

如果不是谷歌浏览器，完全可以使用其他插件，或者直接添加代理，设置为

		http(s): 127.0.0.1:8081

		或者

		SOCKS5: 127.0.0.1:1081

6， 配置 Proxy SwitchyOmega 的自动切换规则。

使用 [配置 Proxy SwitchyOmega 的自动切换规则](https://github.com/ACL4SSR/ACL4SSR/blob/master/SwitchyOmega/%E8%A7%84%E5%88%99%E5%88%97%E8%A1%A8%E8%AE%BE%E7%BD%AE.txt) 

配置如下图：

![自动切换规则](/assets/uploads/2018/08/vpn2.png)

7， 完成后，就可以在谷歌浏览器右上角开启auto switch(自动切换路线)来访问’内外网‘。

###### 注意，记得要把 P-s-i-phon 也开着，因为 P-s-i-phon 是主核心程序。每次启动都是随机连接某个地区的服务器。



