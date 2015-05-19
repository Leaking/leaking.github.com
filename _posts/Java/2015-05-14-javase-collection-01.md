---
layout: post
title: Java容器概览
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

本文主要讲解一些Java容器类的一些共同特点，比如Iterator等等，然后稍微介绍一些其中的一些实现，比如List,Set。



# 容器框架概览


容器框架中包含的接口大致如下图所示

<figure>
  <img src="{{ site.url }}/images/JCF_interface_tree.jpg" alt="search screenshot">
  <figcaption>容器框架接口</figcaption>
</figure>

很明显，平时说的容器，其实包含两个大类，一类是继承于Collection，一类是继承自Map，这篇文章主要讲解前者：Collection。


# Collection一些共同特点
 




关于Collection接口有以下常用点

+ 遍历
+ 对整个容器进行操作
+ 数组操作

#### 遍历

容器遍历方式一般由两种

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

每调用一次next只有，才能调用remove，有且只能调用一次。部分Collections的子接口没有实现该方法。另外，for--each遍历方式中，在遍历过程中不能移除容器元素。

在遍历过程中，修改容器的唯一方式就是使用Iterator的remove方法。所以要过滤容器中的某些元素，
可以使用Iterator、如下

{% highlight java %}
static void filter(Collection<?> c) {
    for (Iterator<?> it = c.iterator(); it.hasNext(); )
        if (!cond(it.next()))
            it.remove();
}
{% endhighlight %}


那么Iterator和Foreach之间是否有什么关系呢？

容器之所以可以使用Foreach语句，其共同点以及原因是，容器实现了Iterable，而Iterable的源码是


{% highlight java %}
import java.util.Iterator;

/** Implementing this interface allows an object to be the target of
 *  the "foreach" statement.
 * @since 1.5
 */
public interface Iterable<T> {

    /**
     * Returns an iterator over a set of elements of type T.
     * 
     * @return an Iterator.
     */
    Iterator<T> iterator();
}
{% endhighlight %}

看到其中的注释了没：

> Implementing this interface allows an object to be the target of the
> "foreach" statement.

所以，其实如果我们自定义一个类，让它实现Iterable接口，实现其中的iterator方法让它返回一个Iterator，那么这个类也可以使用于foreach。

而数组之所以可以应用于foreach，其中的原因我还没研究。



#### 对整个容器进行操作

##### 填充容器

所有容器都有两个构造方法，一个是默认的构造方法，一个是带有一个Collection参数的构造方法，它可以参数Collection其中的元素填充当前容器。（注意哦，这里讲的容器都是Collection以及其子类，不包含Map）

平时填充容器的方式还可以这样：
1，使用Arrays.asList(）方法生成List，然后再调用Collection的addAll方法将刚生成的List加入。不过得注意一点，Arrays.asList()生成的List底层是一个固定大小的数组，所以不能增删元素，只可以修改元素。
2，使用Collections的addAll往一个Collection添加多个元素

以上两种方法的灵活之处是，其参数可以接受可变参数，而且这种方式执行更快（Think in java是这样推荐的）。


##### 容器常用的方法

+ containsAll — 当前容器是否包含某个容器的所有元素

+ addAll — 将某个容器的所有元素添加到当前容器

+ removeAll — 将当前容器中，出现在某个容器中的元素，都移除掉

+ retainAll — 将当前容器中，没有出现在某个容器中的元素，都移除掉，其实就是取交集

+ clear — 移除所有元素.

当前容器调用addAll, removeAll, retainAll这三个方法之后，如果当前容器发生变化，则返回true。

如果要移除容器中元素e，可以这样做

{% highlight java %}
c.removeAll(Collections.singleton(e));
{% endhighlight %}

如果是移除所有为空的元素，则可以

{% highlight java %}
c.removeAll(Collections.singleton(null));
{% endhighlight %}

以上Collections.singleton方法，可以创建一个只包含某个元素的容器。该容器类型时不能修改的Set。



##### 打印容器

数组的打印，不能直接打印数组对象，也不能打印数组对象的toString()结果，而要使用工具类Arrays的方法
Arrays.toString

而容器的打印，只需直接打印容器对象即可，因为容器对象都重写了其中的toString方法。


#### 数组操作

Collection的数组操作有两个方法

+ public Object[] toArray();
+ public <T> T[] toArray(T[] array);

前者只能返回Object类型的数组，而且数组不能强制类型转化。如下

{% highlight java %}
Object[] a = c.toArray();
{% endhighlight %}

如果某个容器的元素类型时String，而你想返回一个String类型的数组，则需要用第二种方法。

{% highlight java %}
String[] a = c.toArray(new String[c.size()]);
{% endhighlight %}


带参数的toArray方法，参数是一个数组，如果数组的长度大于或者小于当前容器，则会自动调整，所以我们可以直接传进去一个大小和容器一样的数组。





## List Interface

List接口除了实现父接口Collection的部分方法，而且包含其他方法，主要有

+ 定位（Positional access），主要有get, set, add, addAll, and remove
+ 查找，主要有indexOf 和 lastIndexOf
+ Iteration遍历：继承了Iterator
+ 区间（Range-view）： 主要有sublist

List接口有两个系统实现的容器类，ArrayList以及LinkedList，前者一般性能比较高，在特定情况下，后者性能比较好，后面再讨论。

以下讲讲上面的四种方法

首先，定位和查找的方法很容易理解和使用，此处就不多说了。

#### ListIterator

先讲讲Iterator的原理，如下图，terator其实就是每次移动一次下标，下标的概念是两个元素之前的位置。

<figure>
  <img src="{{ site.url }}/images/JCF_interface_iterator.jpg" alt="search screenshot">
  <figcaption>Iterator的原理</figcaption>
</figure>


该接口继承于Iterator

{% highlight java %}
public interface ListIterator<E> extends Iterator<E> 
{% endhighlight %}

相比Iterator，ListIterator增加了几个方法

+ hasPrevious 是否有前一个元素（和hasNext相反）
+ previous 获取前一个元素（和next相反）
+ nextIndex 接下来调用next的话，返回元素的下标值
+ previousIndex 接下来调用previous的话，返回元素的下标值
+ remove 每次调用一次previous或者next之后，只能调用一次remove
+ set 每次调用一次previous或者next之后，只能调用一次set
+ add 在当前游标之前插入元素，不同于上面的方法，可以多次调用。


#### 区间（Range-view）

subList(int fromIndex, int toIndex)方法返回一个子序列，子序列和原序列对子元素的修改，会相互影响。

举个例子，以下方法可以删除一个List的若干个元素，并将删除的元素组成的序列返回。

{% highlight java %}
public static <E> List<E> dealHand(List<E> deck, int n) {
    int deckSize = deck.size();
    List<E> handView = deck.subList(deckSize - n, deckSize);
    List<E> hand = new ArrayList<E>(handView);
    handView.clear();
    return hand;
}
{% endhighlight %}

tips：对于List的实现类，比如ArrayList，删除元素时，从末尾删除的效率会比从开头删除的效率高。

#### 序列算法

先强调一下，Collections是一个辅助类，而Collection才是接口。

Collections中有对List进行一系列算法操作的方法。



## List Implementations

List常见的实现类有ArrayList以及LinkedList.

ArrayList的内部实现是一个数组，LinkedList的实现时一个双向链表。前者方便取值，后者方便增删元素。更多情况下，ArrayList的性能都是比较高的。

List还有一个实现类CopyOnWriteArrayList，另外再讲讲。


## Set Interface

Set是一种元素不能重复的容器。常见实现类有三种

+ HashSet 遍历顺序是随机的 
+ TreeSet 遍历顺序按照元素大小
+ LinkedHashSet 遍历顺序按照插入元素的次序

举个实用的小例子，如果你想复制一个容器，并去除其中的重复元素，可以这样

{% highlight java %}
Collection<Type> noDups = new HashSet<Type>(c);
{% endhighlight %}

如果除了要去除重复元素，还要排序，可以这样

{% highlight java %}
Collection<Type> noDups = new LinkedHashSet<Type>(c);
{% endhighlight %}


对Set进行取交集，并集等操作时，如果为了避免影响原先的Set，需要先复制一份Set，如下

{% highlight java %}
Set<Type> union = new HashSet<Type>(s1);
union.addAll(s2);

Set<Type> intersection = new HashSet<Type>(s1);
intersection.retainAll(s2);

Set<Type> difference = new HashSet<Type>(s1);
difference.removeAll(s2);
{% endhighlight %}


先写到这里吧。

参考自：
1, [http://docs.oracle.com/javase/tutorial/collections/](http://docs.oracle.com/javase/tutorial/collections/)

2, 《Think in Java》






