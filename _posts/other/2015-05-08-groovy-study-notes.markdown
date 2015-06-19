---
layout: post
title:  "Groovy!"
date:   2015-05-08 11:00:03
categories: other
type: other
---

**官网**  
http://www.groovy-lang.org/

官网上说groovy是一个多面性（multi-faceted）的java平台语言，它可以像jdk一样安装，也可以当做一个项目在maven那里进行依赖。对于本质还需要再认真理解一下。

我觉得Groovy就是一个脚本版的java，在哪里用到，我目前了解的就是gradle，还有以前在TM实习的时候，那里的推荐算法会用到。Groovy引入了闭包和元编程等出色功能，也是比较难理解的。

**配置环境：**  

eclipse4.4：这个plugin：http://dist.springsource.org/release/GRECLIPSE/e4.4/

像maven一样下载bin包，然后配置环境变量就可以了。

**与Java的不同**  
它会默认import一下这些包，所以可以直接使用这些包里面的类了：  
{% highlight groovy %}
java.io.*
java.lang.*
java.math.BigDecimal
java.math.BigInteger
java.net.*
java.util.*
groovy.lang.*
groovy.util.*
{% endhighlight %}
它还是多方法的（方法名相同），因为它是脚本语言的特性，方法在runtime的时候call，会根据实际的参数类型去调用不同的方法。  
不能使用{}大括号来声明数组了，因为那是闭包，所以使用中括号[]。  


**语法：**

**注释**  
注释和java一样，只不过多了一个Shebang line的注释，其实和shell的第一行一样。  
#!/usr/bin/env groovy

**关键字**  
出了java的那些关键字，还包括const def goto  



既然是脚本，那就肯定是弱类型啦，和js那样了。def是标示定义。  

定义多行变量，使用三个引号：
{% highlight groovy %}
def var="""hello
       world
       groovy!"""
{% endhighlight %}