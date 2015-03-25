---
layout: post
title:  "这个Blog是如何搭建的!"
date:   2013-06-07 11:00:03
categories: other
type: other
---

**Windows下的github-blog搭建**

1.下载git，官方版本，不需要另外的gui版本了，纯命令比较好！呵呵  
地址：http://git-scm.com/downloads  
直接安装完毕即可

2.必须得注册一个github账号。

3.在官网的初始化一个名为XXX.github.io的库。这里的详细步骤以后补充

4.安装ruby（赶紧去学习吧！！！）  
下载地址：http://rubyinstaller.org/downloads/  
安装过程简单，选择配置环境变量和gui tool即可

5.安装Ruby DevKit（还不了解是什么，估计就是ruby的开发工具吧，学了ruby就知道了。貌似是集成linux下的make，gcc等命令工具）  
下载地址：http://rubyinstaller.org/downloads/  
一定要下载对应的ruby版本，不然安装不了jekyll的  
解压路径选择：C:\RubyDevKit\  
然后在这个路径运行这个命令：ruby dk.rb init 会生成一个config.xml配置文件,修改这个配置文件，添加ruby的安装路径 - C:/Ruby200-x64  
最后运行这个命令：ruby dk.rb install

6.安装jekyll  
运行命令 gem install Jekyll  
然后还需要安装Markdown的解释器，这个需要在你的_config.yml里面设置markdown:rdiscount  
Gem install rdiscount

7.运行jekyll时候报错gbk编码：invalid byte sequence in GBK  
由于是windows编码的原因,修改方式如下：  
编辑C:\Ruby200-x64\lib\ruby\gems\2.0.0\gems\jekyll-1.4.2\lib\jekyll目录下的convertible.rb文件
{% highlight ruby %}
把
self.content = File.read_with_options(File.join(base, name),
                                              merged_file_read_opts(opts))
改成：
self.content = File.read_with_options(File.join(base, name), :encoding=>"utf-8",
                                              merged_file_read_opts(opts))
或者在_config.yml文件添加一行encoding: utf-8
{% endhighlight %}

8.运行jekyll serve还是有问题，估计是没有安装python（有几个版本，不太懂，什么portable python，还有distribute 0.6.49，还有settools等等。。。赶紧学python就懂了。）  
这里下载portable python：http://portablepython.com/wiki/PortablePython3.2.1.1/  
安装完毕后把App\Scripts和App目录添加到path环境变量  
再下载distributehttps://pypi.python.org/pypi/distribute#downloads  
解压到C盘，然后在目录里面运行：python distribute_setup.py  
最后运行easy_install Pygments

9.如果运行有问题warning:cannot close fd before spawn，估计是pygments的版本问题：  
{% highlight ruby %}
gem uninstall pygments.rb --version "=0.5.1"
gem install pygments.rb --version "=0.5.0"
{% endhighlight %}

10.一般问题都是版本原因，不要总用最新版本，版本之间兼容问题比较多。


**更新**

ruby dk.rb init
ruby dk.rb install


$gem sources -r http://rubygems.org/
$gem sources -a http://ruby.taobao.org/


**错误**

NoMethodError: undefined method `size' for nil:NilClass
An error occurred while installing pygments.rb (0.6.0), and Bundler cannot
continue.
Make sure that `gem install pygments.rb -v '0.6.0'` succeeds before bundling.

在此只需要将gem目录下cache文件夹内清空，重新安装即可。

如果不知道gem目录的可以使用gem env来查看GEM PATHS值，如下所示d:/Ruby200/lib/ruby/gems/2.0.0
