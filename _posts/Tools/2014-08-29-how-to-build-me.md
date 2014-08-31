---
layout: post
title: "How to Build Me"
modified:
categories: Tools
excerpt:
tags: []
image:
  feature:
date: 2014-08-29T00:23:53+08:00
---

### 怎么新建类别，文章

##1，
测试文章修改，，，-----delete
用新建一个篇文章
其中posts即将作为类别
octopress new post "New Post Title" --dir posts
然后_post文件夹中就会产生相应文件
在每篇博文的md文件中，头部都有注明类别，这个地方很重要！

##2，
在catagory中添加类别

##3，
在根目录下添加posts文件夹，加入一个index.html的布局文件，可以copy自其它分类
完成！（_site中的文件会自动生成）