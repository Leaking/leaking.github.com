---
layout: post
title: "【译】Tasks and Back Stack"
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

总结一下activities和tasks的常见用法

1. 当Activity A启动Activity B，Activity A进入stopped的状态，但是系统保持其状态（比如你滚动视图的滚动位置以及表格中填写的数据）。当用户在Activity B中按下返回键，那么Activity A的状态数据将会还原。

2. 当用户按下home键，当前的activity将会进入stopped状态，该activity所在task也会进入后台，系统保存所有在该task中的activity的状态。当用户在home主页上点击该app的图标，将还原该task的状态，该task回到前台。

3. 当用户按下返回键，当前的activity被弹出堆栈并且被destroy，前一个activity会被resume。注意，一个呗destroy的activity不会保存其状态。

4. activity可以被初始化多次，甚至从其他task中初始化。


##Saving Activity State

如前面所说，当从activity A中跳转到activity B中，activity A进入stopped状态，但是也可能被系统回收资源而destroy，所以为了预防这种情况，得作用保存数据，还原数据的准备。使用onSaveInstanceState()

##Managing Tasks

Android中的Activity和Task的管理基本都是自动化的，但是开发过程中如果想要按照自己的需要操纵Activity和Task，可以使用<activity>标签和为传递给startActivity的intent设置flag属性。

如下，是可以使用的activity标签：

+ taskAffinity
+ launchMode
+ allowTaskReparenting
+ clearTaskOnLaunch
+ alwaysRetainTaskState
+ finishOnTaskLaunch

以下是你经常要使用的flag属性

+ FLAG_ACTIVITY_NEW_TASK
+ FLAG_ACTIVITY_CLEAR_TOP
+ FLAG_ACTIVITY_SINGLE_TOP


##Defining launch modes#

Launch modes可以让你控制activity以怎样的方式启动到当前的task中。

你可以通过以下两种方式定义Launch mode

+ 使用manifest file

+ 使用Intent flags


举个例子，Activity A启动 Activity B的时候，如果要定义Activity B的启动方式，可以通过在manifest file中定义Activity B的launch mode，也可以在Activity A启动Activity B时使用的intent中设置flags。如果manifest和intent中的flags中都设置了launch mode，则intent中的flag优先。

有些launch mode 只能在manifest file中定义，有些只能在intent flags中定义。

# 使用manifest file

可以在manifest file中使用 \<activity\> 标签的launchMode属性定义launch mode

launchMode属性可以有四个值

+ standard (默认方式)

默认情况下，系统通过standard的方式启动一个activity，并且这个activity可以被创建多次，一个task中可以拥有多个
该activity的实例。

+ singleTop

如果某个activity的实例已经处于当前task中的back statck的最上层，则当再次调用该activity时，将不会再次创建一个新的实例，而是将调用栈顶的activity实例的onNewIntent()，并且intent也被传递到onNewIntent()，从而跳转到back stack最上层的实例。如果再创建一个activity时，当前back stack的顶层不是该activity的实例，则按照standard的方式继续创建一个实例

举个例子：

一个back stack中拥有A-B-C-D四个activity，其中D处于最顶层。

如果一个新的创建activityD的intent到来了，如果此时D被设置成"standard"，则新建一个D实例，此时back stack的内容为A-B-C-D-D。

如果此时D被设置成"singleTop",则创建Activity D的intent被传递到栈顶的D实例的onNewIntent()方法，于是跳转到栈顶的activityD实例。此时back stack内容是A-B-C-D。此时按下返回键会返回到C，back stack变为A-B-C

但是，如果实例D中启动一个activity B实例，即时B的启动模式为"signleTop", back stack 都将变为 A-B-C-D-B.



此处讲解一下onNewIntent()方法

<figure>
  <img src="{{ site.url }}/images/onNewIntent.png" alt="search screenshot">
  <figcaption>onNewIntent()</figcaption>
</figure>


> onNewIntent()只有在singleTop启动模式发挥作用时才会被调用，并且此时它将会接收到一个intent.举一个例子，还是上面的back stack，其中内容是A-B-C，此时在C中启动一个D，并且传递的一个intent C，此时新建一个D实例，back stack的状态变为A-B-C-D，而且D中收到intent C。我们定义D的启动模式时singleTop，此时在栈顶的D中启动一个activity D实例，并且传递一个intent D 由于D是singleTop的启动模式，所以此时不会创建新的D实例，而只会调用back stack顶端的activity D实例的onBackIntent() 方法，并且将intentD传递给onNewIntenet()。但是这个时候通过getIntent获取到的依然是intent C，
此时可以利用setIntent()将其更新为intent D。另外，调用完onNewIntent()之后会调用onResume()，你可以在这个时候根据需要刷新界面。



+ singleTask

与singleTop类似，singleTop中的概念类似“复用”，该复用的前提是：要复用的实例必须处于back stack的顶端。而singleTask也有“复用”的作用，但是复用的前提比较宽松：要复用的实例存在当前task中即可，或者存在后台某个task也可以。

注意，本文中，讲到的task和back stack都是一样的意思，因为一个task中只有一个bask stack.



举个例子，back stack 中的内容是A-B-C-D，其中B的启动模式为singleTask，在D中启动一个新的实例B，此时调用堆栈中B实例的onNewIntent()方法，然后back stack就变成A-B，此时C,D都被弹出。

另一个例子，有两个task，一个是A-B,一个是C-D,其中D被声明为D，则在B中启动一个D时，第一个task将会变成A-B-C-D


以上讲的是启动模式为singleTask的activity实例已经存在，可以“复用”的情况。那么当activity实例不存在，则会重新创建一个activity实例，并且放到单独一个task中，但是一般情况下，如果是启动同一个应用程序中activity，该新建的activity实例依然会被放到当前task中。只有启动其它应用程序的activity才会放置到新的task中。

+ singleInstance

singletInstance和singleTask类似，

当back stack中只有一个声明为singleTask的activity实例时，继续启动其它activity实例，则会继续加到同一个task中。

当back stack中只有一个声明为singleInstacne的activity实例时，继续启动其它activity实例，则会在一个新的task中创建activity。


再举一个例子，android的浏览器应用程序中的浏览器activity被定义为singleTask启动模式。所以当你的app试图打开android浏览器时，浏览器的activity不会被放到和你的app的task中，而是重新启动一个task来放置浏览器的activity。或者浏览器已经在后台的某个task中，则直接将该task转到前台呈现给用户。

当我们在同一个task中或者不同task中启动一个activity，按下返回键都将返回到前一个activity。但是当我们启动一个启动模式为singleTask的activity时，如果该activity有一个实例在某个后台task中，则该后台task被转到前台，此时这个task中的activity将会被放到原本task中上端。比如下图


<figure>
  <img src="{{ site.url }}/images/singleTask.png" alt="search screenshot">
  <figcaption>singleTask的特殊back stack示意图</figcaption>
</figure>


如上图所示，当调用到后台task中的Y（Y的启动模式时singleTask），Y所在的整个Task都被放到前台，其中的activity都被放到原本back stack的上端。


# 使用 intent flags

除了manifest file可以定义启动模式，intent flag也可以。如上面提到的，intent flag定义的启动模式会覆盖manifest file中定义的启动模式。 几种常见的intent flag如下


+ FLAG_ACTIVITY_NEW_TASK 

在一个新的task中创建一个activity实例，如果某个已经存在的task中有一个同样的activity，则该task被切换到前台.并且调用该activity的onNewIntent()

该intent flag的作用与上文讲到的“singleTask”一样


+ FLAG_ACTIVITY_SINGLE_TOP

该intent flag与上文讲到的"singleTop"一样

+ FLAG_ACTIVITY_CLEAR_TOP

如果某个activity已经有一个实例在当前task中，则再次试图创建该activity的实例时，则task中在该activity实例上面的activity都将会被弹出并destroy，使该activity实例处于堆栈顶端，并调用其onNewIntent()

launchMode中没有一个属性值与FLAG_ACTIVITY_CLEAR_TOP作用一样


FLAG_ACTIVITY_CLEAR_TOP经常与FLAG_ACTIVITY_NEW_TASK结合使用。它们能跳转到另一个task中某个activity并且响应接收到的intent对象。


##处理 affinities（亲和度）

未遇到使用场景，以后再参考文档学习

##清空 back stack

如果用户离开一个task一段比较长的时间，系统会清除task中除了根部activity之外的其他activity。当用户再次回到该task，只能看到根部activity的状态被保存了。系统这样做是因为超出了某个时间长度之后，用户可能已经不要他们之前操作留下的状态，而将重新进行其他的操作。

以上将的这个系统的特点，可以通过activity的属性进行修改定制

+ alwaysRetainTaskState

如果一个task中的根部activity的这个属性被设置为"true"，则上面描述的系统默认设定将不会发挥作用。即时离开了task很长的时间，其中的activity也将被保存状态

+ clearTaskOnLaunch

如果task中的根部activity的这个属性被设置为“true”，则无论用户离开task的时间是短还是长，task中根部activity以上的activity都将被清除。这恰恰好和alwaysRetainTaskState相反

+ finishOnTaskLaunch


这个属性有点像clearTaskOnLaunch，但是它的作用对象是某个activity，而非整个task。当某个activity的finishOnTaskLaunch属性被设置为“true”，则每当用户离开该task再回来，该activity都将被销毁











