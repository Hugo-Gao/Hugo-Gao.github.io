---
title: SharedPreferences的使用
date: 2016-08-30 12:59:16
tags: [Android]
---



# SharedPreferences的使用#

##简述SharedPreferences
SharedPreferences是一个及其方便的数据存储类，以键值对的形式存储在一个XML文件里，适合储存一些app的配置信息，相比SQLite的优点呢就是操作方便，你可以很容易的用SharedPreferences存储文件

##SharedPreferences对象创建方法
创建SharedPreferences对象有两种方法比较常用
-`SharedPreferences pref=PreferencesManager.getDefaultSharedPreferences.(MainActivity.this)` 这样呢系统就会自动生成一个以当前程序的包名来命名的XML文件。不能直接实例化一个SharedPreferences 对象 
- 可以自定义XML文件的名字，`SharedPreferences pref=getSharedPreferences("myName",""MODE_PRIVATE);`
## SharedPreferences的取出数据 ##
SharedPreferences 必须要用Editor对象来取出数据
```
Editor editor=pref.edit();
editor.putString("name","张三");
editor.putInt("age",13);
```
## SharedPreferences 移除数据 ##
SharedPreferences 非常简单，调用remove方法，参数是键名，

```
editor.remove("age");
```
这样就把age给删除了
## SharedPreferences最关键的一步 ——提交 ##
上面的不管是增加数据也好还是删除数据也好，最后都必须要`editor.commit()` 提交数据否则一切无效。
## 总结 ##
好了，SharedPreferences是非常简单的，不用的让我想起了上学期用C语言做购物系统时被C语言文件储存支配的恐惧，对比SharedPreferences ，此乃神器啊！

