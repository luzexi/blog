---
layout: post
status: publish
published: false
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(四)
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层算法"
excerpt_separator: ===
tags:
- 书籍著作
- Unity3D
- 前端技术
---

### 委托(delegate)与事件(Event)的实质

使用过C或C++的同学都对指针很清楚，指针是个需要谨慎对待的东西，它不仅仅可以指向变量的地址，还可以指向函数的地址。

在更高级的语言中，为了防止程序直接去处理内存操作分配，赋值和销毁引起对内存的破坏，已经没有指针了，只有变量，最多也是引用，虽然跟指针很像，但完全不一样。

在.NET中，在大部分时间里都没有指针的身影，因为指针被封闭在内部函数当中。可是回调函数却依然存在，它是以委托的方式来完成的。委托可以被视为一个更高级的指针，它不仅仅能把地址指向另一个函数，而且还能传递参数，返回值等多个信息。系统还为委托对象自动生成了同步、异步的调用方式，开发人员使用 BeginInvoke、EndInvoke 方法就可以抛开 Thread 而直接使用多线程调用。

创建委托其实就是创建了一个delegate类实例，这个类继承了System.MulticastDelegate类，类实例里有，BeginInvoke、EndInvoke、Invoke三个函数，分别表示，异步开始调用，结束调用，直接调用。

不过我们不能直接写个类来继承System.MulticastDelegate类，因为它不能被继承在明文上，它的父类Delegate类也同样有这个规则，官方文档中就是这么个规则：

		MulticastDelegate is a special class. Compilers and other tools can derive from this class, but you cannot derive from it explicitly. The same is true of the Delegate class.

delegate实例中其实有个变量用来存储函数地址，当变量操作 =(等号) 时，把函数地址赋值给变量存起来。其实这个存储函数地址的变量是个可变数组，你可以认为是个链表，每次直接赋值时会换一个链表。

委托类还重写了 +=，-= 操作符，其实就是对应 MulticastDelegate 的 Combine 和 Remove 方法，当对函数操作 += 和 -= 时，相当于把函数地址推入了链表尾部，和移出了链表。

当委托被调用时，委托实例会把所有链表里的函数依次按顺序用传进来的参数调用一遍。官方文档中就如上述所说：

	A MulticastDelegate has a linked list of delegates, called an invocation list, consisting of one or more elements. When a multicast delegate is invoked, the delegates in the invocation list are called synchronously in the order in which they appear. If an error occurs during execution of the list then an exception is thrown.

所以说，delegate关键字其实只是个修饰用的单词，背后都是由C#编译器来重写代码的，就相当于编译时把那一句换掉，变成继承System.MulticastDelegate的类，并带入参数。

那么什么是event？

event 很简单，它在委托delegate上，又做了一次封装，这次封装的意义是，限制用户直接操作委托变量的权限。

封装后，用户不再能够直接用赋值(=等号操作符)操作来改变委托变量了，只能通过增加或者减少委托的方法来改变委托变量。也就是不再提供‘=’的操作符，只有 += 和 -= 的操作符可供操作。

为什么要限制？因为公开的委托会直接暴露在外，随时被‘=’赋值而清空了前面累加起来的委托链表，造成不可预测的问题。加了 event 后，暴露在外面的委托不再担心随时被清空和重置的危险了，因为经过 event 封装后不再提供赋值操作，只能一个个增加或者一个个减少委托(或者说函数地址)。

### 开箱和装箱




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

委托与事件的实质是类实例

开箱和装箱原理

多态virtual的原理

