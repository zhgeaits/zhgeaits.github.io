---
layout: post
title:  "Android的拉取刷新学习笔记!"
date:   2014-08-05 11:00:03
categories: private
type: private
---

以前做毕设的时候如果去使用的话就更好了。当时对android不是很了解，不知道有下面这两个库：    
https://github.com/johannilsson/android-pulltorefresh  
https://github.com/chrisbanes/Android-PullToRefresh  

当时毕设是模仿微信，用别人的一个东西，不是下拉刷新，这个以后再补充

现在发现公司其实用的是上面的第二个，然后稍作修改成为自己的来使用。然后再加一个无限下拉的控件使用。现在不研究这些控件怎么实现的，以后有时间再去学习。现在就学会使用就OK。

使用其实很简单，github上的页面已有介绍，基本就是在布局设置控件，然后设置listener就完了。

注意：  
这个下拉刷新是包含了一个listview的，如果要设置onTouch事件必须先拿到真正的listview：listView.getRefreshableView()。