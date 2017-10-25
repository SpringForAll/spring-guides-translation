# 简单的YARN程序 

> 原文：[Simple YARN Application][1]
>
> 译者：[UniKrau][author]
>
> 校对：


> 从这里开始

## 简单的YARN应用

本指南主要介绍创建一个Spring Hadoop YARN应用的流程


## 你会学到什么

使用Spring Hadoop和Spring boot构建简单的Hadoop YARN应用

## 你需要准备什么

* 大概15分钟
* 文本编辑器或IDE
* 需要 [JDK 1.6][2]以上的版本
* 编译工具版本[Gradle 2.3+][3] [ Maven 3.0+][4]
* 也可以将代码直接导入到IDE
  * [Spring Tool Suite (STS)][5]
  * [IntelliJ IDEA][6]
     *  使用本地单实例模式，需要Hadoop 2.2.0以上的版本,Apache Hadoop官网也有相关一些[说明][7]

## 怎样完成本指南

像大多数[Spring 入门文章][8]一样，即新手按部就班完成或者有基础的可以跳过这些基本步骤，不过最后，程序是可以跑的.

**如果从基础开始**，参考[配置工程](#set_up)

**如果已经熟悉跳过一些基本步骤**你可以这样


* [下载][download]源码然后使unzip 命令解压，或者使用[Git][9]拷贝一份源代码，克隆命令`git clone` [https://github.com/spring-guides/gs-yarn-basic.git][9]

* 使用 `cd` 命令跳转到代码目录 `gs-yarn-basic/initial`

* 跳转到[创建YARN容器](#CYC)

**当你完成以上步骤**可以到`gs-yarn-basic/complete`目录查看代码

## HADOOP YARN 介绍

如果已经在Hadoop社区泡了一至两年，你大概了知道很多YARN的讨论，并且作为Hadoop MapReduce的下一代版本，称之为 MapReduce V2。
YARN的全称是（Yet Another Resource Negotiator），它的原始设计思想是解决MapReduce组件性能问题。MapReduce v2的基本理念是把JobTracker的功能和Resource Management以及Job Scheduling/Monitoring分成单独守护进程。这个理念是：有个全局的 Resource Manager（RM）和每一个应用程序对应一个Application Master（AM），可以在Hadoop官网上找到[YARN 架构][11]有关YARN组件之间的依赖关系图。


MapReduceV2在原始MapReduce的基础上重写的并且只是作为一个在YARN上运行的应用程序。因此，可以运行与MapReduce模型无关的应用程序。然而由于YARN的API复杂的性，自定义一个基于YARN的应用程序也是有难度的。YARN的API都是低级别的基础架构API，而不是高级别的开发API。


## Spring YARN 介绍

从开发者开始编写YARN应用程序到在Hadoop集群上执行这个程序的整个过程，远远比敲几行 "Hello world"代码复杂的多。

见识一下其中的要点：

* 整个工程的代码结构是什么样的？
* 怎么样编译和打包这个工程是
* 打包好的程序是怎么样配置的
* 最终的程序在YARN上怎么跑的

Spring YARN 和 Spring Boot处理上述几个主题也是有非常清晰的流程

在高层次上，Spring YARN 提供三种不同的组件 [YarnClient][12], [YarnAppmaster][13] and [YarnContainer][14]。但是这些组件都是Spring YARN Application，我们提供所有组件的默认实现但是也给终端用户尽可能多自定义的选项


In a pure Hadoop environment it has always been a cumbersome process to get your own code packaged, deployed and executed on a Hadoop cluster. Should you just put your compiled package in Hadoop’s classpath, or rely on Hadoop’s tools to copy your artifacts into Hadoop during the job submission? What about if your own code depends on some library that isnt already present on Hadoop’s default classpath? Even worse, what about if the dependencies in your code collides with libraries already on Hadoop’s default classpath?


With Spring Boot you can work around all these issues. You either create an executable jar (sometimes called an uber or fat jar) which bundles all dependencies, or a zip package which can be automatically extracted before the code is about to be executed. In the latter case, it’s possible to re-use entries already available on Hadoop’s default classpath.

In this guide we are going to show how these 3 components,  [YarnClient][12], [YarnAppmaster][13] and [YarnContainer][14]and  are packaged into executable jars using Spring Boot. Internally Spring Boot rely heavy on application auto-configuration and Spring YARN adds its own auto-configuration magic. The application developer can then concentrate on his or her own code and application configuration instead of spending a lot of time trying to understand how all the components should integrate with each other.
  
  
##  Set up the project
  
  First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Gradle][15] and [Maven][16] is included here. If you’re not familiar with either, refer to Building Java Projects with Gradle or Building Java Projects with Maven.
  
  We also have additional guides having specific instructions using build systems with Spring YARN. If you’re not familiar with either, refer to [Building Spring YARN Projects with Gradle][17] or [Building Spring YARN Projects with Maven][18].
  
 ### Create the directory structure
  
  In a project directory of your choosing, create the following subdirectory structure:
  
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
  
### 创建Gradle编译文件
 
 Below is the [initial Gradle build file ][19] and the [initial Gradle settings file][20]. But you can also use Maven. The pom.xml file is included[right here][21]. If you are using [Spring Tool Suite (STS)][22], you can import the guide directly.

  `build.gradle`
  
  ```
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
  
  ``include 'gs-yarn-basic-client','gs-yarn-basic-appmaster','gs-yarn-basic-container','gs-yarn-basic-dist'
  ``
  
  In the above gradle build file we simply create three different jars, each having classes for its specific role. These jars are then repackaged by Spring Boot’s gradle plugin to create an executable jar.
 

<h2 id="CYC">创建YARN容器</h2>

Here you create `ContainerApplication`  and `HelloPojo` classes.

`gs-yarn-basic-container/src/main/java/hello/container/ContainerApplication.java`

```
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

In the above `ContainerApplication` , notice how we added the `@Configuration` annotation at the class level and the `@Bean` annotation on the `helloPojo()` method. We have jumped a little bit ahead of what you most likely expect us to do. We previously mentioned `YarnContainer` component which is an interface towards what you’d execute in your containers. You could define your custom `YarnContainer` to implement this interface and wrap all logic inside of that implementation.

However, Spring YARN defaults to a `DefaultYarnContainer` if none is defined and this default implementation expects to find a specific bean type from a `Spring Application Context` having the real user facing logic what container is supposed to do.

`gs-yarn-basic-container/src/main/java/hello/container/HelloPojo.java`

```
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

`HelloPojo` class is a simple `POJO` in a sense that it doesn’t extend any Spring YARN base classes. What we did in this class:

* We added a class level `@YarnComponent` annotation.
* We added a method level `@OnContainerStart` annotation
* We `@Autowired` a Hadoop’s `Configuration` class
`@YarnComponent` is a stereotype annotation, providing a Spring `@Component` annotation. This is automatically marking a class to be a candidate for having `@YarnContainer` functionality.

Within this class we can use `@OnContainerStart` annotation to mark a public method with void return type and no arguments act as an entry point for some application code that needs to be executed on Hadoop.

To demonstrate that we actually have some real functionality in this class, we simply use Spring Hadoop’s  `@FsShell` to list entries from the root of the `HDFS` file system. We needed to have Hadoop’s `Configuration` which is prepared for you so that you can just rely on autowiring for access to it.


### Create a Yarn Appmaster

Here you create an `AppmasterApplication` class.

`AppmasterApplication`

```
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

The application class for `YarnAppmaster` looks even simpler than what we just did for `ClientApplication` . Again the `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application.

One might argue that if you use this type of dummy class to basically fire up your application, could we not use a generic class for this? Well simple answer is yes, we even have a generic `SpringYarnBootApplication` class just for this purpose. You’d define that to be your main class for an executable jar and you’d accomplish this during the gradle build.

In real life, however, you most likely need to start adding more custom functionality to your application component and you’d do that by starting to add more beans. To do that you need to define a Spring `@Configuration` or `@ComponentScan`.` AppmasterApplication` would then act as your main starting point to define more custom functionality. Effectively this is exactly what we do with a `YarnContainer` in section below.


### Create a Yarn Client

Here you create a `ClientApplication` class.

`gs-yarn-basic-client/src/main/java/hello/client/ClientApplication.java`

```
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

* @EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath setting, other beans, and various property settings.
* Specific auto-configuration for Spring YARN components takes place in a same way than from a core Spring Boot.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. From there we simply request a bean of type `YarnClient` and execute its `submitApplication()` method. What happens next depends on application configuration, which we go through later in this guide. Did you notice that there wasn’t a single line of XML?


### Create an Application Configuration

Create a new yaml configuration file for all sub-projects.

```
gs-yarn-basic-container/src/main/resources/application.yml 
gs-yarn-basic-appmaster/src/main/resources/application.yml 
gs-yarn-basic-client/src/main/resources/application.yml
```

```
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
>Pay attention to the `yaml` file format which expects correct indentation and no tab characters.
>
>

Final part for your application is its runtime configuration, which glues all the components together, which then can be executed as a Spring YARN application. This configuration act as source for Spring Boot’s `@ConfigurationProperties` and contains relevant configuration properties which cannot be auto-discovered or otherwise needs to have an option to be overwritten by an end user.

This way you can define your own defaults for your environment. Because these `@ConfigurationProperties` are resolved at runtime by Spring Boot, you even have an easy option to overwrite these properties either by using command-line options, environment variables or by providing additional configuration property files.

### 编译 Application

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

```
gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-client-0.1.0.jar
gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-appmaster-0.1.0.jar
gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-container-0.1.0.jar
```

### 运行application

Now that you’ve successfully compiled and packaged your application, it’s time to do the fun part and execute it on Hadoop YARN.
现在application通过了编译和打包阶段，该准备在YARN上跑了

在项目的根目录下执行以下命令即可

`$ java -jar gs-yarn-basic-dist/target/gs-yarn-basic-dist/gs-yarn-basic-client-0.1.0.jar
`

然后登陆[Resource Manager UI][22]界面（默认端口是8088）查看application的状态.

![Alt rm_ui](/static/rm_ui.jpg)


# 总结

恭喜了，你能够开发Spring YARN的application了

如果想写一个新的指南或者对其他指南有修改意见和建议，请参考[贡献指南说明][20].

[下载完整的源代码][23]


>
>All guides are released with an ASLv2 license for the code, and [an Attribution, NoDerivatives creative commons license][21] for the writing.
>

[1]:https://spring.io/guides/gs/yarn-basic/
[2]:http://www.oracle.com/technetwork/java/javase/downloads/index.html
[3]:http://www.gradle.org/downloads
[4]:https://maven.apache.org/download.cgi
[5]:https://spring.io/guides/gs/sts
[6]:https://spring.io/guides/gs/intellij-idea/
[7]:https://hadoop.apache.org/docs/r2.2.0/hadoop-project-dist/hadoop-common/SingleCluster.html
[8]:https://spring.io/guides
[9]:https://github.com/spring-guides/gs-yarn-basic.git
[10]:https://spring.io/guides/gs/gradle
[11]:https://hadoop.apache.org/docs/r2.2.0/hadoop-yarn/hadoop-yarn-site/YARN.html
[12]:https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/client/YarnClient.html
[13]:https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/am/YarnAppmaster.html
[14]:https://docs.spring.io/spring-hadoop/docs/2.1.0.RELEASE/api/org/springframework/yarn/container/YarnContainer.html
[15]:http://gradle.org/
[16]:https://maven.apache.org/
[17]:https://spring.io/guides/gs/gradle
[18]:https://spring.io/guides/gs/maven
[20]:https://github.com/spring-guides/getting-started-guides/wiki
[21]:https://creativecommons.org/licenses/by-nd/3.0/
[22]:http://localhost:8088/cluster
[23]:https://github.com/spring-guides/gs-yarn-basic.git
[author]:https://github.com/UniKrau
[download]:https://github.com/spring-guides/gs-yarn-basic/archive/master.zip


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



