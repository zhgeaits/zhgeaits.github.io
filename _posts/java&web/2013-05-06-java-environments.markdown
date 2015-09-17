---
layout: post
title:  "JDK环境搭建!"
date:   2013-05-06 14:00:03
categories: java
type: java&web
---

**JDK安装配置**

**windows下配置**

**安装**  
Google一下，到官网找到windows版本下载

**设置环境变量**  
第一步：我的电脑-->属性-->高级-->环境变量   
第二步：配置用户变量:   
{% highlight xml %}
a.新建 JAVA_HOME 
  值为 C:\Program Files\Java\j2sdk1.5.0 （JDK的安装路径）
b.修改 PATH
  添加 %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin
c.新建 CLASSPATH
  值为 .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
{% endhighlight %}
测试  
开始--->运行--->cmd(曹孟德控制台，o(∩∩)o...哈哈)  
输入java --version  
over!~提示你的java版本有木有？  

**Linux下配置**

**安装**   
Google一下，到官网找到Linux版本下载   
下载jdk到你的主目录下，然后用这个命令修改可执行权限：  
{% highlight xml %}
chmod +x jdk1.6.0_31.bin
设置环境变量
sudo vim .bashrc
加入以下：
#要根据你实际的路径来配置啊，不要随便复制有木有
export JAVA_HOME="/home/zhangge/jdk1.6.0_31"
export PATH="$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin"
export CLASSPATH="$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib"
{% endhighlight %}

然后:wq保存退出（：wq是vim命令！～）  

运行命令：  
{% highlight vim %}
source .bashrc
{% endhighlight %}

测试  
在terminal下输入  
{% highlight vim %}
java -version
{% endhighlight %}
