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


一个Activity是一个app的组成成分之一，它提供一个可以让用户进行交互的窗口，比如打电话，拍照，发邮件，看地图，每个activity一般都会有一个充满屏幕的窗口，用户描绘activity的UI。有时，activity拥有的窗口可能小于屏幕大小。

一个APP一般拥有多个activity，它们之间一般拥有低耦合的关系。通常在APP中都用一个“main activity”,当APP启动时这个
“main activity”就会首先呈现在用户面前。每个activity都可以启动其它的activity用来进行其它操作。当新的
activity被启动，前一个activity将被停止（stopped），当时系统将会报错该被停止的activity到一个堆栈中（back 
stack），一个新的activity被启动时它将会被push到back stack的顶部并呈现给用户。back stack秉承LIFO（“last in , first out”），所以，当用户在新的activity中操作结束之后，按下返回键，这个activity将会被弹出back 
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
子类中，你需要重写一些生命周期的回调方法，它们将在activity的状态发生改变时被调用，比如create，stop，resume，destroy，其中两个最重要的回调方法是：

**onCreate()**


你必须实现这个方法，因为系统创建activity时都会调用这个方法。你需要在这个方法中初始化各个组件，最重要的是，在这个方法里你需要调用setContentView()去定义你的activity的UI.


**onPause()**

当用户离开当前activity时首先都会调用这个方法（注意，无论以什么姿势离开，都会先调用这个方法），所以，你可以再这里提交一些需要保存的修改，因为用户可能不会再回来这个activity。

当activity进行切换或者发生预料之外的中断，你可以使用其他几个生命周期的回调方法来提供一个流畅的用户体验，后面讲详细介绍这些回调方法。


## Implementing a user interface


activity的UI是通过许多分层的view对象组成的，view对象都继承于View类，它们都占据一个正方形区域并且能响应用户交互。

Android提供了很多定制好的view，你可以直接使用它们组织你的布局。“Widgets”是其中一种view，它们可以提供一些可见并且可交互的元素在屏幕上，比如buttom,textview,image。而说到“layout”，它继承于ViewGroup，ViewGroup为其子视图（child view）提供一个布局模型，比如linear layout， grid layout，relative layout。另外，你也可以自己自定义自己的view或者view group。

定义布局的最常使用的方式是使用XML布局文件，这样你的布局设计就可以独立于activity的代码，然后将你设计
的布局通过setContentView的方式设置到activity中。当然，你也可以在activity代码中插入view到viewGroup中
，然后将你的根布局传递给setContentView().

关于UI的设计，可以参考[User Interface](http://developer.android.com/guide/topics/ui/index.html)




## Declaring the activity in the manifest




为了让系统能得到你的activity，你需要在manifest文件中声明activity.如下：在application元素中增加一个子元素activity

{% highlight xml %}

<manifest ... >
  <application ... >
      <activity android:name=".ExampleActivity" />
      ...
  </application ... >
  ...
</manifest >

{% endhighlight %}

在这个activity元素中，可以增加诸多属性，比如label,icon,name,theme，其中name属性不能随便进行修改，可以阅读以下这篇文章

[Things That Cannot Change](http://android-developers.blogspot.jp/2011/06/things-that-cannot-change.html)

## Using intent filters

activty元素可以通过定义intent filter来声明其他app或者其他app component如何启动它。具体可以查看如下这篇文章


[Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html)


## Starting an Activity


通过调用startActivity()并传递给它一个intent对象可以启动另一个activity。Intent可以用来描述你即将启动的activity或者你即将执行的操作。甚至可以启动不同application之间的activity。Intent也可以用于传输少量的数据。

以下是通过一个activity启动另一个activity的例子

{% highlight java %}

Intent intent = new Intent(this, SignInActivity.class);
startActivity(intent);

{% endhighlight %}

当你的app要执行一些诸如发邮件，发短信等操作时，你可以启动其它app的activity进行操作，当设备中有多个app中拥有这样的activity，则会弹出一个选择框。当你执行完操作，将返回原本的activity。

如下是一个发送邮件的操作，

{% highlight java %}

Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
startActivity(intent);

{% endhighlight %}

此处recipientArray是接收方的邮件地址，将自动填充到打开的发送邮件的activity中。





##  Starting an activity for a result

这个方面比较熟悉，下面就只是列一个小例子，通过startActivityForResult()获取一个联系人。

{% highlight java %}

private void pickContact() {
    // Create an intent to "pick" a contact, as defined by the content provider URI
    Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
    startActivityForResult(intent, PICK_CONTACT_REQUEST);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
    if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
        // Perform a query to the contact's content provider for the contact's name
        Cursor cursor = getContentResolver().query(data.getData(),
        new String[] {Contacts.DISPLAY_NAME}, null, null, null);
        if (cursor.moveToFirst()) { // True if the cursor is not empty
            int columnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
            String name = cursor.getString(columnIndex);
            // Do something with the selected contact's name...
        }
    }
}

{% endhighlight %}




##  Shutting Down an Activity

终结一个activity可以通过finish()方法，也可以通过finishActivity()
关闭当前activity以及之前启动的activity（堆栈下面的其他activity）

##  Managing the Activity Lifecycle


你在重写各个生命周期方法时，需要先调用其父类的的方法。

关于生命周期，一般能看到三个循环

**entire lifetime**
该循环发生在onCreate()和onDestroy()之间，一些关乎全局的状态(比如布局)需要在onCreate()中创建，而在onDestroy()释放一些持久性的资源。

比如：你在后台下载数据，你可能在onCreate()中创建下载的线程，并且你需要在onDestroy()停止该线程。

**visible lifetime**
该循环发生在onStart()和onStop()之间，在该循环中你始终可以看到该activity并且可以与它交互，onStop是启动其它activity时被调用的。

比如，你可以再onStart()中注册广播(可能你收到广播就需要更新UI了)，并且在onStop()中注销广播。这样一来，你可能会多次执行onStart()和onStop()方法。

**foreground lifetime**
该循环在onResume()和onPause()之间，在这期间，你的activity在所有activity之上，并且拥有用户焦点。一般在以下情况你会跳出这个循环，比如，设备进入睡眠(进入睡眠应该是关闭屏幕了，而不是回到主界面)，或者一个dialog出现了。

由于这个状态切换很频繁，所以在这两个方法中应该执行一些轻量级的代码，避免状态切换太慢。

生命周期的几个循环如下图
<figure>
  <img src="{{ site.url }}/images/activity_lifecycle.jpg" alt="search screenshot">
  <figcaption>生命周期示意图</figcaption>
</figure>

从图中看到，可以中途被杀死的状态有onPause()，onStop()，onDestroy()

所以，一般我们需要在onPause()中选择保存一些需要保存的数据，当然，阻塞过程不该在onPause()中执行，否则会影响状态切换速度。

以上其实在任何生命周期的回调方法中，设备环境极度恶劣的情况下，都随时可能被杀死。





## Saving activity state


举个例子，activityA中有一个edittextA，你从activityA跳转到activityB，再按返回键回到activityA，此时edittextA中的内容依然被保存着。但是有时候可能activity可能被强制回收，此时edittextA中的内容将不会被保存。

上面之所以edittext的内容会被保存，是因为很多组件都默认都会保存其状态，但是你必须给这些组件一个ID值，因为保存其状态时通过键值对保存的。另外，如果你对组件的saveEnabled属性设置为false，则不会有自动保存的功能。

上面是activity进入后台回来之后依然能恢复UI组件的状态，但是当后台的activity被意外地destroy时，则不会恢复UI状态，则需要通过在 onSaveInstanceState()中保存数据，在onCreate()或者onRestoreInstanceState()中恢复数据的方法来实现保存UI状态。在使用onSaveInstanceState()和onCreate()或者onRestoreInstanceState()方法时，需要先各自调用其父类的方法。

onSaveInstanceState()是在系统意外回收activity时会被自动调用，但是用户主动finish掉一个activity时(比如按返回键或者调finish())并不会自动调用onSaveInstanceState().



## Handling configuration changes

当设备的设置发生改变时，需要使用以上保存状态的方法，因为设备状态改变时，activity不会自动保存数据，它们是被destroy再创建的。比如语言环境变化，横屏竖屏切换。

更多的关于运行时的状态变化知识可以参考

[Handling Runtime Changes](https://developer.android.com/guide/topics/resources/runtime-changes.html)



## Coordinating activities


前文我们提到，要保存数据，则最好在onPause()中进行，除了因为它是onPause，onStop，onDestroy三个方法中最先被调用的，还有一个原因，activityA调用onPause之后，可能另一个activityB已经启动了（已经调用了onCreate(), onStart()和onResume()）之后，当前activityA才会调用onStop和onDestroy。而如果此时activityB要使用activityA保存在数据库中的数据，则需要activityA在onPause中即时将数据写入数据库。而不是在onStop或者onDestroy中。

