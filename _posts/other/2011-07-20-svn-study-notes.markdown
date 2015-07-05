---
layout: post
title:  "TortoiseSVN/Subversion 学习、理解和一些重要的使用记录"
date:   2011-07-20 11:00:03
categories: other
type: other
---

# 什么是SVN

svn是Subversion的缩写，因为在命令行里面命令肯定是越短越好的嘛。Subversion是一个版本控制系统，相对于的RCS、CVS，采用了分支管理系统，它的设计目标就是取代CVS。互联网上免费的版本控制服务多基于Subversion。而TortoiseSVN是GUI版本的svn而已，比较好用。svn是比较简单的版本控制器，是单一的集中式的中央仓库，所有人都必须提交过去。比较好理解和使用，具体我就不说了，看我另外一篇scm的blog足以。

[传送门](http://zhgeaits.me/other/2013/03/06/understand-scm.html "scm")

**svn有如下特点**

1. 原子提交。一次提交不管是单个还是多个文件，都是作为一个整体提交的。在这当中发生的意外例如传输中断，不会引起数据库的不完整和数据损坏。
2. 重命名、复制、删除文件等动作都保存在版本历史记录当中。
3. 对于二进制文件，使用了节省空间的保存方法。（简单的理解，就是只保存和上一版本不同之处）
4. 目录也有版本历史。整个目录树可以被移动或者复制，操作很简单，而且能够保留全部版本记录。
5. 分支的开销非常小。
6. 优化过的数据库访问，使得一些操作不必访问数据库就可以做到。这样减少了很多不必要的和数据库主机之间的网络流量。

## svn架构

![alt svn_structure](/image/svn_structure.png "svn_structure")
 
Subversion的版本库可以通过网络访问，从而使用户可以在不同的电脑上进行操作。从某种程度上来说，允许用户在各自的空间里修改和管理同一组数据 可以促进团队协作。因为修改不再是单线进行，开发速度会更快。此外，由于所有的工作都已版本化，也就不必担心由于错误的更改而影响软件质量—如果出现不正确的更改，只要撤销那一次更改操作即可。

## 使用

在windows下的GUI版本的使用比较简单，就算是server端的GUI使用也比较简单，下面我记录的linux的使用：

### 安装

可以采用命令行安装：

>sudo apt-get install subversion

也可以采用编译源码安装（建议，有助于学习）：

编译安装需要安装很多依赖的库，其中包括`libapr`,`libapr-util`,`expat`和`libneon`。先去下载好这些包和最新版本的subversion就可以。

安装过程其实就是那几步，tar解压，configure配置，make编译连接，make install安装。

**安装libapr:**

>tar –zxvf apr-1.4.6.tar  
./configure –prefix=/usr/local/apr  
make  
make install

**安装libapr-util:**

>tar –zxvf apr-util-1.3.4.tar.gz  
./configure –prefix=/usr/local/apr  
make  
make install

**安装expat：**

>tar zxvf expat-2.0.1.tar.gz  
./configure  
make  
make install

**安装libneon:**

>tar zxvf neon-0.28.2.tar.gz  
./configure --prefix=/usr/local/neon  
make  
make install

**安装subversion:**

>./configure --prefix=/usr/local/svn --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr --with-neon=/usr/local/neon  
make  
make install

安装完毕以后执行svn —version就可以看到版本号了

### 配置服务器仓库

**创建一个库：**

>svnadmin create /home/zhangge/svnrepos

**配置`svnserve.conf`：**

>anon-access = read  
auth-access = write  
password-db = passwd  
authz-db = authz  
realm = zhangge

注意这些配置去点\#号以后前面不能留空格

**配置`passwd`:**

在`svnserve.conf`里面指定密码数据库有`passwd`这个文件以`username=password`的形式配置即可。

**配置authz：**

这里配置不同用户的权限，而且最难就是在这里了，关键就在这里：

运行svn时`svnserver -d -r /home/zhangge/svn`，svn这个目录可以是一个库；也可以是一个目录，下面有很多库,如库1：svn1，库2：svn2

如果是一个库，配置按如下1，2说明。如果是一个目录，则需要区分库和库的根目录权限配置,如下面3，4配置

具体配置如下：

>\#1.版本库根目录的权限  
[/]  
*=rw  

>\#2.版本库这个目录的所有权限  
[/testproject1]  
*=rw

>\# svn1这个库的根目录权限  
\#[svn1:/]  
\#*=rw

>\# svn1这个版本库的某个目录权限  
\#[svn1:/testproject1/trunk]  
\#*=rw

>\# 对于svn2这个版本库的配置也是一样的

**最后运行服务端：**

>svnserve -d -r /home/zhangge/svnrepos

### 客户端操作

**1、将文件checkout到本地目录**

>svn checkout path（path是服务器上的目录）  
例如：svn checkout svn://192.168.1.1/pro/domain  
简写：svn co

**2、往版本库中添加新的文件**

>svn add file  
例如：svn add test.php(添加test.php)  
svn add *.php(添加当前目录下所有的php文件)

**3、将改动的文件提交到版本库**

>svn commit -m "LogMessage" [-N] [–no-unlock] PATH  
(如果选择了保持锁，就使用–no-unlock开关)  
例如：svn commit -m "add test file for my test" test.php  
简写：svn ci

**4、加锁/解锁**

>svn lock -m "LockMessage" [–force] PATH  
例如：svn lock -m "lock test file" test.php  
svn unlock PATH

**5、更新到某个版本**

>svn update -r m path  
例如：  
svn update如果后面没有目录，默认将当前目录以及子目录下的所有文件都更新到最新版本。  
svn update -r 200 test.php(将版本库中的文件test.php还原到版本200)  
svn update test.php(更新，于版本库同步。如果在提交的时候提示过期的话，是因为冲突，需要先update，修改文件，然后清除svn resolved，最后再提交commit)  
简写：svn up

**6、查看文件或者目录状态**

>1）svn status path  
显示目录下的文件和子目录的状态，正常状态不显示，有下面这些状态：    
?：不在svn的控制中；M：内容被修改；C：发生冲突；A：预定加入到版本库；K：被锁定

>2）svn status -v path  
显示文件和子目录状态  
第一列保持相同，第二列显示工作版本号，第三和第四列显示最后一次修改的版本号和修改人。  
**注：**`svn status`、`svn diff`和`svn revert`这三条命令在没有网络的情况下也可以执行的，原因是svn在本地的.svn中保留了本地版本的原始拷贝。  
简写：svn st

**7、删除文件**

>svn delete path -m "delete test fle"  
例如：svn delete svn://192.168.1.1/pro/domain/test.php -m "delete test file"  
或者直接svn delete test.php 然后再svn ci -m "delete test file"，推荐使用这种  
简写：svn (del, remove, rm)

**8、查看日志**

>svn log path  
例如：svn log test.php 显示这个文件的所有修改记录，及其版本号的变化

**9、查看文件详细信息**

>svn info path  
例如：svn info test.php

**10、比较差异**

>svn diff path(将修改的文件与基础版本比较)  
例如：svn diff test.php  
svn diff -r m:n path(对版本m和版本n比较差异)  
例如：svn diff -r 200:201 test.php  
简写：svn di

**11、将两个版本之间的差异合并到当前文件**

>svn merge -r m:n path  
例如：svn merge -r 200:205 test.php（将版本200与205之间的差异合并到当前文件，但是一般都会产生冲突，需要处理一下）

**12、SVN 帮助**

>svn help
svn help ci

**以上是常用命令，下面写几个不经常用的**

**13、版本库下的文件和目录列表**

>svn list path  
显示path目录下的所有属于版本库的文件和目录  
简写：svn ls

**14、创建纳入版本控制下的新目录**

>svn mkdir: 创建纳入版本控制下的新目录。  
用法:   
1、mkdir PATH…  
2、mkdir URL…  

>1、每一个以工作副本 PATH 指定的目录，都会创建在本地端，并且加入新增调度，以待下一次的提交。  
2、每个以URL指定的目录，都会透过立即提交于仓库中创建。  
在这两个情况下，所有的中间目录都必须事先存在。

**15、恢复本地修改**

>svn revert: 恢复原始未改变的工作副本文件 (恢复大部份的本地修改)。  
用法: revert PATH…  
注意: 本子命令不会存取网络，并且会解除冲突的状况。但是它不会恢复被删除的目录

**16、代码库URL变更**

>svn switch (sw): 更新工作副本至不同的URL。  
用法:   
1、switch URL [PATH]  
2、switch –relocate FROM TO [PATH…]  
1、更新你的工作副本，映射到一个新的URL，其行为跟“svn update”很像，也会将服务器上文件与本地文件合并。这是将工作副本对应到同一仓库中某个分支或者标记的方法。  
2、改写工作副本的URL元数据，以反映单纯的URL上的改变。当仓库的根URL变动(比如方案名或是主机名称变动)，但是工作副本仍旧对映到同一仓库的同一目录时使用这个命令更新工作副本与仓库的对应关系。

**17、解决冲突**

>svn resolved: 移除工作副本的目录或文件的“冲突”状态。  
用法: resolved PATH…  
注意: 本子命令不会依语法来解决冲突或是移除冲突标记；它只是移除冲突的相关文件，然后让 PATH 可以再次提交。

**18、输出指定文件或URL的内容。**

>svn cat 目标[@版本]…如果指定了版本，将从指定的版本开始查找。  
svn cat -r PREV filename > filename (PREV 是上一版本,也可以写具体版本号,这样输出结果是可以提交的)

## 下面是windows下的GUI版本平时的使用记录

* **TortoiseSVN查看某一个文件的版本历史**  
在项目目录右键show log是查看项目的提交历史。如果选择某个文件右键show log可以看到这个文件的提交历史。

* **回滚revert**  
选择一个文件，然后右键，有revert，点击可以回滚到上一个版本。如果想要回滚到更前的版本，可以选中右键选择show log，然后选中不需要的版本，然后右键选择revert from these versions

* **查看具体每一行是谁修改**  
选择一个文件，然后右键，有一个Blame..的选项，点击即可。

* **合并分支**  
在项目目录下右键选择Merge...，Merge Type我们选择`Merge a range of revisions`就好了,另外两个选项，一个`Reintegrate a branch`是把完整的分支合并到trunk，一个`Merge two different trees`是同时合并两个分支。点击下一步以后，我们在`URL to merge from`选择要合并的分支，在`Revision range to merge`选择要合并的版本，不要勾择`Reverse merge`，点击下一步，`Merge depth`选择`Working copy`，然后只选择`Compare whitespaces`其他不选择，最后点击合并即可了。这些是合并的本地，然后我们还需要提交上去，就完事了。

* **Other**  
在公司里面每次要发布版本了，就禁止提交svn，然后那边构建提交。。。好傻的做法啊，为什么要禁止提交啊，那边的构建系统做得太不好了，那有让程序员不能提交代码的啊。。。应该打一个tag，指定版本来构建发布，然后我提交的就毫无关系了。。。。烦，每次提交都被批。。