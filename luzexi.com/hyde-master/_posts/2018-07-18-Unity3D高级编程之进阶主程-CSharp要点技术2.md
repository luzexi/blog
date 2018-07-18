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

Dictionary字典型数据结构，最关键还是Key是如何映射到内存的，每个Key都要进行一次Hash哈希的运算操作，从而找到自己的位置。

对于不同的关键字可能得到同一哈希地址，即key1 != key2 => F(key1)=F(fey2)，这种现象叫做冲突，在一般情况下，冲突只能尽可能的少，而不能完全避免。因为，哈希函数是从关键字集合到地址集合的映像。通常，关键字集合比较大，它的元素包括多有可能的关键字。既然如此，那么，如何处理冲突则是构造哈希表不可缺少的一个方面。

通常用于处理冲突的方法有：开放定址法、再哈希法、链地址法、建立一个公共溢出区等。

在哈希表上进行查找的过程和哈希造表的过程基本一致。给定Key值，根据造表时设定的哈希函数求得哈希地址，若表中此位置没有记录，则查找不成功；否则比较关键字，若何给定值相等，则查找成功；否则根据处理冲突的方法寻找“下一地址”，直到哈希表中某个位置为空或者表中所填记录的关键字等于给定值时为止。

Dictionary使用的解决冲突方法是拉链法，又称链地址法。

拉链法的原理：将所有关键字为同义词的结点链接在同一个单链表中。若选定的散列表长度为m，则可将散列表定义为一个由m个头指针组成的指针数 组T[0..m-1]。凡是散列地址为i的结点，均插入到以T[i]为头指针的单链表中。T中各分量的初值均应为空指针。

结构图大致如下：


首先我们来看看源码对Dictionary的定义部分，如下:

{% highlight c# %}

public class Dictionary<TKey,TValue>: IDictionary<TKey,TValue>, IDictionary, IReadOnlyDictionary<TKey, TValue>, ISerializable, IDeserializationCallback 
{
    
	private struct Entry {
	    public int hashCode;    // Lower 31 bits of hash code, -1 if unused
	    public int next;        // Index of next entry, -1 if last
	    public TKey key;           // Key of entry
	    public TValue value;         // Value of entry
	}

	private int[] buckets;
	private Entry[] entries;
	private int count;
	private int version;
	private int freeList;
	private int freeCount;
	private IEqualityComparer<TKey> comparer;
	private KeyCollection keys;
	private ValueCollection values;
	private Object _syncRoot;
}

{% endhighlight %}

从继承的类和接口看，Dictionary主要继承了IDictionary接口，ISerializable接口。在使用过程中，主要的接口为，Add, Remove, ContainsKey, Clear, TryGetValue, Keys, Values, 以及[]数组符号形式的接口。其他也包括了常用的Collection中，Count, Contains等。下面的文章中，我们将围绕上述的接口进行解析 Dictionary 底层运作机制。

从定义变量中可以看出，Dictionary是由数组为底层数据结构的类。也就是说，当new Dictionary()后，内部的数组是0个数组的状态。


[Dictionary源码](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs)

### Queue 底层代码

### Stack 底层代码