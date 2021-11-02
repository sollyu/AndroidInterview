# Android Handler 知识点总结


## 优质文章

[👍 Android异步通信：图文详解Handler工作原理_Carson带你学Android-CSDN博客](https://blog.csdn.net/carson_ho/article/details/80175876)

[Android Handler 机制（一）：Handler 运行机制完整梳理 - 灰色飘零 - 博客园](https://www.cnblogs.com/renhui/p/12857876.html)

[Android Handler机制 - MessageQueue如何处理消息_未子涵的博客-CSDN博客](https://blog.csdn.net/lovelease/article/details/81988696)

[Android应用程序消息处理机制 - SegmentFault 思否](https://segmentfault.com/a/1190000002982318)

## 常见问题

{{< admonition type=tip title="提示" open=true >}}
下面是一些常见的问题，可以先尝试的先自我回答一下，点击题目即可查看答案。
{{< /admonition >}}


{{< admonition type=question title="主线程为什么不用初始化Looper？" open=false >}}
因为应用在启动的过程中就已经初始化主线程Looper了。

每个java应用程序都是有一个main方法入口，Android是基于Java的程序也不例外。Android程序的入口在ActivityThread的main方法中：
```java
/**
 * main方法中先初始化主线程Looper，
 * 新建ActivityThread对象，然后再启动Looper，这样主线程的Looper在程序启动的时候就跑起来了。
 * 我们不需要再去初始化主线程Looper。
 */
public static void main(String[] args) {
    ...
    // 初始化主线程Looper
    Looper.prepareMainLooper();
    ...
    // 新建一个ActivityThread对象
    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);

    // 获取ActivityThread的Handler，也是他的内部类H
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    ...
    Looper.loop();
    // 如果loop方法结束则抛出异常，程序结束
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```
{{< /admonition >}}


{{< admonition type=question title="为什么主线程的Looper是一个死循环，但是却不会ANR？" open=false >}}
因为Android 的是由事件驱动的，looper.loop() **不断地接收事件、处理事件**，每一个点击触摸或者说Activity的生命周期都是运行在 Looper.loop() 的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它。

[主线程中的Looper.loop()死循环为什么不会导致ANR?_威威的专栏-CSDN博客](https://blog.csdn.net/u013626215/article/details/88796172)
{{< /admonition >}}


{{< admonition type=question title="Handler如何保证MessageQueue并发访问安全？" open=false >}}
我们可以发现MessageQueue其实是“生产者-消费者”模型，Handler不断地放入消息，Looper不断地取出，这就涉及到死锁问题。

如果Looper拿到锁，但是队列中没有消息，就会一直等待，而Handler需要把消息放进去，锁却被Looper拿着无法入队，这就造成了死锁。

Handler机制的解决方法是循环加锁。在MessageQueue的next方法中：

```java
Message next() {
    ...
    for (;;) {
    ...
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            ...
        }
    }
}
```
{{< /admonition >}}


{{< admonition type=question title="Handler是如何切换线程的？" open=false >}}
使用不同线程的Looper处理消息。
{{< /admonition >}}


{{< admonition type=question title="Handler的阻塞唤醒机制是怎么回事？" open=false >}}
Handler的阻塞唤醒机制是基于Linux的阻塞唤醒机制，主要的唤醒机制是epoll_wait函数。

这个机制也是类似于handler机制的模式。在本地创建一个文件描述符，然后需要等待的一方则监听这个文件描述符，唤醒的一方只需要修改这个文件，那么等待的一方就会收到文件从而打破唤醒。和Looper监听MessageQueue，Handler添加message是比较类似的。

[源码茶舍之没有epoll就没有Handler - 掘金](https://juejin.cn/post/6896495861954510861)
{{< /admonition >}}


{{< admonition type=question title="能不能让一个Message加急被处理？/ 什么是Handler同步屏障？" open=false >}}
可以 / 一种使得异步消息可以被更快处理的机制

注意，同步屏障不会自动移除，使用完成之后需要手动进行移除，不然会造成同步消息无法被处理。

消息屏障启动、关闭的方法：
```java
// 反射插入消息屏障
MessageQueue queue = mHandler.getLooper().getQueue();
Method method = MessageQueue.class.getDeclaredMethod("postSyncBarrier");
method.setAccessible(true);
token = (int)method.invoke(queue);

// 反射移除消息屏障
MessageQueue queue = mHandler.getLooper().getQueue();
Method method = MessageQueue.class.getDeclaredMethod("removeSyncBarrier", int.class);
method.setAccessible(true);
method.invoke(queue, token);
```
[Handler之消息屏障你应该知道的_天亮了的博客-CSDN博客_handler消息屏障](https://blog.csdn.net/my_csdnboke/article/details/109531168)
{{< /admonition >}}



{{< admonition type=question title="一个Thread可以有几个Looper？几个Handler？" open=false >}}
一个线程只能有一个Looper，可以有多个Handler，
在线程中我们需要调用Looper.perpare,
他会创建一个Looper并且将Looper保存在ThreadLocal中，
每个线程都有一个LocalThreadMap，会将Looper保存在对应线程中的LocalThreadMap，
key为ThreadLocal，value为Looper
{{< /admonition >}}


{{< admonition type=question title="Message可以如何创建？哪种效果更好，为什么" open=false >}}

```java
// 最常用的方式，通过Message的无参构造函数创建一个Message 对象
Message message = new Message();
```

```java
// Message.obtain()及其系列的重载方法，我们先看看他的无参方法的源码实现
Message message = Message.obtain();

public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            //直接从本地获取一个message对象
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // clear in-use flag
            //没调用一次该方法，就将新的message对象加入Message池子，sPoolSize就减少一个
            sPoolSize--;
            return m;
        }
    }
    return new Message();
}
```

```java
// new Handler().obtainMessage()及其系列的重载方法，我们先看看他的无参方法的源码实现：
Message message = new Handler().obtainMessage();

public final Message obtainMessage(){
    return Message.obtain(this);
}
// Message.obtain(this)方法具体 如下：
public static Message obtain(Handler h) {
    //方法内部最终还是调用了obtain()方法，又回到,了方式2，这跟方式2创建Message对象是一样的，本质上没有区别。
    Message m = obtain();
    m.target = h;
    return m;
}

```
{{< /admonition >}}

{{< admonition type=question title="Handler是如何实现延时消息的？" open=false >}}

* 消息队列是按消息触发时间排序的

* **设置`epoll_wait`的超时时间，使其在特定时间唤醒**，这里我们先计算当前时间和触发时间有多长，这个差值作为 epoll_wait的超时时间，epoll_wait超时的时候就是消息触发的时候了，就不会继续堵塞，继续往下执行，这个线程就会被唤醒，去执行消息处理

[👍 深入理解Handler(三) --- Handler发送延时消息实现_soso密斯密斯的博客-CSDN博客](https://blog.csdn.net/qq_38366777/article/details/108942036)

[Handler是如何实现延时消息的？ - 简书](https://www.jianshu.com/p/68083d432b3f)
{{< /admonition >}}


{{< admonition type=question title="Handler发送消息的 Delay 可靠吗？" open=false >}}
答案是不靠谱的，引起不靠谱的原因有如下

* 发送的消息太多,Looper负载越高，任务越容易积压，进而导致卡顿
* 消息队列有一些消息处理非常耗时，导致后面的消息延时处理
* 大于Handler Looper的周期时基本可靠（例如主线程>50ms）
* 对于时间精确度要求较高，不要用handler的delay作为即时的依据
{{< /admonition >}}


{{< admonition type=question title="hread、Handler和HandlerThread关系何在？" open=false >}}
Handler：在android中负责发送和处理消息，通过它可以实现其他支线线程与主线程之间的消息通讯。

Thread：Java进程中执行运算的最小单位，亦即执行处理机调度的基本单位。某一进程中一路单独运行的程序。

HandlerThread：一个继承自Thread的类HandlerThread，Android中没有对Java中的Thread进行任何封装，而是提供了一个继承自Thread的类HandlerThread类，这个类对Java的Thread做了很多便利的封装。
{{< /admonition >}}


## 视频资料

{{< bilibili BV1fv411B7bH >}}



