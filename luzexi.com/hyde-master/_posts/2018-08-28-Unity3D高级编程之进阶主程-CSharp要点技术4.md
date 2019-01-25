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

封装后，用户不再能够直接用赋值(=等号操作符)操作来改变委托变量了，只能通过注册或者注销委托的方法来改变委托变量。也就是说被event声明的委托不再提供‘=’的操作符，但仍然有 += 和 -= 的操作符可供操作。

为什么要限制？

因为公开的委托会直接暴露在外，随时会被‘=’赋值而清空了前面累积起来的委托链表，造成不可预测的问题。申明是 event 后，编译器内部重新封装委托，让暴露在外面的委托不再担心随时被清空和重置的危险了，因为经过 event 封装后不再提供赋值操作，只能一个个注册或者一个个注销委托(或者说函数地址)。

https://www.cnblogs.com/dabiaoge/p/4112581.html
### 装箱和开箱

什么是装箱和开箱。

很简单，把值类型实例转换为引用类型实例，就是装箱。

相反，把引用类型实例转换为值类型实例，就是开箱。

再简单点：

int a = 5;

object obj = a;

就是装箱

继续上面代码

a = (int)obj;

就是开箱。

###### 为何需要装箱？

值类型是在栈中分配内存，在声明时初始化才能使用，不能为null。 

引用类型在堆中分配内存，初始化时默认为null。 

值类型有，所有整数，浮点数，bool，以及struct申明的结构

引用类型有，类，接口，委托(委托也是类)，数组以及内置的object与string。

这里又引申出来，为什么要分栈内存和堆内存，简单快速阐述下：

		栈是本着先进后出的数据结构(LIFO)原则的存储机制, 对栈数据的定位比较快速, 而堆则是随机分配的空间, 处理的数据比较多, 无论如何, 至少要两次定位。堆内存的创建和删除节点的时间复杂度是O(logn)。栈创建和删除的时间复杂度则是O(1)，速度更快。

		既然栈速度这么快，全部用栈不就好了。这又涉及到生命周期问题，由于栈中的生命周期是必须确定的，创建后什么时候销毁是一个定量，所以在分配和销毁时不灵活，相反堆内存可以存放生命周期不确定的内存块，满足当需要删除时再删除的需求，所以堆内存相对于引用类型的内存块更适合。

因此在当栈内存和堆内存相互转换时，有了装箱和开箱的过程。大部分时候只有当程序、逻辑或接口需要更加通用的时候才会需要装箱。

比如调用一个含类型为Object的参数的方法，该Object可支持任意为型，以便通用。当你需要将一个值类型(如Int32)传入时，需要装箱。

又比如一个非泛型的容器，同样是为了保证通用，而将元素类型定义为Object。于是，要将值类型数据加入容器时，需要装箱。

我们来看看装箱的内部操作。

装箱： 对值类型在堆中分配一个对象实例，并将该值复制到新的对象中。按三步进行。 

第一步：新分配托管堆内存(大小为值类型实例大小加上一个方法表指针和一个SyncBlockIndex)。 

第二步：将值类型的实例字段拷贝到新分配的内存中。 

第三步：返回托管堆中新分配对象的地址。这个地址就是一个指向对象的引用了。 

拆箱：检查对象实例，确保它是给定值类型的一个装箱值。将该值从实例复制到值类型变量中。
 

###### 装箱/拆箱对执行效率的影响，如何优化

装箱时,生成的是全新的引用对象,这会有时间损耗,也就是造成效率降低。 那该如何做呢？

避免装箱的方法：

  1、通过重载函数来避免。

  2、通过泛型来避免。 

凡事并不能绝对，假设你想改造的代码为第三方程序集，你无法更改，那你只能是装箱了。 对于装箱/拆箱代码的优化，由于C#中对装箱和拆箱都是隐式的，所以，根本的方法是对代码进行分析，而分析最直接的方式是了解原理结何查看反编译的IL代码。

比如：在循环体中可能存在多余的装箱，你可以简单采用提前装箱方式进行优化。


对装箱/拆箱更进一步的了解 装箱/拆箱并不如上面所讲那么简单明了，
比如：装箱时,变为引用对象,会多出一个方法表指针,这会有何用处呢？ 通过示例来进一步探讨。 
例子: 
  Struct A : ICloneable 
  { 
    public Int32 x; 
    public override String ToString() 
    { 
      return String.Format(”{0}”,x); 
    }
    public object Clone() 
    { 
      return MemberwiseClone(); 
    } 
  } 
  static void main() 
  { 
    A a; 
    a.x = 100; 
    Console.WriteLine(a.ToString()); 
    Console.WriteLine(a.GetType()); 
    A a2 = (A)a.Clone(); 
    ICloneable c = a2; Ojbect o = c.Clone(); 
  }
 ：a.ToString()。编译器发现A重写了ToString方法，会直接调用ToString的指令。因为A是值类型，编译器不会出现多态行为。因此，直接调用，不装箱。(注：ToString是A的基类System.ValueType的方法) 
 ：a.GetType()，GetType是继承于System.ValueType的方法，要调用它，需要一个方法表指针，于是a将被装箱，从而生成方法表指针，调用基类的System.ValueType。(补一句，所有的值类型都是继承于System.ValueType的)。 
 ：a.Clone()，因为A实现了Clone方法，所以无需装箱。 5.3：ICloneable转型：当a2为转为接口类型时，必须装箱，因为接口是一种引用类型。 
 ：c.Clone()。无需装箱，在托管堆中对上一步已装箱的对象进行调用。 
附：其实上面的基于一个根本的原理，因为未装箱的值类型没有方法表指针，所以，不能通过值类型来调用其上继承的虚方法。另外，接口类型是一个引用类型。对此，我的理解，该方法表指针类似C++的虚函数表指针，它是用来实现引用对象的多态机制的重要依据。





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

开箱和装箱原理 done

多态virtual的原理

