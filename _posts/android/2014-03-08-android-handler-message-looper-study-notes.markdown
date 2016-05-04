---
layout: post
title:  "Android的消息机制与线程"
date:   2014-03-08 11:00:03
categories: android
supertype: career
type: android
---

## 1 引言

一个进程通常包含一个主线程，而Android的应用是就是一个进程，里面所包含的主线程也就是UI线程，为什么叫UI线程，因为只有这条线程才能够去修改UI，其他线程去修改UI就会发生报错，在里面的代码如下：

{% highlight java %}

void checkThread() {
	if(mThread != Thread.currentThread()) {
		throw new CalledFromWrongThreadException(
		"Only the original thread that created a view hierarchy can touch its views");
	}
}

{% endhighlight %}

用source insight工具查看android源码里面的ViewRootImpl就可以发现这个方法，其中mThread就是主线程。只允许一个线程去修改UI，那么UI控件肯定就不是线程安全的了，只要非UI的多线程去访问必然报错；那为什么不加锁控制？如果那样的话，就会变得很复杂，也会降低效率，体验不好！

这样在UI线程就不能做耗时的操作，如网络，IO等等，放到后台线程去执行，当执行完毕以后又需要修改UI，那么怎么处理？Handler消息机制就是为这个而生的。

## 2 消息机制

Android进程间通信IPC机制的核心是Binder，那么Android的线程间通信的消息机制核心就是Handler了；只要我们弄懂了Handler和Looper，其他的如HandlerThread，AsyncQueryHandler，AsyncTask等等就自然明白了。

### 2.1 使用

首先实现一个内部类，继承Handler，然后重写`handleMessage`方法，父类里面这个方法是空实现，如下：

{% highlight java %}

private class MyHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        switch(msg.what) {
            case 1:
            	//do something to modify UI
                break;
            case 2:
            	//do something to modify UI
                break;
            default:
            	//do something to modify UI
                break;
        }
    }
}

{% endhighlight %}

然后把实例化这个Handler，把它传给后台线程，当后台线程执行完任务以后就可以用它发送消息给UI线程来修改UI了。

{% highlight java %}

Message message = Message.obtain();  
message.obj = data;  
message.what = 1;  
mHandler.sendMessage(message); 

{% endhighlight %}

注意到的是，Handler在UI线程实例化，然后把对象传递给后台线程，后台线程再使用这个Handler对象来发送消息，不要把这里的消息机制想象成了类似于广播的订阅模式了。

### 2.2 第二种用法

除了上面的发送消息Message用法，还可以直接发送Runnable对象的方法。我们可以实例化一个Runnable对象，这个对象包含了View的引用所以能够修改UI，同时，在UI线程直接实例化了一个Handler对象`new Handler()`，然后把这两个对象传给了后台线程，当后台线程执行完任务以后，使用这个handler来post这个Runnable对象即可，如下：

{% highlight java %}

mHandler.post(taskRunnable);

{% endhighlight %}

### 2.3 原理

为什么在别的线程通过handler就能切换到UI线程去执行呢？线程之间通信无非就是通过共享区域，也是临界区，只要把执行的内容放到临界区去即可，可以联想到经典的生产者和消费者模型，在这里，后台线程就是生产者，消息Message就是商品，我们把商品放到了阻塞队列里面去，那么就会触发UI线程就是消费者去消费，就是处理这条消息。消息队列就是一个临界区。

实际上，MessageQueue就是Google自定义实现的阻塞队列，Message是这个队列里面的一个节点，他是用单向的链表实现的，就是说Message里面有next属性。而Looper是管理消息队列的对象，Handler是用来发送消息和处理消息的对象；每个线程最多只能有一个Looper对象，Looper的构造方法是私有，只能够通过它的静态方法来实例化，并存放在ThreadLocal里面来保存线程安全。

关于ThreadLocal，可以把它理解成为像map一样的功能，key便是线程，更多可以直接看源码学习。在Looper里面，sThreadLocal是全局的静态变量。

>static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

当应用启动的时候，Android应用变化调用Looper的下面方法进行创建Looper实例：

{% highlight java %}

/**
 * Initialize the current thread as a looper, marking it as an
 * application's main looper. The main looper for your application
 * is created by the Android environment, so you should never need
 * to call this function yourself.  See also: {@link #prepare()}
 */
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

{% endhighlight %}

由注释可以知道UI线程默认就创建了一个Looper，并存放在ThreadLocal里面；Looper是一个无限循环直到quit调用而退出。要注意的是quit和quitSafely两个方法，前者直接退出，后者要等所有消息处理完毕再退出。

>有个问题，如果调用quitSafely以后还没退出完毕，然后有加入了新消息，那么怎么办？实际上每条Message都是有一个when字段代表消息处理的时间，如果它的时间大于调用quitSafely的时间则不会处理。

我们在实例化Handler的时候，当构造方法参数为空的时候就会默认获取当前线程的Looper，也可以传递一个Looper给构造方法。当发送消息的时候实际上就是把消息放进了消息队列。不管使用方法一还是方法二，实际上都是发送Message对象的，Message里面有一个callback字段名，实际上是Runnable来的，在Handler里面使用方法二的时候实际是会封装成为一个Message的。如下：

{% highlight java %}

private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}

{% endhighlight %}

可以看到Message的target属性就是Handler对象本身。因此，在Looper的loop方法实际上是拿到消息对象以后调用了target的dispatchMessage方法的。

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

        msg.target.dispatchMessage(msg);

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

而dispatchMessage实际上是调用了handleMessage方法和Runnable的run方法的。

我从网上找了下面一张图，其实这图不完整，欠缺处理消息部分。左边的Thread就是我们的工作线程，它持有UI线程的handler对象，handler对象持有UI线程的Looper对象，工作线程用handler来post一个runnable对象或者一个消息的时候，就是加入到MessageQueue消息队列，Looper就是循环消息队列，取出消息交给对应的handler来处理，执行handleMessage方法，或者执行runnable对象。这样更新UI就不会报错，因为Looper是属于UI线程内部的。一般来讲handler也属于UI线程内部的，当然也可以不是，只要在创建handler对象时候传入UI线程的Looper就可以。

![handler][handler]

### 2.4 其他

Handler.removeCallbacksAndMessages(null);  
这个方法是移除所有的callbacks和message。

## 3 Android里面的线程

在移动端来说，由于CPU处理能力没有服务器强，所以尽量在移动端少开线程，不然会占用大量资源，导致崩溃等，所以在Android要正确使用线程和尽量使用线程池。要注意的是在Android中使用Handler其实是顺序队列处理消息，所以有顺序的事情不要用到java的线程池去做，那里并发不保证顺序的，例如把一个cancel和start操作放到线程池，那么就会出现先cancel再start的情况了。

### 3.1 HandlerThread

如果我们想想UI线程那样有一个Looper，用上Android系统的Handler和Looper，那么最好就用HandlerThread了，它是继承Thread的，和普通线程不同的是，它含有一个Looper对象，看它的run方法如下：

{% highlight java %}

@Override
public void run() {
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}

{% endhighlight %}

可以看到run方法里面是Looper在循环，因此这个线程是不会销毁的，除非退出了Looper。这样结合Handler我们就可以使用了，只要在实例化Handler的时候把HandlerThread的Looper传过去就可以了。实际上，这就是一个单线程执行的队列。

### 3.2 AsyncQueryHandler

这个是结合ContentProvider使用的异步查询，适用于查询数据量大的时候，防止ANR的发生。

### 3.3 AsyncTask

AsyncTask我们应该用到的时候比较多，底层是封装Handler和Thread来实现的，而且生命周期比较短，绑定到UI线程。一般来说用于更新UI，例如下载文件更新进度的场景，它的doInBackground是在线程池中执行的，而onProgressUpdate则是在UI线程执行的。

当调用AsyncTask的execute方法的时候，实际上是调用了下面这个方法：

{% highlight java %}

@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
        Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING;

    onPreExecute();

    mWorker.mParams = params;
    exec.execute(mFuture);

    return this;
}

{% endhighlight %}

可以看出，必须在UI线程调用这方法；然后真正是调用了Executor的方法，而Executor的实现如下：

{% highlight java %}

private static class SerialExecutor implements Executor {
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    scheduleNext();
                }
            }
        });
        if (mActive == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}

{% endhighlight %}

当mActive为null的时候就会从mTasks取一个出来执行，执行完毕以后会再取一个执行，实际执行的线程在于THREAD_POOL_EXECUTOR，它就是一个普通的ThreadPoolExecutor。虽然SerialExecutor是一个池，却是串行队列而已。关于FutureTask和Callable不在这里说了，实际上，最终会调用到callable的call方法，call方法会调用到doInBackground方法的。最终会再通过一个Handler来调用到ui线程的finish或者onProgressUpdate方法。

实际上，THREAD_POOL_EXECUTOR的核心线程数是cpu数量+1，最大线程数是cpu数量*2+1，阻塞队列sPoolWorkQueue是128，KEEP_ALIVE这个活跃数却是1；这个线程池只有一个线程运行。另外注意到，这些变量都是static的，进程里面就只有一份，不管new多少个AsyncTask，实际上用的线程池和Handler都是同一个，本质上整个应用就只有一个AsyncTask，而且不能并发执行任务。如果要并发执行，只能自己在外面自定义线程池，然后调用executeOnExecutor方法把线程池传进去。

### 3.4 IntentService

由于Service依然是运行在主线程的，当然这个主线程可以是UI线程也可以是别的进程的主线程，所以耗时的计算仍然需要在别的线程去执行。我们很容易想到的是使用HandlerThread，这个Google已经帮我们做了，叫做IntentService。它继承Service，是一个抽象类，看它的onCreate方法如下：

{% highlight java %}

@Override
public void onCreate() {
    // TODO: It would be nice to have an option to hold a partial wakelock
    // during processing, and to have a static startService(Context, Intent)
    // method that would launch the service & hand off a wakelock.

    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}

{% endhighlight %}

很容易看出它的实现，再看ServiceHandler的实现：

{% highlight java %}

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

{% endhighlight %}

onHandleIntent方法需要我们具体去实现，stopSelf调用的是ActivityManager的stopServiceToken方法，它会等待所有的消息都处理完毕以后才会终止服务的，至于使用，就像普通的服务使用一样调用startService即可。

IntentService的onStart方法会被调用多次，实际上是发送消息给HandlerThread来处理：

{% highlight java %}

@Override
public void onStart(Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}

{% endhighlight %}

因此，IntentService也是串行执行任务的。

[handler]: /image/handler.png