---
layout: post
title:  "关于Http的理解与学习"
date:   2012-05-13 11:00:03
categories: other
type: other
---

既是跻身于互联网之中，又是做web开发的，不得不学习了解http啊。

官方标准规范文档：RFC2616.Google一下，有中文翻译版，已下载PDF。

以前做web的时候只知道在提交form表单的时候可以选择method是get或者post，而且肤浅的理解两者区别是url上的一个参数是否隐藏。顾名思义，get就是获取数据，post就是发布数据，但是提交表单也能是get方法，而且是默认的，为什么？就算提交表单，其实也是获取结果返回数据。
post查询字符串（名称/值对）是在 POST 请求的 HTTP 消息主体中发送的。  
get查询字符串（名称/值对）是在 GET 请求的 URL 中发送的。

所以，下载是get请求，上传就是post请求。

**Get方法：**  
如果是续点下载，需要在头部加上请求数据的其实字节：  
httpGet.addHeader("Range", "bytes=" + gProgressSize + "-");

响应：  
Content-Length: //这个字段是文件的长度  
Content-Range: //这个字段的文件的字节范围  

如果文件url被重定向：  
StatusLine statusLine = response.getStatusLine();  
int statusCode = statusLine.getStatusCode();  
拿到的statusCode是302。新的url放在：response.getHeaders("Location")[0].getValue();  
一般200-300都是正常。  

关于下载，在AlmightZGBox-android里面的ZGDownloadManager可以直接使用。

一个Get的请求实例：  
{% highlight html %}
GET /music/abc.mp3 HTTP/1.1
Host: www.test.com
Connection: keep-alive
Accept-Encoding: identity;q=1, *;q=0
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.16 50.63 Safari/537.36
Accept: */*
Referer: http://abv.cn/music/%E5%85%89%E8%BE%89%E5%B2%81%E6%9C%88.mp3
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Range: bytes=125468-
{% endhighlight %}

响应头部字段：  
{% highlight html %}
HTTP/1.1 206 Partial Content
Date: Tue, 25 Feb 2014 06:05:26 GMT
Server: Apache/2.4.3 (Unix) OpenSSL/1.0.1c PHP/5.4.7
Last-Modified: Sun, 23 Feb 2014 12:53:00 GMT
ETag: "49aa24-4f31255b92300"
Accept-Ranges: bytes
Content-Length: 4827684
Content-Range: bytes 125468-4827684
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: audio/mpeg
{% endhighlight %}

**Post方法：**  
