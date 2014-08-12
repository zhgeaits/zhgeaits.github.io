---
layout: post
title:  "腾达无线网卡832U安装记录"
date:   2013-06-07 17:00:03
categories: linux
type: other
---

>昨天买了一个腾达的无线网卡，其实很久以前就想买一个的，但是因为没有什么特别用处就没有买了，这次想在工作室装一台电脑玩玩，装一下[Linux_mint]，或者装以下[e17]，还有搭建一下[hadoop]环境，重新学习，而且飞哥介绍了一个两个电脑共享剪切板的软件[synergy]。
>
>说到这个软件，在上个学期的时候，我在工作室用双显示屏的时候，因为是我电脑跑双系统，开销有点大，如果跑两台电脑需要两个键盘鼠标感觉很麻烦，就曾经想过在两台电脑之间对接，只是稍微想了一下，感觉需要操作系统底层和硬件的支持，因为我想到实现的功能比较强大的样子，例如不同操作系统之间的软件数据对接！后来只是想了一下就不多想了，感觉这个东西需要去研究，比较高端！没想到真有synergy这个软件，他是通过网络共享剪切板的，这样可以共享鼠标键盘和复制一些文件，功能没有我想到那么强大，我也没有认真去了解这个软件，不过人家毕竟是实现了，对比自己只是想一下，我感觉自己有点想当然，缺乏动手的激情，这本来就是一个很好的创新点子，只是没有付诸于行动而已，唉，这样不行呐！以后必须经常动手实践一些小想法，多点编写代码，写写blog记录创新的想法，就像那个huffman压缩软件一样，虽然小，但是很锻炼！！
>
>说回这个无线网卡，京东买的，上面说了支持linux，一开始以为驱动很好安装，没想到还是有点麻烦的样子，于是就写下blog记录一下咯！  提供的光盘里面基本都是windows的安装软件，连使用手册里面所说的都是windows安装，只有两个文件夹，一个linux，一个mac，分别是各自的驱动程序，于是copy了linux到home目录，里面有三个压缩文件，按照我一贯的作风---全都要试一下！！
>
>于是，解压，编译安装，三个都试过了，看了makefile，看了readme，都没有用，而且还懵了，不知道到底安装哪个！无奈只能Google了一下，发现大家都是去[ralinktech]下载驱动，但是上面好多版本，根本不知道下载哪个！
>
>插上网卡，使用这个命令看了一下芯片
{% highlight bash %}
zhangge@zhangge:linux$ lsusb
Bus 002 Device 008: ID 148f:5372 Ralink Technology, Corp. 
{% endhighlight %}
>可以看到ID是148f:5372，这个就是芯片型号，148f不知道什么，但是RT5372肯定是型号；其实这样才知道腾达是用了别人的芯片的！原来这个驱动在光盘提供的三个里面是有的。
>
{% highlight bash %}
tar -jxvf 2011_0719_RT3070_RT3370_RT5370_RT5372_Linux_STA_V2.5.0.3_DPO.bz2 -C 5372
{% endhighlight %}

>在Readme里面的Build Instructions有说明如果需要加入本地的network-manager支持，必须修改一个头文件  
<pre>
In os/linux/config.mk  
Build for being controlled by NetworkManager or wpa_supplicant wext functions  
Please set 'HAS_WPA_SUPPLICANT=y' and 'HAS_NATIVE_WPA_SUPPLICANT_SUPPORT=y'.
</pre>

>然后就是make, make install，注意到自己可以去查看makefile文件，make的默认执行目标是all目标，删除就执行make uninstall。  
>readme里面的指南有加载模块的方法，而且没有make install步骤，如果执行了make install，那么也就执行了insmod命令。所以完了只要执行以下命令就行：
{% highlight bash %}
modprobe rt5370sta
{% endhighlight %}
>这个命令是加载模块到内核上。我觉得这个命令会把rt5370sta加入到/etc/modules里面，这样重启还是会有的。如果重启以后网卡不能用，那么执行这个命令：
{% highlight bash %}
echo rt5370sta >> /etc/modules
{% endhighlight %}

>看看右上角是不是有两个无线网卡了，执行ifconfig看看是否多了一个网卡！


[ralinktech]: http://www.ralinktech.com/en/
[Linux_mint]: http://www.linuxmint.com/
[hadoop]: http://hadoop.apache.org/
[e17]: http://www.enlightenment.org/
[synergy]: http://synergy-foss.org/zh-cn/
