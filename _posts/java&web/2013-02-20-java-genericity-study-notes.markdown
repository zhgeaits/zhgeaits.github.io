---
layout: post
title:  "Java系列之泛型"
date:   2013-02-20 11:00:03
categories: java
type: java&web
---

>大一刚学java时候接触过泛型，但是不是很理解，只记得<T>使用就好，然后参考飞哥的web项目时候也在Base基类接触过一下，但是还是没有认真学习到泛型的东西。

## 1. 什么是泛型

字面意思就是泛化的类型，就是把类型进行抽象的意思，主要的思想就是编写尽可能广泛应用的代码。试想这样一个问题，如果我编写了一个类，能够适用于多种类型，那么这个类的重用性是非常高。有这样需求的通常是容器类，如链表，表等等。如果我编写了一个链表，但是写死了它的节点类型，那么就无法重用了；过去java是适用多态来解决的，即节点的类型是Object，但是这并不是最好的解决办法，存储的时候会把类型信息丢失了，每次读取节点还需向下转型，很多时候也会带来一些性能损耗。泛型便是要参数化类型，具体类型是通过参数来确定的，它是通过解耦类或者方法与所使用的类型之间的约束实现的。不过，java的类型并不完美，起码与C++的模板，C#的泛型比起来，差太多了，我虽然没学过这两门语言的泛型，但是通过了解java的泛型，java泛型的缺陷。因为，泛型是java1.5才出现了的，也就是说，设计者们必须考虑到兼容过去的版本。但是在jdk9，10的未来，泛型将会有更强的版本出现

## 2. 学会使用

泛型入门非常简单，但是深入了解还是需要费一番功夫的，毕竟设计jdk的那帮人不简单。首先但让是从容器入手。

### 2.1 简单使用

jdk的类库，容器，如List，Map，Set等，过去里面都是存储Object类型的，现在有了泛型支持以后，可以指定里面存储的类型：

>List<String> list = new LinkedList<String>();//jdk1.8以后，直接这样写List<String> list = new LinkedList<>();  
>Map<Integer, String> map = new HashMap<>();  
>Set<Integer> set = new HashSet<>();

### 2.2 自定义使用

如果自己要编写一个工具类，使用到泛型，如下：

{% highlight java %}

public class Animal<T> {
	private T t;
	public Animal(T t) {
		this.t = t;
	}
	public T get() {
		return t;
	}
	//other methods
}

{% endhighlight %}

### 2.3 元组类库

上面的类都只用到了一个泛型T，观察到map是用到了<K,V>两个泛型；而其实是可以使用一组泛型的，称为元组。如：

{% highlight java %}

public class MutipleTuple<A,B,C,D> {
	private A a;
	private B b;
	private C c;
	private D d;
	public MutipleTuple(A a, B b, C c, D d) {
		this.a = a;
		this.b = b;
		this.c = c;
		this.d = d;
	}
}

{% endhighlight %}

### 2.4 接口与继承

接口也是可以使用泛型的，并没有什么特别的，只需要在实现接口的时候传递一个类型参数给接口即可，如：`public class A implements B<String> {}`

继承也是一样，也接口一样的。如果子类或者实现类，也是使用泛型的时候，子类与父类和接口需使用同一个泛型。

## 3 进阶使用

以上简单的使用都是在类上面声明了泛型，并且可以用到方法的返回值和参数中去；但是如果本身类没有用到泛型，那么如何在方法中使用泛型呢？

### 3.1 泛型方法

如果要在方法上使用泛型，那么在方法那里得先声明一下泛型，如下：  

{% highlight java %}

public static <T> T test(T t)；//<T>是声明泛型，这里是任何类型。  
public static <T extends Base> T test(T param);//加入限定  
public static <T extends Base> Info<T> test(Info<T> param);//这样使用也可以，返回值是Info，而Info本身是使用泛型的

{% endhighlight %}

>上面用到的extends，下面会讲到。而方法不一定要static的。

### 3.2 通配符

以上的泛型都是出现在声明或者参数当中的，即类，方法中声明，和在方法参数中使用，这些时候泛型都是采用一些字母，如T(type的缩写)来表示的。当我们在属性或者引用的时候，可以使用通配符`?`。例如，我这样写：

>List<String> list;

这个list是一个引用，这里的List<Sting>也不是一个声明，只是一个引用类型；所以它可以这样使用通配符：

>List<?> list;

而且，在方法上可以这样使用：

{% highlight java %}

public void test(Base<?> temp);//任何类型  
public void test(Base<? extends SecondBase> temp);//规定了上限，只能接收SecondBase及其SecondBase的子类     
public void test(Base<? super String> temp);//规定上限，只能接收String或Object类型的泛型  

//其实Class类也可以是这样的用法，例如：
private HashMap<Class<? extends IBaseCore>, Class<? extends AbstractBaseCore>> coreClasses; 

{% endhighlight %}

### 3.3 边界限定

上面出现的extends和super就是边界的限定，由于java的泛型不能像c++的模板那样直接调一个泛型的未知方法（底层实现的不同，C++在编译具体的模板实现的时候会检查到有这个方法的），java出现了泛型的限定，使得泛型相对的具体化，补偿了擦除的不足。

在声明中只能使用extends，不能使用super，在通配符中可以两者使用。extends后面可以用&来连接多个接口，注意，这并不是多继承，如：

>public class Dog<T extends BaseAnimal & interface1 & interface2> {}

在引用的时候，只有通配符？可以使用extends和super，T泛型不能使用。即：

>List<? extends Zhangge> list1;//正确  
List<T extends Zhangge> list2;//错误

使用容器的时候，如果用到通配符，只能使用super，不能使用extends。如下：

>List<? extends Zhangge> list1;//只能添加null，其他任何对象都不能添加。如此的奇怪  
List<？ super Zhangge> list2;//可以添加Zhangge或者Zhangge的子类。

为什么会这样，可能是因为使用extends的时候，编译器不能安全的检查到添加什么类型，后来使用super解决这个问题，至于深入，待研究罢了。

如果单纯使用<?>，那就是无界通配符了，但是它和原生的又有区别，例如，List和List<?>：List表示持有任何Object类型的原生List；而List<?>表示具有某种特定类型的非原生List，只是我们不知道那种类型是什么而已。根据上面描述，List<?>只能添加null而已。

使用通配符以后（无界或者extends），一般只能get，不能add或者set了。不过可以转型后add或者set。

## 4. 高阶理解

从上面就可以看出，如果不够了解java泛型的话，单纯简单理解就想运用起来，那是比较困难的。也明白了，其实java的泛型是比较局限的。

### 4.1 实例化泛型

试想一下，2.2的代码中，我想要实例化T，怎么办，t = new T();这样吗？在C++和C#中应该是可以的，然而java就不行了，因为T的类型信息被擦除了，这个擦除下面再说，这里先说怎么实例化。

官方是推荐我们使用工厂或者生成器的方法来解决这个问题，下面是一个通用的生成器：

{% highlight java %}

public class BasicGenerator<T> {
	private Class<T> type;
	public BasicGenerator(Class<T> type) {
		this.type = type;
	}
	public T newInstance() {
		try {
			return type.newInstance();
		} catch (InstantiationException | IllegalAccessException e) {
		}
		return null;
	}
	public static <T> BasicGenerator<T> create(Class<T> type) {
		return new BasicGenerator<T>(type);
	}
}

{% endhighlight %}

这样有两点不好，一是T如果没有默认无参数的构造方法就会报错，二是每次使用都必须提供Class对象。

于是，从网上找到一个技巧的方法，据说是用Hibernate那里学到的：

{% highlight java %}

Type genType = getClass().getGenericSuperclass();  
Type[] params = ((ParameterizedType) genType).getActualTypeArguments();  
Class<T> resultType = (Class) params[0];  
T result = resultType.newInstance(); 

{% endhighlight %}

### 4.2 擦除

开头就说了，因为java的设计者要兼容到过去版本的代码，所以做了一个折中，达到迁移兼容性，保留原生的类库。于是乎，在泛型代码内部，无法获取任何有关泛型参数类型的信息，这些类型信息全都被擦除了。也就是说，下面代码是等价的：

>new ArrayList<String>() == new ArrayList<Integer>();

在C++的模板里面，可以直接写t.f();模板是编译通过的，但是在实例化使用这个模板的类的时候，如果对应的泛型T没有f这个方法就会报错。但是，在java中，编译就已经通不过了，就是因为擦除了T的所有信息。

java的泛型不是真正的泛型，是伪泛型，就是说运行期不能将泛型类型与用户定义的普通类型同等对待，例如运行期做反射时无法获得泛型信息。但是C#语言是真正的发支持泛型。

有一个比较蛋疼的例子：

{% highlight java %}

public class ArrayMaker<T> {
	private Class<T> kind;
	public ArrayMaker(Class<T> type) {
		kind = type;
	}
	public T[] create(int size) {
		return (T[])Array.newInstance(kind, size);
	}
}

{% endhighlight %}

上面的代码，虽然存储了Class<T> kind，但是当传到Array.newInstance()来实例化的时候，其实kind的信息是都被删除了的，最后得到的是null数组。

如果要获得数组，必须使用容器：

{% highlight java %}

List<T> list = new ArrayList<T>();
for(int i = 0; i < size; i ++) {
	try {
		list.add(kind.newInstance());
	} catch (InstantiationException | IllegalAccessException e) {
	}
}

{% endhighlight %}

并且不能这样：

>Object[] objs = new Object[size];  
(T[]) list.toArray(objs);

数组不能直接强转，只能一个一个元素地转换类型。
  
## 后记

java的泛型实际上比较多内容，涉及到底层的比较多，而且在以后的java版本中也不会不断的加强，以后我会不断的学习，完善！~