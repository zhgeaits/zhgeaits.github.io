n---
layout: post
title: "关于Windows你不知道的秘密"
date: 2013-06-17 12:49:03
categories: os
type: other
---

快捷键：  
1.win+D:最小化  
2.win+箭头：最大化最小化窗口与移动窗口  
3.win+空格：透视   

追中路由转发，windows命令：tracert，例如 tracert www.baidu.com 可以看到出口ip，直接在百度输入ip也能看到出口ip。这个命令和linux下的traceroute一样。

查看域名ip命令：nslookup www.baidu.com

**Win7无法上网问题**

>以前我的win7也试过无法上网，这些问题也经常有同学问我，修复多了，但是时间一长还是忘记，于是打算写到日志里面以便以后能记住。
>
>一般来说，同学的win7上不了网，就是用360修复网络，或者用QQ管家也行，再不行就去百度一下，总会解决的，实在不行，也不想去研究就会问我或者重装，怎么说大家都是掌握了四大法宝的：关闭重开，卸载重装软件，重启系统，重装系统。
>
>现在说说比较普遍的问题：  
>**1.**有时候拨号成功了，但是不能上网，或者本地连接没有成功，那么诊断本地连接，或者打开IE以后选择修复这个问题，有可能会成功，也有可能会提示错误是远程主机没有指定之类的。这个问题一般是没有设置网关，也没有获取到ip地址，使用ipconfig查看本地可能没有ip地址，这一般要去启动dhcp服务，然后在cmd运行ipconfig\\release和ipconfig\\renew命令获取IP地址就可以了。运行ipconfig看看没有ip地址，运行route print看看路由表，没有网关的话需要添加一个网关。
>
>**2.**拨号成功，本地连接正常，QQ能上，但是打不开网页，测试ping一下百度，失败。这一般是DNS没有配置好，使用nslookup www.baidu.com看看能找到ip地址不。可以试一下dhcp看看能不能获取到dns地址，在设置固定ip那里把ip和dns都改成自动设置，再试一下，如果都不行就只能手动设置dns了，可以设置自己学校的dns，也可能设置一些知名的,如8.8.8.8。DNS问题网上有说是adobe的软件导致的，卸载就行。网上还有其他办法，就是卸载驱动，再重新安装。
>
>**3.**这个有点诡异了，拨号成功，wifi连接成功，QQ能上，ping通了，就是DNS都正常，就是浏览器不能用，各种修复和网上办法都不能成功。那么在网络层是没有问题，网络是成功的，那么就是传输层出问题了。后来才查出来是windows的socket被修改了，奇怪的是QQ竟然能用，QQ到底怎么写的？解决办法就是重新设置win socket，用管理员运行cmd，然后运行netsh winsock reset，完了重启就行。也可以用一些修复软件，如winsockfix，LSPfix等。如果都不行，再去研究，或者只能重装了。
>
>
>**ps：route的使用方法**
>
>ROUTE \[-f\] \[-p\] \[-4|-6\] command \[destination\] \[MASK mask\] \[gateway\] \[METRIC metric\] \[IF interface\]  
>\[-f\] 清除所有网关项的路由表。这个参数慎用。  
>\[-p\] 增加永久路由。在默认情况下，重启系统之后，我们用add命令增加的路由是不会被保存的，-p参数和add命令结合使用的时候，可以增加永久保存路由。永久路由保存在注册表的这个位置：HKEY_LOCAL_MACH/SYSTEM/CurrentControlSet/Services/Tcpip/Parameters/PersistentRoutescommand。  
>\[-4\] IPv4网络  
>\[-6\] IPv6网络  
>\[command\] 共有4个命令：print, add, delete, change  
>\[destination\] 目标地址，结合MASK，可以定义主机或者网段。  
>\[mask\] 定义子网掩码，如果没有定义mask，默认为255.255.255.255，说明destination是一台主机，而不是一个网段。  
>\[gateway\] 定义网关的地址，就是数据的下一跳地址。如果不指定，系统会查找最佳的网关。  
>\[metric\] 定义跳数，这个一般用不到。当到同一目的地有多条路径的时候，系统会选择metric值最小的路由。  
>\[if\] 定义网卡。  
>\[interface\] 网卡的接口号码，在使用route print命令的时候，可以看到该号码。  
>
>route print 查看当前的路由信息  
>route add 10.0.0.0 mask 255.0.0.0 10.1.1.1 增加一条到10.0.0.0/8网络的路由，网关是10.1.1.1  
>route -p add 10.0.0.0 mask 255.0.0.0 10.1.1.1 增加一条永久路由  
>route delete 10.0.0.0 删除10.0.0.0这条路由  
>route change 10.0.0.0 mask 255.0.0.0 10.1.1.111 把网关改成10.1.1.111，注意，change命令只能修改网关或者metric的值  
