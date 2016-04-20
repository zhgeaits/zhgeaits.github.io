---
layout: post
title:  "Linux下的神级文本编辑器vim/vi"
date:   2013-05-02 11:00:03
categories: other
supertype: career
type: other
---

关于vim/vi的介绍网上太多了，我是从鸟哥的私房菜那里学习到vim的，确实它很厉害，迷上了它，学习了它，却没有熟练使用它。

**VIM练级操作命令记录**

{% highlight bash %}
1.cw:				删除从光标所在位置到一个单词结尾的字符
2.0,$,^,g_:			到行头，到行尾，到本行第一个不是blank的字符，到本行最后一个不是blank的字符
3.:e filename,:bn,:bN:		打开一个文件,移动到下一个文件，注意以鸟哥给的不一样
4.saveas filename:		另存为一个文件
5.N<command>:			重复command命令N次
6.100idesc[ESC],.,3.:		插入100个desc，重复插入100个desc，插入3个desc（还是不明白为什么只插入三个）
7.:n<Enter>,w/W,e/E:		到下一个单词/下一个程序语句（空格分隔）的开头，到下一个单词/程序语句的结尾。
8.*,#:				匹配光标当前所在的单词，分别移动到下一个和上一个
9.%:				匹配大中小括号移动,即从一边括号移动到另一边括号。需要把光标先移到括号上.
10.gU2,gu2:			以下2行变大写，小写。
11.y,ye:			y是从光标开始复制，ye是复制到单词最后一个字符
12.3fa:				到下一个为a的字符处，改变3和a即可。F即是相反方向。
9.t,:				到逗号前的第一个字符，同上
10.dt“:				删除所有的内容，直到遇到双引号
11.区域选择<action>a<object>或<action>i<object>，action可以是任何命令，object可以是单词，句子，段落，符号" ' ) } ]
   例如：(map (+) ("foo"))光标位于o，有以下操作:
   vi":选择foo
   va":选择"foo"
   vi):选择"foo"
   va):选择("foo")
   v2i):选择map(+)("foo")
   v2a):选择(map(+)("foo"))
12.块操作:<C-v>。典型的操作：0 <C-v> <C-d> I-- [ESC]。
   0：到行头。<C-v>:开始块操作。<C-d>:向下移动，也可以使用其他移动。I--:I是插入--。[ESC]：为每一行生效。
13.自动提示：<C-n>和<C-p>,在insert模式下可以使用自动补全。
14.宏录制：qa操作序列q，@a，@@。qa把你的操作记录在寄存器a；于是@a会replay被录制的宏；@@是一个快捷键用来replay最新录制的宏。
   示例操作：qaYp<C-a>q,@a,@@,100@@
   qa开始录制，Yp复制行，<C-a>新行增加1，q停止录制，@a回放一次，@@回放一次，100@@回放100次
15.C-v选择块后，<,>,=:分别是左缩进，右缩进，自动缩进
   C-v，可以用j，C-d，方向键，/pattern,%等来选择块，$,A,输入字符，[ESC]:在所选的行后加上字符
{% endhighlight %}


**鸟哥上的操作说明：**

{% highlight bash %}
1.n1, n2 w filename:		将n1到n2行保存到filename
2.hjkl：			左下上右移动，同时可以用方向键,可以结合数字来加快移动
3.C-f,C-b,C-d,C-u：		向下移动一页，向上移动一页，后面两个是半页
4.+,-:				游标移动到非空白的下一列/上一列
5.n<space>:			向右移动n个字符
6.H,M,L:			移动光标到屏幕最上方/中央/最下方那一行的第一个字符
7.G,nG,gg,n<Enter>:		移动光标到最后一行/第n行/第一行/下移n行
8./word, ?word, n, N:		向下/向上搜索word，向下/向上搜索下一个word
9.:n1,n2s/word1/word2/g:	把第n1行到第n2行的word1替换成word2
10.:1,$s/word1/word2/g:		把所有的word1替换成word2
11.:1,$s/word1/word2/gc:	C代表confirm，每次替换都要确认
12.x,X,nx:			删除下一个/前一个字符，连续删除下n个字符
13.dd,ndd,d1G,dG,d$,d0:		删除当前行，删除下n行，删除第一行到当前行，删除当前行到最后一行，删除光标到行末/行首
14.yy,nyy,y1G,yG,y$,y0:		同上，为复制
15.p,P:				粘贴到下一行/上一行
16.J,c:				删除本来的换行符，与下一行整合在一起,可以结合块操作；重复删除多个资料
17.u,C-r,.:			复原/重做上一个动作/重复前一个动作
18.i,I,a,A,o,O:			插入当前光标/插入到当前行的第一个字符，插入到光标下一个字符/插入当前行最后一个字符，向下/向上插入新行
19.r,R:				进入取代模式，r只取代一个字符，R取代一直是取代模式
20.:w,:w!,:q,:q!,:wq:		保存，强保，退出，强退，保存退出
21.ZZ:				保存退出
22.:w filename			另存为
23.:r filename			读入文件内容到光标
24.:n1,n2 w filename		将n1到n2行的内容存到filename
25.! command			暂时离开vim，并执行command，如! ls /home/zhangge
26.set nu, set nonu		显示/取消行号
27.v,V,C-v,y,d			字符区域选择，行区域选择，块区域选择，复制区域，删除区域
28.vim file1 file2, :files	同时打开多个文件,显示目前打开的文件
29.:n,:N			编辑上一个文件，下一个文件
30.:sp filename			split一个窗口打开filename，这里是水平分
31.:vsp filename		vsplit一个窗开打开filename，这里是垂直分
32.C+w+j,C+w+k			在多窗口模式移动到上一个窗口/下一个窗口
{% endhighlight %}

**自己的操作记录：**

{% highlight bash %}
1.K				在命令模式把光标移到一个单词上，然后按大写K则会跳到man页面，相当于:! man word
{% endhighlight %}