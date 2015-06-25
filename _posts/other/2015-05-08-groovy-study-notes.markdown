---
layout: post
title:  "Groovy语言学习笔记"
date:   2015-05-08 11:00:03
categories: other
type: other
---

## 官网  

> [http://www.groovy-lang.org/](http://www.groovy-lang.org/ "官网")

官网上说groovy是一个多面性（multi-faceted）的java平台语言，它可以像jdk一样安装，也可以当做一个项目在maven那里进行依赖。对于本质还需要再认真理解一下。

我觉得Groovy就是一个脚本版的java，在哪里用到，我目前了解的就是gradle，还有以前在TM实习的时候，那里的推荐算法会用到。Groovy引入了闭包和元编程等出色功能，也是比较难理解的。

## 配置环境：  

**通过eclipse4.4安装plugin**

> http://dist.springsource.org/release/GRECLIPSE/e4.4/

**本地安装**

或者像maven一样下载bin包，然后配置环境变量就可以了。然后创建一个Test.groovy文件，直接用groovy命令就能运行了。

## 与Java的不同  
它会默认import一下这些包，所以可以直接使用这些包里面的类了：  

> java.io.*  
java.lang.*  
java.math.BigDecimal  
java.math.BigInteger  
java.net.*  
java.util.*  
groovy.lang.*  
groovy.util.*  

它还是多方法的（方法名相同），因为它是脚本语言的特性，方法在runtime的时候call，会根据实际的参数类型去调用不同的方法。  
不能使用{}大括号来声明数组了，因为那是闭包，所以使用中括号[]。  

## 语法：

### 注释  
注释和java一样，只不过多了一个Shebang line的注释，其实和shell的第一行一样。  

{% highlight groovy %}
#!/usr/bin/env groovy
{% endhighlight %}

### 关键字  
出了java的那些关键字，还包括const def goto  

### 标识符

正常的标识符规则应该和java一样，只不过在点以后可以使用关键字作为标识符，例如：

> foo.as  
foo.assert  
foo.break  
foo.case  
foo.catch

然而groovy还有引号标识符(Quoted identifiers)，这个就和php啊，js一样，其实有点像map里面的key是字符串，我觉得本质应该一样吧：

{% highlight groovy %}
def map = [:]

map."an identifier with a space and double quotes" = "ALLOWED"
map.'with-dash-signs-and-single-quotes' = "ALLOWED"

def firstname = "Homer"
map."Simson-${firstname}" = "Homer Simson"

map.'''triple single quote'''
map."""triple double quote"""
map./slashy string/
map.$/dollar slashy string/$

{% endhighlight %}

### 字符串

三个单引号或者双引号的字符串可以是多行的，如下，String#stripIndent()方法就像java里面的trim一样效果。

定义多行变量，使用三个引号：
{% highlight groovy %}
def var="""hello
       world
       groovy!"""

def startingAndEndingWithANewline = '''
line one
line two
line three
'''
println aMultilineString
println var
{% endhighlight %}

只有使用双引号的字符串才能使用占位符插值，占位符还能够是表达式，这个是GString，例如：

{% highlight groovy %}
def name = 'Guillaume' // a plain string
def greeting = "Hello ${name}"

assert greeting.toString() == 'Hello Guillaume'

def sum = "The sum of 2 and 3 equals ${2 + 3}"
assert sum.toString() == 'The sum of 2 and 3 equals 5'
{% endhighlight %}

关于字符串在插值的时候会有闭包的表达式，还不懂，看了闭包以后在回头看,4.4.2。

GString作为参数传递的时候会隐式调用toString()方法。