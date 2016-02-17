---
layout: post
title:  "Linux下shell编程"
date:   2013-06-10 11:00:03
categories: other
type: other
---

疑问：

{% highlight bash %}

1.为什么virtualbox只能安装i386而不能安装x86_64的系统？ubanutu和centos都是一样。
2.文件系统中，block/inode/superblock和硬盘的物理结构是什么关系？一个block对应一些柱面？
4.在vim 里面%{}是什么意思？P704
5.什么是镜像
6.现在还是不是很明白为什么软件，驱动重新编译以后就可以运行，一定要修改源码的吗？还是因为 编译环境的问题?
7.还是不是很明白自己设置固定IP和路由分配IP有什么区别？不会冲突吗？以谁为准？

{% endhighlight %}

操作与知识点记录：

{% highlight bash %}

1.重启X Window：直接按下[Alt]+[Ctrl]+[Backspace]
2.[Ctrl]+[Alt]+[F1]~[F6]:文字界面登录tty1~tty6
3.[Ctrl]+[Alt]+[F7]：图形界面桌面，或者使用命令：startx
4.显示目前所支持的语言：echo $LANG
5.修改为英文语系：LANG=en_US
6.查看日期：date；其他显示格式：date +%Y/%m/%d/%H:%M
7.查看日历：cal；语法为：cal [[month] year]
8.计算器：bc；退出命令为quit；默认输出结果为整数，如果要确定小数位数，scale=number
9.重要热键之[Tab]:具有命令补全和文件补全功能，
10.重要热键之[Ctrl]+C:中断程序
11.重要热键之[Ctrl]+D:代表键盘输入结束，相当于exit
12.在文档帮助:man。-f选项是查找命令或文件相关说明。 -k选项是查找包含关键字的所有。分别对应相同的两个命令为：whatis, apropos
13.另外一个在线文档帮助：info
14.查看谁在线命令：who
15.查看网络联机状态：nestat –e
16.查看后台执行程序：ps –aux
17.查看文件命令：ls，更多用法，查看man ls或者 info ls
18.更改用户命令：su，如，su zhangge
19.读取文件内容命令：cat
20.新建空文件命令：touch 也可以用dd命令
21.删除文件命令：rm –r，连同文件夹下面的文件全部删除
22.调出目前挂载的设备：df
23.计算目录还有多少容量du是disk usage：du –sb,du给出指定目录及其子目录下所有文件占用硬盘数据块的总数.
24.当使用fdisk分区完后，使用partprobe命令来刷新分区表
25.显示登录用户的信息：last
26.输出文件的前一部分：head
27.输出文件的后一部分：tail
28.取得帐号相关信息：finger account 还有一个命令就是id account
29.显示内核信息：dmesg
30.修改使用者帐号:usermod
31.执行一连串命令：sh –c
32.查看系统资源：free 或者 vmstat 查看动态变化
33.deb包安装命令：sudo dpkg -i packagename.deb
34.语系编码转换：iconv 查看编码用enca
35.环境变量的设置：临时的使用export命令，全局的在/etc/profile修改，用户的在/~/.bashrc修改或者在.profile
36.打开文件夹命令：nautilus
37.压缩:tar -jcv -f filename.tar.bz2 要被要所的目录或则文件
38.查询:tar -jtv -f filename.tar.bz2
39.解压:tar -jxv -f filename.tar.bz2 -C .
40.查看ubuntu版本：sudo lsb_release -a
41.MAC地址转换记录命令查看:arp
42.观察路由表命令：route -n
43.网络服务端口号配置文件：/etc/services
44.查看无线网卡设置：iwconfig 显示列表 iwlist
44.DNS配置文件:/etc/resolv.conf
45.查看域名：nslookup www.baidu.com
46.搜索联机帮助：man命令加-k参数
47.将程序防到后台运行：ctrl + z，观察后台程序:jobs，要切换到前台运行:fg %n，要切换到后台运行:bg %n
48.查看网卡被锁情况：sudo rfkill list,如果被锁就使用：sudo rfkill unblock all，或者首先在键盘上关掉以后再使用这命令
49.查看系统环境变量：env
50.unix实验课用到的cut命令，cut -d ' ' -f 1指的是以空格分隔并取第一个。
51.也可以用awk命令，awk '{print $1}'它默认以空格或者tab键分隔，并取第一个
52.在.bashrc这个文件记录了用户终端配置,如下配置改变路径名显示方式
   if [ "$color_prompt " = yes ]; then
       PS1 ='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\W \[\033[00m\]\$ '
   else
       PS1 ='${debian_chroot:+($debian_chroot)}\u@\h:\W \$ '
   将蓝色的w由小写改成大写，可以表示只显示当前目录名称.
53.~符号代表的是用户的主文件夹，它是个变量！
54.virtualbox 安装centos的增强功能：
   进入Root 终端，依次在终端中执行下列操作：
   # yum install kernel-devel
   # yum install gcc
   # ln -s /usr/src/kernels/2.6.18-274.3.1.el5-i686/ /usr/src/linux
55.terminal终端快捷键：
   Shift+Ctrl+T:新建标签页
   Shift+Ctrl+W:关闭标签页
   Ctrl+PageUp:前一标签页
   Ctrl+PageDown:后一标签页
   Shift+Ctrl+PageUp:标签页左移
   Shift+Ctrl+PageDown:标签页右移
   Alt+1:切换到标签页1
   Alt+2:切换到标签页2
   Alt+3:切换到标签页3
   Shift+Ctrl+N:新建窗口
   Shift+Ctrl+Q:关闭终端
   终端中的复制／粘贴:
   Shift+Ctrl+C:复制
   Shift+Ctrl+V:粘贴
   终端改变大小：
   F11：全屏
   Ctrl+plus:放大
   Ctrl+minus:减小
   Ctrl+0:原始大小
   Alt+F1  打开“应用程序”菜单
   Alt+F2  显示“运行”对话框
   Ctrl+Alt+方向键  切换到方向键所指的工作台
   Ctrl+Alt+D  切换到桌面
   Ctrl+Alt+L  锁定桌面并启动屏幕保护程序
   PrintScreen  截屏
   Alt+PrintScreen  截取当前窗口的图象
   Ctrl+Alt+Fn  切换到终端或桌面(n=1~7)
   &nbsp;  &nbsp;
   Alt+Tab  在不同程序间切换
   Alt+Esc  逆序在不同程序间切换
   Alt+F4  关闭窗口
   Alt+F5  取消最大化窗口 (恢复窗口原来的大小)
   Alt+F7  移动窗口 (注: 在窗口最大化的状态下无效)
   Alt+F8  改变窗口大小 (注: 在窗口最大化的状态下无效)
   Alt+F9  最小化窗口
   Alt+F10  最大化窗口
   Alt+空格  打开窗口的控制菜单 (点击窗口左上角图标出现的菜单)
   &nbsp;  &nbsp;
   Ctrl+N  新建窗口
   Ctrl+C  剪切
   Ctrl+X  复制
   Ctrl+V  粘贴
   Ctrl+Z  撤销上一步操作
   Ctrl+Shift+Z / Ctrl+Y  重做刚撤销的一步操作
   Ctrl+S  保存
   Ctrl+B  书签
56.comm:一行行比较两个文件，参数有1，2，3，具体man去
57.测试网络服务命令netcat：nc host port,如：nc localhost 8080  -u参数是指定UDP测试
58.rm删除一些特殊字符的文件名文件时，例如删除-.markdown，则用rm -- -.markdown

{% endhighlight %}