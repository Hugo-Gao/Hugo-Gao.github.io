---
title: Service的两种启动方法
date: 2016-09-07 13:04:41
tags: [Android]
---



Service是Android的四大组件之一，**四大组件每一件都要在AndroidManifest中进行注册。**， Service可以脱离于Activity运行，也就是说不受Activity的控制，也可以和Activity绑定在一起，与Activity共存亡。这就是Service的两种启动方法。下图是Service的生命周期。![这里写图片描述](http://images.cnitblog.com/blog/325852/201303/24233205-ccefbc4a326048d79b111d05d1f8ff03.png)

----------
## Start方法启动 ##
首先创建一个Service类，继承自Service，取名就叫MyStartService吧，根据上图左可知，服务一旦创建会依次调用onCreate(),onStartCommand(),onDestroy().所以在自己定义的Service类中要把初始化的方法写在onCreate()中，onCreate()在这个Service被销毁前只调用一次，将主要的方法写在onStartCommand()中,这个方法可以被多次调用。启动一个Service的方法和启动一个广播接收器很像，放入Intent中，再调用startService()方法即可。

----------
## Bind方法启动 ##
这个方法与上一个方法的不同是这个方法与Activity是绑定在一起的，与Activity共存亡，当Activity被挂起后台时，Service不会退出。他的生命周期：`context.bindService()->onCreate()->onBind()->Service running-->onUnbind() -> onDestroy() ->Service stop
` 那么接下来就创建一个MyBindService()类继承字Service，如代码所示：

```
public class MyBindService extends Service
{
    @Override
    public void onDestroy()
    {
        super.onDestroy();
        Log.d("info", "BindService-----onDestroy Start");
    }

    @Override
    public void onCreate()
    {
        super.onCreate();
        Log.d("info", "BindService-----onCreate Start");
    }

    @Override
    public boolean onUnbind(Intent intent)
    {
        Log.d("info", "BindService-----onUnbind Start");
        return super.onUnbind(intent);

    }

    public class MyBinder extends Binder
    {
        public MyBindService getService()
        {
            return MyBindService.this;
        }
    }
    @Nullable
    @Override
    public IBinder onBind(Intent intent)
    {
        Log.d("info", "BindService-----onBind Start");
        return new MyBinder();
    }

    public void Play()
    {
        Log.d("iinfo", "播放");
    }

    public void Pause()
    {
        Log.d("iinfo", "暂停");
    }

    public void Previous()
    {
        Log.d("iinfo", "上一首");
    }

    public void Next()
    {
        Log.d("iinfo", "下一首");
    }
}
```
然后在MainActivity中依然用Intent传入bindService()`bindService(intent2, serviceConnection, Service.BIND_AUTO_CREATE);` 那么这里的serviceConnection就很关键了，他就是连接Activity与Service的桥梁，使得在Activity中也可以调用Service中的方法。首先new一个ServiceConnection出来

```
ServiceConnection serviceConnection=new ServiceConnection()
    {
        @Override//当启动源于Service成功连接
        public void onServiceConnected(ComponentName name, IBinder binder)
        {
           bindService =((MyBindService.MyBinder)binder).getService();
        }

        @Override//当启动源于Service意外断开，意外！
        public void onServiceDisconnected(ComponentName name)
        {

        }
    };
```
bindService在MainActivty中定义，可以看到在onServiceConnected中就得到了Service对象，通过这个对象，我们就可以调用我们自己写在Service中的方法。然而这个对象是如何得到的呢？可以看到onServiceConnected有个参数是Ibinder，他就死在Service中的onbinder返回的，于是我们可以定义一个内部类继承Binder，里面仅有一个方法返回this。在onBinder()方法中就可以返回这个类，于是也就等于返回这个Service本身了。

