---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(六) 搜索
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层算法"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### 搜索算法

广度优先搜索和深度优先搜索是最直接的搜索方式，但也效率最差，直接使用广度和深度算法，而不加以优化和改进，会浪费太多CPU，以至于整体效率差劲。

好的搜索算法，都会有数据结构支撑，在数据结构里记录了过去搜索的记录，和当前搜索内容的环境，以达到裁剪优化的目的。

搜索的环境一般有，一个数组中找出某元素，一个2D或3D空间中找出某元素，2D或3D空间中找出某一个范围的所有元素，以及一堆相互连接的结构中找出两点的最短路径。

###### 二分查找算法

二分查找算法是搜索中用的最多，也是最好用的，效率最高的一种。不过它有个前提，前提是查找的数组必须是有序的。

也就是说，在使用二分算法查找前，必须对数组进行排序。而且在每次更改，插入，删除后都要进行排序以保证数组的有序状态。

二分查找算法的步骤是：

1，将数组分为三块，依次是前半部分，中间位，后半部分。

2，将要查找的值和数组的中间位进行比较，若小于中间位则在前半部分查找，若大于中间位则在后半部分查找，等于中值时直接返回。

3，下一个查找范围，跳入1步骤，依次进行递归过程，将当前的范围继续拆分成前半部分，后半部分，和中间位三部分，直到范围缩小到最小，如果还是没有找到，那说明元素并不在数组里面。



###### 四叉树搜索算法

###### 八叉树搜索算法

###### 九宫格裁剪

###### Dijkstra最短路径算法

###### A星算法



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