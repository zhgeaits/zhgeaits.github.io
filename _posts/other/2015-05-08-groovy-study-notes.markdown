---
layout: post
title:  "Groovy!"
date:   2015-05-08 11:00:03
categories: other
type: other
---

我觉得Groovy就是一个脚本版的java，在哪里用到，我目前了解的就是gradle，还有以前在TM实习的时候，那里的推荐算法会用到。Groovy引入了闭包和元编程等出色功能，也是比较难理解的。

环境：

eclipse4.4：这个plugin：http://dist.springsource.org/release/GRECLIPSE/e4.4/

语法：

既然是脚本，那就肯定是弱类型啦，和js那样了。def是标示定义。  

定义多行变量，使用三个引号：
{% highlight groovy %}
def var="""hello
       world
       groovy!"""
{% endhighlight %}