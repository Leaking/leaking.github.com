---
layout: post
title: "【译】Processes and Threads"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
comments: true
share: true
---


原文地址[Processes and Threads](http://developer.android.com/tools/docs/docs/guide/components/processes-and-threads.html)

# Processes and Threads

当一个app的组件（这里一般指四大组件Activity，Service等）启动时，如果此时系统没有其他组件正在运行，则android系统会为该app启动一个新的linux进程，而且该进程中只有一个线程。默认情况下，app中的所有组件，都运行在同个进程中的同个线程（称为主线程）。如果一个app的组件启动时，该app中已经存在一个进程，则该组件将运行在同一个进程中，并且使用同一个执行线程(thread of execution)。其实，你还可以设置同一个app中的各个组件运行在不同的进程中，并且各个进程中可以创建多个线程。

这篇文档主要讨论关于android应用中的进程和线程（Processes and Threads）

## Processes

默认情况下，app中的各个组件时运行在同一个进程中的，并且大多应用应该遵循这个规则。然后，如果你觉得你需要指定某个组件专属于某个进程，则你可以在manifest 文件中做相应配置。

manifest文件中各种组件的标签------\<activity\>, \<service\>, \<receiver\>, \<provider\>都支持属性android:process，这个属性可以为该组件指定一个进程。组件通过这个属性指定了一个自己专属的进程之后，虽然有些组件仍然共享一个进程，但是有些组件将不再与其他组件共享一个进程。甚至，你可以通过设置android:process使得不同app运行在同一个进程里面------前提是不同app要使用想用的linux 用户ID和相同的签名。（android插件的原理貌似涉及到这一点）

\<application\>元素也支持android：process属性，它的作用范围将是所有的组件。

android系统中，某些正在服务于用户的进程需要内存时，如果此时系统内存不足，则系统会考虑关闭某些进程。一个进程被杀死，则进程中的所有组件都将被销毁。被销毁的组件重新启动时，其所在的进程也将重新启动。

当决定该杀死哪个进程时，android系统会衡量各个进程对于用户的重要性。比如，某个进程持有多个不会再显示于屏幕上的activity，则该进程是极有可能会先被杀死的，杀死一个进程的决定因素是运行在该进程中的组件的状态。以下讨论关于选择杀死哪些进程的规则。

### Process lifecycle

android系统会尽可能地维持app的进程，但是有时为了回收内存用于更新更重要的进程，不得不杀死某些进程。为了选择一个进程杀死，系统将进程划分出一个重要性分级，这个划分基于进程中的组件以及组件的状态。当需要回收系统资源时，重要性最低的进程首先被杀死，然后按照重要性依次杀死其他进程。

重要的分级分为5个级别，以下按照重要性从高到底列出各种进程。


+ Foreground  process

用户的当前操作所在的进程就是前台进程。满足以下任何一个条件的进程都是前台进程。

1. 用户当前交互的activity所在的进程（没有消失，而且获得焦点）
2. 某个service与用户当前交互的activity绑定，这个service所在的进程。
3. 某个service运行于前台（也就是调用了startForeground()的service），这些service所在的进程。
4. 某个进程执行了它的任何一个生命周期回调方法（onCreate(),onStart(),onDestroy()），这些service所在的进程。
5. 正在执行某个BroadcastReceiver的onReceive()方法的进程

通常情况下，一个app只会同时拥有很少的前台进程。当系统的内存特别低时，前台进程也将不能继续运行，杀死它们以维持用户UI界面可以正常反应，前台进程都是在最后才被杀死，

+ Visible process

虽然有些进程中没有任何前台组件，当时依然可以对当前屏幕显示的东西有所影响。这种进程就是可见进程，满足以下任何条件的组件都是可见进程。

1. 某个activity虽然不是处于前台（foreground），但是依然可见（visible），一般都是因为visible precess执行了onPause方法。举个例子，某个处于前台的activity启动了一个dialog，此时该activity从前台（foreground）转变为可见（visible）。这种可见activity所在进程就是visible precess.
2. 某个service与某个visible 或者foreground的activity绑定，则该service所在的进程就是visible precess。
一般visible precess不会被杀死，除非系统急需回收资源，并且得保持foreground process运行，则就必须杀死visible process。

+ Service process

Service process 中一般运行一个startService()启动的Service，并且还没变成上面两种进程。虽然service process并不会直接和用户界面有直接的关系，但是通常都在执行某些和用户相关的操作，比如播放音乐或者从网络下载数据。系统一般会保持这些进程运行，除非需要到了需要杀死它们以释放资源的时候。

+ Background process

Background process中拥有用户当前看不见的activity，也就是该activity执行了onStop方法，比如activityA启动了activityB，则activityA就变得不可见。这种进程并不会对用户界面有直接影响，系统当需要为foreground ,visible 或者 service process回收内存时，就会杀死background  process。通常会同时有多个background process在运行，它们被保持在一个LRU（least recently used）队列中。这样太久没呈现到界面上的activity所在的进程就会被优先杀死。如果activity适当地重写它的生命周期回调方法，将可以在进程被杀死时保存数据，并在activity重新出现时恢复数据，通过这种方式，用户就不会感受到进程被杀死。

+ Empty process

Empty process中没有任何活跃的组件。这种进程存在的意义就是为了缓存，它们可以改善下次组件的启动时长。系统杀死这种进程一般是为了平衡这些进程中的缓存以及底层kernel缓存。（这句话不大懂，翻译不好）
当一个进程同时属于以上5种进程中的多种类型，则android系统会将其归类到重要性较高的一种。比如一个进程同时拥有一个service和一个visible的activity，则该进程将被归类到visible process，而不是service process。
另外，一个进程的排序，可能会因为某种依赖关系而上升，比如，进程A中某个组件依赖于进程B中的某个组件，则进程B的重要性排序就不能低于进程A。至少重要性要相同。


如上所述，我们可以知道，运行着service的进程，以及运行着后台activity的进程，前者的重要性排序高于后者。所以需要启动一个耗时操作的activity，这个activity最好启动一个service执行该耗时操作，而不是通过建立一个普通的worker thread。特别是当该耗时操作的时间长于activity的生命周期。比如，一个activity将上传一张图片，可以启动一个service来上传图片，这样，这个service将可以在用户离开activity时仍继续运行。使用service可以保证该耗时操作至少有一个“service process”级别的重要性。无论所在的activity发生了什么。同理，我们最好在broadcast receiver中通过启动一个service执行耗时操作，而不是启动一个thread。

**（上面的耗时操作如果是启动thread，则所在进程的重要性排序只能是“background process”，低于"service process"）**

## Threads

当一个app启动时，系统会为该app创建一个线程，该线程称为“main thread”（主线程），这个线程管理UI界面中各个控件的事件分发，控件交互，绘制。所在该线程的重要性不言而喻。主线程有时也被称为UI线程。

系统并不会为各个控件创建单独的线程（我们这里说的控件区别以上的组件，控件这里指UI控件）。同个进程中的所有控件都在UI线程中（一个进程中只有一个UI线程），系统对每个控件的调用都是发生在UI线程中。所以，系统的回调方法（比如onKeyDown()这种反映用户动作的方法和生命周期回调方法）都运行在进程中的UI线程。

比如：当用户按下屏幕上某个按钮，UI线程会将该touch event分发给该按钮控件，然后按钮控件将依次设置其自身状态为pressed state，然后发出一个把自己设置为invalidate（暂时不可以再被按下）的请求到事件队列中。UI线程处理队列中的请求并且通知按钮组件进行重绘。

当app在相应用户操作时还要执行一些繁杂的操作，UI线程这种单线程的模型的性能将会很差，除非你恰当的进行处理。需要抢到的一点，如果访问网络，访问数据库等耗时操作都在UI线程中执行，将会阻塞整个UI进程。当UI进程被阻塞，事件将不能被分发，重绘也不能进行。从用户的角度来看，此时应用被挂起了，暂停了。当UI线程被阻塞超过几秒钟之后（目前系统设置大概5秒钟），用户将会得到一个臭名昭著的“application not responding”（ANR）的弹出窗口。用户此时只能被迫选择退出app了。

另外，android的UI控件都不是线程安全的，所以对UI组件的修改，不能再worker thread进行，而应该在UI 线程中修改UI控件。综上，android系统UI线程的这种单线程模型有以下两条规则：

+ 不能阻塞UI线程
+ 不能再UI线程之外操控UI控件。

### Worker threads

如上面所描述的单线程模型，UI的反应不该阻塞UI线程，当你需要执行不是短时间的操作，那么你最好就在单独的线程中执行。
比如，以下在一个点击监听器中下载图片，并将图片显示在imageview中，

{% highlight java %}

public void onClick(View v) {  
    new Thread(new Runnable() {  
        public void run() {  
            Bitmap b = loadImageFromNetwork("http://example.com/image.png");  
            mImageView.setImageBitmap(b);  
        }  
    }).start();  
}  

{% endhighlight %}

乍一看，上面的代码似乎没什么问题，我们创建一个新的线程处理网络操作。然后它只符合了上面讲到的关于单线程模型两条规则的第一条，而未被了第二条--------只能在UI线程中操纵UI控件。而上面的代码中，我们在worker thread中修改ImageView而非在UI线程中修改。由于UI控件不是线程安全的，所以将会导致不可预知的错误。

为了解决这个问题，android系统提供了几个方式，让其他线程可以访问UI线程，以下是几个可以起到这个作用的方法：

1. Activity.runOnUiThread(Runnable)
2. View.post(Runnable)
3. View.postDelayed(Runnable, long)

为了解决上面的问题，以上代码片段可以修改成使用View.post(Runnable)，

{% highlight java %}

public void onClick(View v) {  
    new Thread(new Runnable() {  
        public void run() {  
            final Bitmap bitmap = loadImageFromNetwork("http://example.com/image.png");  
            mImageView.post(new Runnable() {  
                public void run() {  
                    mImageView.setImageBitmap(bitmap);  
                }  
            });  
        }  
    }).start();  
}  
{% endhighlight %}

现在，上面的戴拿片段就是线程安全的了，网络访问是在单独的线程中执行，ImageView是在UI线程中修改的。

然后随着耗时操作的逻辑代码变得复杂，上面的这种编码风格会变得复杂，难以维护。为了处理worker thread中复杂的代码，你可以在worker thread中结合使用handler，handler可以处理从UI线程中分发过来的消息。但是，最好的方式是使用AsyncTas，当worker thread需要与UI 线程交互时，AsyncTas能大大简化代码。

### Using AsyncTask

AsyncTask的作用是能让你在UI界面上执行异步任务。它将阻塞过程安排熬worker thread中执行，并且将执行结果发布到UI线程，而且不需要你自己处理上面的thread和handler的问题。

使用AsyncTask，你需要实现一个AsyncTask的子类，并且实现其中的doInBackground() 回调方法，该方法运行在后台一个线程池中。为了更新UI，你需要实现onPostExecute(）方法，该方法接收来自doInBackground() 的结果，并且该方法是运行在UI线程中的，所有你可以放心在onPostExecute(）中更新UI界面。AsyncTask的启动是通过在UI线程中调用其execute()方法。

以下通过AsyncTask实现上面的下载图片更新ImageView的例子。

{% highlight java %}

public void onClick(View v) {  
    new DownloadImageTask().execute("http://example.com/image.png");  
}  
  
private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {  
    /** The system calls this to perform work in a worker thread and 
      * delivers it the parameters given to AsyncTask.execute() */  
    protected Bitmap doInBackground(String... urls) {  
        return loadImageFromNetwork(urls[0]);  
    }  
      
    /** The system calls this to perform work in the UI thread and delivers 
      * the result from doInBackground() */  
    protected void onPostExecute(Bitmap result) {  
        mImageView.setImageBitmap(result);  
    }  
}  
{% endhighlight %}


该代码符合线程安全，而且由于将workers thread和UI  thread中需要完成的任务分割开了，所以代码看起来简洁多了。

如果想要有一个更深入的了解，你可以阅读AsyncTask的API文档。下面先大致列出AsyncTask的要点.


1. 你可以通过泛型制定参数的类型，进度数值，task的结果值。
2. doInBackground()自动在一个worker thread中执行
3. onPreExecute(), onPostExecute(),和onProgressUpdate() 在UI线程中执行
4. doInBackground()执行的结果发送到onPostExecute()
5. 你可以在doInBackground()随时调用publishProgress()，该方法会调用onProgressUpdate()，而且onProgressUpdate()是在UI线程中执行。
6. 你可以在任何线程中任何时间取消AsyncTask。



需要注意一点，当系统在运行时发生设置变化，比如屏幕旋转，则worker thread可能会被迫重新启动。至于如何恰当地在重启前后维持线程任务，以及如何恰当地在activity被销毁时终止线程，这一点可以参考Shelves的例子

### Thread-safe methods

该知识点暂时使用不多，了解有限，以后开发用到了在细读该段落

## Interprocess Communication

IPC ，该知识点暂时使用不多，了解有限，以后开发用到了在细读该段落
