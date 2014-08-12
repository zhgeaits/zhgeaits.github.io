---
layout: post
title:  "这个Blog是如何搭建的!"
date:   2014-08-05 11:00:03
categories: other
type: other
---

ruby dk.rb init
ruby dk.rb install


$gem sources -r http://rubygems.org/
$gem sources -a http://ruby.taobao.org/


NoMethodError: undefined method `size' for nil:NilClass
An error occurred while installing pygments.rb (0.6.0), and Bundler cannot
continue.
Make sure that `gem install pygments.rb -v '0.6.0'` succeeds before bundling.

在此只需要将gem目录下cache文件夹内清空，重新安装即可。

如果不知道gem目录的可以使用gem env来查看GEM PATHS值，如下所示d:/Ruby200/lib/ruby/gems/2.0.0
