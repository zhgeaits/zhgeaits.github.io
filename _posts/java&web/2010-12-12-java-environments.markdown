---
layout: post
title:  "JDK环境搭建"
date:   2010-12-12 10:00:03
categories: java
supertype: career
type: java&web
---

## 1 什么是JDK、JRE和JVM

JDK：Java Development ToolKit(Java开发工具包)。JDK是整个JAVA的核心，包括了Java运行环境(Java Runtime Envirnment)，一堆Java工具(javac/java/jdbc等)和Java基础的类库(即Java API包括rt.jar)。

我们常常用JDK来代指Java API，Java API是Java的应用程序接口，其实就是前辈们写好的一些java Class，包括一些重要的语言结构以及基本图形，网络和文件I/O等等 ，我们在自己的程序中，调用前辈们写好的这些Class，来作为我们自己开发的一个基础。当然，现在已经有越来越多的性能更好或者功能更强大的第三方类库供我们使用。

JRE：Java Runtime Enviromental(java运行时环境)。也就是我们说的JAVA平台，所有的Java程序都要在JRE下才能运行。包括JVM和JAVA核心类库和支持文件。与JDK相比，它不包含开发工具——编译器、调试器和其它工具。

JVM：Java Virtual Mechinal(JAVA虚拟机)。JVM是JRE的一部分，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。JVM有自己完善的硬件架构，如处理器、堆栈、寄存器等，还具有相应的指令系统。JVM 的主要工作是解释自己的指令集(即字节码)并映射到本地的CPU的指令集或OS的系统调用。Java语言是跨平台运行的，其实就是不同的操作系统，使用不同的JVM映射规则，让其与操作系统无关，完成了跨平台性。JVM对上层的Java源文件是不关心的，它关注的只是由源文件生成的类文件(class file)。类文件的组成包括JVM指令集，符号表以及一些补助信息。

![alt text](/image/jdk.png "jdk")

我们开发的实际情况是：我们利用JDK(调用JAVA API)开发了属于我们自己的JAVA程序后，通过JDK中的编译程序(javac)将我们的文本java文件编译成JAVA字节码，在JRE上运行这些JAVA字节码，JVM解析这些字节码，映射到CPU指令集或OS的系统调用。


## 2 windows下配置JDK

### 2.1 安装
  
Google一下，到官网找到windows版本下载

### 2.2 设置环境变量
  
第一步：我的电脑-->属性-->高级-->环境变量
   
第二步：配置用户变量:
   
	a.新建 JAVA_HOME 
	  值为 C:\Program Files\Java\j2sdk1.5.0 （JDK的安装路径）
	b.修改 PATH
	  添加 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
	c.新建 CLASSPATH
	  值为 .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar

测试:

>开始--->运行--->cmd(曹孟德控制台，o(∩∩)o...哈哈)
  
输入:

>java --version
  
over!~提示你的java版本有木有？  

## 3 Linux下配置JDK

### 3.1 安装
   
Google一下，到官网找到Linux版本下载，下载jdk到你的主目录下，然后用这个命令修改可执行权限，然后执行：

{% highlight vim %}
chmod +x jdk1.6.0_31.bin
./jdk1.6.0_31.bin
{% endhighlight %}

### 3.2 设置环境变量

打开控制台的设置文件：

	sudo vim .bashrc

加入以下：

	#要根据你实际的路径来配置啊，不要随便复制有木有
	export JAVA_HOME="/home/zhangge/jdk1.6.0_31"
	export PATH="$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin"
	export CLASSPATH="$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib"

然后:wq保存退出（：wq是vim命令！～）  

运行命令：  

{% highlight vim %}
source .bashrc
{% endhighlight %}

测试，在terminal下输入

{% highlight vim %}
java -version
{% endhighlight %}

## 4 Mac下配置JDK

### 4.1 安装

Google一下，到官网找到mac版本下载，下载jdk到你的主目录下，下载的是一个dmg的镜像，双击以后就会想window那样安装好了。之后直接可以terminal使用java命令了。

### 4.2 配置环境变量

在主目录下用vim创建.bash_profile文件，然后加入下面一行：

>export JAVA_HOME=$(/usr/libexec/java_home)

注意到是用$()包起来的，说明这个是一个命令。再执行：

>sources .bash_profile

上面的java_home不是一个目录，而是一个连接到了下面这个文件：

>/System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/java_home

怎么发现的，首先locate到java在/usr/bin下面，然后ll java发现，它是连接到这里的：

>/System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/java

然后我们再执行java_home，发现了真正的目录是：

>/Library/Java/JavaVirtualMachines/jdk1.8.0_20.jdk/Contents/Home

## 5 JDK相关命令

### 5.1 查看class字节码

>javap -verbose TestClass

这个命令是翻译class字节码为可读性的文件，就算我们不用这个命令，直接打开class文件也是可以查看的，但是就是很难读而已。使用javap命令还带有注释！

`-help`可以方便查看各种使用参数：

	  -version                 版本信息
	  -v  -verbose             输出附加信息
	  -l                       输出行号和本地变量表
	  -public                  仅显示公共类和成员
	  -protected               显示受保护的/公共类和成员
	  -package                 显示程序包/受保护的/公共类
	                           和成员 (默认)
	  -p  -private             显示所有类和成员
	  -c                       对代码进行反汇编
	  -s                       输出内部类型签名
	  -sysinfo                 显示正在处理的类的
	                           系统信息 (路径, 大小, 日期, MD5 散列)
	  -constants               显示最终常量
	  -classpath <path>        指定查找用户类文件的位置
	  -cp <path>               指定查找用户类文件的位置
	  -bootclasspath <path>    覆盖引导类文件的位置
	  