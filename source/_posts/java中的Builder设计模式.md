---
title: java中的Builder设计模式
date: 2017-03-22 15:37:43
tags: [Java,设计模式]
subtitle: 很久前就看到过这种“神奇的”构造器模式，但是一直没有深究，这次就一探Builder模式
---

# 前言

很久前就看到过这种“神奇的”构造器模式，但是一直没有深究，直到最近再使用 OKHTTP 的时候才发现这种用法非常简洁高效，特别适用于属性多的类。

# 旧的设计模式

在以往我们写一个类的时候，假如有十个属性，我们就要去写十个 Get 和 Set 方法，虽然说现在有 IDE 来直接生成 Get 和 Set 方法，但是在写这个类的实例的时候，我们仍然需要去一行一行地去为这个实例设置属性，这样读代码的人看起来会非常累，比如以下这个Student类。

```java
public class Student
{
    private String name;
    private int age;
    private int number;
    private String sex;
    private String address;
    private double GPA;
    private double height;
    private double weight;

    public Student(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    public int getAge()
    {
        return age;
    }

    public void setAge(int age)
    {
        this.age = age;
    }

    public int getNumber()
    {
        return number;
    }

    public void setNumber(int number)
    {
        this.number = number;
    }

    public String getSex()
    {
        return sex;
    }

    public void setSex(String sex)
    {
        this.sex = sex;
    }

    public String getAddress()
    {
        return address;
    }

    public void setAddress(String address)
    {
        this.address = address;
    }

    public double getGPA()
    {
        return GPA;
    }

    public void setGPA(double GPA)
    {
        this.GPA = GPA;
    }

    public double getHeight()
    {
        return height;
    }

    public void setHeight(double height)
    {
        this.height = height;
    }

    public double getWeight()
    {
        return weight;
    }

    public void setWeight(double weight)
    {
        this.weight = weight;
    }

}
```

```java
public static void main(String[] args)
    {
        Student student = new Student("Hugo", 19);
        student.setSex("Man");
        student.setAddress("Chengdu");
        student.setGPA(3.5);
        student.setHeight(180);
        student.setWeight(120);
        student.setNumber(1370000000);
    }
```

也许在 main 方法中这样看起来也还行，但是这是因为 Student 类中属性还不算太多，若是有50个属性那么岂不是要花50行来设置属性？那么在这种需求下，Builder 模式就应运而出。

# Builder 设计模式

所谓 Builder 设计模式就是要链式调用设置类的实例的属性，一个接一个地设置属性，最后调用 build 方法来返回实例。那么改进版地 Student 类如下图所示。

```java
public class Student
{
    private String name;
    private int age;
    private int number;
    private String sex;
    private String address;
    private double GPA;
    private double height;
    private double weight;

    public static class Builder
    {
        private String name;
        private int age;

        private int number;
        private String sex;
        private String address;
        private double GPA;
        private double height;
        private double weight;

        public Builder(String name, int age)
        {
            this.name=name;
            this.age=age;
        }

        public Builder number(int val)
        {
            number = val;
            return this;
        }

        public Builder sex(String val)
        {
            sex=val;
            return this;
        }

        public Builder address(String val)
        {
            address = val;
            return this;
        }

        public Builder GPA(double val)
        {
            GPA=val;
            return this;
        }

        public Builder height(double val)
        {
            height=val;
            return this;
        }

        public Builder weight(double val)
        {
            weight=val;
            return this;
        }

        public Student build()
        {
            return new Student(this);
        }

    }

    private Student(Builder builder)
    {
        name = builder.name;
        age = builder.age;
        number = builder.number;
        sex = builder.sex;
        address = builder.address;
        GPA = builder.GPA;
        weight = builder.weight;
        height = builder.height;

    }
}
```

这是个类包含一个内部类的结构，外部类仅仅提供属性以及一个私有的构造方法，这个构造方法一定要是私有并且只能让内部类访问，否则就失去了封装的意义。内部类的构造方法的参数可以填成这个类的必选的参数，比如这个例子中 Student 类中 name 和 age 属性就是必选属性，其余可缺省属性放在 Builder 类中，构成一个个以属性名命名的方法，并且最后必须要返回 this ，否则就不能进行链式调用了，这就是能进行链式调用的关键。最后 Builder 类用一个 build 方法来调用外部类的构造方法从而能够生成一个实例。以下就是main方法中的方法：

```java
public static void main(String[] args)
    {
        Student student = new Student.Builder("Hugo", 19).number(137000000)
       .sex("Man").address("Chengdu").GPA(3.5).height(180).weight(120).build();
    }
```

可以看到仅仅用了两行代码就完成了这个类的8个属性的设置，而且非常的简洁优雅，阅读代码的人看起来也比较轻松。

# 总结

在旧的设计模式中，因为构造过程被分到了几个调用中，因此在构造过程中这个实例可能处于不一致的状态，这就需要程序员付出额外的努力来确保它的线程安全。然而在 Builder 设计模式中仅仅通过一次调用就能完成所有属性的设置，这就避免了这个问题。

综而言之，如果一个类的属性并不多只有两三个时，使用旧的 Set 和 Get 模式也挺高效，写起来也比较轻松，然而如果一个类的属性有很多，那么为了线程安全和易于阅读，请用 Builder 模式。