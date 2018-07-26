---
layout: post
status: publish
published: true
title: 重写Jekyll的Relate功能
description: "jekyll liquid ruby related"
excerpt_separator: ===
tags:
- 前端技术
- 其他技术
---

Jekyll 的 ‘Relate-相关文章’的功能，写的真的不好用。完全表达不了相关文章的含义。

于是打算修一下 Relate 部分的功能。

网上查了很久，很多人在抱怨，但没人把写好的 Relate 放到网上。

唯一一个Jekyll 的 Relate 插件都是渣的要命的那种，根本没法用。

===

于是就有了重写这个功能的欲望，本想也简单，只是几个for循环而已。

原以为是 Jekyll 是 Ruby 写的，网页上也是 Ruby，还复习了下 Ruby 写法。以前写过很久没用就会忘记。

谁能知道，网页部分不是Ruby语法呢，原来是 Liquid 的语法，他时用 ruby 写的‘模板引擎’。

[Liquid源码](https://github.com/Shopify/liquid)

查了下他的用法，API其实还真的不多，用起来好难受。比如 for循环 就是个要命的点。还有变量申明和运算，和平常用的语言相差有点大的。

### 关于Relate功能，我希望是，有最新的文章链接，之前的文章链接和之后的文章链接，让读者能有更大的概率找到自己想要的文章。

写完后的效果如图：

![relate功能图](/assets/uploads/2018/07/jekyll-relate.png)


源码如下:

[Jekyll Relate github](https://github.com/luzexi/jekyll-relate)

