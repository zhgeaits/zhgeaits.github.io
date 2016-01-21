---
layout: post
title:  "关于Http&DNS的理解与学习"
date:   2012-05-13 11:00:03
categories: other
type: basic
---

既是跻身于互联网之中，又是做web开发的，不得不学习了解http啊。

官方标准规范文档：RFC2616.Google一下，有中文翻译版，已下载PDF。  
查看更多的Content-Type类型，参考RFC1341。更详细的上传表单，参考RFC1867

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
用chrome看到的一个post的实例header：  
{% highlight html %}
POST /Test/upload HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 241
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Origin: http://localhost:8080
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarymfU8VQGnQMKVkr77
Referer: http://localhost:8080/Test/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4
Cookie: JSESSIONID=641280E9BC95D0FF5E06B087F24279B8; CNZZDATA5420382=cnzz_eid%3D177016966-1427339623-%26ntime%3D1431598864
//上面是header，下面开始是Request Payload
------WebKitFormBoundarymfU8VQGnQMKVkr77
Content-Disposition: form-data; name="abc"

asdfzxcv
------WebKitFormBoundarymfU8VQGnQMKVkr77
Content-Disposition: form-data; name="dce"

asdfzxcfv
------WebKitFormBoundarymfU8VQGnQMKVkr77--
{% endhighlight %}

如果上传文件的话，只是Request Payload有区别，如下上传一个pdf：  
{% highlight html %}
------WebKitFormBoundaryG1cZNiwsHjo8vWnE
Content-Disposition: form-data; name="uploadfile"; filename="HTTP??RFC2616???.pdf"
Content-Type: application/pdf


------WebKitFormBoundaryG1cZNiwsHjo8vWnE--
{% endhighlight %}

不明白为什么不是网上所说的boundary有27个-，但是一样的是，header上的boundary和data是-是相差两个。

## DNS

我们都知道DNS就是域名系统(Domain Name System)，提供把域名转换成IP地址的目录服务，但是我们却不知道它的原理和更多机制。其实它是一个应用层的协议，不同的是它不是直接给用户用的，而是给http,ftp,smtp这些协议使用的。

>DNS是一个由分层的DNS服务器实现的**分布式数据库**；是一个允许主机查询分布式数据库的应用层协议；DNS协议运行在UDP之上，使用53号端口。在RFC1034和1035有详细定义。

除了是分布式数据库，还有重要的一点，DNS是通过client/server模式提供服务的。例如：

>浏览器从url中抽取host:zhgeaits.me，然后传给dns应用的client端，通常调用的方法是`getHostByName()`，dns的client就向server发起一个请求，最后会收到一个报文，包含了ip地址，浏览器最后会向这个ip地址的server建立一个tcp的连接。

**DNS的工作机理**

DNS的层次结构：

![alt dns_structure](/image/dns_structure.png "dns_structure") 

第一层是Root服务器，第二层是Top Level Domain服务器，第三层是权威服务器。根服务器全球只有13个，编号是A-M，大部分在美国，中国好像有一个。顶级服务器是负责com,org,net,edu,cn,gov等等这些域名的。权威服务器比较多，一般我们是付费在权威服务器上面买域名（资源记录）的。

除了那三种服务器以后，还有一种称为**本地DNS服务器**，他是由大学，公司，或者组织机构等ISP所提供的，可以理解为代理的DNS服务器，因为我们一般就是配置这些DNS的，发请求都是发到它这里来的。例如拨号电信上网，电信会给我们一个本地DNS服务器地址；还有google的DNS服务器`8.8.8.8`也是属于本地DNS服务器，因为它不是权威的，没有提供域名购买相关服务，只是代理转换的作用。其实在本地服务器也是可以买记录的，没有严格意义上的区分。

正常来说，如果没有本地DNS服务器，那么我们的主机要被访问到，就必须到权威服务器上面去买一条记录，也就是说每一个主机都有自己的权威服务器，世界那么多主机，这样明显不科学的，顶级服务器要记录那么多的权威服务器！然后就有一个不是很权威的本地服务器，它在权威服务器上面有一条记录，然后我们也就可以在本地服务器上面买一条记录了。其实，权威服务器上面买一条记录和在本地服务器上面买一条记录价格差别是很大的。

>可以这样区分本地DNS服务区和权威DNS服务器，告诉用户主机IP地址的是本地DNS服务器，因为它一般距离用户就几个路由，而告诉本地DNS服务器主机IP地址的就是权威服务器。本地服务器一般提供代理查询的服务，而权威服务器提供域名购买的服务。当然本地服务器的缓存也是提供的IP转换，就像权威服务器一样的功能，但是毕竟不是权威的嘛，例如我也可以配置本地的host啊。

DNS的解析流程：

![alt dns_flow](/image/dns_flow.png "dns_flow")

首先我们先本地服务器发起查询请求，本地服务器就去根服务器查询请求，根服务器告诉TLD服务器的地址，我们再去TLD服务器查询，TLD告诉权威服务器的地址，权威服务器最后告诉IP地址是什么。然而，上面说了，其实TLD服务器并不知道所有的权威服务器，TLD只知道中间的权威服务器，中间的权威服务器才知道最后的不是那么权威的权威服务器，所以，一般不是8步，而是10步。

有了DNS缓存就不一样了，不会每次都走上面的流程，上面的每一步都可以缓存，而且，这个是分布式数据库啊，数据同步极其的快速。然后本地的电脑也是可以配置host的嘛。

DNS的记录：

`(Name, Value, Type, TTL)`

TTL就是生命周期，就是缓存时间多久。Name和Value就是一对名字和值了。而Type有四种类型，分别是：

Type=A，其实就Address嘛，存放的就是域名对应IP地址了。 

Type=NS，其实是NameSpace，存放的是权威DNS服务器的域名。

Type=CNAME,其实就是CanonicalName，存放的是规范主机名，例如，其实百度的域名有很多别名的，但是只有一个规范的主机名而已。

Type=MX，存放的是邮件服务器的规范主机名。

基本上理解这些就可以了，更深入的暂时还没有去学习到，以后需要再去学习！

**其他**

一般的url，把host换成ip来请求是没问题的，但是对于如果有反向代理的服务器，那么单纯把域名替换成ip就有问题了，丢失域名无法完成请求的。ip只是包里面的两个字段源IP地址和目的IP地址的值，对于应用层协议，如http，域名host这个字段还是要保留的带过去的。所以在使用apache的httpclient网络包的时候就要注意了，不能直接把url里面的host替换成ip，还需要保留host。  
可以这样做，设置一个host（虚拟主机）过去：
> HttpRequest.getParams().put(ClientPNames.VIRTUAL_HOST, new HttpHost("www.zhangge.com"));  
HttpRequest就是HttpGet，HttpPost等的接口。
  
这是我看源码看出来的，如果对于不同的情况和httpclient版本，还是需要自己去看源码处理实际情况。不过原理就这样，理解就好了。  
对于自己解析域名，httpclient还提供了别的接口，自定义`BasicClientConnectionManager`和`DnsResolver`类，然后传给`DefaultHttpClient`就可以了。