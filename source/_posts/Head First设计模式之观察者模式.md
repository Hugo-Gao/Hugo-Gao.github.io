---
title: Head First设计模式之观察者模式
date: 2017-06-16 16:33:58
tags: [设计模式]
subtitle: 读《Head First设计模式》中的观察者模式的感悟与记录
---

早就对“设计模式”这个词有所耳闻,最早是在大一看《大话数据结构》这本书的背后看到这个系列还有个《大话设计模式》,我当时还以为这个“设计模式”恐怕是给设计师看的吧，当然这是望文生义了。其实设计模式官方解释是：

> 设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。

其实我认为就是大家总结的写代码的套路吧。

# 观察者模式

观察者模式从名字上来看大概就是一种通知与被通知的关系，其实代码思想也与其差不多，其核心思想就是有一个或N个观察者(Observer)和一个(或N个)被观察者(Observable 或 Subject)，观察者以订阅方式来观察被观察者，当被观察者接到更新时(程序员控制或代码自动发出)将通知所有观察者来接受更新的内容。这就有点像一群学生(Observer,观察者)和书店老板(Observable 或 Subject,被观察者)，当书店每次新进漫画杂志时，就会通知所有学生去购买。下面就看看如何用代码来实现这个事件。

## 所需接口

我们需要一共三个接口来定义Observer和Observable的行为，为别是 Observer 和 Observable 本身，还有DisplayElement来定义Observer的输出动作。

```java
public interface Observable
{
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}
```

```java
public interface Observer
{
    void update(String book);
}

```

```java
public interface DisplayElement
{
    void display();
}
```

接口定义的方法都很好理解，Observable 接口定义了将 Observer 对象加入或删除自己的队列里，还有通知队列里的 Observer。当书店有新书进入时，就会调用 Observer 的 update 方法。

## 实现接口

我们需要的类只有两个，当然就是 BookStore 对象和 Student 对象。

```java
public class BookStore implements Observable
{
    private List<Observer> observers;
    private String newBook;
    public BookStore()
    {
        this.observers = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer observer)
    {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer)
    {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers()
    {
        if(newBook!=null)
        {
            for (Observer observer : observers)
            {
                observer.update(newBook);
            }
        }
        newBook=null;
    }

    public void newBookComing(String bookName)
    {
        newBook = bookName;
        notifyObservers();
    }

}
```

```java
public class Student implements DisplayElement , Observer
{
    private String name;
    private List<String> hasBooks;
    private Observable observable;
    public Student(String name,Observable observable)
    {
        this.name = name;
        this.hasBooks = new ArrayList<>();
        this.observable = observable;
        observable.registerObserver(this);
    }

    @Override
    public void update(String book)
    {
        hasBooks.add(book);
        System.out.print(name+"买到了书<"+book+">   ");
        display();
    }

    @Override
    public void display()
    {
        System.out.println(name+"有书->"+hasBooks.toString());
    }
}
```

可以看到 BookStore 就是实现了很简单的 Observable 类的三个接口，其中在 notifyObservers 方法中依次调用队列中的 Observer 对象的 update 方法，BookStore 在 newBookComing 方法中设置新书的名字和调用 notifyObservers 方法。而 Student 则是在 update 方法中将新书加入自己已有的书籍集合中，最后调用 display 方法打印目前所有的书名。值得注意的是 Student 对象是把被观察者在自己的构造函数里传入，然后在构造函数里调用被观察者(这里就是 BookStore)的 registerObserver 方法将自己加入到队列中去。我认为当然也可以不这么做，也可以 `` bookStore.registerObserver(xiaohua); ``   来加入BookStore的队列，RxJava就是这么做的。

## 测试

现在建立一个测试类来跑一跑我们的代码：

```java
public class Main
{
    public static void main(String[] args)
    {
        BookStore bookStore = new BookStore();
        Student xiaoming = new Student("小明", bookStore);
        Student xiaohua = new Student("小华", bookStore);
        Student xiaozhang = new Student("小张", bookStore);
        bookStore.newBookComing("知音漫客");
        bookStore.newBookComing("故事会");
        bookStore.newBookComing("看天下");
    }
}

```

那么我们仅仅用了这么几行代码就跑出了我们想要的结果：

> 小明买到了书<知音漫客>   小明有书->[知音漫客]
>
> 小华买到了书<知音漫客>   小华有书->[知音漫客]
>
> 小张买到了书<知音漫客>   小张有书->[知音漫客]
>
> 小明买到了书<故事会>   小明有书->[知音漫客, 故事会]
> 小华买到了书<故事会>   小华有书->[知音漫客, 故事会]
> 小张买到了书<故事会>   小张有书->[知音漫客, 故事会]
> 小明买到了书<看天下>   小明有书->[知音漫客, 故事会, 看天下]
> 小华买到了书<看天下>   小华有书->[知音漫客, 故事会, 看天下]
> 小张买到了书<看天下>   小张有书->[知音漫客, 故事会, 看天下]

## 总结

如果以后有新书来的话，只需要一行代码 `` bookStore.newBookComing("知音漫客");`` 就可以完成所有的通知动作了。观察者模式将变化的部分与固定的部分分开从而实现了松耦合，这就是观察者模式的威力。java 在自己的 util 包里也内置了一个观察者模式，他的类名与我写的代码的类名是一样的，有所不同的是内置的 Observable 是一个类而不是一个接口，因为他帮我们把 registerObserver ，removeObserver ，notifyObservers 都实现了，其余的使用方法都大同小异。

[Github 源代码地址](https://github.com/Hugo-Gao/Design-pattern/tree/master/Observer) 

