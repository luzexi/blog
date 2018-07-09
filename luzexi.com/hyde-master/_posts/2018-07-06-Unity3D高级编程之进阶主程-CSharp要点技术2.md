---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(二)
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层算法"
excerpt_separator: ===
categories:
- 原创著作
- Unity3D
- 前端技术
tags:
- unity3d
- c#
---

#第一章，C\#要点技术(二)

前文回顾 [《Unity3D高级编程之进阶主程》第一章，C#要点技术(一)](http://www.luzexi.com/%E7%89%88%E6%9D%83%E8%91%97%E4%BD%9C/unity3d/%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF/2018/07/06/Unity3D%E9%AB%98%E7%BA%A7%E7%BC%96%E7%A8%8B%E4%B9%8B%E8%BF%9B%E9%98%B6%E4%B8%BB%E7%A8%8B-%E7%AC%AC%E4%B8%80%E7%AB%A0-C-%E8%A6%81%E7%82%B9%E6%8A%80%E6%9C%AF(%E4%B8%80).html)

前文剖析了 List 的源码，我们明白了 List 是用数组构建而成的，增加，减少，插入的操作，都在数组中进行。我们还分析了大部分 List 的接口，包括Add，Remove，Insert，IndexOf，Find，Sort，ToArray，等等。我们还得出了一个结论，那就是 List 是一个兼容性比较好的组件，但线程并不安全，需要加锁机制来保证线程的安全性，而且 List 在效率方面并没有做优化。

这次我们来对常用的另一个组件Dictionary进行底层源码的分析，看看我们常用的字典树是如何构造而成的，它的优缺点如何。

===

### Dictionary 底层代码




[Dictionary源码](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs)

### Queue 底层代码

### Stack 底层代码