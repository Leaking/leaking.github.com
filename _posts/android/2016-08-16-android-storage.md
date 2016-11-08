---
layout: post
title: "Android存储的坑"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
---

一直以来用到Android中关于*Sdcard*或者*data/data*相关的存储时，以及读写权限问题，难免会有混淆，这里做个总结。


# 总体

正所谓*no-pic-u-say-jj*，一直以来看很多总结*Android*存储的文章，我都很期待有一副图片可以总结以下各种存储。此处简单画了一份。

<figure>
  <img src="{{ site.url }}/images/storage_1.jpeg" alt="search screenshot">
  <figcaption></figcaption>
</figure>   




## /sdcard

该部分不会随着App删除而删除；其他App（同个手机上的程序）以及用户（操作手机的人）可以访问；需要读写权限；不会计入App大小的计算

{% highlight java %}

Environment.getExternalStorageDirectory();

/sdcard/

Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES));

/sdcard/Pictures

{% endhighlight %}

## /sdcard/Android/data/<package-name>

该部分的特点在与4.4开始无需读写权限，会随着App删除而删除，会计入App大小计算

{% highlight java %}

getExternalFilesDir(null);

/sdcard/Android/data/<package-name>/files

getExternalFilesDir(Environment.DIRECTORY_PICTURES);

/sdcard/Android/data/<package-name>/files/Pictures

getExternalFilesDirs(null);

[/sdcard/Android/data/<package-name>/files]

getExternalCacheDir();

/sdcard/Android/data/<package-name>/files

getExternalMediaDirs();

[/sdcard/Android/media/<package-name>]

{% endhighlight %}


## /data/data/<package-name>

无需读写权限，会随着App删除而删除，会计入App大小计算，其他App（同个手机上的程序）以及用户（操作手机的人）不可以访问（除非手机已经root）


{% highlight java %}

getFilesDir();

/data/data/<package-name>/files

getCacheDir();

/data/data/<package-name>/cache

getDir("custom", Context.MODE_PRIVATE);

/data/data/<package-name>/app_custom

{% endhighlight %}
