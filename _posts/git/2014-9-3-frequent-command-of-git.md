---
layout: post
title: "Git常用工作流"
modified:
categories: Git
comments: true
excerpt:
tags: [Git]
date: 2014-9-3 22:41:52
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

近来在电脑上写博客然后提交到Github上时，学习了一点Git的指令。Git值得学习的东西很多，但是经常用到的
却不多。于是我这里记录几条通用的指令的用法，以及整个提交流程。


## 1、查看状态

执行如下代码
{% highlight bash %}
$ git status
{% endhighlight %}

插卡结果又两种，一种时本来就被Git跟踪的文件（可能被修改或者删除），一种是未被Git跟踪的文件（新增加的文件），
Git将其描述为Untracked files

###1.1、被修改或者删除的文件
如下图
<figure>
  <img src="{{ site.url }}/images/delete_modified.jpg" alt="search screenshot">
  <figcaption>修改或者删除的文件</figcaption>
</figure>


###1.2、新增加的文件
如下图
<figure>
  <img src="{{ site.url }}/images/untrack.jpg" alt="search screenshot">
  <figcaption>新增加的文件</figcaption>
</figure>


## 2、 

执行如下代码
{% highlight bash %}
$ git add .
{% endhighlight %}

该指令能将上面**所有**“被修改或者删除的文件”都标记（标记过才能准备push到Github，此处所谓“标记”的用词只是为了方便理解，并非正式用语）

## 3、----------重点

执行如下代码
{% highlight bash %}
$ git add -u
{% endhighlight %}

这条指令能“标记”**所有**被删除的文件

## 4、

执行如下指令
{% highlight bash %}
$ git commit -m "some comments"
$ git push origin master
{% endhighlight %}

执行完如上指令，即完成整个提交过程

