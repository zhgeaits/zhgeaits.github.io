---
layout: post
title:  "Gradle学习笔记"
date:   2014-08-05 11:00:03
categories: other
type: other
---

这个东西和maven相似，可以对比来学习。还需要学习groovy语言，因为gradle是基于这个的。

**Gradle是什么**

和Maven一样，Gradle只是提供了构建项目的一个框架，真正起作用的是Plugin。Gradle在默认情况下为我们提供了许多常用的Plugin，其中包括有构建Java项目的Plugin，还有War，Ear等。与Maven不同的是，Gradle不提供内建的项目生命周期管理，只是java Plugin向Project中添加了许多Task，这些Task依次执行，为我们营造了一种如同Maven般项目构建周期。

领域驱动设计，Gradle本身的领域对象主要有Project和Task。Project为Task提供了执行上下文，所有的Plugin要么向Project中添加用于配置的Property，要么向Project中添加不同的Task。一个Task表示一个逻辑上较为独立的执行过程，比如编译Java源代码，拷贝文件，打包Jar文件，甚至可以是执行一个系统命令或者调用Ant。另外，一个Task可以读取和设置Project的Property以完成特定的操作。

Gradle的Project从本质上说只是含有多个Task的容器，一个Task与Ant的Target相似，表示一个逻辑上的执行单元。我们可以通过很多种方式定义Task，所有的Task都存放在Project的TaskContainer中。Gradle默认为我们提供了dependencies、projects和properties等Task。dependencies用于显示Project的依赖信息，projects用于显示所有Project，包括根Project和子Project，而properties则用于显示一个Project所包含的所有Property。

gradle的bean和delegate机制。bean机制就是默认给属性生成getter和setter方法，但是我们不需要调用setXX和getXX方法，只需像public那样使用即可。delegate是委派机制，虽然理解了，但是由于不懂groovy，也没多少代码量，不知道作用到底是什么。这两个机制的好处就是增加了可读性。

增量式构建机制：如果我们将Gradle的Task看作一个黑盒子，那么我们便可以抽象出输入和输出的概念，一个Task对输入进行操作，然后产生输出。比如，在使用java插件编译源代码时，输入即为Java源文件，输出则为class文件。如果多次执行一个Task时的输入和输出是一样的，那么我们便可以认为这样的Task是没有必要重复执行的。此时，反复执行相同的Task是冗余的，并且是耗时的。为了解决这样的问题，Gradle引入了增量式构建的概念。在增量式构建中，我们为每个Task定义输入（inputs）和输入（outputs），如果在执行一个Task时，如果它的输入和输出与前一次执行时没有发生变化，那么Gradle便会认为该Task是最新的（UP-TO-DATE），因此Gradle将不予执行。一个Task的inputs和outputs可以是一个或多个文件，可以是文件夹，还可以是Project的某个Property，甚至可以是某个闭包所定义的条件。

API：http://www.gradle.org/docs/current/javadoc/org/gradle/api/tasks/TaskContainer.html

**环境搭建**

**使用**

在默认情况下，Gradle将当前目录下的build.gradle文件作为项目的构建文件。

创建task

也可以调用task的构造方法来创建task，如tasks.create(name: 'hello4')。
{% highlight groovy %}
task myTask
task myTask { configure closure }
task myType << { task action }
task myTask(type: SomeType)
task myTask(type: SomeType) { configure closure }
{% endhighlight %}

配置task

Gradle为每个Task默认定义的Property，比如description，logger等，另外，每一个特定的Task类型还可以含有特定的Property，比如Copy的from和to。可以直接在task里面定义属性，如description="hellotest"，也可以通过闭包方式来定义，例如：
{% highlight groovy %}
task myTask{}
myTask.configure {
   description = "this is hello9"
}
{% endhighlight %}

类型

默认每个task都是DefaultTask类型的对象，我们可以显示定义task的类型，如copyFile(type: Copy)，也可以自定义类型。

依赖关系

Task之间可以存在依赖关系，比如taskA依赖于taskB，那么在执行taskA时，Gradle会先执行taskB，然后再执行taskA，taskA(dependsOn: taskB)。也可以用语句来执行依赖关系，如果taskA.dependsOn taskB。



**命令**

gradle tasks：查看Project中所有的Task  
gradle properties：查看Project中所有的Property

**疑惑**  

定义task的时候，"<<"和doLast和doFirst的区别是什么？  