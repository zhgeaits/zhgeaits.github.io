---
layout: post
title:  "Understanding Android's ArrayMap"
date:   2016-05-20 11:00:03
categories: android
supertype: career
type: android
---

## 1 引言

上一篇学习了SparseArray，那都是比较简单的思想，代码量也很少，可用性也比较低，限制了key只能是int类型。今天继续学习另外一个数据结构。

## 2 ArrayMap

由于SparseArray只能用int当作key，并不能完全代替HashMap，所以有了[ArrayMap](https://developer.android.com/reference/android/support/v4/util/ArrayMap.html)，根据官方的描述，它是用一个interger数组来存储每个item的hashcode，用object数组来存kv对。这就避免了创建额外的entry，并且它会控制这个增长的大小，并不会重建hashmap，而是复制移动数组。

针对标准的map，所以里面实现了比较多的东西，如iterator，如果我们并不需要这些，可以去使用[SimpleArrayMap](https://developer.android.com/reference/android/support/v4/util/SimpleArrayMap.html)。

里面使用同样使用了二分查找，所以效率问题跟SparseArray一样，在一千以内性能极佳，过多则不好了。本质的实现其实也是稀疏矩阵。

### 2.1 核心源码学习

这个类的实现比较复杂，代码会比较多，这里主要是展示核心的理解就好了。里面很多核心的思想和算法，如缓存等等的。

先看看它的关键属性和构造方法：

{% highlight java %}

private static final int BASE_SIZE = 4;
/**
 * 一个特殊的值标记immutable
 */
static final int[] EMPTY_IMMUTABLE_INTS = new int[0];

int[] mHashes;
Object[] mArray;
int mSize;
MapCollections<K, V> mCollections;

public ArrayMap() {
    mHashes = ContainerHelpers.EMPTY_INTS;
    mArray = ContainerHelpers.EMPTY_OBJECTS;
    mSize = 0;
}
public ArrayMap(int capacity) {
    if (capacity == 0) {
        mHashes = ContainerHelpers.EMPTY_INTS;
        mArray = ContainerHelpers.EMPTY_OBJECTS;
    } else {
        allocArrays(capacity);
    }
    mSize = 0;
}
private ArrayMap(boolean immutable) {
    mHashes = EMPTY_IMMUTABLE_INTS;
    mArray = ContainerHelpers.EMPTY_OBJECTS;
    mSize = 0;
}

{% endhighlight %}

其中ContainerHelpers是一个二分查找的工具类，和SparseArray一样的；我们默认使用空的构造方法，也可以初始化容量。immutable这个参数没用，后面再研究。mCollections属性也暂时猜测不出来是干什么用的，而mHashes和mArray这两个属性和SparseArray的mKeys和mValues很像，就先猜想一致的。

#### Put方法

最关键的便是put方法，源码如下：

{% highlight java %}

/**
 * KEY不可以为null，如果key已经存在了就会覆盖，返回旧的value，也可能是返回null
 */
@Override
public V put(K key, V value) {
    final int hash;
    int index;
    if (key == null) {
        hash = 0;
        index = indexOfNull();
    } else {
    	//计算key的hashcode
        hash = key.hashCode();
        //转换hashcode为index，先暂时不看这个方法的内部
        index = indexOf(key, hash);
    }
    //如果已经存在旧的值了
    if (index >= 0) {
    	//暂时不明白这个运算，*2+1？跳过
        index = (index<<1) + 1;
        //替换旧值并返回结束
        final V old = (V)mArray[index];
        mArray[index] = value;
        return old;
    }
    //把index转换为反码，这个运算符在上篇介绍过了，实际上这里是把index转换成正数，它是负数才不会进入上一个判断。
    //由这里也猜测得到和SpareArray的逻辑差不多了。因为调用的二分查找方法，如果没找到则返回最后index的反码。
    index = ~index;
    //如果容量不够了则需要扩容
    if (mSize >= mHashes.length) {
    	//这里是计算扩容的大小，如果容量已经大于8了，那么就扩容原本的一半，否则可能扩容8个或者4个。BASE_SIZE=4
        final int n = mSize >= (BASE_SIZE*2) ? (mSize+(mSize>>1))
                : (mSize >= BASE_SIZE ? (BASE_SIZE*2) : BASE_SIZE);

        if (DEBUG) Log.d(TAG, "put: grow from " + mHashes.length + " to " + n);

        final int[] ohashes = mHashes;
        final Object[] oarray = mArray;
        //分配新的数组
        allocArrays(n);
        //然后复制数组
        if (mHashes.length > 0) {
            if (DEBUG) Log.d(TAG, "put: copy 0-" + mSize + " to 0");
            System.arraycopy(ohashes, 0, mHashes, 0, ohashes.length);
            System.arraycopy(oarray, 0, mArray, 0, oarray.length);
        }
        //释放旧的数组
        freeArrays(ohashes, oarray, mSize);
    }
    //可以猜想到由于使用了二分查找，那么和SparseArray一样，是根据key排序的
    //这里判断如果put进来的kv在数组之间，则进行移动数组。
    if (index < mSize) {
        if (DEBUG) Log.d(TAG, "put: move " + index + "-" + (mSize-index)
                + " to " + (index+1));
        System.arraycopy(mHashes, index, mHashes, index + 1, mSize - index);
        System.arraycopy(mArray, index << 1, mArray, (index + 1) << 1, (mSize - index) << 1);
    }
    //设置新值，由这里发现，mHashes数组存放的是key的hash值，它的长度和mSize一致。而mArray存放的是key和value两个值，它的长度是Size的两倍。
    //现在也明白了为什么计算index的时候需要*2+1
    mHashes[index] = hash;
    mArray[index<<1] = key;
    mArray[(index<<1)+1] = value;
    mSize++;
    return null;
}

{% endhighlight %}

上面的逻辑很清晰，看完基本能了解算法了，也能猜测出个大概了：首先把key计算hashcode并转换成一个index，然后判断是否已经存在index，存在则覆盖，否则判断是否需要扩容和移动数组，最后把index值和key-value对设置在mHashes和mArray数组即可。

#### indexOf方法

再看关键方法indexOf()

{% highlight java %}

int indexOf(Object key, int hash) {
    final int N = mSize;
    // 如果size为0，不需要查找，立刻返回
    if (N == 0) {
        return ~0;
    }
    //然后从mHashes数组里面二分查找hash值。
    //显然，mHashes数组存的是key的hashcode，并且是按顺序存放的
    //没找到则立即返回
    int index = ContainerHelpers.binarySearch(mHashes, N, hash);
    if (index < 0) {
        return index;
    }
    //hashcode虽然相同了，但是可能会冲突，还需要比较key是否一致。都一致才立即返回
    //关键知识来了，map里面判断是需要重写hashcode和equals两个方法的。
    if (key.equals(mArray[index<<1])) {
        return index;
    }
    //如果hashcode冲突了，下面开始寻找所有相同的hashcode，看看是否有key也是equals的，存在则返回
    //往后查找
    int end;
    for (end = index + 1; end < N && mHashes[end] == hash; end++) {
        if (key.equals(mArray[end << 1])) return end;
    }
    //往前查找
    for (int i = index - 1; i >= 0 && mHashes[i] == hash; i--) {
        if (key.equals(mArray[i << 1])) return i;
    }
    //如果还是没找到的，则证明又一个冲突产生，把最后查找的位置end设置反码返回，之后将会在这个位置插入新值。
    return ~end;
}

{% endhighlight %}

这个方法的逻辑也很清晰，mHashes数组存放的是排序的key的hashcode，先从这里二分查找，如果找到，还需要调用equals判断，确认相同的key以后就返回；如果hashcode冲突了，则继续查找，依然没找到，则是一个新的冲突，返回反码，将会在这里插入新值。

>其中indexOfNull()方法查找的hashcode是0，也就是说，如果key是null的话，默认存在第一个位置。

#### remove/get方法

再看两个重要的方法，都是经常用到的，源码如下：

{% highlight java %}

public V get(Object key) {
    final int index = key == null ? indexOfNull() : indexOf(key, key.hashCode());
    return index >= 0 ? (V)mArray[(index<<1)+1] : null;
}

public V remove(Object key) {
    int index = key == null ? indexOfNull() : indexOf(key, key.hashCode());
    if (index >= 0) {
        return removeAt(index);
    }

    return null;
}

public V removeAt(int index) {
    final Object old = mArray[(index << 1) + 1];
    if (mSize <= 1) {
        if (DEBUG) Log.d(TAG, "remove: shrink from " + mHashes.length + " to 0");
        freeArrays(mHashes, mArray, mSize);
        mHashes = ContainerHelpers.EMPTY_INTS;
        mArray = ContainerHelpers.EMPTY_OBJECTS;
        mSize = 0;
    } else {
        if (mHashes.length > (BASE_SIZE*2) && mSize < mHashes.length/3) {
            final int n = mSize > (BASE_SIZE*2) ? (mSize + (mSize>>1)) : (BASE_SIZE*2);

            if (DEBUG) Log.d(TAG, "remove: shrink from " + mHashes.length + " to " + n);

            final int[] ohashes = mHashes;
            final Object[] oarray = mArray;
            allocArrays(n);

            mSize--;
            if (index > 0) {
                if (DEBUG) Log.d(TAG, "remove: copy from 0-" + index + " to 0");
                System.arraycopy(ohashes, 0, mHashes, 0, index);
                System.arraycopy(oarray, 0, mArray, 0, index << 1);
            }
            if (index < mSize) {
                if (DEBUG) Log.d(TAG, "remove: copy from " + (index+1) + "-" + mSize
                        + " to " + index);
                System.arraycopy(ohashes, index + 1, mHashes, index, mSize - index);
                System.arraycopy(oarray, (index + 1) << 1, mArray, index << 1,
                        (mSize - index) << 1);
            }
        } else {
            mSize--;
            if (index < mSize) {
                if (DEBUG) Log.d(TAG, "remove: move " + (index+1) + "-" + mSize
                        + " to " + index);
                System.arraycopy(mHashes, index + 1, mHashes, index, mSize - index);
                System.arraycopy(mArray, (index + 1) << 1, mArray, index << 1,
                        (mSize - index) << 1);
            }
            mArray[mSize << 1] = null;
            mArray[(mSize << 1) + 1] = null;
        }
    }
    return (V)old;
}

{% endhighlight %}

可以看到这两个方法的关键都是indexOf，首先找到index然后就能确定value并get到返回了。如果是删除操作，则还会移动数组。

其他的关于`MapCollections<K, V> mCollections`这个属性，其实是为了满足标准的map接口而实现的，以后去研究HashMap的时候再看。

### 2.2 结合SparseArray再理解

回想一下SparseArray，它是稀疏矩阵的实现，用两个数组实现，一个存放key(它的key是int类型)，一个存放value；而key是按顺序排列的，当操作的时候用二分查找实现，kv两个数组必须一一对应。

同理在ArrayMap里面，它也是稀疏矩阵的实现，原理都和SparseArray一样的，用两个数组实现，不过key不再是int类型了，而是任意类型，那么int数组存放的就是key的hashcode，并且object数组不仅仅存放value了，还存放了key，所以，它是int数组大小的两倍，两个数组并不是一一对应的了，不过操作依然是根据int数组来二分查找的，结构如下：

<table>
    <tr>
        <td>int数组</td><td>object数组</td>
    </tr>
        <td>key1的hashcode</td><td>key1</td>
    </tr>
    <tr>
        <td>key2的hashcode</td><td>value1</td>
    </tr>
    <tr>
        <td>null</td><td>key2</td>
    </tr>
    <tr>
        <td>null</td><td>value2</td>
    </tr>
</table>

### 2.3 关于缓存

除了上面所说的属性以外，它还有下面几个属性：

{% highlight java %}

/**
 * Caches of small array objects to avoid spamming garbage.  The cache
 * Object[] variable is a pointer to a linked list of array objects.
 * The first entry in the array is a pointer to the next array in the
 * list; the second entry is a pointer to the int[] hash code array for it.
 */
static Object[] mBaseCache;
static int mBaseCacheSize;
static Object[] mTwiceBaseCache;
static int mTwiceBaseCacheSize;

private static final int CACHE_SIZE = 10;

{% endhighlight %}

我们已经知道BASE_SIZE是4，这里又有一个为10的CACHE_SIZE，用来缓存的吗？另外，这四个static的属性又有什么用？之前分析put方法的时候，跳过了allocArrays()和freeArrays()方法，现在来看看里面的具体实现：

{% highlight java %}

private void allocArrays(final int size) {
    if (mHashes == EMPTY_IMMUTABLE_INTS) {
        throw new UnsupportedOperationException("ArrayMap is immutable");
    }
    //如果要分配的数组大小为base size的两倍，即8
    if (size == (BASE_SIZE*2)) {
        synchronized (ArrayMap.class) {
            if (mTwiceBaseCache != null) {
                final Object[] array = mTwiceBaseCache;
                mArray = array;
                mTwiceBaseCache = (Object[])array[0];
                mHashes = (int[])array[1];
                array[0] = array[1] = null;
                mTwiceBaseCacheSize--;
                if (DEBUG) Log.d(TAG, "Retrieving 2x cache " + mHashes
                        + " now have " + mTwiceBaseCacheSize + " entries");
                return;
            }
        }
    //如果要分配的数组大小为base size，即4
    } else if (size == BASE_SIZE) {
        synchronized (ArrayMap.class) {
            if (mBaseCache != null) {
                final Object[] array = mBaseCache;
                mArray = array;
                mBaseCache = (Object[])array[0];
                mHashes = (int[])array[1];
                array[0] = array[1] = null;
                mBaseCacheSize--;
                if (DEBUG) Log.d(TAG, "Retrieving 1x cache " + mHashes
                        + " now have " + mBaseCacheSize + " entries");
                return;
            }
        }
    }

    mHashes = new int[size];
    mArray = new Object[size<<1];
}

private static void freeArrays(final int[] hashes, final Object[] array, final int size) {
    if (hashes.length == (BASE_SIZE*2)) {
        synchronized (ArrayMap.class) {
            if (mTwiceBaseCacheSize < CACHE_SIZE) {
                array[0] = mTwiceBaseCache;
                array[1] = hashes;
                for (int i=(size<<1)-1; i>=2; i--) {
                    array[i] = null;
                }
                mTwiceBaseCache = array;
                mTwiceBaseCacheSize++;
                if (DEBUG) Log.d(TAG, "Storing 2x cache " + array
                        + " now have " + mTwiceBaseCacheSize + " entries");
            }
        }
    } else if (hashes.length == BASE_SIZE) {
        synchronized (ArrayMap.class) {
            if (mBaseCacheSize < CACHE_SIZE) {
                array[0] = mBaseCache;
                array[1] = hashes;
                for (int i=(size<<1)-1; i>=2; i--) {
                    array[i] = null;
                }
                mBaseCache = array;
                mBaseCacheSize++;
                if (DEBUG) Log.d(TAG, "Storing 1x cache " + array
                        + " now have " + mBaseCacheSize + " entries");
            }
        }
    }
}

{% endhighlight %}

在ArrayMap里面找不到给mTwiceBaseCache这些变量赋值的地方，具体的原理目前我也还没弄懂，先留在这里，以后总会触机顿悟的，到时再来记录。

由此我们也知道了，不管是ArrayMap还是SparseArray都是比较轻量级的数据结构，适合那些数据量很小的场景，如果数据量小的话，使用重量级的数据结构，如HashMap等等，会影响性能；对于数据量大的时候就会性能差了，使用重量级的Java原生数据结构即可。