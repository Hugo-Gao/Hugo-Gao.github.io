---
title: Head First设计模式之单例模式
date: 2017-07-09 10:00:46
tags: [设计模式]
---

单例模式的鼎鼎大名很早以前都听说过了，数据库的连接池就是采用了单例模式，但是一直不知道单例模式到底是什么，我当时想难道是只有一个实例吗，那又怎么可能呢？随便 new 一个不是又有很多实例吗？

> 单例模式的定义：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

## 如何保证只有一个实例

看到单例模式的实现后有点好笑，原本以为是一个多么高大上的代码技巧才能实现整个类只有一个实例，后来才发现原来这么简单，只需要把构造函数设置为私有的。

```java
public class Singleton
{
    private static Singleton uniqueInstance;

    private Singleton()
    {
        System.out.println("Initial");
    }

    public  static Singleton getInstance()
    {
        if (uniqueInstance == null)
        {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
   
}
```

看吧，就是这么简单，构造函数设置为私有的，让这个单例类不能够随便用 new 直接初始化一个实例了，并且设置一个静态 Singleton 并且他也是私有的，他就是这个单例模式中的单例，只能通过自己创建的 getInstance 函数来访问他，这就解决了使用 new 无限创建实例的问题。

## 懒汉式单例模式

但是上面的单例模式并不是线程安全的，假如两个线程同时进入到 if (uniqueInstance == null) 这行代码，那么他们都会同时判断此时 uniqueInstance 没有实例化，那么就会都进入 uniqueInstance = new Singleton();  这行代码，也就意味着会把这个单例实例化两次，这就违反了单例模式的原则，那么就需要把这个函数加上同步锁:

```java
public class Singleton
{
    private static Singleton uniqueInstance;

    private Singleton()
    {
        System.out.println("Initial");
    }

    public  synchronized static Singleton getInstance()
    {
        if (uniqueInstance == null)
        {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

但是 synchronized 对于性能有较大的影响，毕竟我们只是第一次创建的时候才需要关注线程安全。

## 饿汉式单例模式

还有一种办法可以就解决线程安全问题：

```java
public class Singleton
{
    private static Singleton uniqueInstance=new Singleton();

    private Singleton()
    {
        System.out.println("Initial");
    }

    public  synchronized static Singleton getInstance()
    {
        return uniqueInstance;
    }

}
```

这种方法的话就完全不同担心线程同步的问题了，但是却不能做到用时才创建的特性了。那么有没有一种方式既能有很好的性能又能做到用时才创建的特性呢？

## 双重检查加锁

```java
public class Singleton
{
    private volatile static Singleton uniqueInstance;

    private Singleton()
    {
        System.out.println("Initial");
    }

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

看到这里也许会问，为什么要加两次```if (uniqueInstance == null)``` 检查呢？我们用排除法来分析一下。

1. 如果不加第一个```if (uniqueInstance == null)``` ，那么每次进入函数都会遇到 synchronized 锁，这就又导致了性能的损耗，和我们前面的懒汉式单例没有什么区别了。
2. 如果不加第二个```if (uniqueInstance == null)``` ,那么两个线程同时进入第一个```if (uniqueInstance == null) ``` 时，如果这时单例模式的第一次创建，那么由于两个线程同时进入也就会同时判断 ```uniqueInstance == null``` 为true，那么接下来排队进入锁区，没有第二个```if (uniqueInstance == null)``` 控制的话，那么```uniqueInstance = new Singleton();``` 会被两个线程同时执行两次。这就又违反了单例模式的原则。

同时需要注意成员变量 uniqueInstance 一定要设置为 **volatile** ，多个线程才能正确地处理 uniqueInstance 变量。

## 总结

单例模式确保程序中一个类最多只有一个实例，并提供访问这个实例的全局访问点，确定在性能上和资源上的限制，正确的选择不同单例模式的实现以解决多线程的问题。