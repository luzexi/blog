---
layout: post
status: publish
published: true
title: 《Unity3D高级编程之进阶主程》第一章，C#要点技术(二)
description: "unity3d 高级编程 c# 主程 算法 设计模式 动态库 底层源码 Dictionary"
excerpt_separator: ===
categories:
- 版权著作
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

![拉链法结构图](/assets/book/1/lalianfa.png)

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

- - -

了解Add是最直接了解底层数据结构如何运作的方法，我们来看下Add接口的实现。

源代码如下：

{% highlight c# %}

public void Add(TKey key, TValue value)
{
    Insert(key, value, true);
}

private void Initialize(int capacity)
{
    int size = HashHelpers.GetPrime(capacity);
    buckets = new int[size];
    for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;
    entries = new Entry[size];
    freeList = -1;
}

private void Insert(TKey key, TValue value, bool add)
{
    if( key == null ) {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    if (buckets == null) Initialize(0);
    int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
    int targetBucket = hashCode % buckets.Length;

#if FEATURE_RANDOMIZED_STRING_HASHING
    int collisionCount = 0;
#endif

    for (int i = buckets[targetBucket]; i >= 0; i = entries[i].next) {
        if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) {
            if (add) { 
                ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_AddingDuplicate);
            }
            entries[i].value = value;
            version++;
            return;
        } 

#if FEATURE_RANDOMIZED_STRING_HASHING
        collisionCount++;
#endif
    }
    int index;
    if (freeCount > 0) {
        index = freeList;
        freeList = entries[index].next;
        freeCount--;
    }
    else {
        if (count == entries.Length)
        {
            Resize();
            targetBucket = hashCode % buckets.Length;
        }
        index = count;
        count++;
    }

    entries[index].hashCode = hashCode;
    entries[index].next = buckets[targetBucket];
    entries[index].key = key;
    entries[index].value = value;
    buckets[targetBucket] = index;
    version++;

#if FEATURE_RANDOMIZED_STRING_HASHING

#if FEATURE_CORECLR
    // In case we hit the collision threshold we'll need to switch to the comparer which is using randomized string hashing
    // in this case will be EqualityComparer<string>.Default.
    // Note, randomized string hashing is turned on by default on coreclr so EqualityComparer<string>.Default will 
    // be using randomized string hashing

    if (collisionCount > HashHelpers.HashCollisionThreshold && comparer == NonRandomizedStringEqualityComparer.Default) 
    {
        comparer = (IEqualityComparer<TKey>) EqualityComparer<string>.Default;
        Resize(entries.Length, true);
    }
#else
    if(collisionCount > HashHelpers.HashCollisionThreshold && HashHelpers.IsWellKnownEqualityComparer(comparer)) 
    {
        comparer = (IEqualityComparer<TKey>) HashHelpers.GetRandomizedEqualityComparer(comparer);
        Resize(entries.Length, true);
    }
#endif // FEATURE_CORECLR

#endif

}

{% endhighlight %}

代码中展示了几个要点。

首先是初始化，在加入数据前需要对数据结构进行构造。

	if (buckets == null) Initialize(0);

那么构造多大的数据结构才合适呢，看下面这行

	int size = HashHelpers.GetPrime(capacity);

HashHelpers中，primes数值是这样定义的。

{% highlight c# %}

 public static readonly int[] primes = {
        3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,
        1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,
        17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,
        187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,
        1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369};

public static int GetPrime(int min) 
{
    if (min < 0)
        throw new ArgumentException(Environment.GetResourceString("Arg_HTCapacityOverflow"));
    Contract.EndContractBlock();

    for (int i = 0; i < primes.Length; i++) 
    {
        int prime = primes[i];
        if (prime >= min) return prime;
    }

    //outside of our predefined table. 
    //compute the hard way. 
    for (int i = (min | 1); i < Int32.MaxValue;i+=2) 
    {
        if (IsPrime(i) && ((i - 1) % Hashtable.HashPrime != 0))
            return i;
    }
    return min;
}

// Returns size of hashtable to grow to.
public static int ExpandPrime(int oldSize)
{
    int newSize = 2 * oldSize;

    // Allow the hashtables to grow to maximum possible size (~2G elements) before encoutering capacity overflow.
    // Note that this check works even when _items.Length overflowed thanks to the (uint) cast
    if ((uint)newSize > MaxPrimeArrayLength && MaxPrimeArrayLength > oldSize)
    {
        Contract.Assert( MaxPrimeArrayLength == GetPrime(MaxPrimeArrayLength), "Invalid MaxPrimeArrayLength");
        return MaxPrimeArrayLength;
    }

    return GetPrime(newSize);
}

{% endhighlight %}

GetPrime会返回一个需要的size最小的数值。ExpandPrime则会扩大原来size的2倍作为扩展。

从Prime的定义看的出，首次定义size为3，每次扩大2倍，也就是，3->7->17->37->.... 底层数据结构的大小是按照这个数值顺序来扩展的，除非你在创建Directionary时，先定义了他的初始大小。

我们继续看初始化后的内容，对Key做哈希操作获得地址。

		int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
    	int targetBucket = hashCode % buckets.Length;

获得哈希地址后，还需要对哈希地址做余操作，以确定地址落在数据结构长度范围内。

然后就是解决冲突的拉链法了，从for循环开始到后面都是，for循环是检查是否有相同的Key，以及检查数据结构存储的数据是否需要扩大。用得到的哈希地址查看是否有冲突，有冲突则把数据放入链表的头部，无冲突就直接放入。而数据本身Entrie是先后次序放入数组 entries 的。

- - -

了解了Add接口，我们来看看Remove部分。

Remove接口源码：

{% highlight c# %}

public bool Remove(TKey key)
{
    if(key == null) {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    if (buckets != null) {
        int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
        int bucket = hashCode % buckets.Length;
        int last = -1;
        for (int i = buckets[bucket]; i >= 0; last = i, i = entries[i].next) {
            if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) {
                if (last < 0) {
                    buckets[bucket] = entries[i].next;
                }
                else {
                    entries[last].next = entries[i].next;
                }
                entries[i].hashCode = -1;
                entries[i].next = freeList;
                entries[i].key = default(TKey);
                entries[i].value = default(TValue);
                freeList = i;
                freeCount++;
                version++;
                return true;
            }
        }
    }
    return false;
}

{% endhighlight %}

Remove接口相对Add简单的多，先用哈希函数得到哈希值，再做余操作确定地址落在数组范围内，从哈希地址开始，查找冲突的元素的Key是否与需要移除的Key值相同，相同则进行移除操作。

注意，源码中移除操作不对内存进行删减，而是将数值置空，减少了内存的频繁操作。

- - -

我们继续剖析另一个重要的接口 ContainsKey 接口。

源码如下：

{% highlight c# %}

public bool ContainsKey(TKey key)
{
    return FindEntry(key) >= 0;
}

private int FindEntry(TKey key)
{
    if( key == null) {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    if (buckets != null) {
        int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
        for (int i = buckets[hashCode % buckets.Length]; i >= 0; i = entries[i].next) {
            if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) return i;
        }
    }
    return -1;
}

{% endhighlight %}

ContainsKey就是一个查找Key位置的过程。直接调用了 FindEntry 函数，FindEntry 查找Key值位置的方法跟我们前面提到的相同。从用Key值得到的哈希值地址开始查找，查看所有冲突链表中，是否有与Key值相同的值，找到即刻返回该索引地址。

- - -

{% include advertisement_content.html %}

其他接口相对比较简单了，

比如 TryGetValue

{% highlight c# %}

public bool TryGetValue(TKey key, out TValue value)
{
    int i = FindEntry(key);
    if (i >= 0) {
        value = entries[i].value;
        return true;
    }
    value = default(TValue);
    return false;
}

{% endhighlight %}

调用的也是FindEntry的接口，来获取Key对应的Value值。

还有[]操作符的重定义。

{% highlight c# %}

public TValue this[TKey key] {
    get {
        int i = FindEntry(key);
        if (i >= 0) return entries[i].value;
        ThrowHelper.ThrowKeyNotFoundException();
        return default(TValue);
    }
    set {
        Insert(key, value, false);
    }
}

{% endhighlight %}

[]符号在获取元素时也同样使用 FindEntry 函数，Set 时则使用与 Add 相同的 Insert函数，都是同一套方法，即哈希拉链冲突解决方案。

- - -

从源码剖析来看，哈希冲突的拉链法贯穿了整个底层数据结构。所以哈希函数是关键了。哈希函数的好坏直接决定了效率高低。

我们来看看哈希函数的创建过程，比较函数的创建的源码：

{% highlight c# %}

private static EqualityComparer<T> CreateComparer()
{
    Contract.Ensures(Contract.Result<EqualityComparer<T>>() != null);

    RuntimeType t = (RuntimeType)typeof(T);
    // Specialize type byte for performance reasons
    if (t == typeof(byte)) {
        return (EqualityComparer<T>)(object)(new ByteEqualityComparer());
    }
    // If T implements IEquatable<T> return a GenericEqualityComparer<T>
    if (typeof(IEquatable<T>).IsAssignableFrom(t)) {
        return (EqualityComparer<T>)RuntimeTypeHandle.CreateInstanceForAnotherGenericParameter((RuntimeType)typeof(GenericEqualityComparer<int>), t);
    }
    // If T is a Nullable<U> where U implements IEquatable<U> return a NullableEqualityComparer<U>
    if (t.IsGenericType && t.GetGenericTypeDefinition() == typeof(Nullable<>)) {
        RuntimeType u = (RuntimeType)t.GetGenericArguments()[0];
        if (typeof(IEquatable<>).MakeGenericType(u).IsAssignableFrom(u)) {
            return (EqualityComparer<T>)RuntimeTypeHandle.CreateInstanceForAnotherGenericParameter((RuntimeType)typeof(NullableEqualityComparer<int>), u);
        }
    }
    
    // See the METHOD__JIT_HELPERS__UNSAFE_ENUM_CAST and METHOD__JIT_HELPERS__UNSAFE_ENUM_CAST_LONG cases in getILIntrinsicImplementation
    if (t.IsEnum) {
        TypeCode underlyingTypeCode = Type.GetTypeCode(Enum.GetUnderlyingType(t));

        // Depending on the enum type, we need to special case the comparers so that we avoid boxing
        // Note: We have different comparers for Short and SByte because for those types we need to make sure we call GetHashCode on the actual underlying type as the 
        // implementation of GetHashCode is more complex than for the other types.
        switch (underlyingTypeCode) {
            case TypeCode.Int16: // short
                return (EqualityComparer<T>)RuntimeTypeHandle.CreateInstanceForAnotherGenericParameter((RuntimeType)typeof(ShortEnumEqualityComparer<short>), t);
            case TypeCode.SByte:
                return (EqualityComparer<T>)RuntimeTypeHandle.CreateInstanceForAnotherGenericParameter((RuntimeType)typeof(SByteEnumEqualityComparer<sbyte>), t);
            case TypeCode.Int32:
            case TypeCode.UInt32:
            case TypeCode.Byte:
            case TypeCode.UInt16: //ushort
                return (EqualityComparer<T>)RuntimeTypeHandle.CreateInstanceForAnotherGenericParameter((RuntimeType)typeof(EnumEqualityComparer<int>), t);
            case TypeCode.Int64:
            case TypeCode.UInt64:
                return (EqualityComparer<T>)RuntimeTypeHandle.CreateInstanceForAnotherGenericParameter((RuntimeType)typeof(LongEnumEqualityComparer<long>), t);
        }
    }
    // Otherwise return an ObjectEqualityComparer<T>
    return new ObjectEqualityComparer<T>();
}

{% endhighlight %}

源码中，对数字，byte，有‘比较’接口(IEquatable\<T\>)，和没有‘比较’接口的进行了区分对待。

对数字和byte有相应固定的比较函数。

有‘比较’接口(IEquatable\<T\>)的实体，则直接使用GenericEqualityComparer\<T\>来获得哈希函数。

没有‘比较’接口(IEquatable\<T\>)的实体，如果继承了 Nullable\<U\> 接口，则使用一个叫 NullableEqualityComparer 的比较函数来代替。

在C#里所有类都继承了 Object 类，所以如果没有特别的重写 Equals 函数，都会使用 Object 类的 Equals 函数。

{% highlight c# %}

public virtual bool Equals(Object obj)
{
    return RuntimeHelpers.Equals(this, obj);
}

[System.Security.SecuritySafeCritical]  // auto-generated
[ResourceExposure(ResourceScope.None)]
[MethodImplAttribute(MethodImplOptions.InternalCall)]
public new static extern bool Equals(Object o1, Object o2);

{% endhighlight %}

这个 Equals 是很底层的方法没有源码可寻。但我们可以从特性上看到,他应该是获取了更加底层的方法，来获得内存地址。

- - -

Dictionary 不是线程安全的组件，官方源码中进行了这样的解释。

		** Hashtable has multiple reader/single writer (MR/SW) thread safety built into 
		** certain methods and properties, whereas Dictionary doesn't. If you're 
		** converting framework code that formerly used Hashtable to Dictionary, it's
		** important to consider whether callers may have taken a dependence on MR/SW
		** thread safety. If a reader writer lock is available, then that may be used
		** with a Dictionary to get the same thread safety guarantee. 

如果要在多个线程中共享Dictionaray的读写操作，就要自己写lock以保证线程安全。

- - -

### 到这里我们已经全面了解了 Dictionary 的内部构造和运作机制。他是由数组构成，并且由哈希函数完成地址构建，由拉链法冲突解决方案来解决冲突的。

### 从效率上看，new时先确定大致数量会更加高效，另外用数值方式做Key比用类实例方式作为Key值更加有效率。

### 从内存操作上看，大小以3->7->17->37->....的速度，每次增加2倍多的顺序进行，删除时，并不缩减内存。

### Dictionary不是线程安全的，需要进行自己进行lock操作。

[Dictionary源码](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs)





