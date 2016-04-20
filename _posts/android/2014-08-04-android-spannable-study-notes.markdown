---
layout: post
title:  "Android的Spannable, SpannableString，SpannableStringBuilder，ImageSpan学习笔记!"
date:   2014-08-04 11:00:03
categories: android
supertype: career
type: android
---

在聊天的应用里面这个非常有用，输入框显示图片，聊天内容显示表情，图片，语音等等。  

SpannableString实现了CharSequence，Spannable接口   
SpannableStringBuilder实现CharSequence，Spannable，Editable接口  
Editable继承CharSequence，Spannable接口  
ImageSpan继承DynamicDrawableSpan，这个类给Spannable没有了联系。

EditText里面放的一般是SpannableStringBuilder。  
new 一个ImageSpan，然后把图片或者表情传进去。  
然后拿到EditText里面的Editable对象后调用spannable.setSpan(what, start, end, flag);  
what传的就是ImageSpan对象，start, end就是替换edittext里面的字符串。  
flag参数有下面这些，它是用来标识在 Span 范围内的文本前后输入新的字符时是否把它们也应用这个效果：  
Spanned.SPAN_EXCLUSIVE_EXCLUSIVE(前后都不包括)、  
Spanned.SPAN_INCLUSIVE_EXCLUSIVE(前面包括，后面不包括)、  
Spanned.SPAN_EXCLUSIVE_INCLUSIVE(前面不包括，后面包括)、  
Spanned.SPAN_INCLUSIVE_INCLUSIVE(前后都包括)  

除了ImageSpan外，还有StyleSpan，URLSpan等等很多。

TextView一样的做法。

给TextView高亮关键字：  
SpannableString span = new SpannableString(title);  
span.setSpan(new ForegroundColorSpan(getResources().getColor(R.color.top_bar_line_color)), start, end, Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);  
title.setText(span);  
如此的简单！~之前公司的项目，一个搜索功能，以前的人写的时候，在搜索结果页要把关键字高亮，然他就用String.replaceAll()方法来把关键字替换html的<span>标签。然后没有处理到一些情况，如果用户输入正则表达式就会崩溃。因为这个方法就是用正则表达式来替换的。我开始用Matcher.quoteReplacement(searchKey)，但是会抛OOM错，由于转移递归了。后面处理方法就是遍历字符串，用spannable高亮了。

以后慢慢学习补充。。