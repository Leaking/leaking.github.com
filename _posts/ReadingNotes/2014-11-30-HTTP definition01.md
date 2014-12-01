---
layout: post
title: 《HTTP权威指南》Note1-Overview of HTTP
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

## HTTP: The Internet’s Multimedia Courier

首先，HTTP的全称为：Hypertext Transfer Protocol。属于可靠数据传输协议

## Web Clients and Servers

服务器存储某些web内容，客户端向服务器发送一个http request请求某个对象（可能为文档，图片等等），
若请求成功，服务器将返回一个http response，其中包含该对象，对象的内容长度，对象的类型等等信息。

最常见的HTTP客户端是浏览器

## Resources

服务器上web内容都是资源，包括文件，图片等等。网关和搜索引擎也是资源（这一句不大懂）
关于资源，主要有两个知识点，一个是媒体类型，一个是URI，如下

### Media Types

服务器传输给客户端的对象可能是文本文件，图片等各种类型，HTTP协议通过MINE type来标记各种数据类型，客户端根据传输对象的数据类型，选择接收、显示该对象的方式。

如下图
<figure>
  <img src="{{ site.url }}/images/http_overview01.jpg" alt="search screenshot">
  <figcaption>Mine type其实就是平时见到的Content-type</figcaption>
</figure>

Mine type的文本标记方式为 “类型/子类型”，如下几种常见情况

> *  html文本类型：text/html
> *  ASCLL文本：text/plain
> *  jpeg图片：image/jpeg
> *  gif图片：image/gif


### URIs

URI:uniform resource identifier,URI标记着所有资源在互联网上的位置

URIs有两种，一种是URLs，一种是URNs

#### URLs

URL: uniform resource locator,平常说的URI基本默认就是URL。



<figure>
  <img src="{{ site.url }}/images/httpoverview02.jpg" alt="search screenshot">
  <figcaption>URL的解析</figcaption>
</figure>



从上图可以得知，URL一般遵循一种标准格式，这种格式包括三部分

> *  scheme：经常是 HTTP 协议 (http://)
> *  主机地址：比如上面的 www.joes-hardware.com
> *  指定某个资源：比如上面的 /specials/saw-blade.gif

#### URNs

URN:uniform resource name URN的用途是使客户端只要获取URN，无论资源的位置在哪里，都能获取到资源。URN的技术还未成熟，没有普及，所以目前URI基本都是默认为URL

## Transactions（事务）

一个HTTP事务是由一个HTTP请求命令和一个HTTP相应结果组成的，这种通信是具有HTTP报文的格式化数据块实现的。


<figure>
  <img src="{{ site.url }}/images/httpoverview03.jpg" alt="search screenshot">
  <figcaption>事务</figcaption>
</figure>

客户端完成一个任务可能不仅仅只有一个事务。加载一个图片浏览的页面，需要多个事务，首先是加载页面框架的事务，然后是获取各个图片的事务。

### Methods 

HTTP 支持集中请求命令，这种请求命令也称为请求方法，这些方法告知服务器应该执行什么操作，常见方法如下如

<figure>
  <img src="{{ site.url }}/images/httpoverview04.jpg" alt="search screenshot">
  <figcaption>HTTP请求方法</figcaption>
</figure>

### Status Codes（状态码）


HTTP响应报文都会携带一个状态码，并带有一个解释状态码的文本。


## Messages（报文）

报文有两种，一种是请求报文，一种是请求报文，两者格式基本相同。


<figure>
  <img src="{{ site.url }}/images/httpoverview05.jpg" alt="search screenshot">
  <figcaption>HTTP报文格式</figcaption>
</figure>

HTTP报文包含三个部分：

> *  起始行（start line）：如上图，请求报文的起始行包括“方法+资源+HTTP版本”
    而响应报文中则保罗“HTTP版本+状态码及其解释文本”
> *  首部（header）：起始行之后跟随的是首部，首部每行都是一个键值对，键和值用双引号:分开。
      **首部的最后是一个空行**
> *  主体（body）：起始行和首部都是纯文本的格式，而主体则为二进制数据，可以是文本，图片等。


## Connections

HTTP基于TCP连接，TCP是可靠传输协议，前者是应用层协议，后者是传输层协议

<figure>
  <img src="{{ site.url }}/images/httpoverview07.jpg" alt="search screenshot">
  <figcaption>HTTP基于TCP协议</figcaption>
</figure>

## Protocol Versions

目前推荐使用的HTTP版本是1.1，还存在其他0.9，1.0等版本

## Architectural Components of the Web（Web的结构组件）

上文讲到的HTTP客户端和HTTP服务器都是属于HTTP应用程序

还有其他一些重要的HTTP应用程序（比较难理解）

> *  Proxies (代理)
> *  Caches （缓存）
> *  Gateways （网关）
> *  CTunnels （隧道）
> *  Agents （agent代理）

