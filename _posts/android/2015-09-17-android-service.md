---
layout: post
title: "【译】Service"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
comments: true
share: true
---

本文作为阅读官方文档笔记/翻译，原文地址

[http://developer.android.com/guide/components/services.html](http://developer.android.com/guide/components/services.html)

​Service是四大组件（component）之一，它可以负责在后台执行耗时的任务。Service还可以实现跨进程通信（IPC）。Service按照启动方式，可以分为两种形式，分别是`started`以及`bound`。这篇篇文章主要介绍前者

## Started

一个service可以被其他应用组件（比如activity）通过`started`方式启动，这种方式通过调用应用组件的`startService()`方法。一旦service被启动，它将一直运行，即使启动这个service的组件被销毁了，甚至整个app退出了，这种情况，service往往是被销毁了但是会重新生成。通过“started”方式启动的service，往往只是执行一种操作，而与调用者没有更多的交互通信，比如可以在这种service中执行上传，下载的任务，任务完成后，service需要被终止或者自我终止。

## Bound

一个service可以被其他应用组件（比如activity）通过`bound`的方式启动，这种方式通过调用应用组件的`bindService()`方法，这种方式的启动service提供一种`client-server`形式的接口，使service和启动service的组件可以进行通信，另外还可以进行跨进程调用（IPC）。通过`bound`方式启动的service，只有在被绑定的状态下才会运行。另外，一个service可以被多个组件绑定，当所有组件解除绑定后，service也将会被销毁。


# The Basics

启动service，你需要创建一个继承自service的类，或者继承现有service实现类，比如IntentService。

service可以通过以上`started`或`bound`两种方式启动，也可以同时使用以上两种方式。然后重写service部分重要的回调方法。最重要的回调方法有如下几个

* onStartCommand():
当通过startService()的方式启动service时，这个方法就将被调用，并且service将被一直执行下去，任务完成后，你最好调用stopSelf()或者stopService()停止该service。通过bound方式启动service时不需要重写这个方法。

* onBind():
通过bindService()方式启动service，你需要在onBind()返回一个IBinder实例，如果不想绑定service，那么这个方法返回null即可。

* onCreate():
这个方法会在service被首次创建时执行，它在onStartCommand()以及onBind()之前执行，当service是被重启的，或者已经被启动过，那么方法将不会被执行。这个方法适合执行一些初始化的事件。

* onDestroy():
当service被销毁，你应该在这里回收资源，比如thread，监听器，广播接受者等等。

service在系统资源很紧张的情况下会被回收，但是如果service绑定着一个处于前台的`activity`，那么它被回收的概率会降低，如果一个service被声明为`foreground`的，那么被回收的概率基本为0.

通过`startService`方式启动的`service`，虽然可能会被回收，但是也会被重启，所以我们应该处理这个重启的流程，另外，service的重启和`onStartCommand()`的返回值有紧密的关系，这一点后面会提及。

# 在manifest中声明service

声明方式如下

{% highlight xml %}

<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>

{% endhighlight %}


关于声明service时的`android:enabled="true"`和`android:exported="true"`两个属性，这里不描述。接下来讲述启动`service`。



# Creating a Started Service

一个`started`方式启动的`service`，通过调用`startService()`，然后会执行`onStartCommand()`方法。调用`startService`时可以传入一个`Intent`，这个`Intent`会被传递到`onStartCommand()`方法，所以可以借助这个`Intent`传递数据给`service`。

`started`方式启动的`service`会一直在后台运行，所以当`service`的任务执行完之后，需要处理好销毁回收工作，它可以通过调用`stopSelf()`自我关闭，也可以是其他组件调用`stopService()`来终止`service`。

这里将的'service'会在后台一直运行，是指启动`service`的组件被销毁了甚至整个app退出了，`service`可以自动重启，


> `service`默认情况下，会在声明的`application`的进程的`main thread`上运行。所以如果在`service`
> 中执行阻塞性的任务，需要启动一个线程。



建立`service`时，你可以继承自`Service`和`IntentService`

* Service： `service`如果继承自这个类，那么执行阻塞任务时，需要另外启动线程。

* IntentService：这个类是对`service`类的封装，它处理了启动线程的流程，你只需要将你的阻塞任务放在` onHandleIntent()`方法中即可。这个类可以处理同时发起的多个请求，并在所有请求处理完之后自动销毁`service`，然后如果继续发起请求，则将再次启动`service`，这里说的发起请求，就是`startService()`方法启动`service`。


其实IntentService的源码很短，可以看一下


{% highlight java %}

public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler; 
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    /**
     * Creates an IntentService.  Invoked by your subclass's constructor.
     *
     * @param name Used to name the worker thread, important only for debugging.
     */
    public IntentService(String name) {
        super();
        mName = name;
    }

    /**
     * Sets intent redelivery preferences.  Usually called from the constructor
     * with your preferred semantics.
     *
     * <p>If enabled is true,
     * {@link #onStartCommand(Intent, int, int)} will return
     * {@link Service#START_REDELIVER_INTENT}, so if this process dies before
     * {@link #onHandleIntent(Intent)} returns, the process will be restarted
     * and the intent redelivered.  If multiple Intents have been sent, only
     * the most recent one is guaranteed to be redelivered.
     *
     * <p>If enabled is false (the default),
     * {@link #onStartCommand(Intent, int, int)} will return
     * {@link Service#START_NOT_STICKY}, and if the process dies, the Intent
     * dies along with it.
     */
    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        // HandlerThread使得一个普通的Thread变成一个looper线程，然后在其绑定的handler中可以不断处理任务
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    /**
     * You should not override this method for your IntentService. Instead,
     * override {@link #onHandleIntent}, which the system calls when the IntentService
     * receives a start request.
     * @see android.app.Service#onStartCommand
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

    /**
     * Unless you provide binding for your service, you don't need to implement this
     * method, because the default implementation returns null. 
     * @see android.app.Service#onBind
     */
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    /**
     * This method is invoked on the worker thread with a request to process.
     * Only one Intent is processed at a time, but the processing happens on a
     * worker thread that runs independently from other application logic.
     * So, if this code takes a long time, it will hold up other requests to
     * the same IntentService, but it will not hold up anything else.
     * When all requests have been handled, the IntentService stops itself,
     * so you should not call {@link #stopSelf}.
     *
     * @param intent The value passed to {@link
     *               android.content.Context#startService(Intent)}.
     */
    protected abstract void onHandleIntent(Intent intent);
}

{% endhighlight %}


从以上代码可以看出IntnetService主要是借助HandlerThread，将线程编程可以不断处理任务的looper线程。然后你可以不断往IntentService发送请求，知道所有请求处理完毕，IntentService才会被销毁。

在以上代码的`Handler`中，我们可以看到处理任务的最后一行代码`stopSelf(msg.arg1);`，这里`msg.arg1`就是`onStartCommand`中的`startid`,当新的请求来临，这个`startid`会被更新，只有'msg.arg1'是最新的`startid`时，才会销毁`service`，所以避免了有新请求没被处理时，`service`就被销毁，同时，当所有请求被处理完后，`service`就被销毁.IntentService的微妙之处就在这里。


如果你启动一个`service`，而且执行的任务不需要同时启动多个线程一起执行，那么使用I`ntentService`是最方便的。同理，如果你需要在`service`中启动多个线程同时执行任务，那么就只能使用`Service`类了。这时你要做的事情就放在`Service`类中的`onStartCommand()`方法。


 `onStartCommand()`方法的返回值觉得了'service'被销毁后如何重建，它的返回值及对应功能如下：

 ＊ START_NOT_STICKY：不重建`service`。

 ＊ START_STICKY：重建`service`，但是不保存`Intent`，调用`onStartCommand()`。

 ＊ START_REDELIVER_INTENT：重建'`service`，保存`Intent`，调用`onStartCommand()`。

 以上的解析是不考虑有pending intents的情况，


# Starting a Service

通过'started'方式启动service很简单，如下


{% highlight java %}

Intent intent = new Intent(this, HelloService.class);
startService(intent);

{% endhighlight %}

传入数据可以借用`intent`，返回结果可以借用广播的机制。


# Running a Service in the Foreground

这种方式启动service，将会弹出一个不能关闭的通知，这种需求经常见于音乐播放器，你可以借助这个通知，在上面做一些操作，比如音乐播放器的下一首，暂停等等。

为了实现这个功能，可以在onStartCommand()中执行如下代码

{% highlight java %}

Notification notification = new Notification(R.drawable.icon, getText(R.string.ticker_text),
        System.currentTimeMillis());
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
notification.setLatestEventInfo(this, getText(R.string.notification_title),
        getText(R.string.notification_message), pendingIntent);
startForeground(ONGOING_NOTIFICATION_ID, notification);

{% endhighlight %}

# Managing the Lifecycle of a Service



<figure>
  <img src="{{ site.url }}/images/service_lifecycle.png" alt="search screenshot">
  <figcaption>Service生命周期</figcaption>
</figure>






