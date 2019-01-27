---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(五)
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层算法"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

#第一章，C\#要点技术(五) 排序和搜索算法

算法很重要，在程序生涯中，算法是基础能力，很多时候程序的好坏，一方面看的是写程序的经验，另一方面看的是对计算机原理的理解程度，还有一方面看的是对算法的运用到如何程度。

某一处的算法有可能运用的很好，但如果其他地方都用烂算法，对于整体程序效率来说其实也没什么用，效率依然很烂。

在平时的编程时，做到时刻关注算法的效率是区分中、高水平的一个大的关键点。

其中排序和搜索算法最为常用。毫不夸张的说一个项目中有90%的算法都是排序和搜索算法，如果说我们把这90%的算法提高到一个很高效的程度，那么剩下的10%算法其实也没那么重要了。

尽管精益求精是我们写程序人的宗旨，但是假设我们能更快速度摆平90%的算法，那么剩下10%的算法优化已经只是时间问题了。

### 快速排序

快速排序，也叫二分排序，是最最最常用，最最最好用，用的最最最多的排序算法。它的排序方式是：

1.从序列中选一个元素作为基准元素
2.把比基准元素小的元素移到基准元素的左边，把基准元素大的移到右边。
3.对分开来的二个一大一小区块进行递归再筛选，对两个区块同样进行1、2的两个步骤处理。

优化快速排序：
1.如果区间内的所有数字都是相等的，那么岂不是白白排序了么，浪费时间。
2.如果选中的数字不是中位数，那么排序的速度就会降低，比如选中的刚好是最大的或者最小的数字。
https://www.cnblogs.com/vipchenwei/p/7460293.html


### 归并排序
http://www.cnblogs.com/eniac12/p/5329396.html

### 堆排序
http://www.cnblogs.com/eniac12/p/5329396.html
https://blog.csdn.net/coolwriter/article/details/78732728

### 拓扑排序
https://blog.csdn.net/dylan_frank/article/details/52292518

### 非常用排序，计数排序，基数排序，桶排序
https://blog.csdn.net/coolwriter/article/details/78732728

### 广度优先搜索
https://blog.csdn.net/dylan_frank/article/details/52292518

### 深度优先搜索
https://blog.csdn.net/dylan_frank/article/details/52292518

### Dijkstra最短路径算法
http://www.cnblogs.com/biyeymyhjob/archive/2012/07/31/2615833.html

### A星非最优最短路径算法
https://blog.csdn.net/qq_21120027/article/details/52949157

### 树状数组
https://blog.csdn.net/qq_21120027/article/details/50585002

### KMP字符串匹配算法
https://blog.csdn.net/qq_21120027/article/details/50911370

### 凹行包边算法




ugui 字体不能叠加，否则无法合并mesh，因为半透明。

		两个面片，中间叠加半透明面片也是一样的道理。

		因为半透明无法与不透明物体合并


Dictionary 的改进

Heap，二叉最大最小堆

多叉平衡树，B tree，B树

多叉最大最小堆，，Fibonacci heap，斐波纳契堆

递归

字符串匹配

计算几何学，线段相交，多边形合并，裁切，凸包，两矢量拐向判断，


语言相关类：

垃圾回收：概念与策略

委托与事件的实质是类实例 done

拆箱和装箱原理 done

多态virtual的原理
