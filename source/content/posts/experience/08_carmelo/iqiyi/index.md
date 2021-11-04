---
title: "【面试分享】爱奇艺"
date: 2021-11-04T10:24:49+08:00
draft: false
author: "08_carmelo"
authorLink: "https://www.jianshu.com/u/b8dad3885e05"
resources:
- name: "featured-image"
  src: "featured-image.jpg"

tags: ["安卓", "面试分享", "爱奇艺", "iqiyi"]
categories: ["面试分享"]
---

## 说明

以下内容来自[9月Android面试经验分享.md - 简书](https://www.jianshu.com/p/a1cb87be4a88)摘要。

{{< admonition type=tip title="提示" open=true >}}
查看答案前可先自我尝试回答一下，用于增强自己的对问题的理解。
{{< /admonition >}}


## 一面

{{< admonition type=question title="Handler里面的nativePollOnce为什么不会ANR" open=false >}}
这里主要考察的是对epoll_wait的理解，以及和ANR的区别。

nativePollOnce最终会走进epoll_wait函数，当前队列中没有内容的时候，会进入阻塞状态等待内容添加。
ANR的定义为：
1. InputDispatching Timeout：5秒内无法响应屏幕触摸事件或键盘输入事件。
2. BroadcastQueue Timeout ：在执行前台广播（BroadcastReceiver）的onReceive()函数时10秒没有处理完成，后台为60秒。
3. Service Timeout：前台服务20秒内，后台服务在200秒内没有执行完毕。
4. ContentProvider Timeout ：ContentProvider的publish在10s内没进行完。

> 回答：
>
> nativePollOnce会卡住但是只要不是触发在ANR的定义范围内，所有就没有弹出ANR的提示。同时Handler也可以运行中子线程中，更没有ANR的提示。

[源码解读epoll内核机制 - Gityuan博客 | 袁辉辉的技术博客 👍](http://gityuan.com/2019/01/06/linux-epoll/)

[looper阻塞为什么不会造成ANR？_梦否-CSDN博客](https://blog.csdn.net/qq_26460841/article/details/118567231)
{{< /admonition >}}




{{< admonition type=question title="对称加密和非对称加密的区别" open=false >}}
#### 对称加密
加密和解密的`秘钥是同一个`。

常见算法：DES、3DES、Blowfish、IDEA、RC4、RC5、RC6 和 AES

#### 非对称加密
非对称加密算法需要`两个`密钥：公开密钥（publickey）和私有密钥（privatekey）。

常见算法：RSA、ECC（移动设备用）、Diffie-Hellman、El Gamal、DSA（数字签名用）

[对称加密和非对称加密的区别_ETFOX-CSDN博客](https://blog.csdn.net/qq_29689487/article/details/81634057)
{{< /admonition >}}





{{< admonition type=question title="布局嵌套过深会导致什么问题？" open=false >}}
> 答： 会触发`StackOverflowError`错误，也就是会出现`堆栈溢出`的崩溃。
>
> 提示：在嵌套第`22`层的时候就会出现这种崩溃。

[Android布局嵌套太深导致的错误：StackOverflowError - 圣骑士wind - 博客园 👍](https://www.cnblogs.com/mengdd/archive/2013/05/14/3078367.html)

[Android布局优化详解 - 知乎](https://zhuanlan.zhihu.com/p/384628654)
{{< /admonition >}}





{{< admonition type=question title="Java为什么跨平台？ c是跨平台吗" open=false >}}
#### Java跨平台原理
因为字节码是在虚拟机上运行的，而不是编译器。换而言之，`是因为JVM能跨平台`安装，所以相应JAVA字节码便可以跟着在任何平台上运行。只要JVM自身的代码能在相应平台上运行，即JVM可行，则JAVA的程序员就可以不用考虑所写的程序要在哪里运行，反正都是在虚拟机上运行，然后变成相应平台的机器语言，而这个转变并不是程序员应该关心的。

#### C是跨平台的吗？
有的说C语言也可以说是跨平台的：`C语言本身是跨平台的，但程序不是`，如果你的程序只使用C标准的输入输出，那么源代码也是跨平台的，只要用对应平台的编译器编译就可以运行，如果你`使用了平台专有的API，那么就不能跨平台`，比如WINDOWS窗口程序，就调用了WINDOWS的创建窗口，显示窗口等API（这些调用并不一定在你自己的代码中），linux是没有这些API的，所以就无法编译运行。
{{< /admonition >}}








{{< admonition type=question title="App打包过程" open=false >}}
1. 打包资源文件，生成R.java文件
2. 处理aidl文件，生成相应的Java文件
3. 编译项目源代码，生成class文件
4. 转换所有的class文件，生成classes.dex文件
5. 打包生成APK文件
6. 对APK文件进行签名
7. 对签名后的APK文件进行对齐处理

[Android APK打包流程_Davis-CSDN博客_android apk打包流程](https://blog.csdn.net/wangzhongshun/article/details/96160984)
{{< /admonition >}}








{{< admonition type=question title="协程挂起和线程阻塞的区别" open=false >}}
协程的挂起可以理解为`切换线程`，线程的阻塞就是`同步代码中的阻塞`。

> 协程在执行到有 suspend 标记的函数的时候，会被 suspend 也就是被挂起，而`所谓的被挂起，就是切个线程`；
>
> 不过区别在于，挂起函数在执行完成之后，`协程会重新切回它原先的线程`。
>
> 再简单来讲，在 Kotlin 中所谓的挂起，<u>就是一个稍后会被自动切回来的线程调度操作</u>。
>
> 这个「切回来」的动作，在 Kotlin 里叫做 resume，恢复。

[kotlin之协程(二),Kotlin协程是什么、挂起是什么、挂起的非阻塞式 - 简书](https://www.jianshu.com/p/e4e7ae9473de)
{{< /admonition >}}







{{< admonition type=question title="so文件加载流程" open=false >}}
#### System.loadLibrary 加载过程
1. 先到`nativeLibraryDirectories`指向的目录中寻找，是否存在且可用的 so 文件，有则直接加载这里的 so 文件；
2. 上一步没找到的话，则根据当前进程如果是 32 位的，那么依次去`vendor/lib`和`system/lib`目录中寻找；
3. 同样，如果当前进程是 64 位的，那么依次去`vendor/lib64`和`system/lib64`目录中寻找；

#### Native加载过程
1. 调用`dlopen`函数打开so文件
2. 调用`dlsym`函数挂载so内容
3. 触发`init`函数
4. 执行`JNI_OnLoad`函数

[Android 的 so 文件加载机制 - 请叫我大苏 - 博客园 👍](https://www.cnblogs.com/dasusu/p/9810673.html)

[android so加载 - vendanner - 博客园](https://www.cnblogs.com/vendanner/p/4979177.html)

[08-SO加载解析过程 - shuwoom的博客](https://shuwoom.com/?p=351)
{{< /admonition >}}






{{< admonition type=question title="AIDL怎么实现" open=false >}}
1. 定义AIDL文件；——类似于后台给了一个接口文档，里面描述了各种各样的接口，而在AIDL文件中描述的是类似于JAVA接口的方法定义
2. 实现AIDL文件里面定义的接口；——类似于后台实现接口文档的接口控制器，里面定义了接口的具体实现
3. 暴露接口；——这个就像后台开发人员把接口文档给App开发人员，然后App开发人员就知道有哪些接口可以调用来实现业务了
4. 调用；——这个就是App开发人员调用后台的接口来获取过程数据了。
从上面的描述可以看出，这个AIDL实际上就是一个C/S模型，一边是客户端，一边是服务器。

[AIDL文件定义含义与实现详解 - 简书](https://www.jianshu.com/p/111d46bbf521)
{{< /admonition >}}







{{< admonition type=question title="字节码是什么" open=false >}}
Java源代码经过虚拟机编译器编译后产生的文件（即扩展为.class的文件），它不面向任何特定的处理器，只面向虚拟机。

Java语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以Java程序运行时比较高效，而且，由于字节码并不专对一种特定的机器，因此，Java程序无须重新编译便可在多种不同的计算机上运行。

[什么是字节码？采用字节码的最大好处是什么 - 知乎](https://zhuanlan.zhihu.com/p/137021803)
{{< /admonition >}}



## 二面

和一面间隔时间太久了，放弃

