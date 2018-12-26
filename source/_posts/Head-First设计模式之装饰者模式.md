---
title: Head First设计模式之装饰者模式
date: 2017-06-18 16:25:04
tags: [设计模式]
subtitle: 利用装饰者模式快速计算加上各种配方的咖啡的费用
---

假如你接到了一家咖啡店的单子，咖啡店店主要求你写一个系统能快速计算出各种咖啡，还要加上咖啡的不同配料(如摩卡，泡沫，卡布奇诺等等)的费用，如果在以前，可能会这么写——摩卡咖啡不加泡类，摩卡咖啡加泡类，咖啡不加泡类，卡布奇诺咖啡加泡类，卡布奇诺咖啡不加泡类。。。。。如果有一百种咖啡组合，那就要写100种类，如果还要再加上咖啡的大中小杯，那就完全是类爆炸了，手都会写痛的吧。但是如果有装饰者模式就可以拯救这一切，只需要把咖啡的配料类写出来，然后利用类的组合就可以完成这件复杂的事情。

## 定义

> 装饰者模式：在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象。

## 装饰者模式来解决

这就是我们需要建立的类图，可以看到咖啡类和配料类都继承自 Beverage 类。

![](http://img.my.csdn.net/uploads/201205/08/1336484312_7020.jpg)

特别值得注意的是配料类还持有一个 Beverage 的引用，这个引用很关键，就是通过利用这个引用，后来加入咖啡的配料才知道先前的咖啡值多少钱，然后再加上自己这个配料的价钱。

### 超类

```java
public abstract class Beverage
{
    String description="UnKnow Beverage";
    public String getDescription()
    {
        return description;
    }

    public abstract double cost();
}

```

Beverage 是一个抽象类，含有 description 属性来描述这个咖啡有哪些配料，而 cost 方法则是交给后代去实现。

### 咖啡本体类

```java
public class Espresso extends Beverage
{
    Size size;
    public Espresso(Size size)
    {
        description = "Espresso Coffee";
        this.size = size;
    }

    @Override
    public double cost()
    {
        if(size==Size.small)
        {
            return 0.80;
        }else if (size==Size.mid)
        {
            return 1.50;
        }else
            return 2.10;

    }

    @Override
    public String getDescription()
    {
        return size+" Espresso";
    }
}
```

Espresso 就是咖啡本体类，它就是一种具体的咖啡啦，在构造方法里传入需要大杯中杯还是小杯，然后在 cos t里按照杯数的大小计算价钱。

### 配料类

```java
public abstract class CondimentDecorator extends Beverage
{
    public abstract String getDescription();
}
```

CondimentDecorator 继承 Beverage 同样是一个抽象类，**它就是所有配料类的父类**，他将 getDescription 方法变为抽象的要求后代必须自己实现这个方法。

```java
public class Mocha extends CondimentDecorator
{
    Beverage beverage;
    public Mocha(Beverage beverage)
    {
        this.beverage = beverage;
    }
    @Override
    public double cost()
    {
        return 0.20 + beverage.cost();
    }
    @Override
    public String getDescription()
    {
        return beverage.getDescription()+", Mocha";
    }
}
```

Mocha 就是实现的具体配料类，最重要的是它持有 Beverage 类的引用，这个 Beverage 引用很特别，由于我们之前的类的继承关系，它既可以指代配料类，也可以指代咖啡类，通过这个传入的 Beverage 引用，我们就可以获得之前加入的配料信息(如价钱，描述)，然后在 getDescription 和 cost 这两个方法中体现出来。

你还可以根据这个配料类写出其他的配料类如加泡等等。

### 测试程序

现在我们就来写一个咖啡店类(主程序)来测试一下这个程序。

```java
public class CoffeeStore
{
    public static void main(String[] args)
    {
        Beverage beverage = new Espresso(Size.small);
        beverage = new Mocha(beverage);
        beverage = new Soy(beverage);
        beverage = new Whip(beverage);
        System.out.println(beverage.getDescription()+" cost "+beverage.cost());

        Beverage beverage2 = new Espresso(Size.mid);
        beverage2 = new Mocha(beverage2);
        beverage2 = new Soy(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription()+" cost "+beverage2.cost());

        Beverage beverage3 = new Espresso(Size.large);
        beverage3 = new Mocha(beverage3);
        beverage3 = new Soy(beverage3);
        beverage3 = new Whip(beverage3);
        System.out.println(beverage3.getDescription()+" cost "+beverage3.cost());
    }
}
```

其中 Soy，Whip 是我自己写的两个配料类。运行结果如下：

> small Espresso, Mocha, Soy, Whip cost 1.86

> mid Espresso, Mocha, Soy, Whip cost 2.56

> large Espresso, Mocha, Soy, Whip cost 3.16

可以看到 Size 和配料类都起了作用，如果要加新的配料只需要 ```  beverage = new Whip(XXXXX); ``` 即可。

### 扩展

在Java 的 io 包里也运用了装饰者模式，如 InputStream 类，如下如图所示。

![](http://img.my.csdn.net/uploads/201205/19/1337391225_5982.jpg)

## 总结

可以看到，无论是我们自己的咖啡店还是 Java 的 InputStream 类，都巧妙的利用了类的组合这种方法，实践了OO原则。运用装饰者模式可以有效地解决类的数量爆炸的问题。