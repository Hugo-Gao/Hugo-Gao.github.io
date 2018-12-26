---
title: Jar包冲突解决方案调研
date: 2018-08-07 12:05:24
tags: [隔离]
---

## 一.jar包冲突的本质

Java 应用程序因某种因素，加载不到正确的类而导致其行为跟预期不一致。

## 二. jar包冲突的两种情况

1. 第一类jar包冲突问题（同一jar包版本不同）
2. 应用程序依赖的同一个 Jar 包出现了多个不同版本，并选择了错误的版本而导致 JVM 加载不到需要的类或加载了错误版本的类。
3. 出现该问题的三个必要条件：

- 依赖树中出现了同一个jar包的多个版本。
- 该jar包的多个版本之间接口发生了变化（类名，方法签名变化，方法行为变化）
- maven 的仲裁机制选择了错误的版本

1. 第二类jar包冲突问题（不同jar包的同一类版本不同）
2. 同样的类（类的全限定名完全一样）出现在多个不同的依赖 Jar 包中，即该类有多个版本，并由于 Jar 包加载的先后顺序（Maven的路径最短和覆盖优先原则）导致 JVM 加载了错误版本的类。如：假设有 **A** 、 **B** 、 **C** 三个jar包，由于 Jar 包依赖的路径长短、声明的先后顺序或文件系统的文件加载顺序等原因，类加载器首先从 Jar 包 **A** 中加载了该类后，就不会加载其余 Jar 包中的这个类了。
3. 出现该问题的三个必要条件：

- 同一类M出现在了两个（或两个以上）不同的jar包A、B中。
- 类M在A、B中有差异，行为不同。
- 加载的类M不是我们想要的。

## 三.解决方案

**方法一：手动排查**

1. 根据异常堆栈信息确定导致冲突的类名。
2. 通过mvn dependency:tree -Dverbose -Dincludes=<groupId>:<artifactId> 查看是哪个地方引入的jar包的版本。
3. 如果是第一类 Jar 包冲突，则可用 **<excludes>** 排除不需要的 Jar 包版本或者在依赖管理**<dependencyManagement>** 中申明版本。
4. 如果是第二类Jar包冲突，如果可以排除，则用 **<excludes>** 排掉不需要的那个 Jar 包，若不能排，则需考虑 Jar 包的升级或换个别的 Jar 包。

**方法二：通过自定义ClassLoader实现隔离** 在博客 [Java 中隔离容器的实现](http://codemacro.com/2015/09/05/java-lightweight-container/) 中提到了将每个jar包视为多个bundle，通过自定义classloader隔离运行，并且还可以实现多个jar包共享一个类。 该Demo通过启动一个KContainer类运行，KContainer类中主要包含一个BundleList和SharedClassList。每一个Bundle代表一个jar包或者class路径，Bundle类包含一个自定义的BundleClassLoader类（继承UrlClassLoader），由于不同BundleBundleClassLoader不同，可以实现**隔离运行**，而这个BundleClassLoader需要传入一个SharedClassList，classloader在加载一个类时，如果没有加载到，则可以从外部传进来的SharedClassList中加载，这样就实现了**多个jar包共享一个类**。

```
protected Class<?> findClass(String name) throws ClassNotFoundException {
   logger.debug(“try find class {}”, name);
   Class<?> claz = null;
   try {
     claz = super.findClass(name);
  } catch (ClassNotFoundException e) {
     claz = null;
  }
   if (claz != null) {
     logger.debug(“load from class path for {}”, name);
     return claz;
  }
   //如果没有加载到，从共享的类中加载
   claz = sharedClasses.get(name);
   if (claz != null) {
     logger.debug(“load from shared class for {}”, name);
     return claz;
  }
   logger.warn(“not found class {}”, name);
   throw new ClassNotFoundException(name);
}
```

需要共享出去给别人用的类可以通过在类路径下通过一个properties文件指定，在loadBundle的时候加载进SharedClassList。 

**方法三：轻量级隔离容器SOFAArk** SOFAArk同样也是使用不同的类加载器加载冲突的三方依赖包，进而做到在同一个应用运行时共存。 **SOFAArk通过Ark Plugin区分应用中哪些依赖包是需要单独的类加载器加载**。借助 SOFABoot 官方提供的 maven 打包插件，开发者可以把若干普通的 JAR 包打包成 Ark Plugin 供应用依赖或者把普通的 Java 模块改造成 Ark Plugin。应用使用添加 maven 依赖的方式引入 Ark Plugin，运行时，SOFAArk 框架会自动识别应用的三方依赖包中是否含有 Ark Plugin，进而使用单独的类加载器加载。其运行时逻辑图如下：

![image-20180807120802894](https://ws3.sinaimg.cn/large/0069RVTdgy1fu10cfi5mkj31b60jq17q.jpg)

1. SOFAArk 容器处于最底层，负责启动应用。
2. 每个 Ark Plugin 都由 SOFAArk 容器使用独立的类加载器加载，相互隔离。
3. 应用业务代码及其他非 Ark Plugin 的普通三方依赖包，统称为 Ark Biz。需要依赖下层的Ark Plugin。

在Ark Plugin的POM文件中，会配置导出类和导入类的配置。导出类即把 Ark Plugin 中的类导出给 Ark Biz 和其他 Ark Plugin 可见。对于 Ark Plugin 来说，如果需要使用其他 Ark Plugin 的导出类，必须声明为自身的导入类。

 **方法四：阿里的Pandora隔离容器** 阿里的Pandora是闭源的，网上资料比较少。 可以从阿里的一次演讲PPT上得知，Pandora仍然是基于ClassLoader实现的 Pandora这类的隔离容器的缺点：

![image-20180807120821154](https://ws4.sinaimg.cn/large/0069RVTdgy1fu10cp7kxkj31ca0m8n5b.jpg)

1. 使用方式复杂，难以理解。
2. 启动慢，用户无法按需选择
3. 调试困难
4. 部署和运维困难



下一篇文章将着重分析蚂蚁金服的SOFA-ARK容器的使用和源码分析