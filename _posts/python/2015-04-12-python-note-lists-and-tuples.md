---
layout: post
title: 【Python笔记】列表和元祖
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

Python中有一种数据结构叫容器，容器主要有三种，序列，字典，集合。这里的字典就是其他语言中常见的键值对。

序列（sequence）。一共有6种序列，其中的两种是列表和元组（list and tuple）。

列表可以被修改，元组则不能。



##### 2，序列的常用操作


序列可以包含普通类型的数据，也可以包含序列。

序列常用的操作有，索引，分片，加，乘。


## 索引

索引用于访问序列单个元素，序列从左到右的索引从0开始递增，从右到左从-1开始递减

字符串是一种由字符组成的序列，

关于序列的索引，字符串可以直接在字面值之后，后者一个函数的返回结果之后使用索引，如下

{% highlight python %}
>>> 'Quinn'[1]
'u'
>>> fourth = raw_input('Year: ')[3]
Year: 2015
>>> fourth
'5'
{% endhighlight %}



## 分片

+ 索引用于访问序列单个元素，分片则是访问多个元素。

+ 两个索引的分片

分片可以由冒号隔开的两个索引实现。如下

{% highlight python %}

>>> numbers = [1,2,3,4,5,6,7,8,9]
>>> numbers[2:5]
[3, 4, 5]
>>> numbers[-3:-1]
[7, 8]

{% endhighlight %}

冒号两侧的两个索引，分别是分片第一个元素的索引，以及分片最后一个元素的索引加1。换句话说，第一个索引的元素包含在分片中，最后一个不包含在分片中。

两个索引为相等的值，则返回一个空序列。或者左侧索引的元素在右侧索引元素的后面，也返回一个空序列。

注意这种分片方式无论如何都访问不到最后一个元素，只能通过下面两种。

+ 一个索引的分片

{% highlight python %}

>>> numbers = [1,2,3,4,5,6,7,8,9]
>>> numbers[:3]
[1, 2, 3]
>>> numbers[5:]
[6, 7, 8, 9]

{% endhighlight %}

+ 没有索引的分片

获取整个序列

{% highlight python %}
>>> numbers[:]
[1, 2, 3, 4, 5, 6, 7, 8, 9]
{% endhighlight %}



+ 带步长的分片

以上说的分片都是默认步长为1，其实也可以是其他大小，步长的值放在两个索引之后，步长的使用方法如下

{% highlight python %}
>>> numbers[0:10:2]
[1, 3, 5, 7, 9]
>>> numbers[::3]
[1, 4, 7]
>>> numbers[8:3:-2]
[9, 7, 5]
>>> numbers[8:3:2]
[]
{% endhighlight %}


注意，如从右往左取切片，步长必须为负数，否则返回切片为空。



# 序列相加

注意，字符串属于序列，字符串可以和字符串相加，但是字符串不能和列表相加，如下


{% highlight python %}
>>> [1,2,3] + [4,5,6]
[1, 2, 3, 4, 5, 6]
>>> 'Quinn ' + 'Chen'
'Quinn Chen'
>>> [1,2,3] + 'Chen'

Traceback (most recent call last):
  File "<pyshell#38>", line 1, in <module>
    [1,2,3] + 'Chen'
TypeError: can only concatenate list (not "str") to list
{% endhighlight %}


# 序列乘法

乘法主要用于重复元素，如下

{% highlight python %}
>>> 'Quinn' * 5
'QuinnQuinnQuinnQuinnQuinn'
>>> [42] * 5
[42, 42, 42, 42, 42]
>>> [None] * 5
[None, None, None, None, None]
{% endhighlight %}





