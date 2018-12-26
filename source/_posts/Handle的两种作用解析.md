---
title: Handle的两种作用解析
date: 2016-09-09 13:06:21
tags: [Android]
---

​        按照我现在的理解的话，Handle我认为它是Android系统的传送器，文档中是这么的定义的`A Handler allows you to send and process Message and Runnable objects associated with a thread's MessageQueue. ` 他可以在子线程中携带信息跳转到UI线程进行UI刷新，也可以携带信息到任意一个实现了Runnable接口的类中，所以他在android中是十分重要的，特别是子线程与UI线程的沟通。

----------
## 使用Handler进行Runnable ##
如上所说的Handler可以可以携带信息到任意一个实现了Runnable接口的类中，如我现在有一个数组存储了几张图片的resource，我想要让它以时间间隔为1秒进行图片切换，那么现在我定义一个Runnable

```
 private myRunnable runnable=new myRunnable();
    class myRunnable implements Runnable
    {

        @Override
        public void run()
        {
            index++;
            index = index % 3;
            imageView.setImageResource(images[index]);
            handler.postDelayed(runnable, 1000);
        }
    }
```
那么我只需要new一个handle出来，在oncreat方法里调用`handler.postDelayed(runnable, 1000);` 即可，注意Runnable可以用来在子线程中使用，但是并不是Runnable就是子线程了，在没有开启Thread的情况下它仍然在主线程中。

----------
## 在子线程中调用并传送数据 ##
由于我们现在要传送数据到主线程的方法，也就是handler的一个抽象方法 ` handleMessage` 我们就需要在创建Handler的时候就重写他，这个方法的参数是Message，这个对象可以封装任何一个Object，也就是说我们在开启子线程时可以这么做：

```
 new Thread(new Runnable()
        {
            @Override
            public void run()
            {
                try
                {
                    Thread.sleep(2000);
                    Message message = handler.obtainMessage();
                    message.obj=new Person(25,"Hugo");
                    handler.sendMessage(message);//sendToTarget也行
                } catch (InterruptedException e)
                {
                    e.printStackTrace();
                }
            }
        }).start();
```
使用Message保存对象，Message也可以简单的只是保存int，调用arg1，和arg2即可。在sendMessage方法调用后就会调用我们刚才在创建Handler时重写的handleMessage方法。也就是说调到了主线程。

