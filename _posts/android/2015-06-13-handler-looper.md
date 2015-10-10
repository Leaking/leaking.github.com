---
layout: post
title: "详解Android中的Handler和Looper，Message"
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

学习Android时，在请求网络数据或者进行其他耗时操作时，大家都懂得使用`Handler`结合`Thread`去进行耗时操作，并将获得的结果呈现到UI上。而其中的原理就涉及`Looper`，`Handler`，`Message`，`MessageQuene`的关系，其实对于这个知识点，基本从学android开始,每个人就看到来自四面八方的资料，我自己之前也写过博客记录其中的原理，但是依然不够详细，连自己回头看都觉得没法完全看懂。今天决定再来一发，努力把这个知识点讲得简单，详细。



## Looper是什么

首先我们得先了解`Looper`是什么，普通的线程`Thread`一般在执行完当前任务之后就停止执行其他操作了。如果有这样的场景：有断断续续的任务来临（可能没隔10秒，20秒来一个新任务），这些任务都是交付给某个线程去执行，那么普通的线程就无法满足这样的需求。而`Looper`的作用此时就体现了，它可以使你的线程满足这样的场景，`Looper`可以使普通的线程获得轮询的功能，它执行完一个任务后，将轮询是否有新任务来临，如果有新任务，则去处理新的任务，任务执行完后，继续去轮询新的任务。以上就是Looper存在的作用。

这里先大致列出一个简单的例子，也是在网上到处可见的例子，这个例子其实就来自于`Looper`的源码注释里。如下：


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


如上，调用`Looper.prepare();`和`Looper.loop();`之后，当前线程才能变成轮询线程了，我们可以通过`mHandler`的引用，调用其`sendMessage()`方法就可以不断往该线程发送任务。另外，如果你是在其他线程中持有上面的`mHandler`，在其他线程中往当前线程发送任务，那么你就能实现跨线程通信了。有没有觉得这个场景很熟悉，没错，就是平时我们从执行网络操作的工作线程中，往UI线程中发送消息的场景。

这里只是举个最简单的例子，不过已经涵盖了最重要的思想。但是看到这里还是可能看不懂的，放心，后面会讲解Looper的源码。





## Looper解析

`Looper`，`Handler`，`Message`，`MessageQuene`这四个类代码层面的联系，其实有点复杂，因为相互之间有彼此的引用。

先来打开`Looper`的源码看看，可以看到它有如下成员变量


{% highlight java %}


// sThreadLocal.get() will return null unless you've called prepare().
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
private static Looper sMainLooper;  // guarded by Looper.class

final MessageQueue mQueue;
final Thread mThread;

private Printer mLogging;

{% endhighlight %}

这里需要注意的变量有三个`sThreadLocal`,`sMainLooper`,`mQueue`。

+ sThreadLocal：借助`ThreadLocal`类使得每个线程都只对应一个Looper对象（除了UI线程的Looper）。ThreadLocal类的原理这里不展开，大家只需要知道它使得多个线程可以持有同个对象相互独立的引用即可。

+ mQueue：任务队列，`Looper`从这个任务队列中轮询取出任务

+ sMainLooper：UI线程的`Looper`。

接下来看看两个比较重要的方法`prepare()`和`loop()`，首先是`prepare()`方法，如下：

{% highlight java %}


/** Initialize the current thread as a looper.
  * This gives you a chance to create handlers that then reference
  * this looper, before actually starting the loop. Be sure to call
  * {@link #loop()} after calling this method, and end it by calling
  * {@link #quit()}.
  */
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}

{% endhighlight %}


这个方法比较简单，主要是新建一个`Looper`对象，存放在`ThreadLocal`变量中，并初始化消息队列`MessageQueue`.

接下来再看看`loop()`方法


{% highlight java %}


/**
 * Run the message queue in this thread. Be sure to call
 * {@link #quit()} to end the loop.
 */
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

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

        msg.target.dispatchMessage(msg);//查看Message的源码可以看到，它持有一个handler引用，
        //Handler对象的dispatchMessage方法可以将消息最终传递到我们经常看到的handlerMessage的回调方法中。

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

/**
 * Return the Looper object associated with the current thread.  Returns
 * null if the calling thread is not associated with a Looper.
 */
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}

{% endhighlight %}


这个方法主要是获取保存在`ThreadLocal`中的`Looper`，然后获取`Looper`对象中的消息队列`queue`,并在一个`for`循环中轮询消息队列，拿出消息`Message`并进行处理，处理消息的关键代码是这一句`msg.target.dispatchMessage(msg);`，消息对象`Message`中持有一个`Handler`引用，而`Handler`的`dispatchMessage(Message m)`方法将消息最终传递到我们平时经常重写的`handleMessage(Message m)`方法。接下来讲解`Message`，`Handler`的源码。

## Handler原理


首先来看看`Handler`关键的成员变量


{% highlight java %}

final MessageQueue mQueue;
final Looper mLooper;
final Callback mCallback;
final boolean mAsynchronous;
IMessenger mMessenger;

{% endhighlight %}

一个`Handler`只能对应当前线程，也只能对应一个`MessageQueue`，`Looper`。稍微做个总结，一个线程只能有一个`MessageQueue`，一个`Looper`，但是却可以有多个`Handler`。如下图

<figure>
  <img src="{{ site.url }}/images/handler.png" alt="search screenshot">
  <figcaption>Handler,Looper关系图</figcaption>
</figure>


 我们接下来进一步探索`Handler`的源码了，看看究竟是怎么发送和处理消息的。

 首先来看是在哪里处理消息的。从上面将`Looper`的源码时，我们发现，处理消息是在`Handler`的`dispatchMessage`方法中进行，该方法的源码如下：


{% highlight java %}


public void dispatchMessage(Message msg) {
    if (msg.callback != null) { //1，Message对象中有一个callback对象，它是一个Runnable类型
        handleCallback(msg);
    } else {
        if (mCallback != null) { //2，Handler中可以有一个处理消息的回调接口
            if (mCallback.handleMessage(msg)) { //
                return;
            }
        }
        handleMessage(msg);  //3，如果1，2中都为空，则将消息丢给我们经常使用的handlerMessage()方法
    }
}

private static void handleCallback(Message message) {
        message.callback.run();
}

//Handler处理消息的回调接口
public interface Callback {   
        public boolean handleMessage(Message msg);
}

//这个局部变量可以通过Handler构造函数传递进来
final Callback mCallback;


{% endhighlight %}


从上面可以看出，处理消息的地方可以有三种可能

 + Message对象自带的回调接口
 + Handler自带的回调借口
 + Handler的handlerMessage()方法


看完处理消息的地方，再来看看`handler`的几个发送消息的API


 + post(Runnable)
 + postAtTime(Runnable, long)
 + postDelayed(Runnable, long)
 + sendEmptyMessage(int), sendMessage(Message)
 + sendMessageAtTime(Message, long)
 + sendMessageDelayed(Message, long)

 其实上面6个方法，分为`post`和`sendMessage`两种，每种都有三个发送消息的方式，不难发现一一对应。那么，为什么`post`发送的是一个`Runnable`对象，而`sendMessage`发送的是一个`Message`对象。

{% highlight java %}

 public final boolean post(Runnable r){
   return  sendMessageDelayed(getPostMessage(r), 0);
 }


public final boolean postDelayed(Runnable r, long delayMillis){
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}


public final boolean postAtTime(Runnable r, long uptimeMillis){
    return sendMessageAtTime(getPostMessage(r), uptimeMillis);
}


public final boolean sendMessage(Message msg){
    return sendMessageDelayed(msg, 0);
}


public final boolean sendMessageDelayed(Message msg, long delayMillis) {
   if (delayMillis < 0) {
       delayMillis = 0;
   }
   return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}


public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}

{% endhighlight %}

不难发现，以上六个方法，其实最终都是调用`sendMessageAtTime()`。

而其中`post`方式的三个方法中都调用了`getPostMessage()`构造一个`Message`，该方法的源码如下：

{% highlight java %}

private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

{% endhighlight %}

该方法构造一个`Message`并赋值一个处理消息的回调接口，所以不难发现，`post`方式发送的消息，都是在方法传递进来的`Runnable`中进行的。此处`Runnable`仅仅是接口的概念，无关线程。

理解了上面6个发送消息的方法后，看`Handler`的其他发送消息的重载方法就很容易懂了。


## Message原理


首先来看看Message主要的成员变量


{% highlight java %}


public int what; // 用于区别不同Message
public int arg1; // arg1和arg2用于传递简单的整型变量
public int arg2; // arg1和arg2用于传递简单的整型变量
public Object obj; // 用于传递复杂数据
/*package*/ Bundle data; // 可以用于传递复杂数据，需要借助getter和setter方法
/*package*/ Handler target; // 指向发送/处理当前Message的Handler
/*package*/ Runnable callback; // 当使用post方式发送消息时，该参数将被赋值
  
                     
{% endhighlight %}


`Message`类还提供了一系列`obtain()`的重载方法，这一系列方法都是通过在一个`Message`内存池中获取一个`Message`对象，减少内存开销，所以推荐使用该方法，而非每次都适用`new Message()`的方式。


## UI线程其实就是一个轮询线程


大家应该熟悉`Handler`结合`Thread`的用法，并且都是在UI线程按照大致如下的方式建立`Handler`对象，


{% highlight java %}


this.handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case LOAD_SUCCESS:
                //处理成功的情况
                break;
            case LOAD_FAIL:
                //处理失败的情况
                break;
        }

    }
};

{% endhighlight %}



通过上面新建的`Handler`，然后在某个执行耗时操作的线程中，借助`handler`这个引用发送消息给`handleMessage`方法处理。不难发现，UI线程中的`handler`可以接收并处理随时到来的任务，所以其实UI线程就是一个轮询线程了，但是我们前面讲过，调用`Looper.prepare();`和`Looper.loop();`之后，当前线程才能变成轮询线程，但是我们这里并没有看到在UI线程中调用这两个方法啊。不急，我们打开`Handler`的源码看看，如下：


{% highlight java %}

 
  public Handler() {
      this(null, false);
  }

  /**此处还有若干构造方法，此处略去**/

  public Handler(Callback callback, boolean async) {
      if (FIND_POTENTIAL_LEAKS) {
          final Class<? extends Handler> klass = getClass();
          if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                  (klass.getModifiers() & Modifier.STATIC) == 0) {
              Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                  klass.getCanonicalName());
          }
      }

      mLooper = Looper.myLooper();   // 在此处获取Looper对象
      if (mLooper == null) {
          throw new RuntimeException(
              "Can't create handler inside thread that has not called Looper.prepare()");
      }
      mQueue = mLooper.mQueue;
      mCallback = callback;
      mAsynchronous = async;
  }

{% endhighlight %}


我们调用`new Handler`之后，最终会进入`Looper。myLooper`方法，打开其源码：

{% highlight java %}

/**
     * Return the Looper object associated with the current thread.  Returns
     * null if the calling thread is not associated with a Looper.
     */
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }

{% endhighlight %}

从这段代码可以看出，这里并没有去新建并初始化一个`Looper`，紧接着，在`Looper`的源码中又看到一个段代码：

{% highlight java %}

/**
     * Initialize the current thread as a looper, marking it as an
     * application's main looper. The main looper for your application
     * is created by the Android environment, so you should never need
     * to call this function yourself.  See also: {@link #prepare()}
     */
    public static void prepareMainLooper() {
        prepare(false);//此处调用了prepare方法
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

    /**
     * Returns the application's main looper, which lives in the main thread of the application.
     */
    public static Looper getMainLooper() {
        synchronized (Looper.class) {
            return sMainLooper;
        }
    }

{% endhighlight %}


原来真相是这样的，子啊新建`Handler`之前，UI线程已经为你准备好了一个`Looper`了，但是还有一个疑问，上面的代码可以看出，UI线程貌似只调用了`Looper.prepare();`方法，还没调用`Looper.loop();`方法啊，那么后面这个方法，UI线程是在哪里调用的呢？这个只能上网查查资料了，最终发现是在一个叫`ActivityThraad`中调用到，包括上面的`prepareMainLooper`,如下：


{% highlight java %}


public static void main(String[] args) {
        //此处省去其他无关代码

        Looper.prepareMainLooper(); //prepare()方法在这里被调用

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();  //loop()方法在这里被调用，终于找到你了

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }

{% endhighlight %}

到这里基本就理清了为什么UI线程是轮询线程，也是因为UI线程已经默认调用了`Looper.prepare();`和`Looper.loop();`，我们平时才能那么方便的使用`Handler`结合`Thread`去请求网络数据并呈现到UI上。




## 小技巧


某个方法被调用时，我们不知道它是被UI线程调用还是其他子线程调用，我们可以这样


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




再来看看Volley中如何将网络请求结果传回主线程。最重要的一步代码如下，在建立请求队列时，传入了一个ExcutorDelivery对象，该对象中又包含一个Handler对象，这个Handler绑定于主线程，没错，最终就是靠这个传入的handler将网络请求结果回调给主线程。

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















