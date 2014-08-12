---
layout: post
title:  "错误学习之Git"
date:   2013-06-04 00:30:00
categories: other
type: other
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
>
>#2.从工作室的RhodeCode的仓库创建一个git项目以后，克隆下来的时候发现这个错误：

{% highlight vim %}
server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificat
{% endhighlight %}

原因证书问题，这个证书没有正式配置吧，所以只能绕过不验证了，在本机配置一个环境变量,和我們使用hg時候那個問題是一致的。

{% highlight vim %}
export GIT_SSL_NO_VERIFY=1
{% endhighlight %}
