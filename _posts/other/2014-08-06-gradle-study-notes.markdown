---
layout: post
title:  "Gradle学习笔记"
date:   2014-08-05 11:00:03
categories: other
type: other
---

这个东西和maven相似，可以对比来学习。

和Maven一样，Gradle只是提供了构建项目的一个框架，真正起作用的是Plugin。Gradle在默认情况下为我们提供了许多常用的Plugin，其中包括有构建Java项目的Plugin，还有War，Ear等。与Maven不同的是，Gradle不提供内建的项目生命周期管理，只是java Plugin向Project中添加了许多Task，这些Task依次执行，为我们营造了一种如同Maven般项目构建周期。