---
layout: post
title:  "Linux装机配置记录"
date:   2012-03-06 11:00:03
categories: other
supertype: career
type: other
---

经常玩linux，经常装机，就是耗得起，然而很多是通用的东西，而已不断的去做外包的时候也是需要配置服务器的，于是简单记录了，有详细的服务器记录在具体的外包项目下。

## 1. 安装配置

### 1.1设置root密码

>sudo passwd root

### 1.2配置源
用vim打开`/etc/apt/sources.list`，加入以下源（另外还可以自己去找其他的，或者用系统自动添加最快的）

{% highlight bash %}

deb http://ubuntu.dormforce.net/ubuntu/ lucid main restricted universe multiverse
deb http://ubuntu.dormforce.net/ubuntu/ lucid-backports main restricted universe multiverse
deb http://ubuntu.dormforce.net/ubuntu/ lucid-proposed main restricted universe multiverse
deb http://ubuntu.dormforce.net/ubuntu/ lucid-security main restricted universe multiverse
deb http://ubuntu.dormforce.net/ubuntu/ lucid-updates main restricted universe multiverse
deb-src http://ubuntu.dormforce.net/ubuntu/ lucid main restricted universe multiverse
deb-src http://ubuntu.dormforce.net/ubuntu/ lucid-backports main restricted universe multiverse
deb-src http://ubuntu.dormforce.net/ubuntu/ lucid-proposed main restricted universe multiverse
deb-src http://ubuntu.dormforce.net/ubuntu/ lucid-security main restricted universe multiverse
deb-src http://ubuntu.dormforce.net/ubuntu/ lucid-updates main restricted universe multiverse

{% endhighlight %}

然后执行

>sudo apt-get update  
sudo apt-get upgrade


### 1.3安装vim

>sudo apt-get install vim

配置vim来编辑C/java程序:

### 1.4右键打开终端

方法一：

>sudo apt-get install nautilus-open-terminal

方法二：用脚本，把下面的脚本保存成任意名（我的是：打开终端），然后放在主目录的`.gnome2/nautilus-scripts`目录下，当然你可以放一些其他常见的脚本，都可以在右键找到。比如发送到邮件／修改文件权限等等实用的功能。

脚本：

{% highlight bash %}
#!/bin/bash
#
# This script opens a gnome-terminal in the directory youselect.
#
# Distributed under the terms of GNU GPL version 2 or later
#
# Install in ~/.gnome2/nautilus-scripts or ~/Nautilus/scripts
# You need to be running Nautilus 1.0.3+ to use scripts.

# When a directory is selected, go there. Otherwise go tocurrent
# directory. If more than one directory is selected, showerror.
if [ -n "$NAUTILUS_SCRIPT_SELECTED_FILE_PATHS" ]; then
set $NAUTILUS_SCRIPT_SELECTED_FILE_PATHS
if [ $# -eq 1 ]; then
destination="$1"
# Go to file's directory if it's a file
if [ ! -d "$destination" ]; then
destination="`dirname "$destination"`"
fi
else
zenity --error --title="Error - Open terminal here" \
--text="You can only select one directory."
exit 1
fi
else
destination="`echo "$NAUTILUS_SCRIPT_CURRENT_URI" | sed's/^file:\/\///'`"
fi

# It's only possible to go to local directories
if [ -n "`echo "$destination" | grep '^[a-zA-Z0-9]\+:'`" ];then
zenity --error --title="Error - Open terminal here" \
--text="Only local directories can be used."
exit 1
fi

cd "$destination"
exec x-terminal-emulator

{% endhighlight %}

### 1.5安装openssh-server：

>sudo apt-get install openssh-server(已经包含了client)

### 1.6安装ftp图形工具，filezilla: 

>sudo apt-get install filezilla

### 1.7安装输入法：ibus

可以到ibus的googlecode看安装手册，也可以到`wiki.ubuntu.org.cn/IBus`看安装指令：

首先删除系统自带的ibus: 

>sudo apt-get autoremove ibus  
sudo add-apt-reposity ppa:shawn-p-huang/ppa  
sudo apt-get update  
sudo apt-get install ibus ibus-gtk ibus-qt4  
sudo apt-get install ibus-pinyin  
im-switch -s ibus

重启就OK,或者这样安装（推荐）：

>sudo apt-get install ibus ibus-clutter ibus-gtk ibus-qt4 ibus-pinyin  
im-switch -s ibus

重启

原来，如果安装系统时候安装了中文语言包，那么ibus就会直接可以用了。
 
### 1.8安装chrome 

直接google一下chrome下载就可以到官网去下载了，安装完以后直接登录自己的账号同步一下就OK，对于chrome可以有很多插件用，比较常用的有vimum，diigo，有道，印象笔记，goagent等等。
  
### 1.9安装java

去oracle官网下载linux版本的jdk，然后修改chmod +x添加执行权限，再执行解压，然后cp -r复制到/usr/local目录下，最后添加环境变量，具体看我的jdk文档

安装php

安装svn

安装apache

安装maven

安装wine

安装wireshark

配置终端

安装mysql

安装myeclipse

安装hg

安装tomcat

安装Launchy

安装adobe reader
安装wps
安装virtual box
安装dropbox

## 2. 备份

### 2.1备份文件：

{% highlight bash %}

/etc/profile
/etc/environment
/etc/fstab
/etc/rc.local
/home/zhangge/.bashrc
/home/zhangge/.vimrc
/etc/crontab
/home/zhangge/.xhome.ssh
/etc/hosts

{% endhighlight %}

### 2.2先装Ubuntu10.04后再装win8的修复引导程序步骤，主要是安装grub：

(1).刻录ubuntu10.04的liveCD到U盘，然后进入liveCD的ubuntu，查看ubuntu的所在分区信息，可以使用sudo fdisk -l查看

(2).找到挂载/根目录的分区和/boot的分区，如果在都在一个分区就简单，我的在两个分区，/boot在/dev/sda1, /在/dev/sda2

(3).切换到root用户方便操作，sudo -i;然后挂载第二步的分区:

	mkdir /media/tempfix  
	mount /dev/sda2 /media/tempfix  
	mount /dev/sda1 /media/tempfix/boot  

(4).安装grub程序到MBR.

>grub-install --root-directory=/media/tempfix /dev/sda

安装结果显示：Installation finshed.No error reported

(5).重启即可进入ubuntu10.04. 然后更新grub的引导表：sudo update-grub2

## 3.Linux/Unix软件的安装

1.在ubuntu下一般安装方法：

首先配置源，然后使用命令在源里面找到软件安装:sudo apt-get install softwarename

apt是一个很有用的程序：

2.或者去下载deb包，然后双击安装，例如chrome

3.有些是直接编译好的二进制包，直接可以运行

4.有些是bin后缀的，直接运行后解压得到一个目录，例如jdk

5.有些是run后缀的脚本安装，可能里面包含不仅是脚本，例如myeclipse

6.有些是tar的gz,bz2压缩包，直接解压以后就可以用，例如eclipse

7.rmp包的安装

8.最重要最常见最需要掌握的就是下载源码，然后编译安装

在/usr/local下建立一个software目录，以后放置源码文件。一般步骤是：tar, ./configure, make, make install

	tar:就是用tar来解压
	tar -jxv -f filename.tar.bz2 -C .
	tar -zxv -f filename.tar.gz -C .
	configure:软件源码里面一般都有可以执行的configure文件，其实是一个shell脚本，他的作用是对即将安装的软件进行配置，检查当前的环境师傅满足安装软件的依赖关系，然后生成makefile文件。它是autoconf的工具的基本引用。
    它有很多参数，以后慢慢积累，先记录一些常用的：
	    --prefix==dir:相当于设置dir为安装目录
	    --with-mysql=mysqldir:指定mysql所在目录，就是配置一个变量
	    --with-apx2==apache/apx2:指定apache的位置
	make:就是执行makefile里面的默认目标，一般是执行编译和链接的工作。
	make test:是测试一下编译链接后的结果，还可以把报告发送给官方。
	make install:执行安装目标，主要是把编译后的文件和库和配置文件复制到安装目录。
