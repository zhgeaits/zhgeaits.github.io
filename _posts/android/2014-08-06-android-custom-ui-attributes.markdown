---
layout: post
title:  "Android的自定义控件属性学习笔记!"
date:   2014-08-05 11:00:03
categories: android
type: android
---

为控件注册属性，自定义控件属性：
在res/values下创建attrs.xml文件，里面定义好属性名称和类型，然后在布局文件里面引用新的命名空间就可以使用。
然后在自定义的空间里面使用TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.MyView);
有了a的引用就可获取自定义的属性值。
