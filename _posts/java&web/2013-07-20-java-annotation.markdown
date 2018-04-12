---
layout: post
title: "Java系列之注解Annotation"
date: 2013-07-20 13:12:54
categories: java
supertype: career
type: java&web
---

## 1 理解注解

接触注解是在学习Hibernate，Mybatis和JUnit的时候，那时候学习框架，配置信息除了可以写在xml文件，也可以通过注解配置，特别是ORM的框架，通过注解就可以把对象映射到关系表了。而JUnit里面也是通过一个简单的@Test就标记一个单元测试方法了。而其实，在学习Java的时候就已经接触了，因为JDK1.5以后就已经支持注解，并到处都是充斥着注解的痕迹，只不过那时候并不了解，只是当做一个语法。

- @Override 表示覆盖超类的方法，当然也可以不写，如果写了就不可以写错覆盖的方法，不然编译器会报错。另外在我学习OC的时候，终于我觉得Java的这个注解超好用了~
- @Deprecated 表示过时了的方法或者类，如果你使用过时的，编译器会发出警告信息。
- @SuppressWarnings 关闭不当的编译器警告信息。这个我以前在Eclipse很经常使用，用于消除黄色三角感叹号的警告，哈哈~

其实，注解就是元数据，帮助我们在代码中添加信息，使我们在以后的某个时刻非常方便地使用这些数据。老实说，注解是Java学习C#的（有时候觉得其实C#真是蛮强大的，泛型也是比Java的强大！）。至于注解了以后，我们后期是怎么处理这些注解的，Java提供的反射帮助了我们，通过反射可以轻松获得这些注解了。

## 2 基本语法

注解就像一个接口一样，也是一个Java文件，需要我们来编写定义；然而定义一个注解，需要用到jdk提供的四种元注解；加上以上的三个注解，JDK一共提供给我们7个注解。

### 2.1 元注解

- @Target 表示该注解可以用于什么地方。参数填的是ElementType，它的值包括：CONSTRUCTOR构造器，FIELD域声明，LOCAL_VARIABLE局部变量声明，METHOD方法，PACKAGE包声明，PARAMETER参数声明，TYPE类、接口、枚举声明。
- @Retention 表示需要在什么级别保存该注解信息。参数填的是RetentionPolicy，它的值包括：SOURCE源文件，会被编译器丢弃；CLASS注解在class文件可用，但会被VM丢弃；RUNTIME运行期也保留，因此可以通过注解获取注解信息。
- @Documented 将此注解包含在javadoc中。
- @Inherited 允许子类继承父类中的注解。

### 2.2 定义注解

下面定义一个Test注解

{% highlight java %}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ZGTest {
	public int id();
	public String desc() default "";
}

{% endhighlight %}

可以看到定义了一个@ZGTest注解，关键的语法是`@interface`，和接口有点区别。
于是就可以在任何方法上使用如这样使用了`@ZGTest(id = 1)`；这是一个运行期的注解，另外，如果要在所有ElementType上都可以用这个注解，那么就不用写@Target了。

注解里面的属性写法像接口的方法，使用这些属性的时候就像方法一样调用来使用；用public修饰，如果换其他修饰符会怎么样？会报错，只能用public或者不用修饰符。

如果属性有默认值需要用default的写法，因为注解不能用null值的，编译器报错。

属性的类型除了可以是基本类型以外，还可以是class，enum和Annotation，没错，就是可以嵌套注解，不过很少这样用的。

如果属性的名字是`value`的话，就可以省略用k=v这种写法了，直接向Target那样填充一个值就好了。

最后，注解不支持继承！

## 3 处理注解

如果自定义了注解，那么就需要自行编写注解处理器，不然自定义的注解就没用使用的意义了；上面说到使用反射来实现的，且看下面代码：

{% highlight java %}

public class AnnotationTest {
	public static void main(String[] args) {
		test();
	}
	private static void test() {
		for(Method method : AnnotationTest.class.getMethods()) {
			ZGTest zt = method.getAnnotation(ZGTest.class);
			if(zt != null) {
				//to do a lot of things here....
				System.out.println(method.getName());
				System.out.println(zt.desc());
				System.out.println(zt.id());
			}
		}
	}
	@ZGTest(id = 1, desc = "hello world")
	public void add() {
	}
}

{% endhighlight %}

可以看到使用getAnnotation()方法获取注解，通过class就能获取方法，field和注解等等信息，这就是反射的强大之处；然后我们就可以根据实际情况获取得到的注解方法，类，属性等等做更多的处理了。

其实一个class就是一个一系列对象的集合，包括Type，Field，Method等，于是，其实，这个时候我们就可以应用观察者模式来做事情了。

>一个访问者会遍历某个数据结构或一个对象的集合，对其中的每一个对象执行一个操作。该数据结构无需有序，而你对每个对象执行的操作，都是特定于此对象的类型。这就是将操作与对象解耦，也就是说，你可以添加新的操作，而无需向类的定义中添加方法。

典型的观察者模式，我们可以采用注解来实现。一个Android的框架EventBus就是经典的实现，可以去参考。不过EventBus旧的版本是用反射实现的，3.0以后就是不是了，采用了APT技术，效率更高。

## 4 APT处理器

这是jdk提供的命令行工具，可以整合多个注解处理器运行，也需要编写复杂的代码，目前并不需要学习这个东西。