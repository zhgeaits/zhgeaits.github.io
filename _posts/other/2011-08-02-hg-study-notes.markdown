---
layout: post
title:  "轻量级分布式版本控制器Hg/Mercurial教程"
date:   2011-08-02 11:00:03
categories: other
supertype: career
type: other
---

>当时异维工作室使用的就是这个版本控制器，我在星辰刚学了使用svn，对于什么是scm，版本控制都不懂就要开始使用分布式仓库的版本控制器，是非常困难的。第一次使用，是那个碳资产系统上(唉，回不去的光阴啊)。

下面是当年写给工作室的教程：

## 1. 什么是Hg

Hg是Mercurial的缩写，就是一个调用程序，是为了命令更加方便而已，就像svn就是subversion一样。Mercurial 是一个跨平台的分布式版本控制软件。Mercurial主要由Python语言实现，不过也包含一个用C实现的二进制比较工具。

**优点**

* 更轻松的管理。  
传统的版本控制系统使用集中式的 repository，一些和 repository相关的管理就只能由管理员一个人进行。由于采用了分布式的模型，Mercurial 中就没有这样的困扰，每个用户管理自己的 repository，管理员只需协调同步这些repository。

* 更健壮的系统。  
分布式系统比集中式的单服务器系统更健壮，单服务器系统一旦服务器出现问题整个系统就不能运行了，分布式系统通常不会因为一两个节点而受到影响。

* 对网络的依赖性更低。  
由于同步可以放在任意时刻进行，Mercurial 甚至可以离线进行管理，只需在有网络连接时同步。

**安装**

一个快捷的方法：sudo apt-get install mercurial(版本是1.4太老了，下面介绍编译安装)

## 2. 我们的工作流

工作室服务器作为中央仓库，使用scm来创建托管项目，开发人员clone项目到本地，当开发者完成一些功能或者修复一些bug以后，提交更改集到本地仓库，这时，我们是不需要连接上服务器的。然后连接工作室的服务器，拉取中央的head版本到本地，并且更新合并，最后提交本地仓库，再推送到中央服务器。尽量不要让分支太长，也就是要尽早合并，否则你会很egg painful。

## 3. 操作说明

* 初始化项目：

进入项目执行hg init或者直接hg init projectname。这一步一般由组长或者管理员来做。

* 添加文件到版本库：

其实是添加到一个暂存区（加入hg的索引,hg叫做跟踪），提交（commit）以后才会加入版本库，hg add Hello.java，如果add不加参数就是添加所有文件

* 提交一个更改集到版本库：

hg commit -m "add Hello.java as first init"，-m是添加一个提交说明。加-A选项会添加新文件，删除丢失文件。

* 克隆服务器版本到本地：

hg clone https://server/project .

* 本地开发者1提交后推送到服务器：

hg push，因为这是克隆，在.hg/hgrc配置有default目的服务器，否则有填写服务器的地址。

* 本地开发者2从服务器拉取更改集：

hg pull。

* 本地开发者2要把本地仓库更新到工作空间：

hg update.

* 查看本地日志：

hg log可以加-r number指定更改集号码

* 查看状态：

hg status -A

* 查看帮助：

hg help commandname

* 查看最新版本：

hg tip -vp 我们通常把版本库中最新的版本称为tip版本或者简称为tip。

* 撤销跟踪文件：

hg remove filename

* 回复删除的文件:

hg revert filename

## 4. 配置

在主目录下有一个配置文件.hgrc它是一个全局的配置文件，建议加入以下配置
{% highlight xml %}
[web]
push_ssl=false
cacerts=

//以上配置解决push时因为ssl的原因失败问题
//在每一个项目下有一个.hg目录，里面有配置文件hgrc，可以加入以下配置：

[auth]
yourproject.prefix=https://www.xhomestudio.org/hg/yourproject
yourproject.username=zhangge
yourproject.password=zhangge
以上配置指定服务器地址和认证的帐号密码

[ui]
username = zhangge提交版本显示的作者
merge = meld//三路合并工具
	
[tortoisehg]
postpull = update//拉取后更新
vdiff = meld//可视化比较工具
editor = gedit//GUI编辑工具，编辑设置文件
	
[extensions]//打开扩展功能，扩展比较工具
hgext.extdiff =
	
[extdiff]//配置扩展比较工具
cmd.meld =
{% endhighlight %}

## 5. GUI版本的HG

* TortoiseHg  
**第一步：**到官网下载一个安装包：http://tortoisehg.bitbucket.org/download/  
**第二步：**Exe安装包，一直点击下一步，没什么好说的，安装完毕以后需要重启。  
**第三步：**在无中文的路径上创建一个测试目录：TestHg，然后右键，可以看到：  

![hg1][hg1]

Clone就是克隆一个仓库。  
Create Repository Here就是创建一个仓库。  
Explorer Extension Settings就是shell的一些额外设置  
Global Settings就是设置全局参数  
其他不说了，自己看就明白，也可以进入Hg Workbench面板进行操作。  
**第四步：**创建一个仓库，可以看到设置仓库路径，实际的hg命令和一些参数等：  

![hg2][hg2]

成功以后看到这两个文件，以点开头的文件在unix下是隐藏的。.hg目录放了一些配置文件和数据文件。.hgignore是配置提交时的过滤不提交文件。

![hg3][hg3]

**第五步：**在另一个文件夹克隆刚才创建的仓库：

![hg4][hg4]

可以看到你需要填写仓库的路径和到本地的路径和实际的hg命令。你还能看到一些选项，例如克隆制定版本。我们工作时一般的开发你不需要创建库，我们在服务器创建好了，你克隆到本机就可以了。成功以后你可以看到.hg目录，里面有一个hgrc配置文件，打开以后有一个default属性，配置的就是默认的中央仓库。

**第六步：**创建一个java文件，然后提交到本地仓库。这时你在工作空间右键，你会发现多了很多选项，首先就是Hg Commit，这个是提交到本地仓库；其他还有更多的选项操作，而这些操作都可以在Hg workbench完成。

![hg5][hg5]

我们点击Hg Commit后如下：

![hg6][hg6]

左边的工作空间的文件，有一个状态“？”表示为加入hg的管理范围，勾上它表示你要提交；右上框填写提交日志，左下框显示文件的内容。提交后提示你要设置一个用户名：

![hg7][hg7]

点击后进入设置面板：

![hg8][hg8]

可以看到两个版面，一个是全局设置，一个是当前工作空间的设置。点击右上方的编辑文件进行设置，设置内容和hgrc一样。

![hg9][hg9]

保存退出设置面板以后提示：

![hg10][hg10]

意思就是把文件添加到hg的管理范围。提交成功以后，我们右键打开Hg workbench看看：

![hg11][hg11]

这里我为了更加明显的显示基线，所以多提交了两次。我们看到选中的一行就是刚才提交的版本。历史图是每个版本的推进关系，显示了修订版本号，分支是default，描述，版本提交的作者，距离现在的提交时间，标签等。下半部分和提交时的是一样的内容。菜单上的按钮，只要你把鼠标移到那里就会显示内容。常用的是把分支推送到仓库，我们现在推送到中央仓库，然后提示推送完成：

![hg12][hg12]

**第七步：**克隆另外一个仓库作为第二个开发者，然后提交推送一些文件，步骤省略。  
**第八步：**第一个开发者拉取开发者2的版本。打开Hg workbench拉取，结果如下：

![hg13][hg13]

可以看到有一个分支，并且提示当前工作目录不是Head版本，因为当前的工作空间是基于之前本地提交的版本，不是基于从中央仓库拉取的最新版本，所以产生了分支，那么就更新到本地吧。选中拉取的最新版本点击右键：

![hg14][hg14]

注意，当前工作空间基于的版本是2，而且分支基于的版本也是2，所以成功更新。否则就需要合并了。  
**第九步：**开发者1和2分别在本地提交自己版本，然后合并。这正符合分布式的开发工作了流。开发者1提交到本地仓库，开发者2提交本地仓库。然后开发者1推送到中央仓库，最后开发者2推送到中央仓库（记住，在推送前先从中央仓库拉取一下，否则你会推送失败）：

![hg15][hg15]

开发者2拉取以后你看到两个真实的分支了。然后选择分支点击右键，选择Merge with Local进行合并：

![hg16][hg16]

![hg17][hg17]

可以看到一些合并的信息（很有用的哦！）和一些选项！

![hg18][hg18]

![hg19][hg19]

合并完毕后填写合并说明日志然后提交

![hg20][hg20]

最后推送到中央服务器！开发者1这时可以拉取下来看看！  
如果开发者2选择的不是合并而是更新，那么当前工作空间就会切换到拉取下来的分支！就是说拉取下来的版本会作为主线，而你原来的版本成为一个分支！  
**第十步：**解决多个开发者同时修改同一个文件的冲突。我们试试开发者1和2都修改readme.txt然后提交到服务器看看：

![hg21][hg21]

依然产生了一个分支，我们开始合并：

![hg22][hg22]

看到无法合并，需要解决合并冲突！点击以后：

![hg23][hg23]

这是采用了三路合并原则！可以选择Mercurial自己解决，可以选择设置的工具解决，可以选择本地版本，可以选择其他版本，我们选择Kdiff工具查看：

![hg24][hg24]

上左A是本地版本和分支的基节点，上中B是本地版本，上右C是分支版本，下方是你修改保存的新版本。冲突的地方是标红的，选择冲突的区域，然后点击菜单来上面的ABC分别可以把不同版本的内容添加到下面的编辑窗口，然后你就可以编辑修改了，完了以后保存，冲突的文件会跳到以解决冲突窗口，退出后提交：

![hg25][hg25]

对于解决冲突的文件会保留一个.orig的文件，是你原来本地版本的文件。
**第十一步：**从工作室的服务器拉取一个项目试一下，我已经创建好库了：

![hg26][hg26]

提示服务器认证失败：

![hg27][hg27]

那么你可以勾选不要验证服务器证书就可以拉取了！填写帐号密码即可，这个是LDAP的帐号密码，没有的话告诉我，我来分配！

![hg28][hg28]

拉取成功以后，提交一个版本，然后推到服务器看看，依然认证失败：

![hg29][hg29]

那么在全局配置文件里面加入这个，然后重启HG就可以推了。

![hg30][hg30]

这个KDiff用得蛋疼，强烈建议安装一个meld，我在ubuntu下用得爽，windows下和hg配合不起来，所以windows下不建议使用，也还有很多工具自己去找！

Linux安装命令：
>sudo apt-get install meld。

然后按照上面配置就行了。

## 6. 源码托管

> https://bitbucket.org

只要你注册一个帐号就可以登录，然后创建一个HG的库以后就可以克隆这个库了。

## 7. 常见问题

**1.**	本地库的更改集不是服务器更改集的字节点，然后就push到服务器，就是常见的没拉取就推送，服务器的版本与本地的不一样（正常情况是本地的版本比服务器的要新）：

提示：
>abort: push creates new remote heads!  
(did you forget to merge? use push -f to force)

推送终止，因为在服务器会创造多个heads，目前还是正常的！（我用hg heads查看都是正常的）然后pull下来，可以看到多了一个heads，就是多个一个分支，这是可以运行hg heads查看多个heads，再运行hg merge合并，如果没冲突，合并顺利完成，否则会进入编辑阶段，就是所谓的三路合并了（本地版本，服务器版本，本地和服务器的最新共同版本），完成以后就commit，再推送到服务器。

**2.**	我们以前说的没拉就推其实是可以的（实际想说的是没拉取更新就提交一个版本），之前我们的思想还是集中式的，不想产生分支！其实这正是分布式的好处，可以本地提交，不过我们要做的是，分支要尽快合并，除非是开发其他的版本，否则你会很egg painful！

## 8. 参考教程

>http://wenku.baidu.com/view/f9733b1e59eef8c75fbfb317  
http://blog.csdn.net/vagrxie/article/details/4593687  
http://mercurial.selenic.com/wiki/CategoryChinese  

**最后送上两张图，信息量大，呵呵**

![hg_workflow][hg_workflow]

![hg_tutorial][hg_tutorial]


[hg1]: /image/hg1.png
[hg2]: /image/hg2.png
[hg3]: /image/hg3.png
[hg4]: /image/hg4.png
[hg5]: /image/hg5.png
[hg6]: /image/hg6.png
[hg7]: /image/hg7.png
[hg8]: /image/hg8.png
[hg9]: /image/hg9.png
[hg10]: /image/hg10.png
[hg11]: /image/hg11.png
[hg12]: /image/hg12.png
[hg13]: /image/hg13.png
[hg14]: /image/hg14.png
[hg15]: /image/hg15.png
[hg16]: /image/hg16.png
[hg17]: /image/hg17.png
[hg18]: /image/hg18.png
[hg19]: /image/hg19.png
[hg20]: /image/hg20.png
[hg21]: /image/hg21.png
[hg22]: /image/hg22.png
[hg23]: /image/hg23.png
[hg24]: /image/hg24.png
[hg25]: /image/hg25.png
[hg26]: /image/hg26.png
[hg27]: /image/hg27.png
[hg28]: /image/hg28.png
[hg29]: /image/hg29.png
[hg30]: /image/hg30.png
[hg_workflow]: /image/hg_workflow.png
[hg_tutorial]: /image/hg_tutorial.jpg