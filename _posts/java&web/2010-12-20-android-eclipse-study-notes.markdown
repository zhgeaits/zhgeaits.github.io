---
layout: post
title:  "Java开发神器IDE之Eclipse"
date:   2010-12-20 10:10:10
categories: android
supertype: career
type: android
---

## 1. 什么是Eclipse

学习java的时候，必然知道eclipse，仿佛没有了这个东西，就没法编程了一样，可见其重要性。当然，那也是对一个什么都不懂得新手来讲而已。想当年，我还是一个小菜的时候，什么是IDE，毫无概念！那时候还在感叹软件这个神奇的东西，几行代码如何成为软件，这也是对于这个世界的认知在逐渐的成长！浅显的认为软件是一个独立的个体，就像一个物体或者生命一样，却没有从一个构造，层次，功能等层面去认知，纠结于几行代码如何成为一个软件个体！想想当年，虽然可笑，却充满了求知的渴望！

如果是单纯学习java，当然不建议直接就用eclipse这样的东西，不便于理解java，不便于理解编程，编译，运行等等，纯粹是知道代码运行，然后输出结果，那样不如直接学习脚本罢了。IT是一条很长的路，必须一步步走下去，很多计算机的东西需要慢慢的去学习，理解，运用。

IDE的意思是Integration Development Environment的缩写，即是集成开发环境。它包含很多的环境和工具，非常便于我们开发。

## 2. 下载

Google一下，就会找到[官网](http://www.eclipse.org/)，会有最新的版本下载。

## 3. 设置

对代码进行格式化的时候，有一点老是不喜欢，就是格式化以后行宽度很窄，方法换行了。设置：  
>Java->Code Style->Formatter->Edit->Line Wrapper

## 4. Android相关设置

### 4.1 android分析当前页面的布局

这有点像chrome直接看页面的代码，超级方便。。打开DDMS的Perspective，然后在Devices那里有一个Dump View Hierarchy for UI Automator

### 4.2 第三方库问题

eclipse有一个很蛋疼的问题，在android项目使用第三方库的时候，例如sharesdk，腾讯的第三方sdk，我们把jar包放到lib目录下，虽然选择了这个jar加入到build path，但是运行时候报找不到class的异常。。。。。及其蛋疼，必须把libs目录设置为源目录才行。

### 4.3 appcompat包的问题

新建android项目的时候，如果选择的最小sdk小于4.0，则会生成一个appcompat_V7的包，这个好蛋疼。如果adt版本有问题，还不会生成Activity呢。

## 5. 快捷键


<table>
	<tr>
		<td>Shift+Enter</td>
		<td>在当前行的下一行插入空行(这时鼠标可以在当前行的任一位置,不一定是最后) </td>
	</tr>
	<tr>
		<td>Ctrl + Shift + O</td>
		<td>引入imports语句</td>
	</tr>
	<tr>
		<td>Ctrl + Shift + T</td>
		<td>打开Open Type查找类文件</td>
	</tr>
	<tr>
		<td>Ctrl + Shift + F</td>
		<td>格式化代码</td>
	</tr>
	<tr>
		<td>Ctrl + D</td>
		<td>删除本行</td>
	</tr>
	<tr>
		<td>Ctrl + /</td>
		<td>注释本行</td>
	</tr>
	<tr>
		<td>Alt + Shift + j</td>
		<td>添加DOC注释 </td>
	</tr>
	<tr>
		<td>Alt+↓</td>
		<td>当前行和下面一行交互位置(特别实用,可以省去先剪切,再粘贴了)</td>
	</tr>
	<tr>
		<td>Alt+↑</td>
		<td>当前行和上面一行交互位置(同上)</td>
	</tr>
</table>