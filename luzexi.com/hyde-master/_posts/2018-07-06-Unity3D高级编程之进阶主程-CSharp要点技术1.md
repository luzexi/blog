---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(一)
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层算法 List 源码"
excerpt_separator: ===
categories:
- 著作书籍
- 版权著作
- Unity3D
- 前端技术
tags:
- unity3d
- c#
---

#第一章，C\#要点技术(一)

很多老鸟看到C#基础总想跳过，因为看了太多次，次次都一样，有时候看得都想吐，基础里无非是几个语法，或者由继承展开的特性，再加上一些高级特有的属性，看多了当然会吐。而我在这里想写些不一样的东西，我认为能看这本书的，基本上都能做到基础的语法部分已经滚瓜烂熟了。所以我们在基础的语法之上讲些更高级的东西会来得更有趣些，比如算法设计，比如常用组件的底层代码分析，比如设计模式，比如动态库(so文件和dll文件)等等。

===

1。 ## 常用组件底层代码解析

### List 底层代码

我们首先来看看List的构造部分，源码如下：


{% highlight c# %}

public class List<T> : IList<T>, System.Collections.IList, IReadOnlyList<T>
{
    private const int _defaultCapacity = 4;

    private T[] _items;
    private int _size;
    private int _version;
    private Object _syncRoot;
    
    static readonly T[]  _emptyArray = new T[0];        
        
    // Constructs a List. The list is initially empty and has a capacity
    // of zero. Upon adding the first element to the list the capacity is
    // increased to 16, and then increased in multiples of two as required.
    public List() {
        _items = _emptyArray;
    }

    // Constructs a List with a given initial capacity. The list is
    // initially empty, but will have room for the given number of elements
    // before any reallocations are required.
    // 
    public List(int capacity) {
        if (capacity < 0) ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.capacity, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
        Contract.EndContractBlock();

        if (capacity == 0)
            _items = _emptyArray;
        else
            _items = new T[capacity];
    }

    //...
    //其他内容
}

{% endhighlight %}

从源码中可以知道，List 继承于IList，IReadOnlyList，IList是提供了主要的接口，IReadOnlyList提供了迭代接口。

[IList源码](https://referencesource.microsoft.com/#mscorlib/system/collections/ilist.cs,5d74f6adfeaf6c7d)

[IReadOnlyList源码](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/ireadonlylist.cs,b040fb780bdd59f4)

看构造部分，我们明确了，List内部是用数组实现的，而不是链表，并且当没有给予指定容量时，初始的容量为0。

也就是说，可以大概率推测List在Add,Remove时都是用数组拷贝生成新数组的方式工作的。下面我们来看下，我们的猜测是否正确。

Add接口源码：

{% highlight c# %}

// Adds the given object to the end of this list. The size of the list is
// increased by one. If required, the capacity of the list is doubled
// before adding the new element.
//
public void Add(T item) {
    if (_size == _items.Length) EnsureCapacity(_size + 1);
    _items[_size++] = item;
    _version++;
}

// Ensures that the capacity of this list is at least the given minimum
// value. If the currect capacity of the list is less than min, the
// capacity is increased to twice the current capacity or to min,
// whichever is larger.
private void EnsureCapacity(int min) {
    if (_items.Length < min) {
        int newCapacity = _items.Length == 0? _defaultCapacity : _items.Length * 2;
        // Allow the list to grow to maximum possible capacity (~2G elements) before encountering overflow.
        // Note that this check works even when _items.Length overflowed thanks to the (uint) cast
        if ((uint)newCapacity > Array.MaxArrayLength) newCapacity = Array.MaxArrayLength;
        if (newCapacity < min) newCapacity = min;
        Capacity = newCapacity;
    }
}

{% endhighlight %}

每次增加一个元素的数据，Add接口首先检查的是容量还够吗，如果不够则用 EnsureCapacity 来增加容量。

在 EnsureCapacity 中，
		
		int newCapacity = _items.Length == 0? _defaultCapacity : _items.Length * 2;

每次容量不够的时候，整个数组的容量都会扩充一倍，_defaultCapacity 是容量的默认值为4. 所以整个扩充的路线为4，8，16，32，64，128，256，512，1024....

用数组形式作为底层数据结构，好处是使用索引方式提取元素很快，但在扩容的时候就很糟糕了，每次new数组都会造成内存垃圾，这给垃圾回收GC带来了很多负担。一次性扩充更多的数组容量为GC减轻了负担，但数组连续的new也还是GC的不小负担，特别是代码中List频繁使用的Add时。另外，如果数量不得当也会浪费大量内存空间，比如当元素数量为 520 时，List 就会扩容到1024个元素，如果不使用剩余的504个空间单位，就造成了大部分的内存空间的浪费。

我们再来看看Remove接口部分的源码。

{% highlight c# %}

// Removes the element at the given index. The size of the list is
// decreased by one.
// 
public bool Remove(T item) {
    int index = IndexOf(item);
    if (index >= 0) {
        RemoveAt(index);
        return true;
    }

    return false;
}

// Returns the index of the first occurrence of a given value in a range of
// this list. The list is searched forwards from beginning to end.
// The elements of the list are compared to the given value using the
// Object.Equals method.
// 
// This method uses the Array.IndexOf method to perform the
// search.
// 
public int IndexOf(T item) {
    Contract.Ensures(Contract.Result<int>() >= -1);
    Contract.Ensures(Contract.Result<int>() < Count);
    return Array.IndexOf(_items, item, 0, _size);
}

// Removes the element at the given index. The size of the list is
// decreased by one.
// 
public void RemoveAt(int index) {
    if ((uint)index >= (uint)_size) {
        ThrowHelper.ThrowArgumentOutOfRangeException();
    }
    Contract.EndContractBlock();
    _size--;
    if (index < _size) {
        Array.Copy(_items, index + 1, _items, index, _size - index);
    }
    _items[_size] = default(T);
    _version++;
}

{% endhighlight %}

Remove接口中包含了 IndexOf 和 RemoveAt，用 IndexOf 找到元素的索引位置，用 RemoveAt 删除指定位置的元素。删除的原理其实就是用 Array.Copy 对数组进行覆盖。IndexOf 启用的是 Array.IndexOf 接口来查找元素的索引位置，这个接口内部实现是就是按索引顺序从0到n对每个位置的比较，复杂度为O(n)。

再看来 Insert 接口源码。

{% highlight c# %}

// Inserts an element into this list at a given index. The size of the list
// is increased by one. If required, the capacity of the list is doubled
// before inserting the new element.
// 
public void Insert(int index, T item) {
    // Note that insertions at the end are legal.
    if ((uint) index > (uint)_size) {
        ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_ListInsert);
    }
    Contract.EndContractBlock();
    if (_size == _items.Length) EnsureCapacity(_size + 1);
    if (index < _size) {
        Array.Copy(_items, index, _items, index + 1, _size - index);
    }
    _items[index] = item;
    _size++;            
    _version++;
}

{% endhighlight %}

与Add接口一样，先检查容量是否足够，不足则扩容。插入元素时，则以拷贝数组的形式，将数组里的指定元素后面的元素向后移动一个位置。

到这里，可以知道List的Add，Insert，IndexOf，Remove接口都是没有优化过的，如果过于频繁使用的话，会导致CPU使用过高，造成不少内存的冗余，也会使得垃圾回收(GC)时承担了更多的压力。

AddRange，RemoveRange的原理和Add与Remove是一样的，只是多了几个元素，把单个元素变成了以容器为单位的形式进行操作。都是先检查容量是否合适，不合适则扩容，以及Remove时先得到索引位置再进行整体的覆盖删掉的元素。

{% include advertisement_content.html %}

### 我们再来快速的看看其他常用接口的源码是如何实现的。

比如 []的实现，

{% highlight c# %}

// Sets or Gets the element at the given index.
// 
public T this[int index] {
    get {
        // Following trick can reduce the range check by one
        if ((uint) index >= (uint)_size) {
            ThrowHelper.ThrowArgumentOutOfRangeException();
        }
        Contract.EndContractBlock();
        return _items[index]; 
    }

    set {
        if ((uint) index >= (uint)_size) {
            ThrowHelper.ThrowArgumentOutOfRangeException();
        }
        Contract.EndContractBlock();
        _items[index] = value;
        _version++;
    }
}

{% endhighlight %}

[]的实现，直接使用了数组的索引方式获取元素。

再比如 Clear 清除接口

{% highlight c# %}

// Clears the contents of List.
public void Clear() {
    if (_size > 0)
    {
        Array.Clear(_items, 0, _size); // Don't need to doc this but we clear the elements so that the gc can reclaim the references.
        _size = 0;
    }
    _version++;
}

{% endhighlight %}

Clear时并不会删除数组，而是将数组中的元素清空，并置 _size 为 0，表明当前容量为0.


再比如 Contains 包含接口

{% highlight c# %}

// Contains returns true if the specified element is in the List.
// It does a linear, O(n) search.  Equality is determined by calling
// item.Equals().
//
public bool Contains(T item) {
    if ((Object) item == null) {
        for(int i=0; i<_size; i++)
            if ((Object) _items[i] == null)
                return true;
        return false;
    }
    else {
        EqualityComparer<T> c = EqualityComparer<T>.Default;
        for(int i=0; i<_size; i++) {
            if (c.Equals(_items[i], item)) return true;
        }
        return false;
    }
}

{% endhighlight %}

Contains 接口使用的是线性查找方式比较元素，复杂度为O(n)。


再比如 ToArray 转化数组接口

{% highlight c# %}

// ToArray returns a new Object array containing the contents of the List.
// This requires copying the List, which is an O(n) operation.
public T[] ToArray() {
    Contract.Ensures(Contract.Result<T[]>() != null);
    Contract.Ensures(Contract.Result<T[]>().Length == Count);

    T[] array = new T[_size];
    Array.Copy(_items, 0, array, 0, _size);
    return array;
}

{% endhighlight %}

ToArray接口，重新new了一个数组返回出来。

再比如 Find 查找接口

{% highlight c# %}

public T Find(Predicate<T> match) {
    if( match == null) {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.match);
    }
    Contract.EndContractBlock();

    for(int i = 0 ; i < _size; i++) {
        if(match(_items[i])) {
            return _items[i];
        }
    }
    return default(T);
}

{% endhighlight %}

Find接口使用的也是线性查找，对每个元素都进行了比较，算法复杂度为O(n)。

再比如 Enumerator 枚举迭代部分的细节

{% highlight c# %}

// Returns an enumerator for this list with the given
// permission for removal of elements. If modifications made to the list 
// while an enumeration is in progress, the MoveNext and 
// GetObject methods of the enumerator will throw an exception.
//
public Enumerator GetEnumerator() {
    return new Enumerator(this);
}

/// <internalonly/>
IEnumerator<T> IEnumerable<T>.GetEnumerator() {
    return new Enumerator(this);
}

System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() {
    return new Enumerator(this);
}

[Serializable]
public struct Enumerator : IEnumerator<T>, System.Collections.IEnumerator
{
    private List<T> list;
    private int index;
    private int version;
    private T current;

    internal Enumerator(List<T> list) {
        this.list = list;
        index = 0;
        version = list._version;
        current = default(T);
    }

    public void Dispose() {
    }

    public bool MoveNext() {

        List<T> localList = list;

        if (version == localList._version && ((uint)index < (uint)localList._size)) 
        {                                                     
            current = localList._items[index];                    
            index++;
            return true;
        }
        return MoveNextRare();
    }

    private bool MoveNextRare()
    {                
        if (version != list._version) {
            ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
        }

        index = list._size + 1;
        current = default(T);
        return false;                
    }

    public T Current {
        get {
            return current;
        }
    }

    Object System.Collections.IEnumerator.Current {
        get {
            if( index == 0 || index == list._size + 1) {
                 ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);
            }
            return Current;
        }
    }

    void System.Collections.IEnumerator.Reset() {
        if (version != list._version) {
            ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
        }
        
        index = 0;
        current = default(T);
    }

}

{% endhighlight %}

Enumerator 每次都是被new出来了，如果大量使用枚举器比如foreach就会造成大量的垃圾对象，这也是为什么大家说 list 的 foreach 会增加垃圾回收GC的压力了原因了。

最后我们来看看 Sort 排序接口

{% highlight c# %}

// Sorts the elements in a section of this list. The sort compares the
// elements to each other using the given IComparer interface. If
// comparer is null, the elements are compared to each other using
// the IComparable interface, which in that case must be implemented by all
// elements of the list.
// 
// This method uses the Array.Sort method to sort the elements.
// 
public void Sort(int index, int count, IComparer<T> comparer) {
    if (index < 0) {
        ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
    }
    
    if (count < 0) {
        ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.count, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
    }
        
    if (_size - index < count)
        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidOffLen);
    Contract.EndContractBlock();

    Array.Sort<T>(_items, index, count, comparer);
    _version++;
}

{% endhighlight %}

我找到 Array.Sort 源码，使用的是快速排序方式进行排序。也决定了 List 的 Sort 排序部分的效率为O(nlogn)。

以下为 Array.Sort 的使用的算法源码。

{% highlight c# %}

internal static void DepthLimitedQuickSort(T[] keys, int left, int right, IComparer<T> comparer, int depthLimit)
{
    do
    {
        if (depthLimit == 0)
        {
            Heapsort(keys, left, right, comparer);
            return;
        }

        int i = left;
        int j = right;

        // pre-sort the low, middle (pivot), and high values in place.
        // this improves performance in the face of already sorted data, or 
        // data that is made up of multiple sorted runs appended together.
        int middle = i + ((j - i) >> 1);
        SwapIfGreater(keys, comparer, i, middle);  // swap the low with the mid point
        SwapIfGreater(keys, comparer, i, j);   // swap the low with the high
        SwapIfGreater(keys, comparer, middle, j); // swap the middle with the high

        T x = keys[middle];
        do
        {
            while (comparer.Compare(keys[i], x) < 0) i++;
            while (comparer.Compare(x, keys[j]) < 0) j--;
            Contract.Assert(i >= left && j <= right, "(i>=left && j<=right)  Sort failed - Is your IComparer bogus?");
            if (i > j) break;
            if (i < j)
            {
                T key = keys[i];
                keys[i] = keys[j];
                keys[j] = key;
            }
            i++;
            j--;
        } while (i <= j);

        // The next iteration of the while loop is to "recursively" sort the larger half of the array and the
        // following calls recrusively sort the smaller half.  So we subtrack one from depthLimit here so
        // both sorts see the new value.
        depthLimit--;

        if (j - left <= right - i)
        {
            if (left < j) DepthLimitedQuickSort(keys, left, j, comparer, depthLimit);
            left = i;
        }
        else
        {
            if (i < right) DepthLimitedQuickSort(keys, i, right, comparer, depthLimit);
            right = j;
        }
    } while (left < right);
}

{% endhighlight %}

- - -

[List源码](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/list.cs)

### 从源码上看得出来，代码是线程不安全的。所以最好别使用公用的 List 来添加，并发情况下，无法判断 _size++ 的执行顺序。

### List 并不是高效的组件，真实情况是，他比数组的效率还要差的多，他只是个兼容性比较强得组件而已，好用，但效率差。
