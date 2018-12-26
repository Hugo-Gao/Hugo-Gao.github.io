---
title: Servlet必知必会
date: 2017-09-22 11:59:23
tags: [Java,Servlet]
---

## Servlet简述

Servlet 是一个 Java 类，通常在 Web 应用 MVC 模式中担任 Controler 角色，它的任务是得到一个客户的请求，再发回一个响应，在接受客户请求后，调用模型对请求数据进行处理，将处理后的数据设置为请求属性，再发送到控制页面的 JSP 中。下面就通过一次完整的HTTP请求来介绍 Servlet 是如何工作的。

## 一次HTTP请求的到来

容器全盘控制着 Servlet 的一生，当用户点击一个链接比如：http://localhost:8080/testWeb/action.do  后，这个请求到达服务器和容器，Tomcat看到用户请求的是 testWeb 这个Web应用，于是到 testWeb 目录下去找 Web.xml (我们一般称之为部署描述文件,即DD)，在DD中找 servlet-mapping 元素，与之匹配的 url-pattern,根据这个 url-pattern 的 servlet-name 映射到真正的 servlet-class ，容器根据此依据调用相应的 Servlet 类。

```xml
    <servlet>
    	<servlet-name>ActionServletName</servlet-name>
    	<servlet-class>com.gyf.web.ActionServlet</servlet-class>
    </servlet>

    <servlet-mapping>
    	<servlet-name>ActionServletName</servlet-name>
    	<url-pattern>/action.do</url-pattern>
    </servlet-mapping>
```

## Servlet 的生命周期

通过上述过程，容器找到了应该调用的 Servlet ，如果这个 Servlet 类还没有被加载，容器会从头调用Servlet 的生命周期：

1. 首先加载目标类(ActionServlet.class),接着调用Servlet的默认无参构造函数(注意我们不需要去覆盖Servlet的构造函数)。


2. 接着调用 init() 方法，这个方法在 Servlet 的一生中只调用一次，如果你有其他的初始化代码(如得到一个数据库连接)。
3. 接着调用 Service()方法，如果容器当初发现 Servlet 类已经被加载就会跳过前面两个步骤直接进入这个步骤，每次有HTTP请求到来时，都会调用目标 Servlet 的Service 方法，这个Service方法每次调用都会开启一个新线程，根据HTTP请求的类型决定是继续调用doGet(),还是doPost()。Service 方法在 Servlet 的一生中可以调用多次。
4. 最后调用destroy()方法杀死这个 Servlet 类，在这个方法中可以进行垃圾回收清理资源。

![Servlet生命周期](https://i.loli.net/2017/09/22/59c4941846fc0.png)

注意在每个JVM上，每个特定的 Servlet 类只会有一个实例，所以不存在对于Servlet的每个实例这种说法。

![Servlet 的继承结构和方法](https://i.loli.net/2017/09/22/59c497fa9c2db.png)

## ServletConfig 与 ServletContext

我们在 Servlet 输出一些固定信息时，可能会这样做

```java
PrintWriter out=response.getWriter();
out.println("59833576*@qq.com");
```

如果我们要修改邮箱地址怎么办，就只有修改源代码，停止 Web 应用，重新编译，再启动 Web 应用。非常繁琐，在实际生产环境中，能不去动源代码就不去动源代码，那么我们可以用 ServletConfig 与 ServletContext 来解决这个问题。

#### ServletConfig

在部署描述文件中这样写：

```xml
<servlet>
	<servlet-name>ActionServletName</servlet-name>
	<servlet-class>com.gyf.web.ActionServlet</servlet-class>
  	<init-param>
  		<param-name>adminEmail</param-name>
      	<param-value>59833576*@qq.com</param-value>
    </init-param>
</servlet>

<servlet-mapping>
	<servlet-name>ActionServletName</servlet-name>
	<url-pattern>/action.do</url-pattern>
</servlet-mapping>
```
可以看到 init-param 是在 servlet 标签内的，这也意味着只能在该 servlet 类中使用，并不是全局的。

在 Servlet 中我们这么使用：

```java
out.println(getServletConfig().getInitParameter("adminEmail"));
```

注意，不能在构造函数中调用这个方法，在init()后，Servlet 才得到ServletConfig对象。

####ServletContext

ServletContext是全局有效的，这一点从它的部署位置就可以看出来：

```xml
<web-app 
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
    version="2.4">
    <servlet>
    	<servlet-name>Test</servlet-name>
    	<servlet-class>com.gyf.web.GetJarServlet</servlet-class>
    </servlet>
  
    <servlet-mapping>
    	<servlet-name>Test</servlet-name>
    	<url-pattern>/servlet-api.jar</url-pattern>
    </servlet-mapping>
  
    <context-param>
        <param-name>adminEmail</param-name>
        <param-value>59833576*@qq.com</param-value>
    </context-param>

</web-app>
```

context-param 就在web-app 标签下，所以它对所有的 Servlet 都是有效的，在 Servlet 代码中：

```java
out.println(getServletContext().getInitParamter("adminEmail"));
```

要注意区分ServletContext 和 ServletConfig的区别和写法。

##监听者Listener

如果我们想在应用部署时就马上做一个事情要怎么做呢？这是我们就需要一个监听者。监听者分为很多种，每种的用途用法都不一样，比如刚才说的应用部署时就要做一个事情就需要 ServletContextListener。

#### ServletContextListener

```java
package com.gyf;
import javax.servlet.*;

public class MyServletContextListener implements ServletContextListener
{
    public void contextInitialized(ServletContextEvent event)
    {
        ServletContext sc=event.getServletContext();
        String dogBreed=sc.getInitParameter("breed");
        Dog d=new Dog(dogBreed);
        sc.setAttribute("dog",d);
        //得到数据库连接
        //将数据保存进数据库
    }
    public void contextDestroyed(ServletContextEvent event)
    {
        //关闭数据库
    }
}
```

方法很简单，我们只需要扩展 ServletContextListener  接口就行了，并将这个 .class文件 放进classes文件夹，最后在部署描述文件中写上该监听类的名字就行了：

```java
<listener>
        <listener-class>
            com.gyf.MyServletContextListener
        </listener-class>
    </listener>
```

#### 还有很多监听者类可供使用

![](https://i.loli.net/2017/09/22/59c4b9b4e05cf.png)

## 上下文初始化参数线程安全

现在有了一个问题，既然 ServletContext 是全局可见的，那么如何保证保证其线程安全呢？有的同学可能会想在 doPost() 或 doGet() 方法上加 synchronized ，但是仔细想一想，这样做只能保证每个Servlet 只有一个线程在运行，但是一个Web应用可以有很多个Servlet，这样的话仍然不能保证它的线程安全。正确方法应该是这样做的：

```java
synchronized (getServletContext())
{
    getServletContext().setAttribute("foo",22);
    getServletContext().setAttribute("bar",42);
}
```

每次使用ServletContext都要求先获得它的锁，这种方法才奏效