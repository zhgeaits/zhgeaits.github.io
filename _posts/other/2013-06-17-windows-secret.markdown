---
layout: post
title: "关于Windows你不知道的一些小秘密"
date: 2013-06-17 12:49:03
categories: os
supertype: career
type: other
---

## 科大校园里Win7无法上网问题

以前我的win7也试过无法上网，这些问题也经常有同学问我，修复多了，但是时间一长还是忘记，于是打算写到日志里面以便以后能记住。

一般来说，同学的win7上不了网，就是用360修复网络，或者用QQ管家也行，再不行就去百度一下，总会解决的，实在不行，也不想去研究就会问我或者重装，怎么说大家都是掌握了四大法宝的：关闭重开，卸载重装软件，重启系统，重装系统。

现在说说比较普遍的问题：  

**1.**有时候拨号成功了，但是不能上网，或者本地连接没有成功，那么诊断本地连接，或者打开IE以后选择修复这个问题，有可能会成功，也有可能会提示错误是远程主机没有指定之类的。这个问题一般是没有设置网关，也没有获取到ip地址，使用ipconfig查看本地可能没有ip地址，这一般要去启动dhcp服务，然后在cmd运行`ipconfig\release`和`ipconfig\renew`命令获取IP地址就可以了。运行`ipconfig`看看没有ip地址，运行`route print`看看路由表，没有网关的话需要添加一个网关。

**2.**拨号成功，本地连接正常，QQ能上，但是打不开网页，测试ping一下百度，失败。这一般是DNS没有配置好，使用`nslookup www.baidu.com`看看能找到ip地址不。可以试一下dhcp看看能不能获取到dns地址，在设置固定ip那里把ip和dns都改成自动设置，再试一下，如果都不行就只能手动设置dns了，可以设置自己学校的dns，也可能设置一些知名的,如8.8.8.8。DNS问题网上有说是adobe的软件导致的，卸载就行。网上还有其他办法，就是卸载驱动，再重新安装。

**3.**这个有点诡异了，拨号成功，wifi连接成功，QQ能上，ping通了，就是DNS都正常，就是浏览器不能用，各种修复和网上办法都不能成功。那么在网络层是没有问题，网络是成功的，那么就是传输层出问题了。后来才查出来是windows的socket被修改了，奇怪的是QQ竟然能用，QQ到底怎么写的？解决办法就是重新设置win socket，用管理员运行cmd，然后运行`netsh winsock reset`，完了重启就行。也可以用一些修复软件，如`winsockfix`，`LSPfix`等。如果都不行，再去研究，或者只能重装了。

**ps：route的使用方法**

{% highlight bash %}
ROUTE [-f] [-p] [-4|-6] command [destination] [MASK mask] [gateway] [METRIC metric] [IF interface]  
[-f] 清除所有网关项的路由表。这个参数慎用。  
[-p] 增加永久路由。在默认情况下，重启系统之后，我们用add命令增加的路由是不会被保存的，
-p参数和add命令结合使用的时候，可以增加永久保存路由。永久路由保存在注册表的这个位置：
HKEY_LOCAL_MACH/SYSTEM/CurrentControlSet/Services/Tcpip/Parameters/PersistentRoutescommand。  
[-4] IPv4网络  
[-6] IPv6网络  
[command] 共有4个命令：print, add, delete, change  
[destination] 目标地址，结合MASK，可以定义主机或者网段。  
[mask] 定义子网掩码，如果没有定义mask，默认为255.255.255.255，说明destination是一台主机，而不是一个网段。  
[gateway] 定义网关的地址，就是数据的下一跳地址。如果不指定，系统会查找最佳的网关。  
[metric] 定义跳数，这个一般用不到。当到同一目的地有多条路径的时候，系统会选择metric值最小的路由。  
[if] 定义网卡。  
[interface] 网卡的接口号码，在使用route print命令的时候，可以看到该号码。  

route print 查看当前的路由信息  
route add 10.0.0.0 mask 255.0.0.0 10.1.1.1 增加一条到10.0.0.0/8网络的路由，网关是10.1.1.1  
route -p add 10.0.0.0 mask 255.0.0.0 10.1.1.1 增加一条永久路由  
route delete 10.0.0.0 删除10.0.0.0这条路由  
route change 10.0.0.0 mask 255.0.0.0 10.1.1.111 把网关改成10.1.1.111，注意，change命令只能修改网关或者metric的值  
{% endhighlight %}

## 一些windows的快捷键

1.win+D:最小化  
2.win+箭头：最大化最小化窗口与移动窗口  
3.win+空格：透视   

## 网络命令

1. tracert

追踪路由转发，例如:

> tracert www.baidu.com 

可以看到出口ip，直接在百度输入ip也能看到出口ip。这个命令和linux下的`traceroute`一样。

2. nslookup

查看域名ip命令, 例如：

> nslookup www.baidu.com

## 关于dllhost.exe

这是一个诡异的东西，有一次我打开一个目录，里面全是视频，然后这时候系统就几乎是卡住了，一直在占用cpu，貌似是这个目录一直没打开完成，绿色进度条始终没跑完，再有就是视频的图标截图一直没出来完成，里面各种格式的视频都有。打开任务管理器，发现是`dllhost.exe`这个进程占用了100%的cpu，这是什么东西？为什么和视频截图相关？虽然干掉了就什么事都没了，但是很诡异。

>Dllhost.exe是 COM+ 的主进程。正常下应该位于system32目录里面和system32\dllcache目录里面。而system32\win s目录里面是不会有dllhost.exe文件的。

那么显然是系统调用com组件进行视频截图，然后某些视频解码不成功，或者失败了，导致一直死循环去截图，这应该是windows的bug吧！