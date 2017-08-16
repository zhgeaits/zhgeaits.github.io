---
layout: post
title: "Analysing an Interesting Java's Problem About foreach loop"
date: 2017-03-21 13:12:54
categories: java
supertype: career
type: java&web
---

## 背景

前几天，和朋友讨论起下面这个奇怪的Java问题，当时我还没想过来，甚是觉得奇怪，然后也写起代码来测试一下，果然真是！

{% highlight java %}

List<String> datas = new ArrayList<>();
datas.add("1");
datas.add("2");
String target = "1";
for (String temp : datas) {
	if (temp.equals(target)) {
		datas.remove(temp);
	}
}

{% endhighlight %}

问这代码会不会报错？如果把target换成“2”又怎么样？

当时脑袋没怎么想，就说，target是“1”会报错，换成“2”就不会报错。结果当然是反过来的，我们甚是觉得惊讶。自己去写代码的时候才发现，把target换成“2"以后报这个错：

{% highlight java %}

Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
	at java.util.ArrayList$Itr.next(ArrayList.java:851)
	at org.zhangge.testes.TestIterator.test(TestIterator.java:24)
	at org.zhangge.testes.TestIterator.main(TestIterator.java:15)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)

{% endhighlight %}

## 脑袋发热原因

当时没认真想明白foreach循环的本质是什么，我只记得，大学学习计算机编译原理的时候，在c里面，用foreach的话会经过编译器的优化，然后当然效率就高一些咯，但是具体汇编是什么样的，还真没去看过。

又让我想起当年面试的时候被问到Java相关的问题：

>当遍历一个列表，然后满足条件需要修改列表长度的时候怎么做？

菜鸟的我回答说，用两个列表来实现呗，满足条件就复制过去就好了。再被问还有别的方法不，再答是倒序遍历。再要别的方法的时候，想不出来了。面试官给了一个方向，往Iterator方面去想。当时一脸懵逼，噢，原来Iterator是可以实现删除的？难道是复制了一份么？

测试代码如下：

{% highlight java %}

List<String> datas = new ArrayList<>();
datas.add("1");
datas.add("2");
datas.add("3");
String target = "2";
Iterator<String> iterator = datas.iterator();
while (iterator.hasNext()) {
	if (iterator.next().equals(target)) {
		iterator.remove();
	}
	System.out.println(datas);
}

{% endhighlight %}

看了一下remove的接口，说明如下：

>Removes from the underlying collection the last element returned
by this iterator (optional operation).  This method can be called
only once per call to {@link #next}.  The behavior of an iterator
is unspecified if the underlying collection is modified while the
iteration is in progress in any way other than by calling this
method.

只能删除最后一个返回元素，即调用了next()以后的元素。也没有add()等方法，只有remove()方法，需要注意的是，调用的是iterator的remove()方法，不是List的remove()方法。于是也就只记住了这个，并没有去研究为什么，当然在多线程的环境下，使用CopyOnWriteArrayList解决，这都是网上找到的解决方法。

## foreach的本质

上面提到的删除问题，如果是普通的for循环会是怎么样的？下面代码会报错吗？

{% highlight java %}

List<String> datas = new ArrayList<>();
datas.add("1");
datas.add("2");
datas.add("3");
String target = "2";
for (int i = 0; i < datas.size(); i ++) {
	String temp = datas.get(i);
	if (temp.equals(target)) {
		datas.remove(temp);
	}
	System.out.println(datas);
}

{% endhighlight %}

可能我们潜意识会认为，删除了一个元素以后就修改了就会修改列表大小，然后至少会有一个`OutOfIndexBoundsException`，或者是上面的`ConcurrentModificationException`。

事实上是不会报错的，如果我们写过汇编就知道循环的逻辑是怎么实现的，这个用do while循环就比较好理解，而本质上，for循环最终都是转换成do while循环的。每次运行完毕循环体，执行索引加一以后就会执行条件判断。我们发现size()方法返回的只是list里面的size的一个属性值。而其实每次remove一个元素这个值都会修改的。因此，并不会抛出异常，但是下一次循环操作的数据就是脏乱的数据了，因为列表发生了移动。因此就用我上面面试回答的方式来解决。

**foreach**在Java到底是什么呢？思考到这里了，很容易就猜想，foreach循环就是转换成了Iterator来使用。

编写以下的java代码，分别用三种形式来遍历列表：

{% highlight java %}

public class TestIterator {

	private List<String> datas = new ArrayList<>();

	public void testSimpleFor() {
		for (int i = 0; i < datas.size(); i++) {
			String temp = datas.get(i);
			System.out.println(temp);
		}
	}

	public void testForeach() {
		for (String temp : datas) {
			System.out.println(temp);
		}
	}

	public void testIterator() {
		Iterator<String> taskIterator = datas.iterator();
		while (taskIterator.hasNext()) {
			String temp = taskIterator.next();
			System.out.println(temp);
		}
	}
}

{% endhighlight %}

然后，我们在分析一下编译出来的class字节码，关于Class字节码的相关知识，以前写过一篇[blog](http://zhgeaits.me/java/2013/04/08/java-classloader-study-notes.html)。可以去看看，就不详细说了。然后用javap命令来翻译class字节码，使用看以前写[blog](http://zhgeaits.me/java/2010/12/12/java-environments.html)。

执行`javap -c TestIterator`命令，结果如下：

{% highlight java %}

Compiled from "TestIterator.java"
public class org.zhangge.testes.TestIterator {
  public org.zhangge.testes.TestIterator();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: new           #2                  // class java/util/ArrayList
       8: dup
       9: invokespecial #3                  // Method java/util/ArrayList."<init>":()V
      12: putfield      #4                  // Field datas:Ljava/util/List;
      15: return

  public void testSimpleFor();
    Code:
       0: iconst_0
       1: istore_1
       2: iload_1
       3: aload_0
       4: getfield      #4                  // Field datas:Ljava/util/List;
       7: invokeinterface #5,  1            // InterfaceMethod java/util/List.size:()I
      12: if_icmpge     42
      15: aload_0
      16: getfield      #4                  // Field datas:Ljava/util/List;
      19: iload_1
      20: invokeinterface #6,  2            // InterfaceMethod java/util/List.get:(I)Ljava/lang/Object;
      25: checkcast     #7                  // class java/lang/String
      28: astore_2
      29: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
      32: aload_2
      33: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      36: iinc          1, 1
      39: goto          2
      42: return

  public void testForeach();
    Code:
       0: aload_0
       1: getfield      #4                  // Field datas:Ljava/util/List;
       4: invokeinterface #10,  1           // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
       9: astore_1
      10: aload_1
      11: invokeinterface #11,  1           // InterfaceMethod java/util/Iterator.hasNext:()Z
      16: ifeq          39
      19: aload_1
      20: invokeinterface #12,  1           // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
      25: checkcast     #7                  // class java/lang/String
      28: astore_2
      29: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
      32: aload_2
      33: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      36: goto          10
      39: return

  public void testIterator();
    Code:
       0: aload_0
       1: getfield      #4                  // Field datas:Ljava/util/List;
       4: invokeinterface #10,  1           // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
       9: astore_1
      10: aload_1
      11: invokeinterface #11,  1           // InterfaceMethod java/util/Iterator.hasNext:()Z
      16: ifeq          39
      19: aload_1
      20: invokeinterface #12,  1           // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
      25: checkcast     #7                  // class java/lang/String
      28: astore_2
      29: getstatic     #8                  // Field java/lang/System.out:Ljava/io/PrintStream;
      32: aload_2
      33: invokevirtual #9                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      36: goto          10
      39: return
}

{% endhighlight %}

由上面的字节码看出来，testForeach和testIterator生成的字节码一模一样！这样，研究的问题就转换到了Iterator了。其实还是可以延伸一个问题，如果是数组使用foreach会是怎么样的呢？

猜想一下下面的字节码是怎么样的？

{% highlight java %}

public void testArray() {
	String[] array = new String[datas.size()];
	datas.toArray(array);
	for (String temp : array) {
		System.out.println(temp);
	}
}

public void testArrayOppsite() {
	String[] array = new String[datas.size()];
	datas.toArray(array);
	for (int i = 0; i < array.length; i++) {
		String temp = array[i];
		System.out.println(temp);
	}
}

{% endhighlight %}

执行javap翻译class字节码以后：

{% highlight java %}

public void testArray();
Code:
   0: aload_0
   1: getfield      #4                  // Field datas:Ljava/util/List;
   4: invokeinterface #21,  1           // InterfaceMethod java/util/List.size:()I
   9: anewarray     #14                 // class java/lang/String
  12: astore_1
  13: aload_0
  14: getfield      #4                  // Field datas:Ljava/util/List;
  17: aload_1
  18: invokeinterface #23,  2           // InterfaceMethod java/util/List.toArray:([Ljava/lang/Object;)[Ljava/lang/Object;
  23: pop
  24: aload_1
  25: astore_2
  26: aload_2
  27: arraylength
  28: istore_3
  29: iconst_0
  30: istore        4
  32: iload         4
  34: iload_3
  35: if_icmpge     58
  38: aload_2
  39: iload         4
  41: aaload
  42: astore        5
  44: getstatic     #19                 // Field java/lang/System.out:Ljava/io/PrintStream;
  47: aload         5
  49: invokevirtual #24                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
  52: iinc          4, 1
  55: goto          32
  58: return

public void testArrayOppsite();
Code:
   0: aload_0
   1: getfield      #4                  // Field datas:Ljava/util/List;
   4: invokeinterface #21,  1           // InterfaceMethod java/util/List.size:()I
   9: anewarray     #14                 // class java/lang/String
  12: astore_1
  13: aload_0
  14: getfield      #4                  // Field datas:Ljava/util/List;
  17: aload_1
  18: invokeinterface #23,  2           // InterfaceMethod java/util/List.toArray:([Ljava/lang/Object;)[Ljava/lang/Object;
  23: pop
  24: iconst_0
  25: istore_2
  26: iload_2
  27: aload_1
  28: arraylength
  29: if_icmpge     49
  32: aload_1
  33: iload_2
  34: aaload
  35: astore_3
  36: getstatic     #19                 // Field java/lang/System.out:Ljava/io/PrintStream;
  39: aload_3
  40: invokevirtual #24                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
  43: iinc          2, 1
  46: goto          26
  49: return  

{% endhighlight %}

虽然上面的指令有区别，但是明显看到了`arraylength`这条指令，说明差别不是很大，可能有优化，这点我目前就没有深入了解了，要读过jvm规范才知道这些指令的区别了。

## 关于Iterator

到这里，要知道最初那个问题的原因，就必须了解Iterator了。当然，上去google一下ConcurrentModificationException，会发现已经有很多blog分析这个问题了，可是我还是决定自己也读读源码，研究一番。

直接查看ArrayList的iterator():

{% highlight java %}
/**
 * Returns an iterator over the elements in this list in proper sequence.
 *
 * <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
 *
 * @return an iterator over the elements in this list in proper sequence
 */
public Iterator<E> iterator() {
    return new Itr();
}
{% endhighlight %}

上面看到是直接new了一个Itr的类，并且注释里面提到了`fail-fast`。具体是什么东西，暂时先看不懂，直接去看Itr这个类。它是很短的一个内部类，实现了Iterator接口：

{% highlight java %}

/**
 * An optimized version of AbstractList.Itr
 */
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        // update once at end of iteration to reduce heap write traffic
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
{% endhighlight %}

因此看到，它实际操作的还是List的数据，这就推翻了我之前认为iterator复制了一份数据的猜想。

看到hasNext()方法，很简单的判断游标cursor是否等于list的size。再看next()方法，首先执行了方法checkForComodification()，看到里面的实现，如果modCount不等于expectedModCount就抛出了ConcurrentModificationException。接着继续判断游标是否大于size，则抛出NoSuchElementException；最后判断游标是否大于data数组的length，则继续抛出ConcurrentModificationException，这个判断出错概率低，一般认为size和数组的length是相等的，**除非出现多线程的情况，修改了数组还没有修改size，所以它不是线程安全的**。

关键在于modCount和expectedModCount了。

可以看到expectedModCount是Itr的变量，初始值就是等于modCount的。在Itr里面expectedModCount的值只有在执行remove的时候发生了改变，而且也是赋值为modCount。看看modCount的定义如下：

{% highlight java %}

/**
 * The number of times this list has been <i>structurally modified</i>.
 * Structural modifications are those that change the size of the
 * list, or otherwise perturb it in such a fashion that iterations in
 * progress may yield incorrect results.
 *
 * <p>This field is used by the iterator and list iterator implementation
 * returned by the {@code iterator} and {@code listIterator} methods.
 * If the value of this field changes unexpectedly, the iterator (or list
 * iterator) will throw a {@code ConcurrentModificationException} in
 * response to the {@code next}, {@code remove}, {@code previous},
 * {@code set} or {@code add} operations.  This provides
 * <i>fail-fast</i> behavior, rather than non-deterministic behavior in
 * the face of concurrent modification during iteration.
 *
 * <p><b>Use of this field by subclasses is optional.</b> If a subclass
 * wishes to provide fail-fast iterators (and list iterators), then it
 * merely has to increment this field in its {@code add(int, E)} and
 * {@code remove(int)} methods (and any other methods that it overrides
 * that result in structural modifications to the list).  A single call to
 * {@code add(int, E)} or {@code remove(int)} must add no more than
 * one to this field, or the iterators (and list iterators) will throw
 * bogus {@code ConcurrentModificationExceptions}.  If an implementation
 * does not wish to provide fail-fast iterators, this field may be
 * ignored.
 */
protected transient int modCount = 0;

{% endhighlight %}

上面的意思就是，这个变量记录了list结构上的修改次数，结构上的改变就是改变size，否则的话会引起iterator的错误结果。这个属性主要用在了iterator上，并且如果这个值被异常修改，那么iterator就会抛出ConcurrentModificationException，如此一来，就可以fail-fast，这样比非确定性的错误性能高。这样也处理了多线程引发的错误，当多线程操作一个list的时候，当别的线程操作了list，发生了结构的变化，本线程就能即时检查出来，快速抛出异常，而不会去操作脏数据导致数据错误。

也可以从代码里面看到modCount在add，remove等方法都会发生修改。

在ArrayList类的头部还能找到下面这个注释：

	 * <p><a name="fail-fast">
	 * The iterators returned by this class's {@link #iterator() iterator} and
	 * {@link #listIterator(int) listIterator} methods are <em>fail-fast</em>:</a>
	 * if the list is structurally modified at any time after the iterator is
	 * created, in any way except through the iterator's own
	 * {@link ListIterator#remove() remove} or
	 * {@link ListIterator#add(Object) add} methods, the iterator will throw a
	 * {@link ConcurrentModificationException}.  Thus, in the face of
	 * concurrent modification, the iterator fails quickly and cleanly, rather
	 * than risking arbitrary, non-deterministic behavior at an undetermined
	 * time in the future.
	 *
	 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
	 * as it is, generally speaking, impossible to make any hard guarantees in the
	 * presence of unsynchronized concurrent modification.  Fail-fast iterators
	 * throw {@code ConcurrentModificationException} on a best-effort basis.
	 * Therefore, it would be wrong to write a program that depended on this
	 * exception for its correctness:  <i>the fail-fast behavior of iterators
	 * should be used only to detect bugs.</i>

大概也是同样的意思。

## 最初的现象本质

到这里就能解释最初的问题现象了。

{% highlight java %}

List<String> datas = new ArrayList<>();
datas.add("1");
datas.add("2");
String target = "1";
for (String temp : datas) {
	if (temp.equals(target)) {
		datas.remove(temp);
	}
}

{% endhighlight %}

当target是”1”的时候，remove了一个元素之后，modCount和size都发生了改变，再次进入循环需要调用hasNext()方法来判断，此时是不会进入循环的了，因为执行next()方法的时候cursor+1了的，cursor和size相等了。

当target是“2”的情况，remove了一个元素之后，modCount和size都发生了改变，注意到这个时候，size是变小了，size=1; cursor却加了两次，cursor=2；调用hasNext()判断返回时true，依然进入循环，这时候再次调用next()方法就出错了!

**记住原理，不要记住结论！**

到这里我知道了foreach是iterator的本质，而ConcurrentModificationException则是fail-fast机制结果，fail-fast机制则是由modCount引起。只要掌握了这些原理，就可以了解释遇到的问题！不然，我现在问你，问题中的list由两个元素增加到3，4，甚至多个元素的时候，remove哪些会抛异常，哪些不会异常呢？
