---
title: GestureDetector识别手势滑动
date: 2016-09-08 13:05:35
tags: [Android]
---



今天学习了使用GestureDetector进行手势识别，如果要进行手势识别，那么就必然要知道Android系统是如何识别动作的，见下图 ：![这里写图片描述](http://img.blog.csdn.net/20160908151513186)
我就在布局中放一个ImageView，就在这张图片上滑动。

----------
## 触发MotionEvent事件并监听 ##
MotionEvent事件是你手一放上屏幕就出发了的，由onTouchListener监听，由于我是在图片上进行滑动的所以这个监听器由imageview注册`img.setOnTouchListener(new View.OnTouchListener()
        {
            @Override
            public boolean onTouch(View v, MotionEvent event)//捕获触摸屏幕的事件
            {
                myGestureDetector.onTouchEvent(event);//转发onGesturelistenner
                return true;
            }
        });` 
  可以看到会重写这个接口的onTouch()方法，这里的MotionEvent参数就是触摸事件。



----------
## GestureDetector转发至监听器 ##
虽然现在得到了监听对象，但是在这里不能有动作，还需要new一个GestureDetector对象将其转发至OnGestureListener才能有动作`GestureDetector  myGestureDetector = new GestureDetector(MainActivity.this, new myGestureListenner());` 可以看到第二个方法就需要我们的GestureListenner方法，而GestureListenner的实例有OnContextClickListener，OnDoubleTapListener，OnGestureListener和SimpleOnGestureListener。前三种实例都需要你去重写许多用不着的方法，而这里我们只想监听滑动事件，于是我们就定义一个监听器继承SimpleOnGestureListener

```
 class myGestureListenner extends GestureDetector.SimpleOnGestureListener
    {
        @Override//开始事件与结束事件
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY)
        {//识别滑动
            if (e1.getX() - e2.getX() > 50)
            {
                Toast.makeText(MainActivity.this, "从右往左滑", Toast.LENGTH_LONG).show();
            }
            else if (e2.getX()-e1.getX()>50)
            {
                Toast.makeText(MainActivity.this, "从左往右滑", Toast.LENGTH_LONG).show();
            }
            return super.onFling(e1, e2, velocityX, velocityY);
        }
    }
```
在onFling方法就是你需要重写的，写入你自己动作的。两个MotionEvent 则是滑动事件的起始与结束，通过这两个的比较就可以得出滑动。最后再将这个几字写的监听器传入GestureDetector  的构造方法中，在ontouch方法中调用GestureDetector实例的onTouchEvent方法就可以将事件传给SimpleOnGestureListener进行自己的动作了。

