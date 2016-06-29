---
layout: post
title:  "Android的开发环境搭建"
date:   2013-11-11 11:11:11
categories: android
supertype: career
type: android
---

## 1. 引言

开始学习android了，应该乐于去学习新技术，开阔知识面，某某也说我知识面不够广，虽然他不懂我，但是也从某种程度反射出我的缺点。

## 2. 搭建Java环境

因为android开发是基于Java语言的，所以必须首先搭建Java的环境。这个简单了，去看前面的[blog](http://zhgeaits.me/java/2010/12/12/java-environments.html)就可以了。

## 3. 搭建Android-SDK

Android的Software Develop Kit，和JDK一个意思，是Android的开发环境。直接去官网下载，window/linux/mac， 32位/64位，根据自己的系统来。下载完成以后，直接在本地解压即可。然后可以在环境变量里面配置sdk的目录，方便我们在命令行里面直接使用一些命令，如adb等等。

## 4. 使用IDE

IDE就是集成开发环境嘛。目前首选是eclipse。去官网下载最新解压就能用。

## 5. 安装eclipse的插件ADT

ADT就是Android Develop Tool，要使Eclipse可以识别SDK，支持Android，那么先装ADT插件。

首先去官网找到ADT的更新地址，没变的话就是这个 ：

>https://dl-ssl.google.com/android/eclipse/

然后在eclipse里面

>help-> Install new software

add一个repository，剩下就是下一步的事情了。  

基本上是需要翻墙的，装好就能用了。然后就是去Eclipse里面设置SDK的路径就可以。

另外，其实Android的官网是可以直接下载Eclipse-Bundle版本的，直接包含了最新的ADT和SDK的，是一个Android的集成环境。

## 6. 使用IntelliJ IDEA

这个新IDE最近很火，很强大，但是很多地方我不喜欢，还是喜欢eclipse多一点。就连android都要出一个android studio了，这个studio就是改IDEA的。

去官网下载免费版的安装就能用，而且已经集成android了，不用安装adt，详细使用不介绍了，看我的另外blog有介绍。

## 7. 下载API

搞好IDE后，在IDE里面可以打开Android的SDK manager，其实也可以自己去sdk目录打开的。   
然后就是去下载各种API，这个看网速了，不用全部下载，根据你开发需要来下载api，因为实在太大太多了，一般被墙下载不了。所以翻墙工具少不了，还有就是不要去下载img，就是虚拟机模拟器，Android的模拟器简直就是渣。。。不敢用，还是用真机吧！

## 8. 相关命令

Android系统是基于Linux而来的，当我们电脑连接好手机以后，可以在命令行里面直接执行：

>adb shell

这样就连接android的shell，我们就可以执行一些linux的命令了，例如，执行ps,top等。

### 8.1 常用adb命令

1. 指定device来执行adb shell:

>adb -s devicename shell

2. 安装apk：

>adb install -r D:\test.apk //可以加-s devices参数

3. 拉取文件：

>adb pull /mnt/sdcard/yymobile/logs/logs.txt D:/ 

4. 重定向logcat的日志：

>adb logcat > D:/logcat.txt  

5. 查看系统有哪些activity和栈：

>adb shell dumpsys activity

6. 使用top命令查看cpu占用率

adb shell 以后 top -m 10，还可以结合grep来使用

7.查看内存的使用情况

adb shell dumpsys meminfo <package_name>

### 8.2 其他命令

1. 获取debug keystore的sha1值：

>keytool -list -v -keystore .android/debug.keystore -alias androiddebugkey -storepass android -keypass android  

其中，idea的debug.keystore在sdk目录的.android下。
