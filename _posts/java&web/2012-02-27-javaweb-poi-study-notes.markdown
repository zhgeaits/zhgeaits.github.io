---
layout: post
title: "Java操作office文件的解决方案之POI框架"
date: 2012-02-27 01:15:24
categories: javaweb
type: java&web
---

## 1. 什么是POI

POI是apache的开源项目，官方网站是：http://poi.apache.org/。当需要用java语言来读取微软的office文档，并进行修改的写操作时候，poi是我们选择的方案，它是一个函数工具库。记得当时我们是做学校的一个外包项目，需要读取excel表格，并存储数据，于是学习并使用了poi。更多的学习当然自行去官方网站看文档，那里必然是最新的，而且是一手的资料，虽然是英文，但也不算难事。

office有很多，word，excel，powerpoint等等，poi也就对应有很多的组件，而我当时学习使用的正是操作excel的hssf。

## 2. POI的Excel组件

主要是先了解一下excel的结构，一个excel文件包含了多个sheet，每个sheet包含了多行，每行都有多个格子cell，而且cell是可以组合的。然后它还有Header和Footer，最重要的是每个Cell都有样式cellstyle的。好了，这里不列举例子代码，官网上文档太完善了，只记录一些理解和问题罢了，如果以后会有接触到会继续完善的。