---
title: android的消息处理机制（图+源码分析）——Looper,Handler,Message
date: 2016-09-10 13:08:51
tags: [转载,Android]
---

此文章来源：[http://www.cnblogs.com/codingmyworld/archive/2011/09/12/2174255.html](http://www.cnblogs.com/codingmyworld/archive/2011/09/12/2174255.html)

# [android的消息处理机制（图+源码分析）——Looper,Handler,Message](http://www.cnblogs.com/codingmyworld/archive/2011/09/14/2174255.html)

作为一个大三的预备程序员，我学习android的一大乐趣是可以通过源码学习google大牛们的设计思想。android源码中包含了大量的设计模式，除此以外，android sdk还精心为我们设计了各种helper类，对于和我一样渴望水平得到进阶的人来说，都太值得一读了。这不，前几天为了了解android的消息处理机制，我看了**Looper，Handler，Message**这几个类的源码，结果又一次被googler的设计震撼了，特与大家分享。

android的消息处理有三个核心类：Looper,Handler和Message。其实还有一个Message Queue（消息队列），但是MQ被封装到Looper里面了，我们不会直接与MQ打交道，因此我没将其作为核心类。下面一一介绍：

# 线程的魔法师 Looper

Looper的字面意思是“循环者”，它被设计用来使一个普通线程变成**Looper线程**。所谓Looper线程就是循环工作的线程。在程序开发中（尤其是GUI开发中），我们经常会需要一个线程不断循环，一旦有新任务则执行，执行完继续等待下一个任务，这就是Looper线程。使用Looper类创建Looper线程很简单：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

通过上面两行核心代码，你的线程就升级为Looper线程了！！！是不是很神奇？让我们放慢镜头，看看这两行代码各自做了什么。

1)Looper.prepare()

![img](http://pic002.cnblogs.com/images/2011/315542/2011091315284989.png)

通过上图可以看到，现在你的线程中有一个Looper对象，它的内部维护了一个消息队列MQ。注意，**一个Thread只能有一个Looper对象**，为什么呢？咱们来看源码。

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

通过源码，prepare()背后的工作方式一目了然，其核心就是将looper对象定义为ThreadLocal。如果你还不清楚什么是ThreadLocal，请参考[《理解ThreadLocal》](http://blog.csdn.net/qjyong/article/details/2158097)。

2）Looper.loop()

![img](http://pic002.cnblogs.com/images/2011/315542/2011091315590586.png)

调用loop方法后，Looper线程就开始真正工作了，它不断从自己的MQ中取出队头的消息(也叫任务)执行。其源码分析如下：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

除了prepare()和loop()方法，Looper类还提供了一些有用的方法，比如

Looper.myLooper()得到当前线程looper对象：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

getThread()得到looper对象所属线程：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

quit()方法结束looper循环：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

到此为止，你应该对Looper有了基本的了解，总结几点：

1.每个线程有且最多只能有一个Looper对象，它是一个ThreadLocal

2.Looper内部有一个消息队列，loop()方法调用后线程开始不断从队列中取出消息执行

3.Looper使一个线程变成Looper线程。

那么，我们如何往MQ上添加消息呢？下面有请Handler！（掌声~~~）

# 异步处理大师 Handler

什么是handler？handler扮演了往MQ上添加消息和处理消息的角色（只处理由自己发出的消息），即**通知MQ它要执行一个任务(sendMessage)，并在loop到自己的时候执行该任务(handleMessage)，整个过程是异步的**。handler创建时会关联一个looper，默认的构造方法将关联当前线程的looper，不过这也是可以set的。默认的构造方法：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

下面我们就可以为之前的LooperThread类加入Handler：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

加入handler后的效果如下图：

![img](http://pic002.cnblogs.com/images/2011/315542/2011091322483894.png)

可以看到，**一个线程可以有多个Handler，但是只能有一个Looper！**

**Handler发送消息**

有了handler之后，我们就可以使用 `post(Runnable)`, `postAtTime(Runnable, long)`, `postDelayed(Runnable, long)`, `sendEmptyMessage(int)`,`sendMessage(Message)`, `sendMessageAtTime(Message, long)`和 `sendMessageDelayed(Message, long)`这些方法向MQ上发送消息了。光看这些API你可能会觉得handler能发两种消息，一种是Runnable对象，一种是message对象，这是直观的理解，但其实post发出的Runnable对象最后都被封装成message对象了，见源码：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

其他方法就不罗列了，总之通过handler发出的message有如下特点：

1.message.target为该handler对象，这确保了looper执行到该message时能找到处理它的handler，即loop()方法中的关键代码

```
msg.target.dispatchMessage(msg);
```

2.post发出的message，其callback为Runnable对象

**Handler处理消息**

说完了消息的发送，再来看下handler如何处理消息。消息的处理是通过核心方法[dispatchMessage](http://developer.android.com/reference/android/os/Handler.html#dispatchMessage%28android.os.Message%29)([Message](http://developer.android.com/reference/android/os/Message.html) msg)与钩子方法[handleMessage](http://developer.android.com/reference/android/os/Handler.html#handleMessage%28android.os.Message%29)([Message](http://developer.android.com/reference/android/os/Message.html) msg)完成的，见源码

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

可以看到，除了[handleMessage](http://developer.android.com/reference/android/os/Handler.html#handleMessage%28android.os.Message%29)([Message](http://developer.android.com/reference/android/os/Message.html) msg)和Runnable对象的run方法由开发者实现外（实现具体逻辑），handler的内部工作机制对开发者是透明的。这正是handler API设计的精妙之处！

**Handler的用处**

****我在小标题中将handler描述为“异步处理大师”，这归功于Handler拥有下面两个重要的特点：

1.handler可以在**任意线程发送消息**，这些消息会被添加到关联的MQ上。

![img](http://pic002.cnblogs.com/images/2011/315542/2011091409532564.png)              

2.handler是在它**关联的looper线程中处理消息**的。

![img](http://pic002.cnblogs.com/images/2011/315542/2011091409541179.png)

这就解决了android最经典的不能在其他非主线程中更新UI的问题。**android的主线程也是一个looper线程**(looper在android中运用很广)，我们在其中创建的handler默认将关联主线程MQ。因此，利用handler的一个solution就是在activity中创建handler并将其引用传递给worker thread，worker thread执行完任务后使用handler发送消息通知activity更新UI。(过程如图)

![img](http://pic002.cnblogs.com/images/2011/315542/2011091323582123.png)

下面给出sample代码，仅供参考：

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

![img](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)View Code

当然，handler能做的远远不仅如此，由于它能post Runnable对象，它还能与Looper配合实现经典的Pipeline Thread(流水线线程)模式。请参考此文[《Android Guts: Intro to Loopers and Handlers》](http://mindtherobot.com/blog/159/android-guts-intro-to-loopers-and-handlers/)

# 封装任务 Message

在整个消息处理机制中，message又叫task，封装了任务携带的信息和处理该任务的handler。message的用法比较简单，这里不做总结了。但是有这么几点需要注意（待补充）：

1.尽管Message有public的默认构造方法，但是你应该通过Message.obtain()来从消息池中获得空消息对象，以节省资源。

2.如果你的message只需要携带简单的int信息，请优先使用Message.arg1和Message.arg2来传递信息，这比用Bundle更省内存

3.擅用message.what来标识信息，以便用不同方式处理message。