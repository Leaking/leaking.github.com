---
layout: post
title: 《HTTP权威指南》Chapter11-Client Identification and Cookies
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

Web服务器可能同时和很多人在通信，服务器需要知道它正在和谁通信，而不是每个客户端都当做一个匿名的用户，这一章主要就是为了讲解服务器如何辨别它正在和谁通信。


#The Personal Touch

HTTP本来就是一种匿名的，无状态，请求响应形式的协议。服务器难以知道在和哪个客户端通信，而且无法跟踪同个用户先后发出的请求。

现代化的浏览器为了给用户提供个性化的体验，它们实现了跟踪用户先后发出的请求的功能。比如亚马逊就为用户提供了以下个性化的功能：每个用户的称呼和网页内容都不同；根据用户的兴趣推荐商品；自动填充用户个人信息，比如地址电话邮箱等等。

这一章我们将总结几种HTTP识别用户的方法。

+ 使用客户端IP识别客户端
+ 使用用户认证的方式识别用户，比如登陆
+ Fat URLs，在URL中加入识别信息
+ Cookie，能持久保存的识别标志



#HTTP Headers

下面表格中的七个http头部属性，经常用于携带用户信息，我们先介绍前面三个，后面四个在下一小节再仔细介绍

<figure>
  <img src="{{ site.url }}/images/chapter11-image1.jpg" alt="search screenshot">
  <figcaption> HTTP headers carry clues about users</figcaption>
</figure>


"From"一般包含用户电子邮箱，这可以用于鉴别用户身份。为了预防用户收到垃圾邮件，很少浏览器会在头部发出“From”。


“User-Agent”一般是告诉服务器浏览器的版本，以及操作系统的信息。这能使服务器有针对性地返回内容，有利于客户端渲染效果。但是这起不到识别用户身份的作用。以下是Netscape Navigator和IE的“User-Agent”

Navigator 6.2
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.0; en-US; rv:0.9.4) Gecko/20011128
  Netscape6/6.2.1
Internet Explorer 6.01
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)


“Referer”包含用户从哪个页面跳转过来的。它一般不能用户识别用户身份，但是能记录用户个人偏好。

从上面的讨论可知， From, User-Agent, 和 Referer headers都无法用于识别用户身份，下面讨论能更精确识别用户身份的方式。



#Client IP Address

