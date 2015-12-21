---
layout: post
title:  "开源的单元测试框架JUnit学习笔记!"
date:   2011-08-10 11:00:03
categories: java
type: java&web
---

>当时飞哥二号学习完JUnit后告诉我们这个单元测试，那是第一次知道这个概念，飞哥用的还是3.0版本的，很不好用，代码很多。然后暑假我看马士兵的视频，学习了4.0的JUnit来使用。

## 1 单元测试

学校里面学习软件测试这个课程的时候，里面包含的学问是非常大的，单元测试只是里面的一部分而已，如果要精通那些测试，那就是一个出色的测试工程师了。单元测试其实就是白盒测试，在知道代码逻辑分支的前提下设计出一个个的单元测试用例；所以说，其实单元测试也是程序员做的事情！更多测试的理论知识，以后复习软件测试的时候再笔记补充！

单元测试在开发过程中的作用，除了是验证自己做的功能没问题外，主要是用于极限编程的时候，我们会首先构思程序的功能，然后编写思考，编写测试用例，再编写代码，最后进行测试。这样我们考虑到的问题就比较全面，不会因为在写逻辑的时候而忽略掉了一些情况，而实际的情况是一个反复的过程，经常会在编写代码的时候才想到更多的异常情况，不断地去完善测试用例。由此可以知道开发过程中我们的工作量非常大，但是，这样的效果是我们的代码更加稳定，健壮性更好！极限编程结合结对编程是比较好的，编写测试用例和编写功能代码同时进行，而且两个开发者可以进行互补，交替补充！

另外单元测试在重构的时候非常有用，如果之前开发的时候就进行了单元测试，那么在重构的时候跑一边这些用例即可验证重构的结果。

## 2 JUnit

J显然就是java的意思，这个框架是用java实现的，而且主要用于java的开发中。JUnit框架的好处包括不需要认为观察输出确定是否正确，而且是可以同时运行所有的测试用例。官方网站是：http://junit.org/，上面有详细的文档！因为它开源的，所以在github上面直接就有源码了，有机会还是可以去好好学习源码的。

### 2.1 下载配置

在github上面下载junit.jar和hamcrest-core.jar这两个jar包，当然也可以直接使用maven和gradle进行依赖。把这两个jar包放到classpath既可。eclipse上这些都是简单的操作罢了。

### 2.2 编写测试用例

随便创建一个功能类，例如：

{% highlight java %}

public class ZhangGe {

	public int doAdd(int a, int b) {
		return a + b;
	}
}

{% endhighlight %}

然后我们需要编写一个测试类，例如：

{% highlight java %}

import static org.junit.Assert.assertEquals;

import org.junit.BeforeClass;
import org.junit.Test;

public class ZhangGeTest {
	
	private static ZhangGe zg;
	
	@BeforeClass
	public static void beforeClass() {
		zg = new ZhangGe();
	}

	@Test
	public void testDoAdd() {
		int sum = zg.doAdd(10, 5);
		assertEquals(15, sum);
	}
}

{% endhighlight %}

然后eclipse右键，选择Run as JunitTest就可以看到输出的结果了。可以看到junit是通过注解一个个的方法来实现的：

1.	@Test: 测试方法
2.	@Ignore: 被忽略的测试方法
3.	@Before: 每一个测试方法之前运行
4.	@After: 每一个测试方法之后运行
5.	@BeforeClass: 所有测试开始之前运行
6.	@AfterClass: 所有测试结束之后运行

而hamcrest还是有很多高级的用法的，例如assertThat这个方法，以后会逐渐使用记录在此。