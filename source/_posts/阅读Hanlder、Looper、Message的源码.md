---
title: 阅读Hanlder、Looper、Message的源码
date: 2018-09-17 14:01:49
tags:
  - Handler
categories:
  - Android
---

## 经典例子

首先是创建一个 Handler

```java
private Handler mHandler = new Handler() {

	@Override
	public void handleMessage(Message msg) {
		switch (msg.what) {
			case 0:
				mTextView.setText("更新了UI");
				break;
		}
	}
};
```

在子线程中更新UI

```java
new Thread(new Runnable() {
	@Override
	public void run() {
		try {
			Thread.sleep(1000);  // 在子线程做一段耗时操作，比如进行网络请求
			mHandler.sendEmptyMessage(0);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}).start();
```

<!-- more -->

## 找到看源码的切入点

从这个经典的例子中来读源码，源码的切入点就可以从 `Handler` 的无参构造方法开始看起。

```java
public Handler() {
	this(null, false);
}
```

这个无参构造方法调用了另外一个构造方法 `this(null, false);`。去除方法体里面其他的无关代码，得到关键代码如下

```java
/**
 * Use the {@link Looper} for the current thread with the specified callback interface
 * and set whether the handler should be asynchronous.
 *
 * Handlers are synchronous by default unless this constructor is used to make
 * one that is strictly asynchronous.
 *
 * Asynchronous messages represent interrupts or events that do not require global ordering
 * with respect to synchronous messages.  Asynchronous messages are not subject to
 * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
 *
 * @param callback The callback interface in which to handle messages, or null.
 * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
 * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
 *
 * @hide
 */
public Handler(Callback callback, boolean async) {
	// ...
	
	mLooper = Looper.myLooper();
	if (mLooper == null) {
		throw new RuntimeException(
			"Can't create handler inside thread that has not called Looper.prepare()");
	}
	mQueue = mLooper.mQueue;
	
	// ...
}
```

看一下这个两句话是什么意思

```java
mLooper = Looper.myLooper();
if (mLooper == null) {
	throw new RuntimeException(
		"Can't create handler inside thread that has not called Looper.prepare()");
}
```

看一下 `Looper.myLooper()` 的实现

```java
/**
 * Return the Looper object associated with the current thread.  Returns
 * null if the calling thread is not associated with a Looper.
 */
public static @Nullable Looper myLooper() {
	return sThreadLocal.get();
}
```

只是返回一个 `Looper` 对象而已，也就是获取了当前线程保存的 `Looper` 实例，看一下 `sThreadLocal` 这个变量是什么东西，用 `CTRL-F` 查找一下代码，整个 `Looper` 类中只有这几个地方有用到 `sThreadLocal` 变量

```java
// sThreadLocal.get() will return null unless you've called prepare().
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

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
```

主要看 `prepare(boolean quitAllowed)` 这个方法就可以了。`sThreadLocal` 是一个 `ThreadLocal` 对象，可以在一个线程中存储变量。

```java
1 public static final void prepare() {  
2 	 if (sThreadLocal.get() != null) {  
3 	 	 throw new RuntimeException("Only one Looper may be created per thread");  
4 	 }  
5 	 sThreadLocal.set(new Looper(true));  
6 }
```

第 5 行新建一个 `Looper` 对象放入 `sThreadLocal` 对象。2~4 行判断 `sThreadLocal` 中是否已经有 `Looper` 对象了，如果有就会抛出异常，说明一个线程只能有一个 `Looper` 对象。

看一下 `new Looper(true)` 这句话做了什么事情

```java
private Looper(boolean quitAllowed) {
	mQueue = new MessageQueue(quitAllowed);
	mThread = Thread.currentThread();
}
```

创建了一个消息队列 `MessageQueue`，还有获取了当前线程的实例。

到这里就不知道要看什么了，那就看一下类的注释好了，说不定有线索

```java
/**
  * Class used to run a message loop for a thread.  Threads by default do
  * not have a message loop associated with them; to create one, call
  * {@link #prepare} in the thread that is to run the loop, and then
  * {@link #loop} to have it process messages until the loop is stopped.
  *
  * <p>Most interaction with a message loop is through the
  * {@link Handler} class.
  *
  * <p>This is a typical example of the implementation of a Looper thread,
  * using the separation of {@link #prepare} and {@link #loop} to create an
  * initial Handler to communicate with the Looper.
  *
  * <pre>
  *  class LooperThread extends Thread {
  *      public Handler mHandler;
  *
  *      public void run() {
  *          Looper.prepare();
  *
  *          mHandler = new Handler() {
  *              public void handleMessage(Message msg) {
  *                  // process incoming messages here
  *              }
  *          };
  *
  *          Looper.loop();
  *      }
  *  }</pre>
  */
public final class Looper {
	// ...
}
```

注释说 `Looper` 类是用来为一个线程创建一个消息循环用的。线程默认是没有消息循环的，可以调用 `prepare()` 创建消息循环，调用 `loop()` 去处理消息。通常情况下是用 `Handler` 类和消息循环交互。

这段注释提到了 `prepare()` 和 `loop()` 方法。`prepare()` 方法刚才已经看过了，那就看一下 `loop()` 方法做了什么。去掉一些无关的语句，得到

```java
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

	// ...

	for (;;) {
		Message msg = queue.next(); // might block
		if (msg == null) {
			// No message indicates that the message queue is quitting.
			return;
		}

		// ...

		try {
			msg.target.dispatchMessage(msg);
		} finally {
			// ...
		}

		// ...

		msg.recycleUnchecked();
	}
}
```

这段代码的大概意思是

> 无限循环去遍历消息队列，取出队列中的 `Message` 对象，然后调用 `msg.target.dispatchMessage(msg);` ，然后再把 `Message` 回收了

`msg.target.dispatchMessage(msg);` 这句话不明白，进入 `Message` 类看一下是什么东西。

```java
Handler target;
```

从这里可以知道 `Handler` 和 `Message` 有一腿。

再看一下 `Handler` 类中 `dispatchMessage(msg)` 方法的实现

```java
/**
 * Handle system messages here.
 */
public void dispatchMessage(Message msg) {
	if (msg.callback != null) {
		handleCallback(msg);
	} else {
		if (mCallback != null) {
			if (mCallback.handleMessage(msg)) {
				return;
			}
		}
		handleMessage(msg);
	}
}
```

这里应该就是调用各种回调方法了。从刚才写的经典例子中，我没有写任何的回调方法，因此这个方法体最后就进入了 `handleMessage(msg);` 方法中。看一下这个方法做了什么

```java
/**
 * Subclasses must implement this to receive messages.
 */
public void handleMessage(Message msg) {
}
```

这个方法就是经典例子中重写的 `handlerMessage` 方法

```java
private Handler mHandler = new Handler() {

	@Override
	public void handleMessage(Message msg) {
		switch (msg.what) {
			case 0:
				mTextView.setText("更新了UI");
				break;
		}
	}
};
```

现在梳理一下

* Looper 在 `prepare()` 方法中创建一个 `Looper` 实例和一个消息队列
* Looper 在 `loop()` 方法中不断地遍历消息队列，调用 `MessageQueue.next()` 取出 `Message`，然后调用和 `Message` 绑定的 `Handler` 的 `dispatchMessage` 方法，去调用各种回调方法

现在几个疑问

* `Message` 是怎么创建的？
* `Message` 是怎么进入 `Looper` 的消息队列的？
* `Handler` 是怎么和 `Message` 绑定在一起的？

## Message 是怎么创建的？

在使用 `Handler` 异步更新 UI 的时候，都会创建 `Message` 携带我们想要传递的数据。`Message` 有几个属性可以携带数据：

`what`、`arg1`、`arg2`、`obj` 。用法如下

```java
Message msg = mHandler.obtainMessage();
msg.what = 1;
msg.arg1 = 2;
msg.arg2 = 3;
msg.obj = myObj // myObj 是我们想通过 Message 携带的对象
mHandler.sendMessage(msg);
```

看一下 `mHandler.obtainMessage()` 做了什么

```java
/**
 * Returns a new {@link android.os.Message Message} from the global message pool. More efficient than
 * creating and allocating new instances. The retrieved message has its handler set to this instance (Message.target == this).
 *  If you don't want that facility, just call Message.obtain() instead.
 */
public final Message obtainMessage()
{
	return Message.obtain(this);
}
```

又调用了 `Message.obtain(this)`，再进入看看

```java
/**
 * Same as {@link #obtain()}, but sets the value for the <em>target</em> member on the Message returned.
 * @param h  Handler to assign to the returned Message object's <em>target</em> member.
 * @return A Message object from the global pool.
 */
public static Message obtain(Handler h) {
	Message m = obtain();
	m.target = h;

	return m;
}
```

可以看到，为了提高性能，先调用 `Message m = obtain();` 从消息池中取出一个 `Message`，然后再将 `Handler` 实例赋值给 `Message` 的 `target` 属性，这样就将 `Handler` 和 `Message` 绑定在一起了。

## Message是怎么和Handler绑定在一起的？

从上面的

```java
public static Message obtain(Handler h) {
	Message m = obtain();
	m.target = h;

	return m;
}
```

我们已经知道了

## Message 是怎么进入 Looper 的消息队列的？

看一下刚才写的代码

```java
Message msg = mHandler.obtainMessage();
msg.what = 1;
msg.arg1 = 2;
msg.arg2 = 3;
msg.obj = "想要携带的对象";
mHandler.sendMessage(msg);
```

`Message` 已经通过 `Handler.obtainMessage()` 获取到了，然后就是调用 `Handler.sendMessage(Message msg)` 发出消息。看一下 `mHandler.sendMessage(msg);` 做了什么

```java
/**
 * Pushes a message onto the end of the message queue after all pending messages
 * before the current time. It will be received in {@link #handleMessage},
 * in the thread attached to this handler.
 *  
 * @return Returns true if the message was successfully placed in to the 
 *         message queue.  Returns false on failure, usually because the
 *         looper processing the message queue is exiting.
 */
public final boolean sendMessage(Message msg)
{
	return sendMessageDelayed(msg, 0);
}
```

又调用了另外一个方法，再点进去看看

```java
/**
 * Enqueue a message into the message queue after all pending messages
 * before (current time + delayMillis). You will receive it in
 * {@link #handleMessage}, in the thread attached to this handler.
 *  
 * @return Returns true if the message was successfully placed in to the 
 *         message queue.  Returns false on failure, usually because the
 *         looper processing the message queue is exiting.  Note that a
 *         result of true does not mean the message will be processed -- if
 *         the looper is quit before the delivery time of the message
 *         occurs then the message will be dropped.
 */
public final boolean sendMessageDelayed(Message msg, long delayMillis)
{
	if (delayMillis < 0) {
		delayMillis = 0;
	}
	return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

怎么又调用了另外一个方法，再点进去看看

```java
/**
 * Enqueue a message into the message queue after all pending messages
 * before the absolute time (in milliseconds) <var>uptimeMillis</var>.
 * <b>The time-base is {@link android.os.SystemClock#uptimeMillis}.</b>
 * Time spent in deep sleep will add an additional delay to execution.
 * You will receive it in {@link #handleMessage}, in the thread attached
 * to this handler.
 * 
 * @param uptimeMillis The absolute time at which the message should be
 *         delivered, using the
 *         {@link android.os.SystemClock#uptimeMillis} time-base.
 *         
 * @return Returns true if the message was successfully placed in to the 
 *         message queue.  Returns false on failure, usually because the
 *         looper processing the message queue is exiting.  Note that a
 *         result of true does not mean the message will be processed -- if
 *         the looper is quit before the delivery time of the message
 *         occurs then the message will be dropped.
 */
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
```

从这个方法体可以看出，是把刚才我们创建的 `Message` 加入队列中去。这里有句话

```java
MessageQueue queue = mQueue;
```

这个消息队列是哪里来的？`Handler` 类中怎么会有消息队列？用 `CTRL-F` 查找一下 `mQueue =` 在哪里出现过

```java
/**
 * Use the {@link Looper} for the current thread with the specified callback interface
 * and set whether the handler should be asynchronous.
 *
 * Handlers are synchronous by default unless this constructor is used to make
 * one that is strictly asynchronous.
 *
 * Asynchronous messages represent interrupts or events that do not require global ordering
 * with respect to synchronous messages.  Asynchronous messages are not subject to
 * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
 *
 * @param callback The callback interface in which to handle messages, or null.
 * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
 * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
 *
 * @hide
 */
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
	mQueue = mLooper.mQueue;
	mCallback = callback;
	mAsynchronous = async;
}
```

在 `Handler` 的这个构造方法中找到了 `mQueue` 被赋值的地方

```java
mLooper = Looper.myLooper();
if (mLooper == null) {
	throw new RuntimeException(
		"Can't create handler inside thread that has not called Looper.prepare()");
}
mQueue = mLooper.mQueue;
```

它是通过调用 `Looper` 的静态方法 `myLooper()` 获取到 `Looper` 对象，然后再通过 `Looper` 对象进而获取到 `Looper` 的消息队列的。

知道了 `Handler` 中的 `mQueue` 是怎么获取到的，那就再接着往下看

```java
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
```

这个方法又调用了 `enqueueMessage(queue, msg, uptimeMillis)`，点进去看一下做了什么

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
	msg.target = this;
	if (mAsynchronous) {
		msg.setAsynchronous(true);
	}
	return queue.enqueueMessage(msg, uptimeMillis);
}
```

在这里，这个方法调用了消息队列 `MessageQueue` 的 `enqueueMessage()` 方法把 `Message` 放进去。

`enqueueMessage()` 这个方法的方法体代码太多，我看不懂。不过只需要知道一件事，这个方法的指责是将 `Message` 放入 `MessageQueue` 就好了。

## 梳理Looper、Handler、Message、MessageQueue关系

- Looper 在 `prepare()` 方法中创建一个 `Looper` 实例和一个消息队列
- Looper 在 `loop()` 方法中不断地遍历消息队列，调用 `MessageQueue.next()` 从消息队列中取出 `Message`，然后调用和 `Message` 绑定的 `Handler` 的 `dispatchMessage` 方法，去调用各种回调方法
- `Handler` 的构造方法会获得当前线程中的 `Looper` 实例，进而和 `MessageQueue` 关联在一起
- `Handler` 调用 `sendMessage(Message msg)`，会将 `msg.target` 赋值为自己，然后放入 `MessageQueue` 中
- 我们在构造 `Handler` 实例的时候，会重写 `handleMessage` 方法，也就是 `msg.target.dispatchMessage(msg)` 最终调用的方法

## Looper.prepare()又是谁来调用的？

从刚才的源码中，可以知道是通过调用 `Looper.prepare()` 来创建 `Looper` 对象。同时，从经典例子中发现，我们没有去调用 `Looper.prepare()` 方法。那么 `Looper.prepare()` 又是谁来调用的？

查了下资料，Android 应用程序的入口是 `ActivityThread` 的` main` 函数，那就去看一下 `main` 函数是怎么实现的，可以看到，在方法体中有一句 `Looper.prepareMainLooper();`

```java
public final class ActivityThread {

	public static void main(String[] args) {
		// ...

        Looper.prepareMainLooper();

        // ...
		
        Looper.loop();		
    }
}
```

看一下这个方法的方法体

```java
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
```

它调用了 `prepare()` 创建了 `Looper` 实例。

以上就是 UI 线程 `Looper` 创建过程

## 参考资料

* [Android 异步消息处理机制 让你深入理解 Looper、Handler、Message三者关系](http://blog.csdn.net/lmj623565791/article/details/38377229)
* [Android应用程序入口点究竟是什么？](https://www.zhihu.com/question/30648827)