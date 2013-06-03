---
layout: post
title:  "错误学习之Git"
date:   2013-06-04 00:30:00
categories: try-and-error
type: try-and-error
---

>#1.fault: cannot use bin as an exclude file git.#
>刚想提交一个java项目的时候发现提示这个错误，以为是\.gitignore配置错误了，里面的配置过滤的是bin\\，明显是正常的，  
>然后google了一下，只有两条记录，一条是stackoverflow上面的，没帮助！另外一条被墙了，打不开。想了一下后去看看全局配置，  
>发现全局的\.gitignore配置也是对的，而且把全局和项目的这个bin都删掉，还是提示错误，最后看了一下全局的**\.gitconfig**文件，发现里面这条配置：  

{% highlight vim %}
[core]
	excludefile=bin
{% endhighlight %}

>这个估计是在学习git的时候配置的，它的值只能是file，不能是文件夹，所以出错拉，删掉就好了！
