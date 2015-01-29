---
layout: post
title: "【译】Intents and Intent Filters"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android,xmpp,IM]
comments: true
share: true
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

tips:*该本博客的翻译类似笔记，大多着重意译博主觉得需要记住的知识。*

一个app中的三个最重要的组件------activity，service，broadcast receiver，都是借助intent启动的。Intent messaging is a facility for late run-time binding between components in the same or different applications.intent对象本身是一个数据结构，这个数据结构描述某个操作，或者是描述某种已经发生的事情。以上说到的几个组件，它们使用intent的方式各不相同。

