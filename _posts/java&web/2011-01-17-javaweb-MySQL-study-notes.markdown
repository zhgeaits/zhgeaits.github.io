---
layout: post
title: "MySQL学习记录"
date: 2011-01-17 01:15:24
categories: javaweb
type: java&web
---

问题:

1.问题一：设计数据库表的时候，有多个timestamp属性字段，导入时候，报这个错：  
ERROR 1293 (HY000): Incorrect table definition; there can be only one TIMESTAMP column with CURRENT_TIMESTAMP in DEFAULT or ON UPDATE clause  
原因是：如果有一个字段是timestamp default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,则必须是第一出现的，后面出现的timestamp字段不用显示设置默认值。否则必须显示设置默认值，而且不能多个是default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP，只允许一个。