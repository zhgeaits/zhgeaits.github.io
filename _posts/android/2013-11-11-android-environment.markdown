---
layout: post
title:  "Android的开发环境搭建!"
date:   2013-11-11 11:11:11
categories: android
type: android
---

开始学习android了，应该乐于去学习新技术，开阔知识面，某某面试官也说我知识面不够广，虽然他不懂，但是也反射我的缺点。

**搭建Java环境**  
这个简单了，去看我的blog的java页面有介绍。

**搭建android-sdk**  
android的software develop kit，和jdk一个意思，是android的开发环境。  
直接去官网下载，window/linux， 32位/64位，根据自己的系统来。下载完成以后，直接在本地解压即可。

**使用IDE**  
IDE就是继承开发环境嘛。目前首选是eclipse，不用介绍了，可以去看我的blog有这一篇。去官网下载最新解压就能用。

**安装eclipse的插件ADT**  
adt就是android develop tool，要做eclipse可以识别sdk，支持android，那么先装adt插件。  
首先去官网找到ADT的更新地址， 没变就是这个 ：https://dl-ssl.google.com/android/eclipse/  
然后在eclipse的help-> Install new software那里，add一个repository，剩下就是下一步的事情了。  
装好就能用了。然后就是去eclipse里面设置sdk的路径就可以。

**使用IntelliJ IDEA**  
这个新IDE最近很火，很强大，但是很多地方我不喜欢，还是喜欢eclipse多一点。就连android都要出一个android studio了，这个studio就是
改IDEA的。  
去官网下载免费版的安装就能用，而且已经集成android了，不用安装adt，详细使用不介绍了，看我的另外blog有。

**使用android官方IDE**  
就是前面是的android studio，还有就是eclipse-bundle，就是捆绑了adt的eclipse啦。我觉得还是自己搞，不用官方的IDE好。

**下载API**  
搞好IDE后，在IDE里面可以打开android的SDK manager，其实也可以自己去sdk目录打开的。   
然后就是去下载各种API，这个看网速了，不用全部下载，根据你开发需要来下载api，因为实在太大太多了，一般被墙下载不了。  
所以翻墙工具少不了，还有就是不要去下载img，就是虚拟机模拟器，android的模拟器简直就是渣。。。不敢用，还是用真机吧！

**ADB**  
跑起项目以后，在命令行运行adb shell以后就能运行linux命令了，例如，执行ps,top等命令