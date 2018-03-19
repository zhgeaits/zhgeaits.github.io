---
layout: post
title:  "Android的AsyncTask学习笔记!"
date:   2014-02-07 14:00:03
categories: private
type: private
---

AsyncTask是异步的后台线程，不是进程，其实具体要看源码才知道。它干完了创建，管理，同步等所有工作，所以我们用起来十分方便。

把它理解为一个异步的任务，在后台执行处理，处理完毕以后进行UI更新。而且最好用在生命周期较短的UI，或者需要在UI显示进度和结果的地方。
对于生命周期较长的操作，如下载数据，最好用Service了。AsyncTask是绑定在UI线程的，所以，它必须在UI线程里面创建和执行。
一旦activity被销毁或重新创建，则asynctask会被取消，就是只能执行一次。一般使用AsyncTask作为内部类使用，更方便更新UI。

使用：  
继承AsyncTask即可，有三个泛型参数：  
第一个是doInBackground的参数，也是excute传的参数。  
第二个是onProgressUpdate的参数。  
第三个是doInBackground的返回参数。  
第一个和第二个都是泛型参数。

四个重要方法：  
onPreExecute()和onPostExecute()是task执行前和后去执行的。一般可以是创建对话框和销毁对话框。  
doInBackground()是主要执行业务的地方。当处理一定进度以后调用publishProgress()方法，会触发onProgressUpdate(),我们重写这个方法来更新UI进度条即可。


总之，asynctask是非常轻量级的后台异步任务线程。。。