---
layout: post
title: "Jekyll搭建博客"
modified:
categories: Tools
comments: true
excerpt:
tags: [jekyll]
date: 2014-08-29T00:23:53+08:00
comments: true
share: true
---


终于搭建好了这个博客，记录一下安装过程

> * （必须）安装ruby,Devkit
> * （必须）安装Jekyll
> * （非必须）Python，


其中安装Jekyll的时候，我总是遇到问题，结果发现问题是天朝的网络在作祟
先运行如下代码即可

{% highlight ruby %}
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://ruby.taobao.org/
$ gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org
# 请确保只有 ruby.taobao.org
$ gem install rails
{% endhighlight %}



以下是我参考的网站，根据这几个网站完全能搞懂如何在Github上搭建博客


1、[http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#install-pygments](http://yizeng.me/2013/05/10/setup-jekyll-on-windows/#install-pygments)
2、[http://www.cnblogs.com/hutaoer/archive/2013/02/06/3078873.html](http://www.cnblogs.com/hutaoer/archive/2013/02/06/3078873.html)
3、[http://blog.fens.me/jekyll-bootstarp-github/](http://blog.fens.me/jekyll-bootstarp-github/)
4、[http://www.jekyllbootstrap.com/usage/jekyll-quick-start.html](http://www.jekyllbootstrap.com/usage/jekyll-quick-start.html)
5、[http://ruby.taobao.org/](http://ruby.taobao.org/)

