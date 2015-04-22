---
layout: post
title:  "SVN，TortoiseSVN，Subversion学习，理解，使用记录"
date:   2011-07-20 11:00:03
categories: other
type: other
---

* **TortoiseSVN查看某一个文件的版本历史**  
在项目目录右键show log是查看项目的提交历史。如果选择某个文件右键show log可以看到这个文件的提交历史。

* **回滚revert**  
选择一个文件，然后右键，有revert，点击可以回滚到上一个版本，如果想要回滚到更前的版本，可以选中右键选择show log，然后选中不需要的版本，然后右键选择revert from these versions

* **查看具体每一行是谁修改**  
选择一个文件，然后右键，有一个Blame..的选项，点击即可。

* **Other**  
在公司里面每次要发布版本了，就禁止提交svn，然后那边构建提交。。。好傻的做法啊，为什么要禁止提交啊，那边的构建系统做得太不好了，那有让程序员不能提交代码的啊。。。应该打一个tag，指定版本来构建发布，然后我提交的就毫无关系了。。。。烦，每次提交都被批。。