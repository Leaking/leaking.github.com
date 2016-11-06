---
layout: post
title: 【Python笔记】列表和元祖
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
categories: python
tags: [sample-post]
comments: true
share: true
---

# 1,大致介绍

Python中有一种数据结构叫容器，容器主要有三种，序列，字典，集合。这里的字典就是其他语言中常见的键值对。

序列（sequence）。一共有6种序列，其中的两种是列表和元组（list and tuple）。

列表可以被修改，元组则不能。



# 2，序列的常用操作


序列可以包含普通类型的数据，也可以包含序列。

序列常用的操作有，索引，分片，加，乘。长度，最大值，最小值。


### 索引

索引用于访问序列单个元素，序列从左到右的索引从0开始递增，从右到左从-1开始递减

字符串是一种由字符组成的序列，

关于序列的索引，字符串可以直接在字面值之后，或者在一个函数的返回结果之后使用索引，如下

{% highlight python %}
>>> 'Quinn'[1]
'u'
>>> fourth = raw_input('Year: ')[3]
Year: 2015
>>> fourth
'5'
{% endhighlight %}



### 分片

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



### 序列相加

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


### 序列乘法

乘法主要用于重复元素，如下

{% highlight python %}
>>> 'Quinn' * 5
'QuinnQuinnQuinnQuinnQuinn'
>>> [42] * 5
[42, 42, 42, 42, 42]
>>> [None] * 5
[None, None, None, None, None]
{% endhighlight %}


### 成员资格

为了检查一个元素是否存在于某个序列中，可以用in运算符，如下

{% highlight python %}
>>> name = 'quinn chen'
>>> 'q' in name
True
>>> 'x' in name
False
>>> 'qu' in name
True
{% endhighlight %}

如上，字符串比较特别，自从2.3版本开始，Python开始支持检查多余1个元素的字符串检查成员资格，比如上面的最后一个例子。

### 长度，最大值，最小值

{% highlight python %}
>>> len(name)
10
{% endhighlight %}

同理，和len类似的内建函数还有max，和min，相信大家一下子就知道这两个函数的作用。

# 3，列表

列表不同于元祖和字符串，列表能改变内容，而元祖和字符串不能。

这一节讲两部分，首先是list方法，元素赋值，删除元素，分片赋值方法，然后是列表的方法（列表对象的方法）。

### list方法

如果想将一个字符串转化为列表，可以这样子做


{% highlight python %}
>>> name = 'quinn chen'
>>> name
'quinn chen'
>>> list(name)
['q', 'u', 'i', 'n', 'n', ' ', 'c', 'h', 'e', 'n']
{% endhighlight %}

### 元素赋值

{% highlight python %}
>>> namelist = list(name)
>>> namelist
['q', 'u', 'i', 'n', 'n', ' ', 'c', 'h', 'e', 'n']
>>> namelist[2] = 'm'
>>> namelist
['q', 'u', 'm', 'n', 'n', ' ', 'c', 'h', 'e', 'n']
{% endhighlight %}

注意，此处元素赋值是对列表进行，而不能直接对字符串进行该操作（因为字符串不能被修改）。

### 删除元素

{% highlight python %}
>>> del namelist[0]
>>> namelsit
['u', 'm', 'n', 'n', ' ', 'c', 'h', 'e', 'n']
{% endhighlight %}

del只能对列表进行操作，不能读字符串进行操作（因为字符串不能被修改）。



### 分片赋值

分片赋值可以用长度相同的分片替代，甚至是长度不同的分片，甚至是长度为0的分片，这样就是删除的效果了，也可以是用长度不为0的分片，替代某个长度为0的分片，这样相当于插入。看如下代码



{% highlight python %}
>>> namelist
['u', 'm', 'n', 'n', ' ', 'c', 'h', 'e', 'n']
>>> name = list('quinn')
>>> name
['q', 'u', 'i', 'n', 'n']
>>> name[2:] = 'aa'
>>> name
['q', 'u', 'a', 'a']
>>> name[2:4] = []
>>> name
['q', 'u']
>>> name[1:1] = 'man'
>>> name
['q', 'm', 'a', 'n', 'u']
>>> name[::2] = 'ttt'
>>> name
['t', 'm', 't', 'n', 't']
{% endhighlight %}

注意，带步长的分片，赋值时，分片长度要对应，不能大于或者小于


### 列表方法

方法不同于函数，方法的调用方式一般如下：

对象.方法（参数）

在这里大概介绍几个常用方法，使用这些方法时遇到问题可以常看文档。

+ append

该方法是在列表自身添加一个元素，只能添加一个元素，即使添加一个列表，这个列表将作为一个元素添加在末尾


{% highlight python %}

>>> lst = [1,2,3]
>>> lstend = [4,5]
>>> lst.append(lstend)
>>> lst
[1, 2, 3, [4, 5]]
>>> lst.append(9)
>>> lst
[1, 2, 3, [4, 5], 9]

{% endhighlight %}

+ count

计算某个元素在列表中出现的次数

+ extend

在列表末尾一次性追加另一个序列的多个值。

该方法作用于前面说到的相加类似（用加号）。二者区别在于后者返回一个全新列表，而前者不是。

extend可以添加多个元素，而append只能添加一个元素，

看如下例子

{% highlight python %}

>>> c = [1,2,3]
>>> d = [4,5,6]
>>> c+d
[1, 2, 3, 4, 5, 6]
>>> c
[1, 2, 3]
>>> c.extend(d)
>>> c
[1, 2, 3, 4, 5, 6]
>>> c.append(d)
>>> c
[1, 2, 3, 4, 5, 6, [4, 5, 6]]

{% endhighlight %}

+ index

该方法返回某个元素在列表中第一次出现的位置，不存在则抛出异常。

+ insert

该方法用于在某个位置插入一个新元素，该元素可以是序列或者普通值


{% highlight python %}

>>> c.insert(2,d)
>>> c
[1, 2, [4, 5, 6], 3, 4, 5, 6, [4, 5, 6]]

{% endhighlight %}


+ pop

弹出某个位置（默认是最后一个）的元素，并返回

{% highlight python %}
>>> c
[1, 2, [4, 5, 6], 3, 4, 5, 6, [4, 5, 6]]
>>> c.pop(4)
4
>>> c
[1, 2, [4, 5, 6], 3, 5, 6, [4, 5, 6]]
>>> c.pop(2)
[4, 5, 6]

{% endhighlight %}

+ remove

移除列表中某个值的第一个匹配项，不存在匹配项则抛出异常，该方法不同于pop，不会返回移除的元素

+ reverse

改变列表自身（没有返回值），颠倒顺序

+ sort

改变列表自身（没有返回值），进行排序


{% highlight python %}
>>> a = [4,3,2,7,8,6]
>>> a
[4, 3, 2, 7, 8, 6]
>>> b = a.sort()
>>> b   //None
>>> a
[2, 3, 4, 6, 7, 8]
{% endhighlight %}


如果想对某个序列排序，然后保留一个排序前和排序后的副本，可以这样做

{% highlight python %}
>>> a = [4,3,6,8,1,9]
>>> b = a[:]
>>> b.sort()
>>> b
[1, 3, 4, 6, 8, 9]
>>> a
[4, 3, 6, 8, 1, 9]

{% endhighlight %}

上面创造副本使用了分片，如果仅仅使用b = a。则两个引用指向同个列表，而不是创建一个新的副本。




# 4，元组

元组和列表一样，都是序列，但是元组不能修改，列表可以修改。元组用圆括号表示，列表用中括号表示。

如果只是用逗号隔开几个值，也会自动创建元组，如下


{% highlight python %}

>>> x = 1,2,3
>>> x
(1, 2, 3)
>>> 1,2,3
(1, 2, 3)
>>> (4,5,6)
(4, 5, 6)

{% endhighlight %}

要创建只有一个值的元组，需要添加一个逗号

{% highlight python %}
>>> (42)
42
>>> (42,)
(42,)
{% endhighlight %}

再看一个结合乘法的例子

{% highlight python %}

>>> 4*(20+4,)
(24, 24, 24, 24)

{% endhighlight %}

### tuple函数

该函数与list函数一样，以一个序列为参数，把它转化为元组


{% highlight python %}
>>> tuple([2,3,4])
(2, 3, 4)
>>> tuple((2,3,4))
(2, 3, 4)
>>> tuple('234')
('2', '3', '4')
>>> tuple(2,3,4)

Traceback (most recent call last):
  File "<pyshell#7>", line 1, in <module>
    tuple(2,3,4)
TypeError: tuple() takes at most 1 argument (3 given)
{% endhighlight %}

注意，只能接收一个值，不能接收多个值。

### 元组的操作

元组由于是不能修改的序列，所以常见操作只有创建元组，访问元组，分片，这些都和列表一样。



