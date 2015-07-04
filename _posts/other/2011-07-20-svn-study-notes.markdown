---
layout: post
title:  "TortoiseSVN/Subversion 学习、理解和一些重要的使用记录"
date:   2011-07-20 11:00:03
categories: other
type: other
---

# 什么是SVN

svn是Subversion的缩写，因为在命令行里面命令肯定是越短越好的嘛。TortoiseSVN是GUI版本的svn而已，比较好用。svn是比较简单的版本控制器，是单一的集中式的中央仓库，所有人都必须提交过去。比较好理解和使用，具体我就不说了，看我另外一篇scm的blog足以。

[传送门](http://zhgeaits.me/other/2013/03/06/understand-scm.html "scm")

下面我就记录一些平时的记录

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