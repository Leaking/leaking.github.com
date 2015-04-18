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


# 4,  字典的方法

字典方法很多，不需要记住，大概知道有哪些即可，需要用时，可以查看文档。

+ clear方法

属于原地操作，清空字典。

+ copy方法和deepcopy函数

copy是方法是“浅复制”，如果副本的某个值被修改，则原始字典不受影响，如果副本的某个值被原地修改（比如一个列表添加一个元素），则原始字典一起变化。

deepcopy函数时“深复制”，副本无论做什么改变都不会影响原始字典。


看如下例子，对于“浅复制”，不是原地操作的修改会不会影响原始字典


{% highlight python %}

>>> a = {}
>>> a['name'] = 'quinn'
>>> a['girls'] = [1,2,3,4]
>>> a
{'girls': [1, 2, 3, 4], 'name': 'quinn'}
>>> b = a.copy();
>>> b['name'] = 'chen'
>>> b['girls'] = [1,2,3,4,5]
>>> b
{'girls': [1, 2, 3, 4, 5], 'name': 'chen'}
>>> a
{'girls': [1, 2, 3, 4], 'name': 'quinn'}


{% endhighlight %}

对于“浅赋值”，如果进行“原地操作”，则会影响原始字典

{% highlight python %}
>>> b = a.copy()
>>> b['girls'].append(5)
>>> a
{'girls': [1, 2, 3, 4, 5], 'name': 'quinn'}
>>> b
{'girls': [1, 2, 3, 4, 5], 'name': 'quinn'}
>>> 

{% endhighlight %}


“深复制”就容易理解了，无论对副本进行什么操作，都不会影响原始字典。但是注意，deepcopy是函数，不是方法，使用语法如下


{% highlight python %}
b = deepcopy(a)
{% endhighlight %}


