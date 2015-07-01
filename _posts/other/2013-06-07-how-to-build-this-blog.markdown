---
layout: post
title:  "这个Blog是如何搭建的"
date:   2013-06-07 11:00:03
categories: other
type: other
---

## Windows下的github-blog搭建

**1.**下载git，官方版本，不需要另外的gui版本了，纯命令比较好！呵呵！地址：

> http://git-scm.com/downloads
  
直接安装完毕即可

**2.**必须得注册一个github账号。

**3.**在官网的初始化一个名为`XXX.github.io`的库,XXX就是你的github账号。这里的详细步骤以后补充

**4.**安装ruby（赶紧去学习吧！！！）下载地址：

> http://rubyinstaller.org/downloads/
  
安装过程简单，选择配置环境变量和gui tool即可。我下载的版本是：Ruby 1.9.3

**5.**安装Ruby DevKit（还不了解是什么，估计就是ruby的开发工具吧，学了ruby就知道了。貌似是集成linux下的make，gcc等命令工具）下载地址：

> http://rubyinstaller.org/downloads/
  
一定要下载对应的ruby版本，不然安装不了jekyll的!我下载的版本是：DevKit-tdm-32-4.5.2

解压路径选择：

> C:\RubyDevKit\

然后在这个路径运行这个命令：

> ruby dk.rb init

之后会生成一个`config.xml`配置文件,一般里面就是ruby的路径，如果路径不对的话（64位）则自行修改这个配置文件，添加ruby的安装路径

> `- C:/Ruby193`
  
最后再运行这个命令：

> ruby dk.rb install

**6.**安装jekyll，运行命令:

> gem install jekyll
  
然后还需要安装Markdown的解释器，这个需要在你的`_config.yml`里面设置

> markdown:rdiscount

然后运行：

> gem install rdiscount

**7.**如果使用语法高亮插件Pygments，则需要安装Python(有几个版本，不太懂，什么portable python，还有distribute 0.6.49，还有settools等等。。。赶紧学python就懂了。）

这里下载portable python：

> http://portablepython.com/wiki/PortablePython3.2.1.1/
  
安装完毕后把`App\Tools\Scripts`和`App`目录添加到path环境变量

再下载distribute:

> https://pypi.python.org/pypi/distribute#downloads
  
解压到C盘，然后在目录里面运行：

> python distribute_setup.py
  
最后运行

>easy_install Pygments

**8.**如果使用另一个语法高亮引擎Rouge。它是ruby写的，所以不需要再安装python了。

>gem install rouge

然后在`_config.xml`配置：
>highlighter: rouge

我还没使用过rouge所以不知道语法怎么样的。

**9.**其实在执行完第5步以后，我自己的blog不需要再执行其他步骤的了，因为使用bundle更方便，我在项目下配置了Gemfile:

>source 'https://ruby.taobao.org'  
gem 'github-pages'

然后安装bundle:

>gem install bundle

最后update一下就可以了：

>bundle update

完了以后就去跑一下：

>bundle exec jekyll serve --watch

具体详情看jekyll的官网：
>http://jekyllrb.com/docs/github-pages/

**10.**一般问题都是版本原因，不要总用最新版本，版本之间兼容问题比较多。


## 命令

> ruby dk.rb init  
ruby dk.rb install  
gem sources -l  
gem sources -remove http://rubygems.org/  
gem sources -a http://ruby.taobao.org/

指定在某个源来安装，例如
> gem install jekyll -source http://ruby.taobao.org

## TroubleShooting

**问题1**

>NoMethodError: undefined method `size' for nil:NilClass
An error occurred while installing pygments.rb (0.6.0), and Bundler cannot
continue.
Make sure that `gem install pygments.rb -v '0.6.0'` succeeds before bundling.

在此只需要将gem目录下cache文件夹内清空，重新安装即可。如果不知道gem目录的可以使用`gem env`来查看GEM PATHS值，如下所示

> d:/Ruby200/lib/ruby/gems/2.0.0

**问题2**

下面一个网络的问题，在天朝底下没法解决，然后我就1.93这个版本的ruby就好了。

>Unable to download data from https://rubygems.org/ - Errno::ECONNABORT ED: An established connection was aborted by the software in your host machine. - SSL_connect (https://rubygems.org/quick/Marshal.4.8/jekyll-2.5.3.gemspec.rz)

当然还有别的解决方法，那就是去掉官方的源，然后用淘宝的源，详细看上面的命令，或者直接看淘宝的ruby网页使用。

**问题3**

如果运行ruby命令出现下面报错：
> The 'fast-stemmer' native gem requires installed build tools.

那就是RubyDevKit没有配置成功。
只需要执行上面第五步的命令就可以了，不过可能需要先执行review再执行install了：

> ruby dk.rb init  
ruby dk.rb review  
ruby dk.rb install

**问题4**

运行jekyll时候报错gbk编码：

> invalid byte sequence in GBK
  
由于是windows编码的原因,修改方式如下,编辑:

> C:\Ruby200-x64\lib\ruby\gems\2.0.0\gems\jekyll-1.4.2\lib\jekyll目录下的convertible.rb文件

{% highlight ruby %}
把
self.content = File.read_with_options(File.join(base, name),
                                              merged_file_read_opts(opts))
改成：
self.content = File.read_with_options(File.join(base, name), :encoding=>"utf-8",
                                              merged_file_read_opts(opts))
或者在_config.yml文件添加一行encoding: utf-8
{% endhighlight %}

**问题5**

运行有问题：

> warning:cannot close fd before spawn

估计是pygments的版本问题：

>gem uninstall pygments.rb --version "=0.5.1"  
gem install pygments.rb --version "=0.5.0"