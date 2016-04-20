---
layout: post
title:  "Linux下进行连接vpn"
date:   2012-03-06 11:00:03
categories: other
supertype: career
type: other
---

由于工作搭建了vpn服务器，所以，从此以后再也不用花钱上网了，冻结了校园的账号，只保留了能链接校园内网的功能，然后就能链接到工作室了！

### 1. 先安装VPN客户端
 
>sudoapt-getinstall pptp-linux

### 2. 然后命令行下拨号连接VPN服务器

>sudo pptpsetup --create testvpn --server 123.45.67.88 --username kk --password fku --encrypt --start

其中：

>–create 后的是创建的连接名称，可以为任意名称;  
–server 后接的是vpn服务器的IP;  
–username 是用户名  
–password 是密码，在这也可以没这个参数，命令稍后会自动询问。这样可以保证账号安全  
–encrypt 是表示需要加密，不必指定加密方式，命令会读取配置文件中的加密方式  
–start 是表示创建连接完后马上连接，如果你不想连，就不写  

### 3. 以后要连接VPN或断开VPN

>pon testvpn # VPN的“连接名称"

>poff # 断开VPN连接

### 4. 设置全部流量走VPN通道

把下面两行加入`/etc/ppp/ip-up`中或者直接命令行输入也行，删除默认网关及把VPN服务器作为默认网关，也就是改变路由策略，把所以传输流量通过VPN线路来走。

>route add default gw 192.168.0.101

>route del default gw 原来的网关 # 原来的默认网关地址可通过 route 命令来获取


### 5.其他

route add default dev ppp0

route add -net 192.168.2.0 netmask 255.255.255.0 ppp0
