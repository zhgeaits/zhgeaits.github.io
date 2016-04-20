---
layout: post
title:  "Android的View，ViewGroup，Window，WindowManager学习笔记!"
date:   2014-02-05 11:00:03
categories: android
supertype: career
type: android
---

学习android如果纯粹是学会了简单的开发，是做不出来令人惊讶的界面。不理解这些重要的内部结构，就会在写UI的时候蛋疼无比。



**View和ViewGroup**  



**dialog的edittext无法弹出键盘问题**  
//只用下面这一行弹出对话框时需要点击输入框才能弹出软键盘  
window.clearFlags(WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM);  
//加上下面这一行弹出对话框时软键盘随之弹出  
window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);  

暂时就只学习到这里，日后继续补充。

**PopWindow**  
日后再学习