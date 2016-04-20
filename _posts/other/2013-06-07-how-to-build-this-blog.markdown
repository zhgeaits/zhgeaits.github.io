---
layout: post
title:  "这个Blog是如何搭建的"
date:   2013-06-07 11:00:03
categories: other
supertype: career
type: other
---

## 1 什么是github博客

git是一个重量级的分布式版本控制器，详情看我另外一篇blog；而github则是一个综合性的scm平台，即github是你提交代码的后台，除了管理代码，它还有更多的功能，例如协同开发，建立实验室，建立项目page等等。所以它也是一个开源社区，每个人都可以分享自己的代码，由这个project-page就衍生出了github-page，可以建立个人的blog了。而这个blog是运行在jekyll之上的一个project，只要按照一定的规则建立一个jekyll project，github就会去build这个project，帮你生成一个站点了。

## 2 使用github

### 2.1 注册github

这个不用教了，直接注册一个账号就好了。

### 2.2 安装git

下载git，官方版本，不需要另外的gui版本了，纯命令比较好！呵呵！地址：

> http://git-scm.com/downloads
  
直接安装完毕即可

### 2.3 新建一个project

在官网的初始化一个名为`XXX.github.io`的库,XXX就是你的github账号。这里的详细步骤以后补充

### 2.4 什么是jekyll

它的官网是http://jekyllrb.com/。其实jekyll是一个ruby编写的工具，它为blog而生，它的功能就是把通过Markdown（或者Textile）以及Liquid编写的模板转化成一个完整的可发布的静态网站，当然也可以直接解析html网页文件。所以要搞一个这个东西，就先学习ruby语言，然后学会jekyll项目的目录结构，还学习Markdown之类的标记语言。要在本地跑起来jekyll的站点就必须先搭建ruby环境，安装jekyll。

## 3 Linux下搭建github-blog环境

Todo here

## 4 Windows下搭建github-blog环境

虽然不喜欢windows，但是还是很常用的，也要在windows下搭建这个环境，方便编辑。

### 4.1 安装ruby（赶紧去学习吧！！！）

下载地址：

> http://rubyinstaller.org/downloads/
  
安装过程简单，选择配置环境变量和gui tool即可。我下载的版本是：Ruby 1.9.3

### 4.2 安装Ruby DevKit

还不了解Ruby DevKit是什么，估计就是ruby的开发工具吧，学了ruby就知道了。貌似是集成linux下的make，gcc等命令工具。

下载地址：

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

### 4.3 安装jekyll

运行命令:

> gem install jekyll
  
然后还需要安装Markdown的解释器，这个需要在你的`_config.yml`里面设置

> markdown:rdiscount

然后运行：

> gem install rdiscount

### 4.4 使用语法高亮插件Pygments

如果使用语法高亮插件Pygments，则需要安装Python(有几个版本，不太懂，什么portable python，还有distribute 0.6.49，还有settools等等。。。赶紧学python就懂了。）

这里下载portable python：

> http://portablepython.com/wiki/PortablePython3.2.1.1/
  
安装完毕后把`App\Tools\Scripts`和`App`目录添加到path环境变量

再下载distribute:

> https://pypi.python.org/pypi/distribute#downloads
  
解压到C盘，然后在目录里面运行：

> python distribute_setup.py
  
最后运行

>easy_install Pygments

### 4.5 使用语法高亮引擎Rouge

如果使用另一个语法高亮引擎Rouge。它是ruby写的，所以不需要再安装python了。

>gem install rouge

然后在`_config.xml`配置：
>highlighter: rouge

我还没使用过rouge所以不知道语法怎么样的。

### 4.6 使用Bundle一次性搞完

其实在安装Ruby DevKit以后，我自己的blog不需要再执行其他步骤的了，因为使用bundle更方便，我在项目下配置了Gemfile:

>source 'https://ruby.taobao.org'  
gem 'github-pages'

然后安装bundle:

>gem install bundle

最后update一下就可以了：

>bundle update

完了以后就去跑一下：

>bundle exec jekyll serve --watch

具体Bundle详情看jekyll的官网：

>http://jekyllrb.com/docs/github-pages/

### 4.7 TroubleShooting

一般问题都是版本原因，不要总用最新版本，版本之间兼容问题比较多。

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

## 5 Mac下搭建github-blog环境

### 5.1 更新gem

为了保证少掉错误，最好更新gem到最新版本

>sudo gem update --system

### 5.2 修改源

	gem sources -l
	gem sources --remove http://rubygems.org
	gem sources -a https://ruby.taobao.org/
	gem sources --add http://gems.github.com

### 5.3 安装xcode

因为xcode包含了不少东西，其中我们需要ruby，所以先去appstore安装xcode。但是这个ruby并没有开发工具包的，所以需要安装Ruby on Rails。

还需要在命令行安装xcode的这个工具集合，不然会缺少很多工具导致无法编译：

	xcode-select --install

>其实安装完xcode-select以后，可以先试一下直接安装bundler和rails，如果不行再用rvm来安装。

### 5.4 安装Homebrew

Homebrew是Mac OSX上的软件包管理工具，能在Mac中方便的安装软件或者卸载软件。

	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

如果已经有了homebrew，那么可以先更新升级一下

	brew update
	brew upgrade

然后安装rails必须的一些第三方库

	brew install libxml2 libxslt libiconv

### 5.4 安装rvm

xcode包含的ruby是没有开发工具的，完全不能用来安装jekyll的，需要用rvm来安装ruby。gem是ruby的程序，用来安装啊ruby软件的。而rvm是装各种版本ruby的，是个ruby版本管理器。bundle是rails框架里面安装Gemfile指定的各种库的工具，相当于一个套件。

	curl -L https://get.rvm.io | bash -s stable

完了以后会有提示，其实已经加入环境变量了的，但是当前的shell还是没有更新，所以可以执行：

	source ~/.rvm/scripts/rvm
	rvm --version

### 5.5 安装ruby

需要指定一个ruby版本，例如2.2.3

	rvm install 2.2.3

有可能安装不了，因为被墙了，所以下载不了二进制包，提示：

>No binary rubies available for: osx/10.10/x86_64/ruby-2.2.3

可以查看这个remote文件`~/.rvm/config/remote`，获取下载地址，然后翻墙下载，再用mount命令安装：

	rvm mount ~/Downloads/ruby-2.2.3.tar.bz2

最后设置使用ruby的版本：

	rvm 2.2.3 --default
	ruby -v
	gem -v

再安装bundler和rails

	gem install bundler  
	gem install rails

### 5.6 安装jekyll

	sudo gem install jekyll

跑一个：

	jekyll serve --watch

### 5.7 使用Bundle一次性搞完

安装完ruby，即5.5步骤以后，可以直接使用bundle来搞了。

	gem install bundle

最后切换到blog目录update一下就可以了：

	bundle update

完了以后就去跑一下：

	bundle exec jekyll serve --watch

具体Bundle详情看jekyll的官网：
>http://jekyllrb.com/docs/github-pages/

### 5.8 TroubleShooting

一般问题不是网络就是ruby的开发工具没安装，即没安装ruby on rails。只要使用rvm成功安装ruby即可。

**问题1**

安装完xcode以后虽然有ruby，然后就安装jekyll，报下面的错，原因就是没有ruby开发工具，使用rvm安装

	ERROR:  Error installing jekyll:
	ERROR: Failed to build gem native extension.

    /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby -r ./siteconf20151014-4264-1hl3zb6.rb extconf.rb

**问题2**

安装完xcode以后虽然有ruby，然后就安装bundle，报下面的错，原因就是没有ruby开发工具，使用rvm安装

	An error occurred while installing RedCloth (4.2.9), and Bundler cannot continue.
	Make sure that `gem install RedCloth -v '4.2.9'` succeeds before bundling.

## 6 相关命令

	ruby dk.rb init  
	ruby dk.rb install  
	gem sources -l  
	gem sources -remove http://rubygems.org/  
	gem sources -a https://ruby.taobao.org/
	gem sources -a http://gems.github.com

指定在某个源来安装，例如
> gem install jekyll -source http://ruby.taobao.org