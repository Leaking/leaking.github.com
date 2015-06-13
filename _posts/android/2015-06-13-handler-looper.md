---
layout: post
title: "浅谈Android中Handler，Looper机制"
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

这么寂寞的博客，我想我接下来书写时，得用做笔记，或者写论文的态度写，而非给别人看的态度，毕竟，根本木有人来看啊，有木有！T,T


## Volley中Handler的应用

这两天又看了点Volley的源码，想到一个问题，Volley将网络请求放到一个异步的请求队列中，那么返回结果是如何从子线程中返回到主线程来的呢？以平时接触的代码，基本能确定是和handler的机制有关，细思极恐，貌似到现在也还没认真的探索过handler的机制，翻看了一下别人的文章，其实挺难理解，最后是在看Looper，Handler，Message几个类的源码之后才茅塞顿开。

先来看看Volley中如何将网络请求结果传回主线程。最重要的一步代码如下，在建立请求队列时，传入了一个ExcutorDelivery对象，该对象中又包含一个Handler对象，这个Handler绑定于主线程，没错，最终就是靠这个传入的handler将网络请求结果回调给主线程。

{% highlight java %}

    /**
     * Creates the worker pool. Processing will not begin until {@link #start()} is called.
     *
     * @param cache A Cache to use for persisting responses to disk
     * @param network A Network interface for performing HTTP requests
     * @param threadPoolSize Number of network dispatcher threads to create
     */
    public RequestQueue(Cache cache, Network network, int threadPoolSize) {
        this(cache, network, threadPoolSize,
                new ExecutorDelivery(new Handler(Looper.getMainLooper())));
    }

{% endhighlight %}


## Looper，Handler，Message的关系

网上已经有很多文章讲解这三者的关系了，最开始一直看不懂，最重要的一点是Looper存在的理由，其他文章都说，一个普通的Thread执行完一个任务之后，Thread就结束了，但是，使用Looper能使其变成一个一直在轮询检查新任务的线程，一个任务一个任务地去执信。这种线程，为什么要存在呢？其实，我们的UI线程就是这样的，它一直在轮询新的消息。而你确实也可以使用Looper建立一个轮询的线程,如下，这段代码到处文章都引用到了。

{% highlight java %}

class LooperThread extends Thread {
      public Handler mHandler;

      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };

          Looper.loop();
      }
  }

{% endhighlight %}


Looper这个类中的方法基本都是static的，而且其中使用了ThreadLocal，这个类的作用大致是：让每个异步线程，对某个对象拥有单独一个备份，正是通过如此，使得每个Thread调用Looper.prepare()之后，回使每个线程绑定一个Looper，prepare方法就不贴出来了，来贴一下loop方法

{% highlight java %}


public static void loop() {
        final Looper me = myLooper(); //借助ThreadLocal,获取当前线程的Looper
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue; // 获取消息队列,后面会讲到，handler持有looper引用，从而可以添加消息到looper中的消息队列中


        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();
        // 上面两句不懂，但是无碍


        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);  //查看Message的源码可以看到，它持有一个handler引用，
            //dispatchMessage方法可以将消息最终传递到我们经常看到的handlerMessage的回调方法中。

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }

{% endhighlight %}



从上面的代码和注释，我们注意到，Looper，Handler，Message，MessageQueue基本都持有相互之间的引用，正是这种错综复杂的关系，才会使得理解起来最开始我懵了，打开源代码就明朗了很多。

首先，Handler类中，持有一个Looper对象，它就是当前线程的Looper，而Looper中有MessageQueue，handler就是借助Looper引用从而获得MessageQueue的引用，从而可以往消息队列中添加消息，可以参考下面的代码，另外，不难理解，一个线程只可以有一个looper，但是可以有多个handler。


{% highlight java %}

 public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue; // 获得消息队列的引用
        mCallback = callback;
        mAsynchronous = async;
    }

{% endhighlight %}


## 小技巧


某个方法呗调用时，我们不知道它是悲UI线程调用还是其他子线程调用，我们可以这样


 {% highlight java %}

 if (Looper.myLooper() != Looper.ge tMainLooper()) {
                // If we finish marking off of the main thread, we need to
                // actually do it on the main thread to ensure correct ordering.
                Handler mainThread = new Handler(Looper.getMainLooper());
                mainThread.post(new Runnable() {
                    @Override
                    public void run() {
                        mEventLog.add(tag, threadId);
                        mEventLog.finish(this.toString());
                    }
                });
                return;
  }

{% endhighlight %}

## 以上！












