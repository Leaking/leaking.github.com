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

## Paths

path是指你需要到某个主机的哪个位置获取资源，

Each path segment can have its own params component.

## Parameters