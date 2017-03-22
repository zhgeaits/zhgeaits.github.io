---
layout: post
title:  "Understanding Android's SparseArray"
date:   2016-05-20 11:00:03
categories: android
supertype: career
type: android
---

## 1 引言

使用更好的数据结构，能让程序的性能更佳，虽然java里面提供了很多的数据结构，但是很多性能并不是很好，我们在做Android开发的时候经常调用的是Java的API，Android并针对性能这里开发了一些工具。如使用SparseArray和ArrayMap来在特定的场景代替HashMap。而其实，网上有不少相关的资料学习了，为什么我还要记录一份，原因网上的资料写得并不是很适合自己，并不能很好理解其中的原理，再就是经常忘记，要去网上查也麻烦，不如就当作笔记写在blog里面，平时忘记以后能快速翻查并回忆其中的原理。

## 2 SparseArray

观其名，Sparse是稀疏的意思，合起来就是稀疏数组，这不就是大学时候学习的稀疏矩阵吗？矩阵就是二维数组，而计算机里面二维数组的物理存储结构就是一维数组。

### 2.1 稀疏矩阵

完全可以在百科里面科普到这个知识点，当一个矩阵里面没用到的元素非常多，通常元素值为0，那么它就是一个稀疏矩阵，至于要多少才算是稀疏，我们可以定义一个稠密度。

可以想到如果矩阵是一个稀疏矩阵的时候，我们会浪费很多的内存空间来存储这个矩阵。于是改用一个n*3的矩阵来存储，第一行数metedata，即第一个(x,y,z)存储的分别是原矩阵的行数和列数，z代表是原矩阵存储的元素个数。接下来的第2至(z+1)行，分别存储了元素的坐标和值。

由于太简单了，所以直接语言描述即可，无需画图，用下表说明并加上想象即可。

<table>
    <tr>
        <td>原矩阵行数</td><td>原矩阵列数</td><td>原矩阵元素个数</td>
    </tr>
    <tr>
        <td>x1</td><td>y1</td><td>value1</td>
    </tr>
    <tr>
        <td>x2</td><td>y2</td><td>value2</td>
    </tr>
</table> 

### 2.2 概述

直接在Google的官方文档就有[SparseArray](https://developer.android.com/reference/android/util/SparseArray.html)的介绍了，然后在android的SDK也是可以直接打开类看到源码，我是基于API 22来学习的。

它不是二元的泛型，即不能用KV的结构，因此也不能完全代替HashMap。根据官网的描述，它是map intergers to objects的，所以key是int类型的。由于里面使用的是int而不是Integer，就少了自动装箱和拆箱的过程，并没有了额外的Entry实体来存储，内存性能会优于HashMap，更重要的是，里面使用了二分查找算法，在时间复杂度方面优于HashMap，但是，因为用了数组存储，并且每次操作都会二分查找，即要求排序，会对数组重新排列，所以在数据量大的时候性能会很差，根据二分查找，性能至少低50%。根据官方给的数据，建议在一千以内(hundreds of items)的数量时候使用最佳。

>关于HashMap的学习网上非常多资料，以后如果有时间，我得重新学习源码并写一篇blog吧。

### 2.3 源码学习

直接打开源码，代码量非常的少，可以看到有下面的属性和构造方法：

{% highlight java %}

private static final Object DELETED = new Object();
private boolean mGarbage = false;
private int[] mKeys;
private Object[] mValues;
private int mSize;

public SparseArray() {
    this(10);
}
public SparseArray(int initialCapacity) {
    if (initialCapacity == 0) {
        mKeys = EmptyArray.INT;
        mValues = EmptyArray.OBJECT;
    } else {
        mValues = ArrayUtils.newUnpaddedObjectArray(initialCapacity);
        mKeys = new int[mValues.length];
    }
    mSize = 0;
}

{% endhighlight %}

可以看到属性非常少，默认的容量大小是10个元素，使用的`ArrayUtils.newUnpaddedObjectArray`方法来创建对象数组看不到源码，实际上是native的调用，根据名字可以判断是对内存的分配有针对性的进行优化。那些属性现在还不太明白，看到下面的操作自然就明白了。

最先用到的肯定是put方法，往里面添加数据，其中key是int类型：

{% highlight java %}

public void put(int key, E value) {
	//这里是二分查找，从mKeys里面找到key，并返回索引。
    int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

    if (i >= 0) {
    	//找到的话直接设置
        mValues[i] = value;
    } else {
    	//恢复索引
        i = ~i;

        //如果这个位置的元素已经删除了，则直接替换
        if (i < mSize && mValues[i] == DELETED) {
            mKeys[i] = key;
            mValues[i] = value;
            return;
        }

        //如果需要gc回收对象，回收以后再次二分查找
        if (mGarbage && mSize >= mKeys.length) {
            gc();

            // Search again because indices may have changed.
            i = ~ContainerHelpers.binarySearch(mKeys, mSize, key);
        }

        //扩张mKeys数组，插入key到i位置
        mKeys = GrowingArrayUtils.insert(mKeys, mSize, i, key);
        //扩张mValues数组，插入value到i位置
        mValues = GrowingArrayUtils.insert(mValues, mSize, i, value);        
        mSize++;
    }
}

{% endhighlight %}

代码逻辑都看起来比较简单了，使用两个数组来分别存储key和value，在这里，如果不了解稀疏数组之前，我经常会犯一个错误想法：以为Object数组的索引便是int类型的key，或者另外一种错误的想法，以为key数组存储的是object数组的索引。实际上，正确的是应用稀疏数组的思想：int数组存储的就是key，object数组存储的是value，两个数组必须一一对应，所以形成KV对。之所以会产生错误想法，也是由于误解了二分查找，二分查找比较的是数组的元素，而返回的是数组的索引，且看源码：

{% highlight java %}

static int binarySearch(int[] array, int size, int value) {
    int lo = 0;
    int hi = size - 1;

    while (lo <= hi) {
        final int mid = (lo + hi) >>> 1;
        final int midVal = array[mid];

        if (midVal < value) {
            lo = mid + 1;
        } else if (midVal > value) {
            hi = mid - 1;
        } else {
            return mid;  // value found
        }
    }
    return ~lo;  // value not present
}

{% endhighlight %}

通常二分查找的实现用递归，这里用迭代，虽然理解没递归容易，但是少了栈叠堆，性能会好一些。除2没有用除法符号，而是用移位运算，右移一位，而是用的是无符号移位，即右移以后左边补0，如果是有符号移位，则左边补上符号位。二分法查找算法简单，就是不断减半，去中间值判断，因为是排好序的了。如果找到就直接返回索引，如果没找到就用`~`这个符号对索引运算再返回。

>我们知道java是没有无符号数的，它全都是有符号的，所以一个字节表示的是-127到128的范围，在大学记得求一个负数的二进制是先求正数的二进制，然后取反码，再+1。`~`符号是对一个数进行求反码，所以对应的10进制数就是其负数-1。

这里如果没查找到元素，则返回一个补码，即负数，之后还是可以还原回正数。

如果最后没查找到key，但是最后查找停止的位置的元素已经是删除了的，那么就可以直接替换结束了。问题是什么时候values数组的值会变成`DELETED`，mGarbage会变为true？查看代码发现：

{% highlight java %}
public void delete(int key) {
    int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

    if (i >= 0) {
        if (mValues[i] != DELETED) {
            mValues[i] = DELETED;
            mGarbage = true;
        }
    }
}
public void removeAt(int index) {
    if (mValues[index] != DELETED) {
        mValues[index] = DELETED;
        mGarbage = true;
    }
}
{% endhighlight %}

可以看出来，删除一个元素的时候就会去掉这个元素的引用，并且设置mGarbage等待下一次gc回收。查看gc的代码如下：

{% highlight java %}
private void gc() {
    int n = mSize;
    int o = 0;
    int[] keys = mKeys;
    Object[] values = mValues;

    for (int i = 0; i < n; i++) {
        Object val = values[i];

        if (val != DELETED) {
            if (i != o) {
                keys[o] = keys[i];
                values[o] = val;
                values[i] = null;
            }

            o++;
        }
    }

    mGarbage = false;
    mSize = o;
}
{% endhighlight %}
可以看出，gc方法做的事情就是找到`DELETE`的元素，然后把数组整体移动，最后修改mSize的值和设置mGarbage为false。

再看会put方法的最后调用的是GrowingArrayUtils.insert()方法，实际上，也是移动数组，如果数组变大了，就会new新的数组出来，然后调用System.arraycopy()方法来复制数组。但是实际上，这里可能GrowingArrayUtils的类包装了底层的实现，做了更多的优化，看不到源代码了。

最后看获取数据的方法：

{% highlight java %}
public E get(int key) {
    return get(key, null);
}

@SuppressWarnings("unchecked")
public E get(int key, E valueIfKeyNotFound) {
    int i = ContainerHelpers.binarySearch(mKeys, mSize, key);

    if (i < 0 || mValues[i] == DELETED) {
        return valueIfKeyNotFound;
    } else {
        return (E) mValues[i];
    }
}
{% endhighlight %}
非常的简单，首先是拿到key去二分查找获取索引，然后根据索引再去Object数组获取value值返回。

### 2.4 结合稀疏矩阵再理解

由2.1知道稀疏矩阵改用一个3列的矩阵存储，前面两列存储的是元素在原矩阵的坐标，第三列存储的是元素的值。其实这三列就是三个数组，第几行则是数组的下标索引。而存储map的kv对的时候，因为并不是矩阵，所以无需用到三列的矩阵存储，两列即可，一列存储key，另外一列存储value，即我们的int数组和object数组，如下表所示：

<table>
    <tr>
        <td>int数组</td><td>object数组</td>
    </tr>
    <tr>
        <td>key1</td><td>value1</td>
    </tr>
    <tr>
        <td>key2</td><td>value2</td>
    </tr>
</table> 