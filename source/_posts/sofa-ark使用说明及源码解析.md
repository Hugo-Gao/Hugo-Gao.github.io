---
title: sofa-ark使用说明及源码解析
date: 2018-08-07 12:23:34
tags: [隔离]
---

## 一.使用方法及示例

> 简介：当引入二方依赖包或三方依赖包时，可能出现外部依赖jar包与自己的工程需要依赖的冲突，或者多个二方三方依赖包互相冲突。这时候就需要一个隔离容器对他们进行隔离，其依赖的原理就是jvm认为不同classloader加载的类即使包名类名相同，也认为他们是不同的。sofa-ark将需要隔离的jar包打成plugin，对每个plugin都用独立的classloader去加载。

>  （温馨提示：若对sofa-ark不太了解的，最好先去看看[官方文档](https://alipay.github.io/sofastack.github.io/)，简单了解下）

使用的基本步骤：

1. 在会发生冲突的jar包的POM文件加入sofa-ark提供的maven插件，将其打成特定格式的jar包（plugin）。
2. 在外部工程按照约定引入jar包。如果外部工程想打包成可执行的jar（fat-jar），还需要加入特定的maven插件。
3. 直接运行。



名词解释：

- Ark Container: Ark 容器，是组件 SOFAArk 的核心，运行 Ark 包时，Ark 容器会最先启动，负责应用运行时的管理，**主要包括构建 Ark Plugin 和 Ark Biz 的类导入导出关系表**、启动并初始化 Ark Plugin 和 Ark Biz、管理 Ark Plugin 服务的发布和引用等等。
- Biz：即业务工程，该工程引用一个或多个外部jar包。
- plugin：会发生冲突的外部依赖jar包经过提供的Maven插件打成的fat-jar包。运行时，由独立的类加载器加载，因此有隔离需求的 Jar 包建议打包成 Ark Plugin 供应用依赖。

### 1.0 sofa-ark原理

在sofa-ark中，使用container容器启动外部工程（Biz）和冲突jar包（plugin），sofa-ark的plugin maven插件将冲突的jar包打包成为plugin，外部工程只能引用plugin exported出来的类，并且这个类是由独立的`PluginClassLoader`加载的，从而解决了jar包冲突的问题。

![image-20180724204016919](https://ws4.sinaimg.cn/large/006tNc79gy1ftl8h1oa5cj317i0wqk2s.jpg)

### 1.1 冲突示例

![jar冲突](https://ws4.sinaimg.cn/large/006tKfTcly1ftn86vbz40j30de0b53yl.jpg)

### 1.2 安装sofa-ark

从[sofa-ark官网](https://github.com/alipay/sofa-ark ) 将整个工程下载下来，在最外层pom.xml所在路径执行`mvn package`和`mvn install` 将sofa-ark依赖jar包安装到本地。

### 1.3 创建基础依赖Jar包（冲突的包）

自己创建myjar工程，先后正常打包两个版本安装到本地maven仓库。

如:v1版本

![image-20180724185922039](https://ws4.sinaimg.cn/large/006tNc79ly1ftl5k1wwwij31j40kethb.jpg)

v2版本

![image-20180724190102751](https://ws1.sinaimg.cn/large/006tNc79gy1ftl5lsuopuj31f00iu473.jpg)

### 1.4 创建service包（plugin）

创建如图所示两个工程：

<img src="https://ws2.sinaimg.cn/large/006tNc79gy1ftl5qc7lkvj30ne1304b8.jpg" width="50%" height="50%" />

在`myJarservice-v1`和`myJarservice-v2`的pom中分别引用之前写的两个基础依赖jar包。然后在`MyJarService1`和`MyJarService2`中分别引用对应版本的myjar包里的方法。

如MyJarService1.java：

![image-20180724190836319](https://ws3.sinaimg.cn/large/006tNc79ly1ftl5to23zyj31g20lo48b.jpg)

MyJarService2.java

![image-20180724191026890](https://ws1.sinaimg.cn/large/006tNc79ly1ftl5vkwevlj31h00lc48c.jpg)

**注意：这里两个Service引用的是不同版本的myjar。**

接下来需要对两个工程打包，和平常打包不一样的是需要加入sofa-ark-plugin的maven插件。两个工程下的pom.xml都要加入：

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>com.alipay.sofa</groupId>
                <artifactId>sofa-ark-plugin-maven-plugin</artifactId>
                <version>0.4.0-SNAPSHOT</version>
                <executions>
                    <execution>
                        <id>default-cli</id>
                        <goals>
                            <goal>ark-plugin</goal>
                        </goals>

                        <configuration>
                            <!-- configure exported class -->
                            <exported>
                                <!-- configure class-level exported class -->
                                <classes>
                                 
           <class>com.netease.sofaservice.MyJar1Service</class>
                                </classes>
                            </exported>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

在`exported`标签里写出要对外提供的方法，外部要引用的所有方法都必须写在这里，可以以类(`<classes>`)为单位和包(`packages`)为单位导出。

到parent工程路径下`mvn package`和`mvn install` 即可。

### 1.5 外部工程引用(Biz)

新建一个工程,在pom.xml引入以下依赖：

```xml
<dependencies>
        <dependency>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-ark-support-starter</artifactId>
            <version>0.4.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.netease</groupId>
            <artifactId>myjarservice-v1</artifactId>
            <classifier>ark-plugin</classifier>
            <version>1.0</version>
        </dependency>

        <dependency>
            <groupId>com.netease</groupId>
            <artifactId>myjarservice-v2</artifactId>
            <classifier>ark-plugin</classifier>
            <version>1.0</version>
        </dependency>

        <dependency>
            <groupId>com.netease</groupId>
            <artifactId>myjarservice-v1</artifactId>
            <version>1.0</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>com.netease</groupId>
            <artifactId>myjarservice-v2</artifactId>
            <version>1.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

```

**注意要添加`<classifier>`标签**，因为IDE识别不了ark-plugin的jar包，所以需要再引入一个范围为`provided`的jar包。

然后pom.xml还要添加maven插件：

```xml
<build>
        <plugins>
            <plugin>
                <groupId>com.alipay.sofa</groupId>
                <artifactId>sofa-ark-maven-plugin</artifactId>
                <version>0.4.0-SNAPSHOT</version>
                <executions>
                    <execution>
                        <id>default-cli</id>

                        <!--goal executed to generate executable-ark-jar -->
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                        <configuration>
                            <!--specify destination where executable-ark-jar will be saved, default saved to ${project.build.directory}-->
                            <outputDirectory>./</outputDirectory>

                            <!--default none-->
                            <arkClassifier>executable-ark</arkClassifier>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>
```

然后再随便写个类引入两个版本service包的类使用即可，在main函数入口处需要加上一句：

`SofaArkBootstrap.launch(args);`

![image-20180724203423465](https://ws2.sinaimg.cn/large/006tNc79gy1ftl8awqnjdj31h20l8wqs.jpg)

可以看到打印结果:

![image-20180724203546696](https://ws1.sinaimg.cn/large/006tNc79gy1ftl8ccs12fj31kw0cch3i.jpg)

## 二.sofa-ark-plugin-maven-plugin插件原理分析

### 2.1 查看plugin jar包内容

使用`sofa-ark-plugin-maven-plugin `  Maven插件即可将jar包打包成可在container中隔离加载的jar包（如myjarservice-v1-1.0-ark-plugin.jarr和myjarservice-v2-1.0-ark-plugin.jar）。进入本地maven仓库myJarservice-v1工程所在位置，可以看到maven打了两个包：一个是maven自带的插件打的普通的包：`myjarservice-v1-1.0.jar` ，另一个是sofa-ark提供的maven插件打的包`myjarservice-v1-1.0-ark-plugin.jar`，打开``myjarservice-v1-1.0-ark-plugin.jar`` 可以看到如下目录：

<img src="https://ws4.sinaimg.cn/large/006tKfTcgy1ftlymw3guzj30ik0o40xn.jpg" width="50%" height="50%" />

相关目录说明：

- `com/alipay/sofa/ark/plugin/mark` ：标记文件，标记该 Jar 包是 `sofa-ark-plugin-maven-plugin` 打包生成的 `Ark Plugin` 文件。
- `META-INF/MANIFEST.MF` ：记录插件元信息,**其中包含要导出的和导入的类**。

```properties
Manifest-Version: 1.0
groupId: com.netease
artifactId: myjarservice-v1
version: 1.0
priority: 100
pluginName: myjarservice-v1
activator: 
import-packages: 
import-classes: 
import-resources: 
export-packages: 
export-classes: com.netease.sofaservice.MyJar1Service
export-resources: 
```

- `conf/export.index` ：插件导出类索引文件；为了避免在运行时计算`MANIFEST.MF` 中`export-packages` 下面具体的导出类，在打包生成 `Ark Plugin` 时，会生成插件所有导出类的索引文件，缩短 `Ark Container` 解析配置时间。
- `lib/` : lib 目录存放插件工程依赖的普通 Jar 包，一般包含插件需要和其他插件或者业务有隔离需求的 Jar 包；插件配置的导出类都包含在这些 Jar 包中。

### 2.2 分析sofa-ark-plugin-maven-plugin源码

进入下载的sofa-ark源码中的`ark-plugin-maven-plugin`工程，可以看到ArkPluginMojo继承了

```java
@Mojo(name = "ark-plugin", defaultPhase = LifecyclePhase.PACKAGE, requiresDependencyResolution = ResolutionScope.RUNTIME)
public class ArkPluginMojo extends AbstractMojo {
@Parameter(defaultValue = "${project.artifactId}")
    public String                   pluginName;

    @Parameter(defaultValue = "100", property = "sofa.ark.plugin.priority")
    protected Integer               priority;

    @Parameter
    protected String                activator;

    @Parameter
    protected ExportConfig          exported;

    @Parameter
    protected ImportConfig          imported;
...
```

`@Mojo` 就是一个 goal，可以绑定到某个 `phase` (这里是package)执行，@Parameter 是从外面传进来的参数，可以直接获取xml中配置的参数。Maven插件标准要求必须重写`execute()`方法，插件执行时主要就是执行`execute()`。

```java
    @Override
    public void execute() throws MojoExecutionException 
    {
        Archiver archiver;//zip归档
        archiver = getArchiver();
        outputDirectory.mkdirs();
        String fileName = getFileName();
        File destination = new File(outputDirectory, fileName);
        archiver.setDestFile(destination);
        Set<Artifact> artifacts = project.getArtifacts();
        artifacts = filterExcludeArtifacts(artifacts);
        Set<Artifact> conflictArtifacts = filterConflictArtifacts(artifacts);
        addArkPluginArtifact(archiver, artifacts, conflictArtifacts);
        addArkPluginConfig(archiver);
      	archiver.createArchive();
        projectHelper.attachArtifact(project, destination, getClassifier());
    }
```

这里的将`execute()`方法进行了精简，这个方法主要做了一下几件事情：

1. 建立一个zip格式的归档，用来保存引入的jar包和其他文件，建立输出路径。
2. 获取引入的所有依赖（Artifacts），并且将需要exclude的包排除出去。
3. 将所有依赖写入zip归档中的lib目录下

![image-20180725153520102](https://ws1.sinaimg.cn/large/006tKfTcgy1ftm5a2etwkj31gu0im164.jpg)

1. 将配置信息写入zip归档中，包括之前提到的`export.index`，`MANIFEST.MF`，`mark`![image-20180725153659204](https://ws2.sinaimg.cn/large/006tKfTcly1ftm5bs3tk1j31du0a87b4.jpg)

经过上述步骤后即把依赖的dependence和配置文件都写入zip中了，然后将其转换为jar后缀即可。

## 三. Sofa-ark原理分析

### 3.1 初始化ArkContainer

前面讲解了plugin插件如何工作，这节讲述外部工程是如何引用运用plugin插件打包而成的plugin jar包来解决隔离冲突的。可以从main方法加入的`SofaArkBootstrap.launch(args)`进行单步跟踪，这句代码主要是**将Container启动起来**，然后让Container去加载Plugin和Biz。在`launch`方法里通过反射调用了`SofaArkBootstrap` 的`remain`方法，在`remain`方法里主要干了两件事：

```java
 private static void remain(String[] args) throws Exception {// NOPMD
        URL[] urls = getURLClassPath();
        new ClasspathLauncher(new ClassPathArchive(urls)).launch(args, getClasspath(urls),
            entryMethod.getMethod());
    }
```

1. 获取classpath下的所有jar包，包括jdk自己的jar包和maven引入的jar包。
2. 将所有**依赖jar包和自己写的启动类及其main函数**以url的形式传入`ClasspathLauncher`，`ClasspathLauncher`反射调用`ArkContainer`的`main`方法，并且使用`ContainerClassLoader`加载`ArkContainer`。至此，就开始启动ArkContainer了。

### 3.2 启动ArkContainer

接着就运行到了`ArkContainer`中的main方法，传入的参数args即之前`ClasspathLauncher`传入的url

```java
public static Object main(String[] args) throws ArkException
    {
            //使用LaunchCommand将传入的参数按类型分类
            LaunchCommand launchCommand = LaunchCommand.parse(args[ARK_COMMAND_ARG_INDEX], Arrays.copyOfRange(args, MINIMUM_ARGS_SIZE, args.length));
            //ClassPathArchive将传入依赖的Jar包分类，并提供获得plugin和biz的filter方法
            ClassPathArchive classPathArchive = new ClassPathArchive(launchCommand.getEntryClassName(), launchCommand.getEntryMethodName(), launchCommand.getEntryMethodDescriptor(), launchCommand.getClasspath());
            return new ArkContainer(classPathArchive, launchCommand).start();
            }
    }
```

这个方法主要做了一下几件事：

1. 使用`LaunchCommand`将传入的参数分类，将classpath的url和自己写的启动类的main方法提取出来![image-20180726151249913](https://ws2.sinaimg.cn/large/006tKfTcly1ftna8yerlej31kw0bttjo.jpg)
2. 将`LaunchCommand`传入`ArkContainer`并启动:

在`ArkContainer.start()`中：

```java
public Object start() throws ArkException
    {
        if (started.compareAndSet(false, true))
        {
            arkServiceContainer.start();
            Pipeline pipeline = arkServiceContainer.getService(Pipeline.class);
            pipeline.process(pipelineContext);
        }
        return this;
    }
```

`arkServiceContainer`中包含了一些Container启动前需要运行的Service，这些Service被封装到一个个的`PipelineStage`中，这些`PipelineStage`又被封装成List到一个`pipeline`中。主要包含这么几个`PipelineStage`，依次执行：

1. `HandleArchiveStage`筛选所有第三方jar包中含有mark标记的plugin jar，说明这些jar是sofa ark maven插件打包成的需要隔离的jar。从jar中的export.index中提取需要隔离的类，把他们加入一个`PluginList`中，并给每个plugin，分配一个独立的`PluginClassLoader`。同时以同样的操作给Biz也分配一个`BizClassLoader`
2. `DeployPluginStage` 创建一个map，key是需要隔离的类，value是这个加载这个类使用的PluginClassLoader实例。
3. `DeployBizStage` 使用BizClassLoader反射调用Biz的main方法。



至此，Container就启动完了。后面再调用需要隔离的类时，由于启动Biz的线程已经被换成了BizClassLoader，在loadClass时BizClassLoader会首先看看在`DeployPluginStage`创建的Map中是否有PluginClassLoader能加载这个类，如果能就委托PluginClassLoader加载。就实现了不同类使用不同的类加载器加载。