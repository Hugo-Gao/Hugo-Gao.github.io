---
title: Spring总结二--配置验证码
date: 2018-01-31 15:48:06
tags: [Spring]
---

在许多网页中我们都可以看到验证码的存在，验证码就是用来进行人机识别的，防止脚本或爬虫无限制地请求网页导致资源浪费，本篇博客就是介绍如何在Spring和Springboot中配置验证码模块。

<!-- more-->

本博客使用的验证码包wiki地址https://code.google.com/archive/p/kaptcha/



## Maven导包

首先在Maven中导入使用验证码所需要使用到的包

```xml
<dependency>
            <groupId>com.github.penggle</groupId>
            <artifactId>kaptcha</artifactId>
            <version>2.3.2</version>
</dependency>
```

## Web.xml配置Servlet参数

接着我们进入Web.xml,来配置验证码相关的Servlet和具体的参数，就按照普通Servlet的配置方法，Servlet的类名为com.google.code.kaptcha.servlet.KaptchaServlet，在servlet-mapping中配置/Kaptcha截获验证码请求到Servlet，最后在Servlet中配置init-param参数。

```xml
<servlet>
        <servlet-name>Kaptcha</servlet-name>
        <servlet-class>com.google.code.kaptcha.servlet.KaptchaServlet</servlet-class>
        <!-- 有无边框 -->
        <init-param>
            <param-name>kaptcha.border</param-name>
            <param-value>no</param-value>
        </init-param>
        <!-- 图片颜色 -->
        <init-param>
            <param-name>kaptcha.textproducer.font.color</param-name>
            <param-value>red</param-value>
        </init-param>
        <!-- 图片宽度 -->
        <init-param>
            <param-name>kaptcha.image.width</param-name>
            <param-value>125</param-value>
        </init-param>
        <!-- 使用那些字符产生验证码 -->
        <init-param>
            <param-name>kaptcha.textproducer.char.string</param-name>
            <param-value>ACDEFHKPRSTWX345679</param-value>
        </init-param>
        <!-- 图片高度 -->
        <init-param>
            <param-name>kaptcha.image.height</param-name>
            <param-value>50</param-value>
        </init-param>
        <!-- 字体大小 -->
        <init-param>
            <param-name>kaptcha.textproducer.font.size</param-name>
            <param-value>43</param-value>
        </init-param>
        <!-- 干扰线的颜色 -->
        <init-param>
            <param-name>kaptcha.noise.color</param-name>
            <param-value>black</param-value>
        </init-param>
        <!-- 字符个数 -->
        <init-param>
            <param-name>kaptcha.textproducer.char.length</param-name>
            <param-value>4</param-value>
        </init-param>
        <!-- 字符字体 -->
        <init-param>
            <param-name>kaptcha.textproducer.font.names</param-name>
            <param-value>Arial</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>Kaptcha</servlet-name>
        <url-pattern>/Kaptcha</url-pattern>
    </servlet-mapping>
```

## 网页使用

在前端网页里只要向'项目地址/Kaptcha'发起请求就可以获得验证码了，具体代码如下：

```html
 <img id="captcha_img" alt="点击更换" title="点击更换"
                  onclick="changeVerifyCode(this)" src="../Kaptcha" />
```

由于随机产生的验证码可能不是很清楚，所以最好加一个点击事件点击验证码就可以更换一张验证码，js代码如下：

```js
function changeVerifyCode(img) {
    img.src="../Kaptcha?"+Math.floor(Math.random()*100);
}
```

## 后端验证

用户填写了验证码，向服务器发起了request，这个request就包含了用户输入的验证码，后台的工作就是需要验证验证码是否填写正确了，如果填写错误则需要立即返回错误信息告知用户，验证码的正确内容是存在session的Constants.KAPTCHA_SESSION_KEY中，所以我们只需要取出正确的验证码内容和用户输入的验证码内容就可以完成验证。

```java
public class CodeUtil
{
    public static boolean checkVerifyCode(HttpServletRequest request)
    {
        String verifyCodeExpected= (String) request.getSession().getAttribute(Constants.KAPTCHA_SESSION_KEY);
        String verifyCodeActual = HttpServletRequestUtil.getString(request, "verifyCodeActual");
        if (verifyCodeActual == null || !verifyCodeActual.toLowerCase().equals(verifyCodeExpected.toLowerCase()))
        {
            return false;
        }
        return true;
    }
}
```

可以写一个工具类来复用代码

## SpringBoot的验证码配置

SpringBoot的配置其实和Spring的配置是差不多的，只不过SpringBoot崇尚去xml化，以上所有在xml上书写的内容都需要在代码中配置。

首先在application.properties中把要用的参数信息提前写好

```properties
#Kaptcha相关
kaptcha.border=no
kaptcha.textproducer.font.color=red
kaptcha.image.width=135
kaptcha.textproducer.char.string=ACDEFHKPRSTWX345679
kaptcha.image.height=50
kaptcha.textproducer.font.size=43
kaptcha.noise.color=black
kaptcha.textproducer.char.length=4
kaptcha.textproducer.font.names=Arial
```

其次我们需要在@Configuration配置文件中自行配置一个Servlet来取代之前在Web.xml中的操作，其实具体操作很简单也和之前很相似,声明一个映射特定路径的 Servlet ，或是需要配置初始化参数的话，需要使用`ServletRegistrationBean`。

```java
@Bean(name="captchaProducer")
    public ServletRegistrationBean servletRegistrationBean() throws ServletException
    {
        ServletRegistrationBean servlet = new ServletRegistrationBean(new KaptchaServlet(), "/Kaptcha");
        servlet.addInitParameter("kaptcha.border", border);
        servlet.addInitParameter("kaptcha.textproducer.font.color", fcolor);
        servlet.addInitParameter("kaptcha.image.width", width);
        servlet.addInitParameter("kaptcha.textproducer.char.string", cString);
        servlet.addInitParameter("kaptcha.image.height", height);
        servlet.addInitParameter("kaptcha.textproducer.font.size", fsize);
        servlet.addInitParameter("kaptcha.noise.color", nColor);
        servlet.addInitParameter("kaptcha.textproducer.char.length", clength);
        servlet.addInitParameter("kaptcha.textproducer.font.names", fnames);
        return servlet;
    }
```

至此SpringBoot的验证码就配置完了。