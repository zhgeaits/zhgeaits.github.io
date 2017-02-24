---
layout: post
title:  "iOS学习笔记之CocaPods"
date:   2015-09-28 11:00:03
categories: ios
supertype: career
type: ios
---

## 1。 什么是CocoaPods

CocoaPods和Maven一样，是用来管理依赖的，它是ruby编写的，所以有机会还是有必要学习一下Ruby的。因为有过[Maven](http://zhgeaits.me/other/2011/11/20/maven-study-notes.html)，Ant，[Gradle](http://zhgeaits.me/other/2014/08/05/gradle-study-notes.html)的经验，也很容易懂了，在Java的世界里面，由于什么都崇尚开源，所以就是百花齐放的状况，连构建工具，依赖工具都是一个个，而且一个比一个好，每次出新的一个都说比之前谁家的好多少。而在iOS的世界里面，我发现，几乎所有的项目都用CocoaPods，也不见得还有其他的好用的工具，好吧，只有一个，也简单咯！

## 2. 安装CocoaPods

通常使用Maven和gradle，由于是开源的，我们说的是配置环境，其实也就是安装，只不过我们都是下载然后解压，再指定环境变量就可以使用命令了。而由于CocoaPods是Ruby写的，所以在mac环境里面就需要安装到系统里了。

### 2.1 搭建Ruby环境

我在搭建这个blog的时候最初接触到Ruby的，然后也学会了搭建Ruby的环境，[看这里就知道了](http://zhgeaits.me/other/2013/06/07/how-to-build-this-blog.html)。

>注意的是，现在淘宝的源已经不维护了，需要按照指示切换到Ruby-China的源即可。

### 2.2 安装CocoaPods

搭建好Ruby环境以后就可以安装CocoaPods了，命令如下：

>sudo gem install cocoapds

有时候安装并不会那么容易成功，曾经我安装过cocoapods，但是版本太老了，需要升级，执行完毕以后，发现这样的错误：

>ERROR:  While executing gem ... (Gem::DependencyError)
>    Unable to resolve dependencies: cocoapods requires cocoapods-core (= 1.2.0), cocoapods-downloader (< 2.0, >= 1.1.3), cocoapods-trunk (< 2.0, >= 1.1.2), molinillo (~> 0.5.5), xcodeproj (< 2.0, >= 1.4.1)

然后需要升级一下gem，执行命令如下：

>sudo gem update --system

完成以后再次安装cocoapods，发现还是有错：

>ERROR:  While executing gem ... (Errno::EPERM)
>    Operation not permitted - /usr/bin/xcodeproj

然后继续Google，找到一个方法，指定目录安装即可，其他方法没有测试

>sudo gem install -n /usr/local/bin cocoapods

最后执行安装命令就一切结束了：

>pod setup

## 3. 使用CocodPods

就跟Maven的pom.xml和Gradle的build.gradle一样，总是需要一个配置文件，那么我们就需要在iOS项目里面新建一个Podfile文件，内容如下：

	platform :ios  
	pod 'Reachability',  '~> 3.0.0'  
	pod 'SBJson', '~> 4.0.0'  
	  
	platform :ios, '7.0'  
	pod 'AFNetworking', '~> 2.0' 

实际上，这里依赖了三个项目Reachability、SBJson和AFNetworking，其中AFNetworking的平台是7.0。

然后执行命令：

>pod install

就会去下载这三个库的了，并且生成*.xcworkspace、Podfile.lock文件和Pods目录。显然*.xcworkspace就是项目文件，代替了之前的*.xcodeproj，而Pods目录下存放的就是依赖的库的代码。

最后双击*.xcworkspace文件就可以用xcode打开项目了。

## 4. 关于Podfile

Podfile.lock里面记录的是各个依赖项目的版本。创建项目的人会使用`pod install`来安装项目依赖，然后生成了Podfile，然后提交到版本库，当别人checkout出来的时候，也是执行`pod install`命令，那么就会直接根据Podfile.lock里面的版本来install了，否则就不知道安装那个版本，如果默认安装最新版本，那么各个开发成员之间的依赖版本就不一样了。

如果我们更新了某个依赖的版本号，那么我就需要执行命令来更新Podfile.lock

>pod update

## 5. 制作一个依赖项目