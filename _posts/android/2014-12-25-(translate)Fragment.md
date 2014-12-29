---
layout: post
title: "【译】Fragment"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---


## Fragments

Fragment属于Activity的一个部分，它能让你在一个activity中建立多屏（muti-pane）。并且一个fragment可以再多个activity中复用。fragment拥有自己的生命周期，自己的输入事件。你可以在activity运行期间动态加入或者移除一个fragment。


fragment必须嵌入在activity中而且其生命周期直接受宿主activity生命周期的影响。比如一个activity进入paused状态时，其中的所有fragment也会进入paused。当activity进入destroyed，其中的所有fragment也会。在activity运行期间，你可以加入移除fragment，并且可以为它们建立一个back stack，这个back stack由activity维护，这个back stack允许用于按下返回键弹出一个fragment。

当你将fragment当做布局的一部分，你可以再xml布局文件插入fragment，或者在你的代码中插入。然后，其实你可以使用一个没有UI的fragment，拥有后台线程。


### Design Philosophy

利用fragment，你可以更好的处理多屏显示，如下图

<figure>
  <img src="{{ site.url }}/images/fragments.png" alt="search screenshot">
  <figcaption>fragment的设计原理</figcaption>
</figure>


关于利用fragment适应屏幕旋转，可以参考

[Supporting Tablets and Handsets](https://developer.android.com/guide/practices/tablets-and-handsets.html)


### Creating a Fragment

创建Fragment需要继承Fragment或者其现成的子类。创建一个Fragment和创建一个Activity非常相似，它们拥有很多一样的生命周期回调方法。创建一个Fragment，你至少实现以下几个方法：

**onCreate()**

在这个方法里，你需要对一些特殊的数据进行初始化，即时fragment从paused或者stopped恢复过来，这些数据也会保持不变。

**onCreateView()**

当fragment开始绘制自己的UI时，就会回调这个方法，并且返回一个view对象，当一个fragment不提供UI时（只是跑后台进程），这个方法也可以返回null。

**onPause()**

和activity的生命周期类似，你需要在fragment中的onPause()保存一些数据（比如写入数据库）。调用onPause时，fragment可能进入后台不可见，也可能被销毁destroy。

APP至少要实现以上三个方法。另外，其他生命周期的回调方法下文将会讲解。

平时你需要了解几个常用的Fragment的子类

**DialogFragment**
**ListFragment**
**PreferenceFragment**

## Adding a user interface

Fragment的UI创建在onCreateView()中进行，如下

{% highlight java %}

public static class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}

{% endhighlight %}

关于在该方法中使用savedInstanceState下文会详细讲解。

inflate的第三个参数true或者false，还不大懂。

##Adding a fragment to an activity

往activity中添加fragment有两种方法，一种是通过XML文件，一种是通过代码。

**Declare the fragment inside the activity's layout file.**


{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment android:name="com.example.news.ArticleListFragment"
            android:id="@+id/list"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
    <fragment android:name="com.example.news.ArticleReaderFragment"
            android:id="@+id/viewer"
            android:layout_weight="2"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
</LinearLayout>


{% endhighlight %}

**Or, programmatically add the fragment to an existing ViewGroup**

在activity中动态的加入一个fragment，只需要指定一个fragment所在的viewgroup的ID，具体方法如下：

首先你需要获得一个FragmentTransaction对象。

{% highlight java %}

FragmentManager fragmentManager = getFragmentManager()
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();

{% endhighlight %}

然后使用add()方法，指定将fragment插入到哪个viewgroup。


{% highlight java %}

ExampleFragment fragment = new ExampleFragment();
fragmentTransaction.add(R.id.fragment_container, fragment);
fragmentTransaction.commit();

{% endhighlight %}

add()的第一个参数是fragment将要插入的viewgroup的ID。使用FragmentTransaction之后，你调用commit()方法，FragmentTransaction做出的修改才会起作用。

## Adding a fragment without a UI

用一个没有UI的fragment运行后台程序，参考ApiDemo中的FragmentRetainInstance.java


### Managing Fragments






















