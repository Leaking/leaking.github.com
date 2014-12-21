---
layout: post
title: "【译】Activities"
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


## Activities

一个Activity是一个app的组成成分之一，它提供一个可以让用户进行交互的窗口，比如打电话，拍照，发邮件，看地图，每个act
ivity一般都会有一个充满屏幕的窗口，用户描绘activity的UI。有时，activity拥有的窗口可能小于屏幕大小。

一个APP一般拥有多个activity，它们之间一般拥有低耦合的关系。通常在APP中都用一个"main activity",
当APP启动时这个"main activity"就会首先呈现在用户面前。每个activity都可以启动其它的activity用来进行其它操作。当新的
activity被启动，前一个activity将被停止（stopped），当时系统将会报错该被停止的activity到一个堆栈中（back 
stack），一个新的activity被启动时它将会被push到back stack的顶部并呈现给用户。back stack秉承LIFO（"last in , first 
out"），所以，当用户在新的activity中操作结束之后，按下返回键，这个activity将会被弹出back 
stack，并且被desrotyed，接下来前一个activity将会重新出现（resume）。关于back stack的文章可以参考

[Tasks and Back Stack](http://developer.android.com/guide/components/tasks-and-back-stack.html)


当一个activity因为启动了新的activity而被停止（stopped），activity的生命周期回调方法将会被触发。当activity的状态发
生改变时，比如，当系统对一个activity进行create，stop， resume，destroy，这些情况都会触发回调方法。在这些回调方法里
面，你可以执行一些与当前状态变化相关的操作。比如，当activity进入stopped状态时，你可以释放activity一些比较占内存的
对象，比如网络连接，数据库连接，当activity被resume时，你可以重新获取一些必要的资源并恢复一些被中止的操作。

下文将讨论如何创建和使用activity，并会详细地分析activity的生命周期，这样一来，你就可以合理地处理activity中的生命周
期切换。

## Creating an Activity

为了创建一个activity，你需要创建一个Activity类的自雷（或者直接继承一个现有的Activity的子类）。在你创建的Activity的
子类中，你需要重写一些生命周期的回调方法，它们将在activity的状态发生改变时被调用，比如create，stop，resume，destro
y，其中两个最重要的回调方法是：

**onCreate()**

你必须实现这个方法，因为系统创建activity时都会调用这个方法。你需要在这个方法中初始化各个组件，最重要的是，在这个方
法里你需要调用setContentView()去定义你的activity的UI.


**onPause()**

当用户离开当前activity时首先都会调用这个方法（注意
，无论以什么姿势离开，都会先调用这个方法），所以，你可以再这里提
交一些需要保存的修改，因为用户可能不会再回来这个ac
tivity。

当activity进行切换或者发生预料之外的中断，你可以使
用其他几个生命周期的回调方法来提供一个流畅的用户体
验，后面讲详细
介绍这些回调方法。

## Implementing a user interface

activity的UI是通过许多分层的view对象组成的，view对
象都继承于View类，它们都占据一个正方形区域并且能响
应用户交互。

android提供了很多定制好的view，你可以直接使用它们
组织你的布局。"Widgets"是其中一种view，它们可以提
供一些可见并且可交互的元素在屏幕上，比如buttom,
textview,image。而说到"layout"，它继承于ViewGroup,
ViewGroup为其子视图（child 
view）提供一个布局模型，比如linear layout， grid 
layout，relative layout。另外，你也可以自己自定义
自己的view或者view group。

定义布局的最常使用的方式是使用XML布局文件，这样你
的布局设计就可以独立于activity的代码，然后将你设计
的布局通过setContentView的方式设置到activity中。当
然，你也可以在activity代码中插入view到viewGroup中
，然后将你的根布局传递给setContentView().








