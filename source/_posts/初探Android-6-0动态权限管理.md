---
title: 初探Android 6.0动态权限管理
date: 2016-11-01 13:19:23
tags: Android
subtitle: 由于6.0以前的系统都是在安装时询问权限，而6.0以后运行时询问，也就是动态权限管理，在6.0以上的系统分为Normal Permissions，和Dangerous Permissions。
---



我的APP简记在豌豆荚上线之后，我叫同学帮我下载测试一下，结果却惊奇的发现在我的小米2s测试机上跑得好好的，在他们同样是小米，同样是MIUI8的手机上却不能调出拍照，会直接退出程序，也就是传说中的闪退，于是我把手机连上Android Studio调试，结果出现了这样的Log：open failed: EACCES (Permission denied)；
       后来我想到小米2s和小米5虽然都是MIUI8，但是好像Android 版本不同，2s好像是Android 5.0，5则是6.0以上，我记得6.0以上的系统好像要动态申请权限，而不是只是简单的在AndroidManifest中申明就行了，于是我上网查了下。
​       

----------
## 权限分类 ##
由于6.0以前的系统都是在安装时询问权限，而6.0以后运行时询问，也就是动态权限管理，在6.0以上的系统分为Normal Permissions，和Dangerous Permissions。Normal Permissions就是不用动态申请的权限，而Dangerous Permissions就是需要动态申请的权限。
Normal Permissions：

```
ACCESS_LOCATION_EXTRA_COMMANDS
ACCESS_NETWORK_STATE
ACCESS_NOTIFICATION_POLICY
ACCESS_WIFI_STATE
BLUETOOTH
BLUETOOTH_ADMIN
BROADCAST_STICKY
CHANGE_NETWORK_STATE
CHANGE_WIFI_MULTICAST_STATE
CHANGE_WIFI_STATE
DISABLE_KEYGUARD
EXPAND_STATUS_BAR
GET_PACKAGE_SIZE
INSTALL_SHORTCUT
INTERNET
KILL_BACKGROUND_PROCESSES
MODIFY_AUDIO_SETTINGS
NFC
READ_SYNC_SETTINGS
READ_SYNC_STATS
RECEIVE_BOOT_COMPLETED
REORDER_TASKS
REQUEST_INSTALL_PACKAGES
SET_ALARM
SET_TIME_ZONE
SET_WALLPAPER
SET_WALLPAPER_HINTS
TRANSMIT_IR
UNINSTALL_SHORTCUT
USE_FINGERPRINT
VIBRATE
WAKE_LOCK
WRITE_SYNC_SETTINGS
```
Dangerous Permissions：
![这里写图片描述](http://img.blog.csdn.net/20161101092452396)

----------
## 动态申请 ##
例如现在我要调用相机，那么我就要知道我现在的系统是否时大于6.0.就调用如下方法：

```
private boolean OverLollipop(){
        return(Build.VERSION.SDK_INT>Build.VERSION_CODES.LOLLIPOP_MR1);
    }
```
如果返回True，就继续动态申请权限操作，如果是Lollipop以下，不需要动态申请，就直接调用相机即可。
然后申请权限：

```
String[] perms = {"android.permission.CAMERA"};

int permsRequestCode = 200; 

requestPermissions(perms, permsRequestCode);
```
这个requestPermissions和startActivityForResult很像，会返回到onRequestPermissionsResult方法

```
@Override

public void onRequestPermissionsResult(int permsRequestCode, String[] permissions, int[] grantResults){

    switch(permsRequestCode){

        case 200:

            boolean cameraAccepted = grantResults[0]==PackageManager.PERMISSION_GRANTED;
            if(cameraAccepted){
                //授权成功之后，调用系统相机进行拍照操作等
            }else{
                //用户授权拒绝之后，友情提示一下就可以了
            }

            break;

    }

}
```
在这个方法里进行拍照操作即可。

----------
## 优化逻辑 ##
但是难道用户每次都需要被这样弹出的申请权限的对话框打扰吗？很明显我们需要判断以前用户有没有允许过权限，如果之前就允许过了，就不用再申请了，那么我们把OverLollipop也集成到hasPermission中。

```
private boolean hasPermission(String permission){

        if(OverLollipop()){

            return(checkSelfPermission(permission)== PackageManager.PERMISSION_GRANTED);

        }

        return true;

    }
```
那么这样的话就不用重复申请权限了。


Thanks to [Android6.0运行时权限的处理及解决办法](https://segmentfault.com/a/1190000005065457)

[我的Github](https://github.com/Hugo-Gao)

