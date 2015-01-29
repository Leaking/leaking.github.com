---
layout: post
title: 《HTTP权威指南》Note2-URLs and Resources
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
categories: readingnotes
tags: [readingnotes]
image:
  feature: http_definitive_guide.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

URLs(Uniform resource locators)是互联网资源的标准化名称

## Navigating the Internet’s Resources

像上一篇博客[“《HTTP权威指南》Note1-Overview of HTTP”](http://leaking.github.io/readingnotes/HTTP%20definition01/)中提到的，URLs只是URI的一个子集，另一个子集是URN，URL强调资源的位置，URN抢到资源的名称，当某个资源在互联网的位置发生改变，旧的URL将不能再访问该资源，而URN则依然可以。由于URN还未普及，我们一般默认URL就是URI。

URL有统一的格式：scheme://server location/path


URL被应用于除了HTTP之外的其他协议，比如FTP，电子邮件等等。URL是众多网络协议的访问点（原文是access point，翻译有点怪）。

## URL Syntax

URL的语法格式（详细）如下


{% highlight html %}
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
{% endhighlight %}

一般没有哪个URL同时拥有以上9个部分，一般用于scheme，host，path.
接下来讲解以详细语法格式中的9个组成部分。

### Schemes: What Protocol to Use

scheme一般表示访问某个资源是通过哪种协议。它不区分大小写，用冒号:隔开后面的字段，所以
HTTP://等同于http://

### Hosts and Ports

host和port指定你得通过互联网中哪台主机的哪个端口获取某个资源。


### Usernames and Passwords

用户名和密码，一般运用于FTP协议，举几个例子，如下：


{% highlight html %}
ftp://ftp.prep.ai.mit.edu/pub/gnu
ftp://anonymous@ftp.prep.ai.mit.edu/pub/gnu
ftp://anonymous:my_passwd@ftp.prep.ai.mit.edu/pub/gnu
http://joe:joespasswd@www.joes-hardware.com/sales_info.txt
{% endhighlight %}


第一个URL中，没有账户名也没有密码，则默认插入“anonymous”作为账号，而密码，也会插入默认值，不同浏览器的默认密码不同，IE是“IEUser”，Netscape Navigator 是 “mozilla”。
第二个URL中，有账号，则只需要自动插入默认密码。
第三个和第四个URL中，拥有账号也有密码。

通常账号密码通过“@”字符与后续的URL组成部分分隔开，而账号密码之间则用冒号":"分开。

### Paths

path是指你需要到某个主机的哪个位置获取资源，

path中的每一级，都可以有自己的参数（Each path segment can have its own params component）

## Parameters

还记得参数Parameters放在哪个位置吗？再看看URL的完整格式，它与path之间用分号“;”隔开。

{% highlight html %}
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
{% endhighlight %}

参数都以键值对的方式出现，如下例子

{% highlight html %}
ftp://prep.ai.mit.edu/pub/gnu;type=d
{% endhighlight %}

向前面提到的，path的各个级别都可以有自己的参数，如下


{% highlight html %}
http://www.joes-hardware.com/hammers;sale=false/index.html;graphics=true
{% endhighlight %}

路径在hammers级别有参数sale，值为false，路径在index.html有参数graphics，其值为true


### Query Strings

Query Strings的例子如下，不大理解，这个和get方法后面的参数不是一样吗

{% highlight html %}
http://www.joes-hardware.com/inventory-check.cgi?item=12731
{% endhighlight %}


### Fragments

这个概念是指从服务器获取某个对象，显示在浏览器之后，直接滚动显示到某个片段。

注意，该语法不会使服务器只返回某个对象的片段，服务器仍然返回整个对象，只是浏览器默认滚动到某个片段的位置。

如下语句

{% highlight html %}
http://www.joes-hardware.com/tools.html#drills
{% endhighlight %}

该网页www.joes-hardware.com/tools.html的源代码为：


{% highlight html %}
<HTML>

<HEAD><TITLE>Joe's Tools</TITLE></HEAD>

<BODY>

<H1>Tools Page</H1>

<H2>Hammers</H2>

<P>Joe's Hardware Online has the largest selection of 
<A HREF="./hammers.html">hammers</A> on the earth.</P>

<H2><A NAME=drills></A>Drills</H2>

<P>Joe's Hardware has a complete line of cordless and corded drills,
as well as the latest in plutonium-powered atomic drills, for those
big around the house jobs.</P> ...

</BODY>

</HTML>

{% endhighlight %}


