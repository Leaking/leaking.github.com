---
layout: post
title: Java集合框架01
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
categories: Java
tags: [Java]
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

# 集合框架概览

集合框架主要包含三个部分
 
+ 接口
+ 实现
+ 算法，算法指常用的排序，查找等


# 接口

集合框架中包含的接口大致如下图所示

<figure>
  <img src="{{ site.url }}/images/JCF_interface_tree.jpg" alt="search screenshot">
  <figcaption>集合框架接口</figcaption>
</figure>



下面介绍以上各个接口

## Collections
 
关于Collections接口有以下两个常用点

+ 遍历
+ 对整个集合进行操作
+ 数组操作

#### 遍历

遍历方式一般由两种

+ for-each
+ iterator

1, for-each

比如：

{% highlight java %}

for (Object o : collection)
    System.out.println(o);

{% endhighlight %}

2, Iterator

先看看Iterator接口的源码

{% highlight java %}
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove(); //optional
}
{% endhighlight %}


hasNext以及next两个方法比较容易理解，需要注意的是remove方法的使用。

每调用一次next只有，才能调用remove，有且只能调用一次。部分Collections的子接口没有实现该方法。另外，for--each遍历方式中，在遍历过程中不能移除集合元素。

在遍历过程中，修改集合的唯一方式就是使用Iterator的remove方法。所以要过滤集合中的某些元素，
可以使用Iterator、如下

{% highlight java %}
static void filter(Collection<?> c) {
    for (Iterator<?> it = c.iterator(); it.hasNext(); )
        if (!cond(it.next()))
            it.remove();
}
{% endhighlight %}


#### 对整个集合进行操作

常见的方法有如下

+ containsAll — 当前集合是否包含某个集合的所有元素

+ addAll — 将某个集合的所有元素添加到当前集合

+ removeAll — 将当前集合中，出现在某个集合中的元素，都移除掉

+ retainAll — 将当前集合中，没有出现在某个集合中的元素，都移除掉，其实就是取交集

+ clear — 移除所有元素.

当前集合调用addAll, removeAll, retainAll这三个方法之后，如果当前集合发生变化，则返回true。

如果要移除集合中元素e，可以这样做

{% highlight java %}
c.removeAll(Collections.singleton(e));
{% endhighlight %}

如果是移除所有为空的元素，则可以

{% highlight java %}
c.removeAll(Collections.singleton(null));
{% endhighlight %}

以上Collections.singleton方法，可以创建一个只包含某个元素的集合。该集合类型时不能修改的Set。

#### 数组操作

Collection的数组操作有两个方法

+ public Object[] toArray();
+ public <T> T[] toArray(T[] array);

前者只能返回Object类型的数组，而且数组不能强制类型转化。如下

{% highlight java %}
Object[] a = c.toArray();
{% endhighlight %}

如果某个集合的元素类型时String，而你想返回一个String类型的数组，则需要用第二种方法。

{% highlight java %}
String[] a = c.toArray(new String[c.size()]);
{% endhighlight %}


带参数的toArray方法，参数是一个数组，如果数组的长度大于或者小于当前集合，则会自动调整，所以我们可以直接传进去一个大小和集合一样的数组。