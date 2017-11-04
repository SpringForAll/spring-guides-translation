# 简单的YARN程序

> 原文：[Simple YARN Application](https://spring.io/guides/gs/yarn-basic/)
>
> 译者：[UniKrau](https://github.com/UniKrau)
>
> 校对：


> 从这里开始

## 简单的YARN应用

本指南主要介绍创建一个Spring Hadoop YARN应用程序的流程


## 你会学到什么

使用Spring Hadoop和Spring Boot构建简单的Hadoop YARN应用

## 你需要准备什么

* 大概15分钟
* 文本编辑器或IDE
* 需要 [JDK 1.6](http://www.oracle.com/technetwork/java/javase/downloads/index.html)以上的版本
* 编译工具版本[Gradle 2.3+](http://www.gradle.org/downloads) [ Maven 3.0+](https://maven.apache.org/download.cgi)
* 也可以将代码直接导入到IDE
  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA(https://spring.io/guides/gs/intellij-idea/)
     *  使用本地单实例模式，需要Hadoop 2.2.0以上的版本,Apache Hadoop官网也有相关一些[说明(https://hadoop.apache.org/docs/r2.2.0/hadoop-project-dist/hadoop-common/SingleCluster.html)

### 怎样完成本指南

像大多数[Spring 入门文](https://spring.io/guides)一样，即新手按部就班完成或者有基础的可以跳过这些基本步骤，不过最后，程序是可以跑的.

**如果从零基础开始**，参考[配置工程](#set_up)

**如果有基础则跳过一些基本步骤**你可以这样


* [下载](https://github.com/spring-guides/gs-yarn-basic/archive/master.zip)源码然后使unzip 命令解压，或者使用[Git](https://spring.io/understanding/Git)拷贝一份源代码，克隆命令`git clone` [https://github.com/spring-guides/gs-yarn-basic.git](https://github.com/spring-guides/gs-yarn-basic.git)

* 使用 `cd` 命令跳转到代码目录 `gs-yarn-basic/initial`

* 跳转到[创建YARN容器](#CYC)

**当你完成以上步骤**可以到`gs-yarn-basic/complete`目录查看代码

## HADOOP YARN 介绍

在Hadoop社区泡了一至两年，你大概了知道很多YARN的讨论，并且YARN作为Hadoop MapReduce的下一代版本，也称为 MapReduce V2。
YARN的全称是（Yet Another Resource Negotiator），它的最初设计思想是解决MapReduce组件性能问题。MapReduce V2的基本理念是把JobTracker的功能和Resource Management以及Job Scheduling/Monitoring分成单独守护进程。因此最后是这样的：有个全局的 Resource Manager（RM）和每一个应用程序对应一个Application Master（AM），当然可以在Hadoop官网上找到[YARN 架构](https://hadoop.apache.org/docs/r2.2.0/hadoop-yarn/hadoop-yarn-site/YARN.html)了解有关YARN组件之间的依赖关系图。


MapReduce V2 是在最初的MapReduce代码的基础上重写的。重写的结果是把它当为一个在YARN上运行的应用程序。所以在YARN上可以运行与MapReduce模型无关的应用程序。然而由于YARN的API复杂的性，开发一个基于YARN的应用程序也是有难度的。YARN的API都是低级别的基础架构API，而不是高级别的开发应用API。


## Spring YARN 介绍

开发者从开始编写YARN应用程序到在Hadoop集群上运行的整个过程，远远比敲几行"Hello world"代码复杂的多。

需要考虑其中的以下几个要点：

* 整个工程的代码结构是什么样？
* 怎么样编译和打包这个工程？
* 打包好的程序需要怎么样配置？
* 最终的程序在YARN上怎么跑？

因此，Spring YARN 和 Spring Boot进行了二次开发Hadoop YARN

在高层次上，Spring YARN 提供三种不同的组件 [YarnClient](https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/client/YarnClient.html), [YarnAppmaster](https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/am/YarnAppmaster.html) and [YarnContainer](https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/container/YarnContainer.html)。这些组件都是Spring YARN Application，我们提供所有组件的默认实现同样也给终端用户尽可能多自定义的选项。


在Hadoop集群环境下进行开发、打包、发布、运行程序是项较为繁重的工作。举个例子，job提交的过程当中，只是把编译好的包放在Hadoop的classpath下或者通过Hadoop的tool拷贝已经编译的 artifacts jar包到Hadoop集群 ？但是如果代码所依赖的库不在Hadoop默认的classpath？甚至，其依赖的库和已经存在Hadoop默认classpath的库有冲突呢？


Spring Boot可以解决这些问题，比如开发一个uber或者fat jar包的时候，有两种方式解决依赖性问题：一、通过Spring Boot解决所有的依赖性问题。二、自动提取zip形式执行包并且保存在Hadoop的默认classpath下，而且第二方式可以重复使用。

使用Spring Boot [YarnClient](https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/client/YarnClient.html), [YarnAppmaster](https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/am/YarnAppmaster.html) and [YarnContainer](https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/container/YarnContainer.html)这个三个组件把application打包成可执行的jar。可以知道，Spring Boot内部通过应用程序的自动化配置和Spring YARN添加其自动化配置去解决依赖性问题。因此，应用开发者可以集中应用本身的开发和配置上。而不需要花太多时间去理解所有的组件之间如何集成的

###  配置工程

首先配置基本的编译脚本。可以用熟悉的编译系统构建Spring app。[Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/)都可以用来构建代码。如果你不熟悉，请参[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) or [Building Java Projects with Maven](https://spring.io/guides/gs/maven).


如果不熟悉而且想知道关于构建Spring YARN的系统说明指南，请参考 [Building Spring YARN Projects with Gradle](https://spring.io/guides/gs/gradle-yarn) or [Building Spring YARN Projects with Maven](https://spring.io/guides/gs/maven-yarn ).

 #### 创建目录结构

  在选定的目录下，创建以下目录结构

  ```
  ├── gs-yarn-basic-appmaster
  │   └── src
  │       └── main
  │           ├── resources
  │           └── java
  │               └── hello
  │                   └── appmaster
  ├── gs-yarn-basic-container
  │   └── src
  │       └── main
  │           ├── resources
  │           └── java
  │               └── hello
  │                   └── container
  ├── gs-yarn-basic-client
  │   └── src
  │       └── main
  │           ├── resources
  │           └── java
  │               └── hello
  │                   └── client
  └── gs-yarn-basic-dist
  ```

比如在类nix系统，可以有以下操作  

  ```
  mkdir -p gs-yarn-basic-appmaster/src/main/resources
  mkdir -p gs-yarn-basic-appmaster/src/main/java/hello/appmaster
  mkdir -p gs-yarn-basic-container/src/main/resources
  mkdir -p gs-yarn-basic-container/src/main/java/hello/container
  mkdir -p gs-yarn-basic-client/src/main/resources
  mkdir -p gs-yarn-basic-client/src/main/java/hello/client
  mkdir  gs-yarn-basic-dist
  ```

#### 创建Gradle编译文件

以下是[initial Gradle build file ](https://github.com/spring-guides/gs-yarn-basic/blob/master/initial/build.gradle)和[initial Gradle settings file](https://github.com/spring-guides/gs-yarn-basic/blob/master/initial/settings.gradle)文件内容，如果你使用 [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
可以直接导入目录。

  `build.gradle`

  ```groovy
  buildscript {
      repositories {
          maven { url "http://repo.spring.io/libs-release" }
      }
      dependencies {
          classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.3.RELEASE")
      }
  }

  allprojects {
      apply plugin: 'base'
  }

  subprojects { subproject ->
      apply plugin: 'java'
      apply plugin: 'eclipse'
      apply plugin: 'idea'
      version =  '0.1.0'
      repositories {
          mavenCentral()
          maven { url "http://repo.spring.io/libs-release" }
      }
      dependencies {
          compile("org.springframework.data:spring-yarn-boot:2.4.0.RELEASE")
      }
      task copyJars(type: Copy) {
          from "$buildDir/libs"
          into "$rootDir/gs-yarn-basic-dist/target/gs-yarn-basic-dist/"
          include "**/*.jar"
      }
      assemble.doLast {copyJars.execute()}
  }

  project('gs-yarn-basic-client') {
      apply plugin: 'spring-boot'
  }

  project('gs-yarn-basic-appmaster') {
      apply plugin: 'spring-boot'
  }

  project('gs-yarn-basic-container') {
      apply plugin: 'spring-boot'
  }

  project('gs-yarn-basic-dist') {
      dependencies {
          compile project(":gs-yarn-basic-client")
          compile project(":gs-yarn-basic-appmaster")
          compile project(":gs-yarn-basic-container")
          testCompile("org.springframework.data:spring-yarn-boot-test:2.4.0.RELEASE")
          testCompile("org.hamcrest:hamcrest-core:1.2.1")
          testCompile("org.hamcrest:hamcrest-library:1.2.1")
      }
      test.dependsOn(':gs-yarn-basic-client:assemble')
      test.dependsOn(':gs-yarn-basic-appmaster:assemble')
      test.dependsOn(':gs-yarn-basic-container:assemble')
      clean.doLast {ant.delete(dir: "target")}
      jar.enabled = false
  }

  task wrapper(type: Wrapper) {
      gradleVersion = '1.11'
  }
  ```

  `settings.gradle`

```groovy
include 'gs-yarn-basic-client','gs-yarn-basic-appmaster','gs-yarn-basic-container','gs-yarn-basic-dist'
```

以上的gradle编译文件，简单的创建了三个不同的jar包，各自的类文件都有他们的角色。Spring Boot的插件把这些jar打包成一个可执行的jar。

<h4 id="CYC">创建YARN容器</h4>

分别创建 `ContainerApplication`  and `HelloPojo` 2个类.

`gs-yarn-basic-container/src/main/java/hello/container/ContainerApplication.java`

```java
package hello.container;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
public class ContainerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ContainerApplication.class, args);
	}

	@Bean
	public HelloPojo helloPojo() {
		return new HelloPojo();
	}

}
```

这个 `ContainerApplication` 类，添加了类级别的 `@Configuration` 注解，以及 `@Bean` 注解在 `helloPojo()` 方法上。这些内容虽然有些超前，但是不难理解。之前提到的`YarnContainer` 组件其实是一个需要实现的接口。所以可以自定义一个实现这个接口的`YarnContainer`来包装其逻辑实现。


然而，Spring YARN 有一个默认的 `DefaultYarnContainer` ，当没有自实现 `YarnContainer` 的时候，这个默认 `DefaultYarnContainer` 从 `Spring Application Context` 找到指定的bean类型，而 `Spring Application Context` 包含类真实用户container的逻辑。

`gs-yarn-basic-container/src/main/java/hello/container/HelloPojo.java`

```java
package hello.container;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.hadoop.fs.FsShell;
import org.springframework.yarn.annotation.OnContainerStart;
import org.springframework.yarn.annotation.YarnComponent;

@YarnComponent
public class HelloPojo {

	private static final Log log = LogFactory.getLog(HelloPojo.class);

	@Autowired
	private Configuration configuration;

	@OnContainerStart
	public void publicVoidNoArgsMethod() throws Exception {
		log.info("Hello from HelloPojo");
		log.info("About to list from hdfs root content");

		FsShell shell = new FsShell(configuration);
		for (FileStatus s : shell.ls(false, "/")) {
			log.info(s);
		}
		shell.close();
	}

}
```

`HelloPojo` 类是个简单的 `POJO` ，这样的场景下，它不需要继承Spring YARN的基类，这个类包含以下内容:

* 添加一个 `@YarnComponent` 类级别的注解
* 添加一个 `@OnContainerStart` 方法级别的注解
* 添加一个 `@Autowired` 注解到Hadoop的 `Configuration` 类 `@YarnComponent` 是一个 stereotype 的注解, 为Spring提供了一个 `@Component` 注解. This is automatically marking a class to be a candidate for having  functionality.
它能自动标记一个类为候选类——具有 `@YarnContainer`的功能。

如果想在Hadoop上执行这个方法，那么这个类模版是这样的：要用 `@OnContainerStart` 注解标记一个 `void` 类型返回值的无参数公有方法。这个方法只是作为一个application代码的实体点，

为了掩饰这一点，在这个类中特意添加了一些真实的功能，使用Spring Hadoop `@FsShell` 列举  `HDFS` 跟文件系统的目录树。并且Hadoop的 `Configuration` 能方便地访问 `HDFS` 文件系统。

#### 创建一个YARN Appmaster

新建一个 `AppmasterApplication` 类.

`AppmasterApplication`

```java
package hello.appmaster;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class AppmasterApplication {

	public static void main(String[] args) {
		SpringApplication.run(AppmasterApplication.class, args);
	}

}
```

这个 `YarnAppmaster` 比起 `ClientApplication` 简单类许多。值得强调一次是：它 `main()` 调用了Spring Boot的 `SpringApplication.run()` 进行启动一个程序。


那么为什么要使用这样的类来封装程序呢，而不用一般的类封装呢。理由是：即使有这样一个 `SpringYarnBootApplication` 类进行封装。但是还是需要在可执行的jar包主类中定义。gradle编译必须需要指定。


真实的情况是，需要添加许多自定义功能到程序的组件并且添加更多的beans。但是很方便，只需定义Spring `@Configuration` or `@ComponentScan`。当然` AppmasterApplication`也有同样的功能。 在以下部分的 `YarnContainer`中， 它就是明显的例子。

#### 创建一个YARN客户端

新建一个 `ClientApplication` 类.

`gs-yarn-basic-client/src/main/java/hello/client/ClientApplication.java`

```java
package hello.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.yarn.client.YarnClient;

@EnableAutoConfiguration
public class ClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ClientApplication.class, args)
			.getBean(YarnClient.class)
			.submitApplication();
	}

}
```

* `@EnableAutoConfiguration` 能让Spring Boot 添加基于classpath配置的beans、其他的beans和各种各样的属性设置
* Spring Boot的核心也会以相同的方式为Spring YARN 组件加载特定自动化配置

这个 `main()` 调用Spring Boot’s `SpringApplication.run()`方法启动一个程序，从简单 `YarnClient` bean请求和执行 `submitApplication()`方法，接下运行流程都要依赖程序的配置文件，后来的指南会讲述相关的内容。配置文件不是只有一个单独XML文件。

#### 创建一个程序配置文件

根据所有的子工程创建一个新的 yaml 配置文件

```
gs-yarn-basic-container/src/main/resources/application.yml
gs-yarn-basic-appmaster/src/main/resources/application.yml
gs-yarn-basic-client/src/main/resources/application.yml
```

```yaml
spring:
    hadoop:
        fsUri: hdfs://localhost:8020
        resourceManagerHost: localhost
    yarn:
        appName: gs-yarn-basic
        applicationDir: /app/gs-yarn-basic/
        client:
            files:
              - "file:gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-container-0.1.0.jar"
              - "file:gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-appmaster-0.1.0.jar"
            launchcontext:
                archiveFile: gs-yarn-basic-appmaster-0.1.0.jar
        appmaster:
            containerCount: 1
            launchcontext:
                archiveFile: gs-yarn-basic-container-0.1.0.jar
```


>
>
>注意 `yaml` 文件的格式，它缩进方式不是tab字符。
>
>

最后一部分是运行配置，把所有的组件能融合在一块，作为一个可执行的Spring YARN 程序，这个配置为Spring Boot `@ConfigurationProperties`提供配置源，内容包含了相关的配置属性，有些是不能可修改的属性也有些是用户可以修改的


这种方式，方便自定义默认的环境变量。因为程序运行的时候，Spring Boot会解析 `@ConfigurationProperties` ，当然还有更简单的方式，比如命令行或者定义配置属性文件覆盖默认的属性值。

#### 编译 Application

如果gradle编译工具可以使用 `clean` and `build` 两个命令

`./gradlew clean build`


跳过所有测试命令:

`./gradlew clean build -x test`

如果是maven编译工具可以执行 `clean` and `package`

`mvn clean package
`

跳过所有测试命令:

`mvn clean package -DskipTests=true
`

gradle编译成功后，有以下三个jar包

```groovy
gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-client-0.1.0.jar
gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-appmaster-0.1.0.jar
gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-container-0.1.0.jar
```

#### 运行application

Now that you’ve successfully compiled and packaged your application, it’s time to do the fun part and execute it on Hadoop YARN.
现在application通过了编译和打包阶段，该准备在YARN上跑了

在项目的根目录下执行以下命令即可

```groovy
$ java -jar gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-client-0.1.0.jar

```


然后登陆[Resource Manager UI](http://localhost:8088/cluster)界面（默认端口是8088）查看application的状态.

![Alt rm_ui](/translated/static/rm_ui.jpg)


### 总结

恭喜了，你能够开发Spring YARN的application了


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
