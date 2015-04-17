---
layout: post
title: 【Python笔记】字典
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
categories: python
tags: [sample-post]
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

# 1,大致介绍

其实字典就是其他语言中的Map，一种键值对数据结构。字典的键可以为任何不可变类型，比如浮点型，字符串，元组

# 2，字典的创建

列表用[]中括号，元组用小括号()，字典用大括号{}。

{% highlight python %}

>>> dictdemo = {'age':12,'name':'quinn'}
>>> dictdemo
{'age': 12, 'name': 'quinn'}
>>> dictdemo['name']
'quinn'

{% endhighlight %}

创建字典还可以用dict函数，有以下两种方式


{% highlight python %}

>>> items = [('name','quinn'),('age',24)]
>>> items
[('name', 'quinn'), ('age', 24)]
>>> d = dict(items)
>>> d['name']
'quinn'

>>> d = dict(name = 'quu', age = 42)
>>> d
{'age': 42, 'name': 'quu'}
>>> d['age']
42


{% endhighlight %}


# 3， 字典的常用操作

+ len(d):返回d中“键值对”的个数
+ d[k]：返回k键对应的值
+ k{k]=v：修改k键对应的值，如果k键不存在，则添加一对键值对
+ del d[d]: 删除某个键值对
+ k in d:检查k键对应的键值对是否存在。

看看下面一个例子，注意序列和字典的区别



{% highlight python %}

>>> x ={}   // 字典通过大括号创建
>>> x[3] = 5
>>> x[3]
5
>>> x = []   // 列表通过中括号创建
>>> x[3] = 5

Traceback (most recent call last):
  File "<pyshell#18>", line 1, in <module>
    x[3] = 5
IndexError: list assignment index out of range

{% endhighlight %}

