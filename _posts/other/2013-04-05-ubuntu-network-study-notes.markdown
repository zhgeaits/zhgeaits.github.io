---
layout: post
title:  "Ubuntu下的网络设置"
date:   2013-04-05 11:00:03
categories: other
supertype: career
type: other
---

自从痛恨windows的蓝屏以后，一直都在ubuntu，也非常喜欢使用它，这里记录了一些ubuntu的网络设置。

IP地址配置文件： 

>/etc/network/interfaces

打开后里面可设置DHCP或手动设置静态ip；前面auto eth0，让网卡开机自动挂载。

### 1. 以DHCP方式配置网卡

>sudo vim /etc/network/interfaces

并用下面的行来替换有关eth0的行:

>auto eth0  
iface eth0 inet dhcp

用下面的命令使网络设置生效:

>sudo /etc/init.d/networking     restart

也可以在命令行下直接输入下面的命令来获取地址

>sudo dhclient eth0

### 2. 为网卡配置静态IP地址

>sudo vim /etc/network/interfaces

并用下面的行来替换有关eth0的行:# The primary network interface

>auto eth0  
iface eth0 inet static  
address 192.168.3.90  
gateway 192.168.3.1  
netmask 255.255.255.0 

将上面的ip地址等信息换成你自己就可以了。用下面的命令使网络设置生效:

>sudo /etc/init.d/networking restart

### 3. 设定第二个IP地址(虚拟IP地址)

>sudo vim /etc/network/interfaces

在该文件中添加如下的行:

>auto eth0:1  
iface eth0:1 inet static  
address 192.168.1.60  
netmask 255.255.255.0  
network x.x.x.x  
broadcast x.x.x.x  
gateway x.x.x.x  

根据你的情况填上所有诸如address,netmask,network,broadcast和gateways等信息。用下面的命令使网络设置生效:

>sudo /etc/init.d/networking restart


### 4.主机名称配置文件(/bin/hostname)

使用下面的命令来查看当前主机的主机名称:

>sudo /bin/hostname

使用下面的命令来设置当前主机的主机名称:

>sudo /bin/hostname newname

系统启动时,它会从/etc/hostname来读取主机的名称.


### 5.DNS配置文件

首先,你可以在/etc/hosts中加入一些主机名称和这些主机名称对应的IP地址,这是简单使用本机的静态查询。

要访问DNS服务器来进行查询,需要设置/etc/resolv.conf文件.

>sudo vi /etc/resolv.conf  
nameserver 202.96.128.68  
nameserver 61.144.56.101  
nameserver 192.168.8.220  

重新设置网络，以启用新设置

>sudo /etc/init.d/networking restart
