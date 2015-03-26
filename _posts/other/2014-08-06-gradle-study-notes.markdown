---
layout: post
title:  "新秀项目构建神器之Gradle学习笔记"
date:   2014-08-05 11:00:03
categories: other
type: other
---

这个东西和maven相似，可以对比来学习。还需要学习groovy语言，因为gradle是基于这个的。

**Gradle是什么**

Gradle和maven，ant，ivy等都是项目构建工具，Gradle只是提供了构建项目的一个框架，真正起作用的是Plugin。Gradle在默认情况下为我们提供了许多常用的Plugin，其中包括有构建Java，android项目的Plugin等。与Maven不同的是，Gradle不提供内建的项目生命周期管理，只是java Plugin向Project中添加了许多Task，这些Task依次执行，为我们营造了一种如同Maven般项目构建周期。

Gradle是基于DSL（领域特定语言）语法的自动化构建工具。什么是领域驱动设计，现在还是难于理解。Gradle本身的领域对象主要有Project和Task。Project为Task提供了执行上下文，所有的Plugin要么向Project中添加用于配置的Property，要么向Project中添加不同的Task。一个Task表示一个逻辑上较为独立的执行过程，比如编译Java源代码，拷贝文件，打包Jar文件，甚至可以是执行一个系统命令或者调用Ant。另外，一个Task可以读取和设置Project的Property以完成特定的操作。

Gradle的Project从本质上说只是含有多个Task的容器，一个Task与Ant的Target相似，表示一个逻辑上的执行单元。我们可以通过很多种方式定义Task，所有的Task都存放在Project的TaskContainer中。Gradle默认为我们提供了dependencies、projects和properties等Task。dependencies用于显示Project的依赖信息，projects用于显示所有Project，包括根Project和子Project，而properties则用于显示一个Project所包含的所有Property。

gradle的bean和delegate机制。bean机制就是默认给属性生成getter和setter方法，但是我们不需要调用setXX和getXX方法，只需像public那样使用即可。delegate是委派机制，虽然理解了，但是由于不懂groovy，也没多少代码量，不知道作用到底是什么。这两个机制的好处就是增加了可读性。

增量式构建机制：如果我们将Gradle的Task看作一个黑盒子，那么我们便可以抽象出输入和输出的概念，一个Task对输入进行操作，然后产生输出。比如，在使用java插件编译源代码时，输入即为Java源文件，输出则为class文件。如果多次执行一个Task时的输入和输出是一样的，那么我们便可以认为这样的Task是没有必要重复执行的。此时，反复执行相同的Task是冗余的，并且是耗时的。为了解决这样的问题，Gradle引入了增量式构建的概念。在增量式构建中，我们为每个Task定义输入（inputs）和输入（outputs），如果在执行一个Task时，如果它的输入和输出与前一次执行时没有发生变化，那么Gradle便会认为该Task是最新的（UP-TO-DATE），因此Gradle将不予执行。一个Task的inputs和outputs可以是一个或多个文件，可以是文件夹，还可以是Project的某个Property，甚至可以是某个闭包所定义的条件。

API：http://www.gradle.org/docs/current/javadoc/org/gradle/api/tasks/TaskContainer.html

**环境搭建**

配置环境比较简单，只需要下载以后解压，配置环境变量Gradle_HOME，然后加入到PATH即可。

**使用**

在默认情况下，Gradle将当前目录下的build.gradle文件作为项目的构建文件。

* 创建task

也可以调用task的构造方法来创建task，如tasks.create(name: 'hello4')。构造方法入下：  
{% highlight groovy %}
task myTask
task myTask { configure closure }
task myType << { task action }
task myTask(type: SomeType)
task myTask(type: SomeType) { configure closure }
{% endhighlight %}

* 配置task

Gradle为每个Task默认定义的Property，比如description，logger等，另外，每一个特定的Task类型还可以含有特定的Property，比如Copy的from和to。可以直接在task里面定义属性，如description="hellotest"，也可以通过闭包方式来定义，例如：
{% highlight groovy %}
task myTask{}
myTask.configure {
   description = "my name is zhangge's task"
}
{% endhighlight %}

* 依赖关系

Task之间可以存在依赖关系，比如taskA依赖于taskB，那么在执行taskA时，Gradle会先执行taskB，然后再执行taskA，taskA(dependsOn: taskB)。也可以用语句来执行依赖关系，如果taskA.dependsOn taskB。

* 使用java插件

apply plugin: 'java'

java项目的目录结构和maven的一致，也可以自己去定义修改。

* 依赖管理

配置maven的仓库：
{% highlight groovy %}
repositories {
   mavenCentral()
}
{% endhighlight %}  

使用本地maven仓库：  
{% highlight groovy %}
repositories {
    mavenLocal()
}
{% endhighlight %}  

在USER_HOME/.m2下的settings.xml文件中的配置会覆盖存放在M2_HOME/conf下的settings.xml文件中的配置。如果没有settings.xml配置文件，Gradle会使用默认的USER_HOME/.m2/repository地址。  
gradle的本地缓存库在：USER_HOME/.gradle  
但是这个库不像maven的那样，而且也不能像maven那样进行修改。

* 自定义依赖

Gradle将对依赖进行分组，每一组依赖称为一个Configuration，在声明依赖时，我们实际上是在设置不同的Configuration。分组完以后就给不同的组设置不同的依赖。

{% highlight groovy %}
configurations {
   myDependency
}
dependencies {
   myDependency 'org.apache.commons:commons-lang3:3.0'
   myDependency 'com.android.support:appcompat-v7:20.+'
}
{% endhighlight %}  

一般来说，java和android插件都有compile和testCompile分组，为这两个来配置依赖就可以。  
Gradle还允许我们声明对其他Project或者文件系统的依赖：  

{% highlight groovy %}
dependencies {
   compile project(':ProjectB')
   compile files('spring-core.jar', 'spring-aap.jar')
   compile fileTree(dir: 'deps', include: '*.jar')
}
{% endhighlight %}  

* 构建多个项目

要创建多Project的Gradle项目，我们首先需要在根Project中加入名为settings.gradle的配置文件，该文件应该包含各个子Project的名称。比如，我们有一个根Project名为root-project，它包含有两个子Project，名字分别为sub-project1和sub-project2。root-project本身也有自己的build.gradle文件，同时它还拥有settings.gradle文件位于和build.gradle相同的目录下。此外，两个子Project也拥有他们自己的build.gradle文件。  
settings.gradle里面配置：  

{% highlight groovy %}
include 'sub-project1', 'sub-project2'
{% endhighlight %}

在Gradle中，我们可以通过根Project的allprojects()方法将配置一次性地应用于所有的Project，当然也包括定义Task：

{% highlight groovy %}
allprojects {
   apply plugin: 'android' 
   task allTask << {
      println project.name
   }
}
{% endhighlight %}

除了allprojects()之外，Project还提供了subprojects()方法用于配置所有的子Project（不包含根Project）:

{% highlight groovy %}
subprojects {
   task subTask << {
      println project.name
   }
}
{% endhighlight %}

* 类型及自定义task类型

默认每个task都是DefaultTask类型的对象，我们可以显示定义task的类型，如copyFile(type: Copy)，也可以自定义类型。

可以在build.gradle里面直接定义task类型：  
{% highlight groovy %}
class HelloWorldTask extends DefaultTask {
    @Optional
    String message = 'I am zhangge'

    @TaskAction
    def hello(){
        println "hello world $message"
    }
}
task hello(type:HelloWorldTask)
task hello1(type:HelloWorldTask){
   message ="I am a programmer"
}
{% endhighlight %}

另外，还可以在buildSrc下面创建一个helloworld.task来定义，或者新创建一个project来定义，这些以后用到才去记录。现在只需要知道就好了。

* 自定义plugin


**构建android项目**  

官方指南：http://tools.android.com/tech-docs/new-build-system/user-guide  
例子（例如兼容eclipse使用）：https://github.com/Goddchen/Android-Gradle-Examples

使用android studio创建android项目就已经默认支持gradle了，而且生成的文件非常方便。  
buildscript{}：project的运行环境  
android{}：设置编译android项目的参数

打包签名apk，首先我们得要有一个keystore..-_-!!，然后在android{}里加入这些：  
{% highlight groovy %}
signingConfigs {
    myConfig {
        storeFile file("test.keystore")
        storePassword "test"
        keyAlias "test"
        keyPassword "test"
    }
}
    
buildTypes{
    release {
        signingConfig  signingConfigs.myConfig
        runProguard true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.txt'
    } 
}
{% endhighlight %}

* 打多渠道包：  


**命令**

gradle tasks：查看Project中所有的Task  
gradle properties：查看Project中所有的Property  
apply plugin：应用插件  
gradle clean：清除gradle 插件  
gradle build：构建android 项目

**疑惑**  

定义task的时候，"<<"和doLast和doFirst的区别是什么？    
闭包这些还是需要好好理解一下，因为没有学习过groovy，所以比较难理解。