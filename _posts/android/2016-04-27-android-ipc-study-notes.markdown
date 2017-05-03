---
layout: post
title:  "Android的进程间通信IPC机制"
date:   2016-04-27 11:00:03
categories: android
supertype: career
type: android
---

## 1 引言

### 1.1 什么是进程

在大学学习计算机操作系统的时候，我们知道系统资源申请和调度执行是分开的，进程是资源申请和拥有单位，而线程是调度执行的基本单位；进程包括了程序、数据集合和进程控制块三个部分。IPC的意思Inter-Process Commnunication，进程间通信，Linux里面的IPC方式有管道，信号量，共享内存等等，而Android是基于Linux的，那么我们在上层应用进程通信的方式是有所区别的，我猜测本质上和底层的Unix是一样的，不过是包装不同而已。

### 1.2 关于序列化

当跨进程通信传输对象的时候，肯定是需要对对象进行序列化的，不管是网络传输还是持久化到存储设备，都是需要序列化的；在Java里面支持Serializable接口，使用很简单的，只需要实现这个接口，并生成serialVersionUID即可，显然，这个UID是用来标识对象的，当反序列化的时候，会根据这UID来比较，如果相等就会成功。

关于序列化的实现，Java里面用的是ObjectInputStream和ObjectOutputStream来对一个对象进行反射的操作，然后把属性写到文件。要注意的是static和transient的关键字不属于对象，是不对参与序列化过程的。具体的实现，我在大学的时候曾经写过一个序列化工具，可以参考。

#### 1.2.1 Android的Parcelable接口

Java的Serializable接口是比较通用的，但是它的效率不会很高，一是使用了反射，二是主要它进行的都是IO操作，于是Android提供了自己的接口，它主要是在内存上进行序列化的，并且序列化的属性都是用户指定的，所以它的效率相对来说会高一些。看下面的例子即可：

{% highlight java %}

public class TestObject implements Parcelable {
    public int id;
    public String name;
    public TestObject(int id, String name) {
        this.id = id;
        this.name = name;
    }
    @Override
    public int describeContents() {
        return 0;
    }
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(id);
        dest.writeString(name);
    }
    public static final Parcelable.Creator<TestObject> CREATOR = new Parcelable.Creator<TestObject>() {
        @Override
        public TestObject createFromParcel(Parcel source) {
            TestObject to = new TestObject(source.readInt(), source.readString());
            return to;
        }
        @Override
        public TestObject[] newArray(int size) {
            return new TestObject[size];
        }
    };
}

{% endhighlight %}

## 2 Android中的多进程模式

在Android里面，启动应用的时候回默认启动一个进程，进程的名字就是应用的包名；我们也知道Android是以组件的形式来划分的，四大组件是必须依附在进程上运行的，当第一次启动的时候需要创建进程，会比较慢，但是当下次启动组建的时候，如果进程还在那速度就会很快了；组件没有进程就运行不了，进程没有任何组建也会有被回收的风险。

使用多进程的正常方式只有一种，那就是在`AndroidManifest.xml`里面配置四大组建的时候，加一个参数：

>android:process=":processname"

这样就直接指定这个组建在另外一个进程运行了，这个进程的名字是“packagename:processname”，这种命名方式说明这是一个私有进程，其他应用的组件不能跟它跑在同一个进程了；如果不用“冒号”这种命名方式，直接是字符串，那么就不是私有进程了。

另外一种创建多进程的方式就是利用JNI技术，在C底层调用系统的`fork()`这个系统接口。

Android为每个应用/进程都分配了一个独立的虚拟机，因此他们的地址空间是不一样的了；而且每个进程都有自己的Application了，这时候我们会发现Application调用onCreate了两次，但是属于不同的进程，只不过一个是有UI线程的。这时候内存共享不行了，如static不是全局的了，使用不了单例模式了，更别说线程同步问题了。SharedPreference也用不了了，因为它是基于XML和内存缓存实现的，并发的会出现问题的，数据不可靠。

## 3 Android里面的IPC

当使用多线程的时候，我们都需要处理不少事情，那使用多进程更是一个复杂的事情，而在Android里面进程间通信有以下几种方式。

### 3.1 文件共享

这是想到最直接简单的方式了，既然共享不了内存，那就共享文件。进程A把数据序列化写到文件，然后进程B就可以读取这些数据和反序列化这些对象了；但是由于Linux里面是没有文件锁的，如果高并发肯定会出现问题！当然，我们是有很多方式可以解决问题，但是没有必要去做，因为Android提供了更好的方式来使用。

### 3.2 Socket

网络也是比较容易想到的方式了，通常我们使用Socket进行的是跨主机跨进程通常，那么本机跨进程就可以了。这就是Java原生的方法，跟Android没有多大关系，大学时期也写过很多Socket的项目，比较熟悉了。

### 3.3 Intent/Bundle

学习Android的时候，知道启动一个组件是需要用到Intent的，而Intent中是带了Bundle数据的，Bundle是可以带基本类型和支持序列化的对象的。在单进程的情景下可以使用Bundle传递数据，那么跨进程的时候也是支持的，Android系统的Framework底层是支持实现了的。例如，从一个进程的Activity启动另外一个进程的Activity的时候，Intent就可以带Bundle数据过去，并且在Result回来的时候也是可以带Bundle数据了。

因此，这种方式是适用于Android原生的四大组件间通信的。

### 3.4 ContentProvider

这是Android的四大组件之一，内容提供者，它的作用是为别人提供内容，那必然是支持跨进程的，具体使用方法就不举例子了。它底层的实现基于Binder的。

### 3.5 Messenger

注意这个单词不是`Messager`！但是它的确和消息相关，里面结合[Handler](http://zhgeaits.me/android/2014/03/08/android-handler-message-looper-study-notes.html)来使用，Android在底层是封装了AIDL实现的，所以它是比较常用和方便的一种IPC方式。

跨进程通信是C/S架构的模式，即客户端与服务端之间的通信，Android里面充当服务端角色的是Service组件，通常我们是启动一个进程来运行Service组件，客户端绑定Service组件，它们之间用Messenger来通信。

在AndroidManifest文件配置Service为多进程，然后服务端Service的实现：

{% highlight java %}

public class TestMessengerService extends Service {
    private static class MessengerHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            String message = msg.getData().getString("msg");
            Log.i("TestMessengerService", message);

            Messenger client = msg.replyTo;
            Message replyMessage = Message.obtain();
            Bundle replyBundle = new Bundle();
            replyBundle.putString("replymsg", "hahaha");
            replyMessage.setData(replyBundle);
            try {
                client.send(replyMessage);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    }
    private final Messenger mMessenger = new Messenger(new MessengerHandler());
    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }
}

{% endhighlight %}

可以看出是Messenger是封装了Binder的，然后需要传递一个Handler进去来处理消息。实际上Handler没参与跨进程，只是实现了消息的转发。当收到客户端的消息以后，同时拿出客户端的Messenger进行回复消息。

客户端bindService()以后，实际上拿到的是上面服务端返回的Binder对象，然后构造一个Messenger，实现如下：

{% highlight java %}

private Messenger mService;
private Messenger mClientMessenger = new Messenger(new MessageHandler());
private static class MessageHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        String message = msg.getData().getString("replymsg");
        Log.i("MainActivity", message);
    }
}
private ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mService = new Messenger(service);
        Message msg = Message.obtain();
        Bundle bundle = new Bundle();
        bundle.putString("msg", "test");
        msg.setData(bundle);
        msg.replyTo = mClientMessenger;
        try {
            mService.send(msg);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
    }
};

{% endhighlight %}

当与Service连接成功以后，通过Binder对象构造一个Messenger，然后就可以用来发送消息了，同时也要创建一个Messenger对象设置到replyTo属性，这样服务就能拿到来回复消息了。理解到Messenger是单向通信的，如果要双向发消息必须需要两个Messenger，每一个Messenger都创建了Binder对象。

另外也理解到Handler没有用来跨进程，因此传递数据最好不要用到Message里面的属性，特别是obj这个属性，就算设置的Parcelable对象也没法传输，所以我们应该把数据放置在Bundle里面。

### 3.6 AIDL

最后这个是相对比较底层和复杂的使用了，通常用它来开发RPC的功能。AIDL的意思是Android Interface Description Language，一种简单的Android接口描述语言。简单来说，我们就是用它来定义跨进程方法调用的接口。

#### 3.6.1 关于Binder

AIDL的底层实现也是Binder，所以说，Binder是Android里面IPC的核心，关于它的底层实现是什么需要深入研究源码才能够明白。目前我只是理解了它的使用原理；在Android里面，它是一个类，可以理解它为一个通道，当客户端连接服务器的时候，服务器就创建了一个Binder对象，这个Binder对象包含了服务器的业务调用，然后把这个Binder对象返回给客户端，客户端就可以使用这个Binder对象进行RPC的调用了，可以理解到这个RPC调用是会挂起线程的。同理，如果客户端也创建了一个Binder对象，同样把这个对象传给了服务端，那么服务端也能回调到客户端的方法了。

也就是说，服务端为客户端打开了一个通道，客户端也为服务端打开了一个通道，这些通道都是单向的调用关系。一个通道其实就是一个线程执行，当服务端的Binder业务方法被调用的时候，它是在另外一条线程执行，同样，当客户端的Binder方法被回调的时候，也是在一条Binder线程里面执行的；另外，通道是可以打开多个的，就是说，服务端可以为不同的业务创建不同的Binder对象，让它们各自进行不同的RPC调用，其实就是不同的线程在运行。

#### 3.6.2 使用AIDL

对于需要传输的序列化对象，也是需要一个aidl文件对象的，如上面的TestObject对象，想要的`TestObject.aidl`文件如下：

{% highlight java %}

package org.zhangge.test;

parcelable TestObject;

{% endhighlight %}

然后定义一个业务的接口`TestServiceCore.aidl`，IDE是支持新建的，内容如下：

{% highlight java %}

package org.zhangge.test;

import org.zhangge.test.TestObject;

interface TestServiceCore {
    List<TestObject> getList();
    void send(in TestObject obj);
}

{% endhighlight %}

注意到`send()`方法里面有一个`in`的关键字，表示输入。然后就是服务端的实现了，在Service里面创建Binder的实例，然后在onBind()方法里面return就可以了，如下：

{% highlight java %}

private Binder mBinder = new TestServiceCore.Stub() {
	@Override
	public List<TestObject> getList() throws RemoteException {
		// to do here
	}
	@Override
	public void send(TestObject obj) throws RemoteException {
		// to do here
	}
}

{% endhighlight %}

而客户端在连接成功的方法`onServiceConnected()`参数拿到这个Binder对象，然后转化为TestServiceCore接口，就可以进行RPC调用了。如下：

{% highlight java %}

@Override
public void onServiceConnected(ComponentName name, IBinder service) {
    TestServiceCore core = TestServiceCore.Stub.asInterface(service);
    try {
        List<TestObject> list = mService.getList();
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}

{% endhighlight %}

如果服务端要回调客户端，那么就再新建一个aidl，然后在客户端里面像服务端那样new出这个Binder回调对象，这个Binder类也是继承Stub的。最后在TestServiceCore那里新增一个接口把这个对象传递给服务端，通常是在onServiceConnected()方法回调的时候传递给服务器。服务端拿到客户端的Binder对象就可以回调客户端了。

其中还有一个问题，表面上传递了这个回调的Binder对象过去，实际上是经过了序列化过程的，所以客户端和服务端之间的并不是同一个对象，所以在服务端存放这些回调对象的时候需要使用`RemoteCallbackList`这个类，它实际上里面的key存储使用了底层的Binder。通常在Service里面new一个RemoteCallbackList对象出来，他是一个泛型，我们制定的类型便是客户端的binder的AIDL文件类。当客户端把他的binder传递过来的时候，就调用RemoteCallbackList.register()方法来注册进去即可。相应也还有unregister()方法。

#### 3.6.3 分析AIDL

我们创建了`TestServiceCore.aidl`文件以后，在gen目录会自动生成一个接口文件的，如下所示：

{% highlight java %}

public interface TestServiceCore extends android.os.IInterface
{
    /** Local-side IPC implementation stub class. */
    public static abstract class Stub extends android.os.Binder implements org.zhangge.test.TestServiceCore
    {
        private static final java.lang.String DESCRIPTOR = "org.zhangge.test.TestServiceCore";
        /** Construct the stub at attach it to the interface. */
        public Stub()
        {
            this.attachInterface(this, DESCRIPTOR);
        }
        /**
         * Cast an IBinder object into an org.zhangge.test.TestServiceCore interface,
         * generating a proxy if needed.
         */
        public static org.zhangge.test.TestServiceCore asInterface(android.os.IBinder obj)
        {
            if ((obj==null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin!=null)&&(iin instanceof org.zhangge.test.TestServiceCore))) {
                return ((org.zhangge.test.TestServiceCore)iin);
            }
            return new org.zhangge.test.TestServiceCore.Stub.Proxy(obj);
        }
        @Override public android.os.IBinder asBinder()
        {
            return this;
        }
        @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) 
        				throws android.os.RemoteException
        {
            switch (code)
            {
                case INTERFACE_TRANSACTION:
                {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_getList:
                {
                    data.enforceInterface(DESCRIPTOR);
                    java.util.List<org.zhangge.test.TestObject> _result = this.getList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                case TRANSACTION_send:
                {
                    data.enforceInterface(DESCRIPTOR);
                    org.zhangge.test.TestObject _arg0;
                    if ((0!=data.readInt())) {
                        _arg0 = org.zhangge.test.TestObject.CREATOR.createFromParcel(data);
                    }
                    else {
                        _arg0 = null;
                    }
                    this.send(_arg0);
                    reply.writeNoException();
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }
        private static class Proxy implements org.zhangge.test.TestServiceCore
        {
            private android.os.IBinder mRemote;
            Proxy(android.os.IBinder remote)
            {
                mRemote = remote;
            }
            @Override public android.os.IBinder asBinder()
            {
                return mRemote;
            }
            public java.lang.String getInterfaceDescriptor()
            {
                return DESCRIPTOR;
            }
            @Override public java.util.List<org.zhangge.test.TestObject> getList() 
            				throws android.os.RemoteException
            {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<org.zhangge.test.TestObject> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(org.zhangge.test.TestObject.CREATOR);
                }
                finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
            @Override public void send(org.zhangge.test.TestObject obj) 
            				throws android.os.RemoteException
            {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((obj!=null)) {
                        _data.writeInt(1);
                        obj.writeToParcel(_data, 0);
                    }
                    else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_send, _data, _reply, 0);
                    _reply.readException();
                }
                finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }
        static final int TRANSACTION_getList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_send = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }
    public java.util.List<org.zhangge.test.TestObject> getList() throws android.os.RemoteException;
    public void send(org.zhangge.test.TestObject obj) throws android.os.RemoteException;
}

{% endhighlight %}

可以看到，当我们创建Binder对象的时候，实际上new的是Stub这个抽象类，它实现了我们定义的接口，所以我们要具体实现接口方法；当把这个对象传递给客户端的时候，客户端调用了Stub的静态方法asInterface()来把Binder进行转换，可以看到这个方法里面调用了queryLocalInterface()方法查询本地接口，就是从本进程里面寻找，如果能找到就返回这个Binder对象，如果找不到，就返回新创建的Proxy这个对象。

所以说，如果不是跨进程调用的话，那么直接就是Binder对象的方法调用了，否则就是调用了Proxy对象的方法。而Proxy对象实际上是包装了Service返回的Binder对象的。首先，这里分别把定义的业务接口相应定义了int常量，然后在调用Proxy的方法时候把这些int常量相应传递下去，服务端就会根据int常量去执行不同的业务方法。实际上它调用的是Service的Binder对象的`transact()`方法，这个方法还包含了两个Parcel参数，一个是传递的参数，一个是返回的参数，因此，当调用`transact()`方法的时候线程是会被挂起的。

调用transact方法以后，Binder对象在服务端会响应调用`onTransact()`方法的，在这里根据传递的code来判断调用哪个业务方法，最后把数据设置到reply里面。注意到这个方法是运行在服务端的Binder线程里面的。

#### 3.6.4 Binder连接断了

如果客户端与服务端的连接断了会回调到`onServiceDisconnected()`方法，但是如果Binder断了，回调哪个方法？

通常我们在客户端创建`IBinder.DeathRecipient`这个接口实例，然后在onServiceConnected以后调用binder.linkToDeath()传这个接口对象即可，这个方法的第二个参数是一个标记位，好像没什么用，直接传0。相应的还有`unLinkToDeath()`方法。DeathRecipient这个接口只有一个方法，就是当服务端死了，binder断了，就会回调这个方法，所以我们用这个方法来监听服务端的binder是否断了，然后在里面调用unLinkToDeath和重新绑定服务。

这是在客户端监听服务端死亡，反过来，使用同样的方法也是可以在服务使用DeathRecipient来监听跟客户端之间是否断开连接了的，只要使用客户端的binder来调用linkToDeath()即可。另外，上面说到RemoteCallbackList这个类，实际上我们是可以继承自己实现的，因为里面有一个空实现的方法`onCallbackDied()`，可以看到注释说，就是当callback死掉的时候就会回调，因此，我们也是可以在这里监听客户端是否死掉了。

最后，Service还有一个方法onTaskRemoved()，也是可以监听到客户端是否已经死掉了。我们只要在重写这个方法就可以了，然后可以做很多事情，例如使用AlarmManager来再次启动服务等等。