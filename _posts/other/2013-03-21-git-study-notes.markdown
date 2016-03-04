---
layout: post
title:  "重量级分布式版本控制器Git教程"
date:   2013-03-21 11:00:03
categories: other
type: other
---

**一、什么是Git：**

Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的分布式版本控制软件。分布式和集中式的最大区别在于开发者可以本地提交。每个开发者机器上都有一个服务器的数据库。

**索引：**

Git需要将代码的变化显示的与下一次提交进行关联。举个例子，如果你对一个文件继续了修改，然后想将这些修改提交到下一次提交中，你必须将这个文件提交到索引中，通过git add file命令。这样索引可以保存所有变化的快照。

新增的文件总是要显示的添加到索引中来。对于那些之前已经提交过的文件，可以在commit命令中使用-a 选项达到提交到索引的目的。

下图是经典的git开发过程。

![git_workflow][git_workflow]

**从一般开发者的角度来看git有以下功能：**

* 从服务器上克隆数据库（包括代码和版本信息）到单机上。

* 在自己的机器上创建分支，修改代码。

* 在单机上自己创建的分支上提交代码。

* 在单机上合并分支。

* 新建一个分支，把服务器上最新版的代码fetch下来，然后跟自己的主分支合并。

* 生成补丁（patch），把补丁发送给主开发者。

* 看主开发者的反馈，如果主开发者发现两个一般开发者之间有冲突（他们之间可以合作解决的冲突），就会要求他们先解决冲突，然后再由其中一个人提交。如果主开发者可以自己解决，或者没有冲突，就通过。

* 一般开发者之间解决冲突的方法，开发者之间可以使用pull命令解决冲突，解决完冲突之后再向主开发者提交补丁。

**从主开发者的角度（假设主开发者不用开发代码）看，git有以下功能：**

* 查看邮件或者通过其它方式查看一般开发者的提交状态。

* 打上补丁，解决冲突（可以自己解决，也可以要求开发者之间解决以后再重新提交，如果是开源项目，还要决定哪些补丁有用，哪些不用）。

* 向公共服务器提交结果，然后通知所有开发人员。

**二、安装：**

ubuntu下命令安装：

> sudo apt-get install git-core

**三、配置：**

你可以在.gitconfig文件中防止git的全局配置。文件位于用户的home目录。上述已经提到每次提交都会保存作者和提交者的信息，这些信息都可以保存在全局配置中。

**用户信息**  

{% highlight bash %}
#通过如下命令来配置用户名和Email, 服务器默认地址
git config --global user.name "Example Surname"
git config --global user.email "your.email@gmail.com"
git config --global push.default "matching"
#获取Git配置信息，执行以下命令
git config --list
{% endhighlight %}

**高亮显示**

{% highlight bash %}
#以下命令会为终端配置高亮
git config --global color.status auto
git config --global color.branch auto
{% endhighlight %}

**忽略特定的文件**

可以配置Git忽略特定的文件或者是文件夹。这些配置都放在.gitignore文件中。这个文件可以存在于不同的文件夹中，可以包含不同的文件匹配模式。为了让Git忽略bin文件夹，在主目录下放置.gitignore文件，其中内容为bin。 同时Git也提供了全局的配置，core.excludesfile。

**使用.gitkeep来追踪空的文件夹**

Git会忽略空的文件夹。如果你想版本控制包括空文件夹，根据惯例会在空文件夹下放置.gitkeep文件。其实对文件名没有特定的要求。一旦一个空文件夹下有文件后，这个文件夹就会在版本控制范围内。

**四、操作：**

**创建仓库、添加文件和提交更改，查看日志**

每个Git仓库都是放置在.git文件夹下.这个目录包含了仓库的所有历史记录，.git/config文件包含了仓库的本地配置。

{% highlight bash %}
#以下将会创建一个Git仓库，添加文件倒仓库的索引中，提交更改。
git init
git add .
git commit -m "Initial commit"
git log
{% endhighlight %}

**diff命令与commit更改，查看状态**

通过git diff命令，用户可以查看更改。通过改变一个文件的内容，看看git diff命令输出什么，然后提交这个更改到仓库中，commit的-a参数不会提交未加索引的文件；

{% highlight bash %}
#比较两个更改集：
git diff changeset1 changeset2
#比较当前分支与另一个分支：
git diff branchname
#比较索引与工作空间：
git diff
#比较上一次提交与工作空间：
git diff HEAD
#比较上一次提交与所以：
git diff --cached
echo "This is a change" > test01
echo "and this is another change" > test02
git diff
git status
git commit -a -m "These are new changes"
{% endhighlight %}

**更正提交的信息 - git amend**

{% highlight bash %}
#通过git amend命令，我们可以修改最后提交的的信息  
#撤销上一次提交
git commit --amend -m "More changes - now correct"
{% endhighlight %}

**设置一个远端的Git仓库**

我们将创建一个远端的Git仓库。这个仓库可以存储在本地或者是网络上。  
远端Git仓库和标准的Git仓库有如下差别：一个标准的Git仓库包括了源代码和历史信息记录。我们可以直接在这个基础上修改代码，因为它已经包含了一个工作副本。但是远端仓库没有包括工作副本（工作空间），只包括了历史信息。可以使用--bare选项来创建一个这样的仓库。
{% highlight bash %}
git clone --bare . ../remote-repository.git
{% endhighlight %}

**推送更改到其他的仓库**
{% highlight bash %}
#做一些更改，然后将这些更改从你的第一个仓库推送到一个远端仓库 
git push ../remote-repository.git
{% endhighlight %}

**添加远端仓库**

除了通过完整的URL来访问Git仓库外，还可以通过git remote add命令为仓库添加一个短名称。当你克隆了一个仓库以后，origin表示所克隆的原始仓库。即使我们从零开始，这个名称也存在。 
{% highlight bash %}
git remote add origin ../remote-repository.git 
git push origin
#通过以下命令查看已经存在的远端仓库 
git remote
{% endhighlight %}

**克隆仓库**
{% highlight bash %}
#通过以下命令在当前目录下创建一个新的仓库 
git clone ../remote-repository.git .
{% endhighlight %}

**拉取（Pull）更改**

通过拉取，可以从其他的仓库中获取最新的更改。在第二个仓库中，做一些更改，然后将更改推送到远端的仓库中。然后第一个仓库拉取这些更改。这里注意到这个pull命令包含了fetch和merge，相当于hg的（pull和update）。
{% highlight bash %}
git pull ../remote-repository.git/
{% endhighlight %}

**还原更改**
{% highlight bash %}
#如果在你的工作副本中，你创建了不想被提交的文件，你可以丢弃它。 
touch test04
echo "this is trash" > test04
git clean -n
git clean -f
#你可以提取老版本的代码，通过提交的ID。git log命令可以查看提交ID 
git log
git checkout commit_name
#如果你还未把更改加入到索引中，你也可以直接还原所有的更改 
echo "nonsense change" > test01
git checkout test01
#如果更改已经加到索引，使用reset命令恢复index
echo "another nonsense change" > test01
git add test01
git reset HEAD test01
git checkout test01
#以上两条命令可以合并为git checkout HEAD test01
#也可以通过revert命令进行还原操作 
git revert commit_name
#即使你删除了一个未添加到索引(这里的索引和hg的add添加hg管理范围不一样)和提交的文件，你也可以还原出这个文件
rm test01
git checkout test01
#如果你已经添加一个文件到索引中，但是未提交。可以通过git reset file 命令将这个文件从索引中删除 
touch incorrect.txt
git add .
git reset incorrect.txt
#如果你删除了文件夹且尚未提交，可以通过以下命令来恢复这个文件夹 。译者注：即使已经提交，也可以还原
git checkout HEAD — your_dir_to_restore
{% endhighlight %}

**标记**
Git可以使用对历史记录中的任一版本进行标记。这样在后续的版本中就能轻松的找到。一般来说，被用来标记某个发行的版本  
可以通过git tag命令列出所有的标记，通过如下命令来创建一个标记和恢复到一个标记
{% highlight bash %}
git tag version1.6 -m 'version 1.6'      
git checkout <tag_name>
{% endhighlight %}

**分支**

通过分支，可以创造独立的代码副本。默认的分支叫master。Git消耗很少的资源就能创建分支。Git鼓励开发人员多使用分支
{% highlight bash %}
#下面的命令列出了所有的本地分支，当前所在的分支前带有*号
git branch 
#如果你还想看到远端仓库的分支，可以使用下面的命令
git branch -a
#可以通过下面的命令来创建一个新的分支，然后切换到分支去
git branch testing
git checkout testing
echo "Cool new feature in this branch" > test01
git commit -a -m "new feature"
#commit的-a参数自动提交修改的代码和自动删除在索引不在工作空间的文件
git checkout master
cat test01
{% endhighlight %}

**合并**

通过Merge我们可以合并两个不同分支的结果。Merge通过所谓的三路合并来完成。分别来自两个分支的最新commit和两个分支的最新公共commit  
可以通过如下的命令进行合并 
{% highlight bash %}
git merge testing
{% endhighlight %}
一旦合并发生了冲突，Git会标志出来，开发人员需要手工的去解决这些冲突。解决冲突以后，就可以将文件添加到索引中，然后提交更改

**删除本地和远程分支**
{% highlight bash %}
#删除分支的命令如下： 
git branch -d testing
git push origin —delete <branch name>
{% endhighlight %}

**推送（push）一个分支到远端仓库**

默认的，Git只会推送匹配的分支的远端仓库。这意味在使用git push命令默认推送你的分支之前，需要手工的推送一次这个分支。 
{% highlight bash %}
git push origin testing
git checkout testing
echo "News for you" > test01
git commit -a -m "new feature in branch"
git push
{% endhighlight %}

**解决合并冲突**

如果两个不同的开发人员对同一个文件进行了修改，那么合并冲突就会发生。而Git没有智能到自动解决合并两个修改
{% highlight bash %}
#下面会产生一个合并冲突 
cd ~/repo01
touch mergeconflict.txt
echo "Change in the first repository" > mergeconflict.txt
git add . && git commit -a -m "Will create merge conflict 1"
cd ~/repo02
touch mergeconflict.txt
echo "Change in the second repository" > mergeconflict.txt
git add . && git commit -a -m "Will create merge conflict 2"
git push
cd ~/repo01
git push
git pull origin master
#Git将冲突放在收到影响的文件中，文件内容如下： 
<<<<<<< HEAD
Change in the first repository
=======
Change in the second repository
>>>>>>> b29196692f5ebfd10d8a9ca1911c8b08127c85f8
#上面部分是你的本地仓库，下面部分是远端仓库。现在编辑这个文件，然后commit更改。另外的，你可以使用git mergetool命令
git mergetool
git commit -m "merged changes"
{% endhighlight %}

**Checkout命令总结：**

`checkout`命令通常用来从仓库中取出文件，或者在分支中切换。如果指定了一个checkout版本，则把文件复制到工作空间和index区，如`git checkout HEAD~ files`，分支没有变化。如果指定了分支名字，则切换分支，如`git checkout main`。如果没有指定版本和分支，仅仅是文件名，则从index复制到工作空间，若工作空间没有更新，则从最新的版本复制。如果既没有指定文件名，也没有指定分支名，而是一个标签、远程分支、SHA-1值或者是像master~3类似的东西，就得到一个匿名分支，称作detached HEAD，这个匿名分支是危险的，一旦提交了新的版本然后切换到其他分支，匿名分支的工作就会丢失，因此这时需要指定这个匿名分支的名字`git checkout -b newname`。

**Reset命令总结：**

`reset`用来撤销最后一次`git add files`，你也可以用`git reset`撤销所有暂存区域文件。`reset`命令把当前分支指向另一个版本，并且有选择的变动工作目录和索引。也用来在从历史仓库中复制文件到索引，而不动工作目录。如果不给选项，那么当前分支指向到那个提交。如果用--hard选项，那么工作目录也更新，如果用--soft选项，那么都不变。如果没有给出提交点的版本号，那么默认用HEAD。这样，分支指向不变，但是索引会回滚到最后一次提交，如果用--hard选项，工作目录也同样。如果给了文件名(或者-p选项), 那么工作效果和带文件名的`checkout`差不多，除了索引被更新。

**和HG的区别：**

* hg的add是添加文件到hg的管理范围，这样以后hg每次提交时就不需要add了；Git的add是添加文件到索引区，而且每次提交都要先add到索引区。

* hg要pull了以后，再update到工作空间，pull是把服务器的更改集拉到本地仓库。Git是用fetch是把服务器的更改集拉到本地仓库，然后用merge合并到工作空间，pull命令合并了这两个命令。

**五、托管库：**

> https://github.com/

**六、GUI版本TortoiseGit**

**七、引用：**

Git一分钟教程：

>http://blog.enjoyrails.com/2008/12/31/git一分钟教程

Git中文教程：
>http://www.open-open.com/lib/view/open1328928294702.html

Git教程：
>http://www.cnblogs.com/zhangjing230/archive/2012/05/09/2489745.html

Git入门教程：
>http://hi.baidu.com/eehuang/item/22283e220437a80d76272cb7

图解Git：
>http://my.oschina.net/xdev/blog/114383

**八、常见错误**

**问题1**

> fault: cannot use bin as an exclude file git.

刚想提交一个java项目的时候发现提示这个错误，以为是\.gitignore配置错误了，里面的配置过滤的是bin\\，明显是正常的，然后google了一下，只有两条记录，一条是stackoverflow上面的，没帮助！另外一条被墙了，打不开。想了一下后去看看全局配置，  发现全局的\.gitignore配置也是对的，而且把全局和项目的这个bin都删掉，还是提示错误，最后看了一下全局的**\.gitconfig**文件，发现里面这条配置：  

{% highlight vim %}
[core]
	excludefile=bin
{% endhighlight %}

这个估计是在学习git的时候配置的，它的值只能是file，不能是文件夹，所以出错拉，删掉就好了！

**问题2**  
从工作室的RhodeCode的仓库创建一个git项目以后，克隆下来的时候发现这个错误：

{% highlight vim %}
server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificat
{% endhighlight %}

原因证书问题，这个证书没有正式配置吧，所以只能绕过不验证了，在本机配置一个环境变量,和我們使用hg時候那個問題是一致的。

{% highlight vim %}
export GIT_SSL_NO_VERIFY=1
{% endhighlight %}


**附上git命令大图**

<img src="/image/git_command.jpg" width="1100px">


[git_workflow]: /image/git_workflow.jpg
[git_command]:   =1000x