---
layout: post
title:  "Android开发IDE之IntelliJ IDEA使用，快捷键，常用技巧!"
date:   2014-07-18 11:00:03
categories: android
type: android
---

**什么是IntelliJ IDEA**

IntelliJ IDEA也是一个IDE，对于我习惯了eclipse的来说，使用起来是很不习惯的，但是到网上搜索有很多比较两者的优劣的文章，我就不多说了。
来到新公司，大家都用这个，就用吧，用着就会习惯的了。再说虽然Android官方主要的开发IDE还是eclipse，但是android已经在intellij的基础上
修改出一个android studio了，目前还是测试版。。。。
主要还是先要记一下常用的快捷键。

**DIY settings**  
给标签tab设置一个星※，*，就像eclipse那样，如果一个文件，tab，被修改了，就显示一个*，然后按下ctrl+s保存以后这个星就消失；  
在settings->Editor->Editor tabls->Mark modified tabs with asterisk，勾上就好了。  
这里还有两个设置，当关闭一个tab的时候，激活上次打开的那个，则勾上：when closing editor, active the most recently tabs.  
再勾上show tabs in single row，让它显示在一行，两行闪来闪去，实在受不了。。。

设置固定行长度，settings->Editor->Code Style->Right margin(columns)，如果想要在输入的时候自动wrap，可以勾选Wrap when typing reaches right margin。设置了这个不一定生效，要再到Code Style->Java->Wrapping and Braces勾选Ensure right margin is not exceeded(超出)才行。

新版的idea会提示这个override is not allowed when implementing interface method，就是说如果实现一个接口使用了@Override的注解的话，就会报错了，想要取消这个提示，在Inspections是找不到的。其实是这是jdk1.6一下的提示错误，1.6以上包括1.6的jdk是不会报错的了，所以只要把project的language level改一下就OK了：在Project Structure | Project 那里修改 Project language Level 为 6.0 - @Override in interfaces，如果还是不行，那是因为其他model没有使用的是整个project的level，那么就在Modules那里把每个module的level都同样修改即可。但是可能修改以后运行项目就报错了：  
Information:java: javacTask: 源发行版 1.7 需要目标发行版 1.7  
Error:java: Compilation failed: internal java compiler error  
原因是对应模块的java compiler不对，那么就去settings->java Compiler那里修改一下即可。

**编码**  
在eclipse里面直接项目右键就能打开属性设置项目的编码，intellij的话，File->settings->File encodings也可以设置了。

**导入eclipse项目**  
直接import项目，选择eclipse就可以了。然后libs的jar包，需要选中以后右键选择"Add as library"就像eclipse那样"Add to build path"，但是eclipse的貌似默认会依赖libs下的jar包。  
然后要保持编码一致就不会乱码了，基本上都使用utf-8比较好，以后少用gbk编码。

**如果在Run/Debug Configuration里面Launch default Activity选项中说找不到**  
是因为Java类没有编译成为class文件，所以找不到，这时候需要rebuild一下项目，就算rebuild完了也是找不到的，因为没有更新，所以需要在File那里点击Invalidate Caches/Restart。

**Troubleshooting**

同事跑公司项目报这个错：

Error:Android Pre Dex: classes.jar UNEXPECTED TOP-LEVEL EXCEPTION

然后下面的Exception stack几乎没有用，主要说是zip file is empty。。。
那就是说这个classes.jar是空的。。。说明intellij在dex的时候没成功什么的。很蛋疼。。。。搞来搞去都找不到原因，然后我就搜索项目下的所有classes.jar文件，
然后把0kb的删除，再跑项目，竟然奇迹的跑起来了。。。。好吧，无语了。。。。再说这个异常在我机器却没有。。。
但是，上次我这里跑yy2.0一直抛一个错，现在我都没解决呢。。。晕死了。。

**找不到包**  
如果项目找不到一些类，无法编译，那么就是可能没找到jar包，在项目的project structure 那里去看看，在SDKs那块，例如有时候依赖maven的android22，但是maven却没成功下载, 那么可以在classpath那里手动选择本地sdk的android.jar。

**一些常用快捷键**

我比较喜欢使用ctrl+w来关闭tab，这个在eclipse使用太好了，不明白这里为毛用ctrl+F4，实在手不够长啊。。。。。到settings->keymap：搜一下close，在Editor Tabs那里改改就好。当然可以在这里自定义自己的快捷键。

在Help->default keymap referrence可以找到官网的默认快捷键。

<table>
	<tr>
		<td>Alt+F7:  </td>
		<td>Find Usages</td>
	</tr>
	<tr>
		<td>Ctrl+J： </td>
		<td>自动补全</td>
	</tr>
	<tr>
		<td>Alt+上下箭头： </td>
		<td>在方法间移动</td>
	</tr>
	<tr>
		<td>Alt+左右箭头：</td>
		<td>在标签间移动</td>
	</tr>
	<tr>
		<td>Ctrl+H： </td>
		<td>可以看到类的继承和实现关系</td>
	</tr>
	<tr>
		<td>Ctrl＋Shift＋N：</td>
		<td>可以快速打开文件</td>
	</tr>
	<tr>
		<td>Ctrl＋N：</td>
		<td>可以快速打开类</td>
	</tr>
	<tr>
		<td>Ctrl＋P：</td>
		<td>方法参数提示</td>
	</tr>
	<tr>
		<td>Ctrl＋Q：</td>
		<td>显示注释文档</td>
	</tr>
	<tr>
		<td>Ctrl＋E：</td>
		<td>显示最近编辑的文件</td>
	</tr>
	<tr>
		<td>Ctrl＋Shift＋Backspace：</td>
		<td>跳转到上次编辑的地方</td>
	</tr>
	<tr>
		<td>Ctrl＋Alt + 左右箭头：</td>
		<td>移动光标到上次所在地方</td>
	</tr>
	<tr>
		<td>Ctrl＋Alt + L：</td>
		<td>格式化代码</td>
	</tr>
	<tr>
		<td>Ctrl+Shift+上下键：</td>
		<td>移动选中的代码</td>
	</tr>
	<tr>
		<td>Alt + Insert：</td>
		<td>Generate</td>
	</tr>
	<tr>
		<td>Ctrl + D：</td>
		<td>复制行</td>
	</tr>
	<tr>
		<td>Ctrl + X：</td>
		<td>删除行</td>
	</tr>
	<tr>
		<td>Ctrl + O：</td>
		<td>重载方法Override</td>
	</tr>
	<tr>
		<td>Ctrl + I：</td>
		<td>实现方法Implement</td>
	</tr>
	<tr>
		<td>Ctrl + Alt + 空格：</td>
		<td>提示，相当于eclipse的Alt+/</td>
	</tr>
	<tr>
		<td>Ctrl + Shift + F：</td>
		<td>在整个项目查询字符</td>
	</tr>
	<tr>
		<td>Double click shift：</td>
		<td>双击shift键，超级查询，可以查到jar包里面的东西</td>
	</tr>
	<tr>
		<td>Ctrl + F12</td>
		<td>查询方法</td>
	</tr>
	<tr>
		<td>Ctrl + Alt + O</td>
		<td>优化import自动去除无用的import语句</td>
	</tr>
	<tr>
		<td>Ctrl + G</td>
		<td>跳到某一行，相当于eclipse的Ctrl+L</td>
	</tr>
	<tr>
		<td>Ctrl + Tab/Ctrl + Shift + Tab</td>
		<td>可以选择界面的某一个区域，某一个版面</td>
	</tr>
	<tr>
		<td>Shift + Esc</td>
		<td>收起最小化</td>
	</tr>
	<tr>
		<td>Shift + F10</td>
		<td>run application</td>
	</tr>
	<tr>
		<td>Shift + F5</td>
		<td>自己定义的，没默认，Attach debugger to android process</td>
	</tr>
	<tr>
		<td>Alt + E</td>
		<td>自己定义的，默认是End键，移动到行尾。这个实在没有eclipse自动跳到右括号去方便啊。</td>
	</tr>
	<tr>
		<td>Alt + U</td>
		<td>自己定义的，没有默认，Clone caret Above。向上复制光标，可以多行编辑，和vim一样，超爽。</td>
	</tr>
	<tr>
		<td>Alt + D</td>
		<td>自己定义的，没有默认，Clone caret below。向下复制光标，可以多行编辑，和vim一样，超爽。</td>
	</tr>
	<tr>
		<td>Ctrl + Alt + I</td>
		<td>Indent，自动跳到适合位置，不用多次tab键，超爽。</td>
	</tr>
</table>

 
