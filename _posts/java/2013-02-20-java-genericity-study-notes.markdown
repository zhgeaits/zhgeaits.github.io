---
layout: post
title:  "Java泛型学习笔记!"
date:   2013-02-20 11:00:03
categories: java
type: java
---

大一刚学java时候接触过泛型，但是不是很理解，只记得<T>使用就好，然后参考飞哥的web项目时候也在Base基类接触过一下，但是还是没有认真学习到泛型的东西。现在来总结一下常用的地方。

一般用法：  
一般就是在类，或者接口那里用上，例如：List<String>，或者Map<Integer, String>。然后也可以在自己的Base类使用<T>。

如果使用了Base<T>以后，在方法参数可以这样：  
{% highlight java %}
public void test(Base<?> temp);//任何类型  
public void test(Base<? extends SecondBase> temp);//规定了上限，只能接收SecondBase及其SecondBase的子类     
public void test(Base<? super String> temp);//规定上限，只能接收String或Object类型的泛型  
其实Class类就是这样的用法，例如：HashMap<Class<? extends IBaseCore>, Class<? extends AbstractBaseCore>> coreClasses;  

在方法上使用泛型：  
如果要在方法上使用泛型，那么在方法那里得先声明一下泛型，如下：  
public static <T> T test(T t)；//<T>是声明泛型，这里是任何类型。  
public static <T extends Base> T test(T param);//加入限定  
public static <T extends Base> Info<T> test(Info<T> param);//这样使用也可以
{% endhighlight %}

对于数组，可变参数都可以在泛型用上。
   