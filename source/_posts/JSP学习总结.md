---
title: JSP学习总结
date: 2017-09-14 19:12:55
tags: [Java,JSP]
---

这个学期在从 Android 转向 Java 后端，在学习《Head First Servlet & JSP》时遇到了一点困难，书上花了将近两百页来介绍JSP和其相关的语言语法特性，导致脑容量完全不够，记不住这么多东西，所以写一篇博客来记录一下这部分的内容。

## JSP到底是什么

那么 JSP 到底是个什么，在 Servlet 组成的 MVC 中模型中，JSP 就代表着 View ,JSP控制着视图显示，一切逻辑在 Servlet(Controller) 和Model 中解决完毕后，转发到 JSP 中生成视图文件(HTML),大家也都知道在 Java 中写 HTML 代码是很麻烦的，因为有很多转义字符需要转义，所以人们想不如在 HTML 中写 Java 算了，于是就诞生了JSP。人们编写的 JSP 会被转换为.java文件，然后编译为 .class,最后加载并初始化为一个 Servlet。

## scriplet

scriplet是最简单的JSP元素了，它只需要在 HTML中插入<% %>标记，在标记中你可以任意输入 Java 代码。

## 指令

JSP一共有三个指令，分别是 page,taglib,include。

### 1.page指令

page指令一共有13个属性，但是常用的就那么几个。import 属性：这个属性用来导包，用来在 scriplet 中使用。contentType属性：如果不把这个属性设置为UTF-8，那么显示中文就会出现问题。

代码可以是这样  <%@ page import="foo.*,java.util.*" contentType="UTF-8" %>

### 2.taglib 指令

目前我学的 taglib 指令只有一个作用：用来导入EL函数。当我们在EL中想要调用其它 Java 类时，就会用到这个指令，步骤如下。

a.创建你想要调用的类:

```java
package foo;

public class DiceRoller
{
    public static int rollDice()
    {
        return (int) ((Math.random()*6)+1);
    }
}
```

 b.创建TLD库描述文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<taglib xmlns="http://java.sun.com/xml/ns/j2ee"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/j2ee/dtds/web-jsptaglibrary_2_0.xsd"  
        version="2.0">
<tlib-version>1.2</tlib-version>
<uri>DiceFunctions</uri>
    <function>
        <name>rollIt</name>
        <function-class>foo.DiceRoller</function-class>
        <function-signature>
            int rollDice()
        </function-signature>
    </function>
</taglib>
```

c.JSP:

```jsp
<%@ taglib prefix="mine" uri="DiceFunctions" %>
<html>
<body>
Hello world
${mine:rollIt()}
</body>
</html>
```

我们创建TLD的目的就是为了让EL语言在TLD中找要找的函数，并且这个函数必须是静态的，taglib 指令为<%@ taglib prefix="mine" uri="DiceFunctions" %> ,可以看到 uri 属性与 TLD中的 uri 标签对应，代表 mine 这个命名空间调用 rollIt()函数。

### 3.include指令

如果我们有很多个 JSP 都有重复的地方怎么办呢，比如说都有页眉与页尾，那么就可以把这些地方抽取出来作为公用部分，使用include 指令来复用这些页面。

 <%@ include file="Header.jsp" %> 

## 表达式

表达式是形如<%= out.println("Hello World") %> 的一串代码，标签内放置 Java 代码，但是值得注意的是，这段 Java 代码必须有返回值，不能是 void 的。并且最后不能有分号。

## 声明

我们说了，JSP最后仍然会转换为Java代码，并且前面介绍的 scriplet 所定义的变量是局部变量，那么如果我想定义一个全局变量怎么办呢，这就需要用到声明，形如<%! int i=0;%> 注意这句代码里就必须要有分号了。标签类不仅可以定义变量，也可以定义一个函数。

## 隐式对象

第一次看到隐式对象有点不明白是什么东西，后来明白了，说白了就是我们在创建一个Servlet时，我们能够得到 request，response，ServletContext,ServletConfig,Session等等东西，那么JSP最后也是会变成Servlet的，我们在JSP中如何使用这些隐式对象呢。原来JSP早就为我们创建好了。证据呢，请看容器将JSP转化为Servlet的源码：



```java
    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
```

可以看到我们可以直接使用session，application，config，out，pageContext 等隐式对象而不用自己声明了，而request 和response 也是生成的——jspService()函数的参数，我们也可以直接使用，因为在Servlet 中的requestDispatcher.forward(request,response) 参数传到这里来了。

```java
 public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
```

## JSP标准动作

JSP标准动作可以不用脚本，也就是不使用Java代码来完成一些编程工作，例如我们在 request 中有一个"person"属性，对应一个foo.Person 对象，如果我们想在JSP中获取这个对象，我们可以很轻松的使用 scriptlet 来写一段 Java 代码获取。但是对于不懂Java 的前端人员怎么办呢？我们就可以使用标准动作：

```jsp
<jsp:useBean id="person" class="foo.Person" scope="request">
  <jsp:setProperty name="person" property="name" value="Fred" />
</jsp:useBean>
Person is <jsp:getProperty name="person" property="name" />
```

上面的这串代码分别做到:

< jsp:useBean > 声明使用foo.Person类，这个对象存在request，他的键为"person"。

< jsp:setProperty > 做到如果 request 没有这个对象，那么就自己创建一个，名为 person ，并且把他的name属性默认地设置为Fred。

 < jsp:getProperty > 做到打印出 person 对象的 name 属性。

```jsp
<jsp:include page="Header.jsp" >
  <jsp:param name="subTitle" value="This is a subtitle" />
</jsp:include>
```

< jsp:include > 标签作用与上面讲的 include 指令很相似，都是复用重复的的jsp页面。< param>标签可以向被包含的jsp传送一个参数，在被包含的标签用EL语言\${param.subtTtle} 就可以获得了。与 include 指令不同的是， include 指令只适用于静态页面，因为使用 include 指令就相当于直接把被包含的页面代码原封不动的搬到目标JSP代码中，而< jsp:include > 标签可以适用与动态元素，因为它是动态的将被包含的代码响应的页面放在目标位置。

## EL语言

EL语言十分强大，它几乎可以做所有标准动作能做到的事情，而且还更加简单，例如 request 中有一个person 对象，person 对象有一个 dog 对象，dog 对象 有一个 name 性质，我们要打印这个 dog 的 name 怎么办呢。使用 EL 就非常简单 \${perosn.dog.name} 就完成了这个任务，在EL表达式中，我们用request 的Attribute的键直接作为EL的对象使用。例如在Servlet中  request.setAttribute("person",xiaohong) 那么我们在 EL 中直接只用 person 来代替 xiaohong 这个 Person对象了，同理，如果request.setAttribute("map",aHashMap) 将一个 Map 作为参数，那么我们直接 \${map["key"]}或者 \${map.key},这两者的区别是\${map.key} 点号后面必须符合java 命名规则。

## EL 的隐式对象

EL语言这么强大，当然也有自己的隐式对象了：

|      EL隐式对象      |                    作用                    |
| :--------------: | :--------------------------------------: |
|    pageScope     |               页面作用域的**属性**               |
|   requestScope   |               请求作用域的**属性**               |
|   sessionScope   |               会话作用域的**属性**               |
| applicationScope |               应用作用域的**属性**               |
|      param       | {param.id} 相当于request.getParameter("id") |
|   paramValues    |            如果参数的值为一个集合，应该用此对象            |
|      header      |       相当于request.getHeader("...");       |
|   headerValues   |           适用于header的某个值为集合的情况            |
|    initParam     |            调用web.xml中的上下文初始参数            |
|      cookie      |                 获取cookie                 |
|   pageContext    |         可以通过pageContext获取以上所有对象          |

再次提醒一定要注意 requestScope 与 param 的区别，也就是属性和参数的区别，参数是用户提交的表单了的parameter，而属性是编程人员自己设置的Attribute。

## 获取上下文参数的几种方法

### 1.脚本

```jsp
<%= application.getInitParameter("mainEmail")%>
```

### 2.EL使用initParam

```jsp
${initParam.mainEmail}
```

###3.EL使用pageContext

```jsp
${pageContext.request.getServletContext().getInitParameter("mainEmail")}
```

## 获取表单参数的几种方法

### 1.脚本

```jsp
 <%= request.getParameter("foo")%>
```

### 2.EL使用param

```jsp
${param['foo']}
```

### 3.EL使用pageContext

```jsp
${pageContext.request.getParameter("foo")}
```
