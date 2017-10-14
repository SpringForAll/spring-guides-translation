> 原文：[Scheduling Tasks](https://spring.io/guides/gs/scheduling-tasks/)
> 
> 译者：[rhwayfun][1]
>
> 校对：[carlzhangweiwen][2]

# Spring 定时任务

本指南将一步步引导您在Spring中使用定时任务

## 你将要构建什么

构建一个应用，实现的功能为，每隔5秒打印出当前时间。这点可以通过Spring注解`@Scheduled`完成。

## 你需要什么

 - 大约需要15分钟
 - 一个您喜爱的文本编辑器或者IDE（集成开发工具）
 - [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或更高版本
 - [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
 - 您也可以直接导入代码到IDE中：
    - [Spring Tool Suite (STS)][3]
    - [IntelliJ IDEA][4]

## 如何完成这份指南
和其他[Spring使用指南][5]一样，你可以从零开始一步步完成，或者略过那些你已經熟悉的項目創建步骤(setup steps)。不管哪种方式你最终都会得到一份工作代码(working code)。

如果从零开始，请移步[Build with Gradle][https://spring.io/guides/gs/scheduling-tasks/#scratch]。。

如果打算跳过基本步骤，按照如下操作：

 - [下载][6]并解压这份指南的源码包，或者使用[Git][7]克隆：
    `git clone https://github.com/spring-guides/gs-scheduling-tasks.git`
 - 进入目录 `gs-scheduling-tasks/initial`
 - 跳到 `创建定时任务章节`

当你完成以上操作，你可以在 `gs-scheduling-tasks/complete` 根据代码检查结果。

## 使用 Gradle 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Gradle][8] 或者 [Maven][9] 构建的代码。如果你对两者都不是很熟悉，你可以先去看下[如何使用 Gradle 构建 Java 项目][10]或者[如何使用 Maven 构建 Java 项目][11]。

### 创建 Gradle 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

### 创建 Gradle 构建文件

下面是一份[初始化Gradle构建文件][12]

build.gradle

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.7.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-spring-boot'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    // tag::jetty[]
    compile("org.springframework.boot:spring-boot-starter-web") {
        exclude module: "spring-boot-starter-tomcat"
    }
    compile("org.springframework.boot:spring-boot-starter-jetty")
    // end::jetty[]
    // tag::actuator[]
    compile("org.springframework.boot:spring-boot-starter-actuator")
    // end::actuator[]
    testCompile("junit:junit")
}
```

[Spring Boot gradle 插件][13] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][14].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用 Maven 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Maven][15] 构建的代码。如果你对 Maven 还不是很熟悉，你可以先去看下[如何使用 Maven 构建 Java 项目][16].

### 创建 Maven 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-scheduling-tasks</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

[Spring Boot Maven 插件][17] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][18].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用你的 IDE 进行构建

*   [如何在Spring Tool Suite中构建][19].

*   [如何在IntelliJ IDEA中构建][20].

## 创建定时任务
既然已经搭建好了项目，下面就可以开始创建定时任务了。

`src/main/java/hello/ScheduledTasks.java`

```java
package hello;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledTasks {

    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", dateFormat.format(new Date()));
    }
}
```

`@Scheduled`注解定义一个方法的执行周期。**注意**：这个例子使用了`fixedRate`，
它指定了每次方法调用开始执行的间隔。有[其他的选项][21]，比如`fixedDelay`，它指定了从每次方法调用结束开始计算时间的调用间隔。你也可以[使用`@Scheduled(cron=". . .")`表达式实现更复杂的定时任务][22]

## 开启定时功能
虽然计划任务可以嵌入在Web应用程序和WAR包中，通过创建一个独立的应用程序，下面演示了一种更简单的方法。 您将所有内容都打包在一个可执行的JAR中，通过Java的`main()`方法就可以执行。

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class Application {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class);
    }
}
```
`@SpringBootApplication`是一个方便的注解，组合了以下注解：

 - `@Configuration` 将类标记为应用上下文获取bean定义的地方
 - `@EnableAutoConfiguration` 告诉Spring Boot从类路径、其他bean和属性配置中添加bean
 - 正常情况下如果是基于Spring MVC构建的应用需要添加注解`@EnableWebMvc`，但是Spring Boot当发现类路径下有**spring-webmvc**依赖时会自动添加这个注解。这个注解的作用标记一个应用为Web应用并且会自动激活一些功能，比如创建`DispatcherServlet`
 - `@ComponentScan` 告诉Spring在`hello`包下扫描其他的组件、配置和服务，从而让Spring找到Controller。

`main()`方法使用Spring Boot的`SpringApplication.run()`启用一个应用。不知道你是否注意到没有一行**XML**配置，也没有**web.xml**文件。这个web应用100%使用Java实现，你不需要任何添加额外的配置。

`@EnableScheduling`注解会在后台创建一个任务执行器，没有它定时任务无法正常工作。

**构建一个可执行JAR包**

你可以使用Maven或者Gradle在命令行中启动这个应用。也可以构建一个包含所有依赖、类文件和资源文件的可执行JAR包，然后运行它。这使得在整个开发生命周期中，在不同的环境部署一个服务变得简单了。

如果你使用的是Gradle，可以通过运行脚本`./gradlew bootRun`来启动应用。或者在Maven使用`./mvnw clean package`打包成JAR文件，然后运行：

`java -jar target/gs-scheduling-tasks-0.1.0.jar`

> 上面的程序会生成一个可执行的JAR，你也可以选择构建传统的WAR包。

从日志输出可以看到这是在后台的线程在输出，你应该可以看到你的定时任务是5秒输出一次的。

```
[...]
2016-08-25 13:10:00.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:00
2016-08-25 13:10:05.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:05
2016-08-25 13:10:10.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:10
2016-08-25 13:10:15.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:15
```

## 总结
恭喜！ 您创建了一个有定时任务功能的应用程序。可以发现，实际代码比构建文件更少！ 这份代码适用于任何类型的应用。

## 了解更多

下面的指南可能对你也有帮助：

*   [Building an Application with Spring Boot][23]

*   [Creating a Batch Service][24]
 
> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可][25]协议进行许可。


  [1]: https://github.com/happyxiaofan
  [2]: https://github.com/carlzhangweiwen
  [3]: https://spring.io/guides/gs/sts
  [4]: https://spring.io/guides/gs/intellij-idea/
  [5]: https://spring.io/guides
  [6]: https://github.com/spring-guides/gs-scheduling-tasks/archive/master.zip
  [7]: https://spring.io/understanding/Git
  [8]: http://gradle.org/
  [9]: https://maven.apache.org/
  [10]: https://spring.io/guides/gs/gradle
  [11]: https://spring.io/guides/gs/maven
  [12]: https://github.com/spring-guides/gs-scheduling-tasks/blob/master/initial/build.gradle
  [13]: https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin
  [14]: https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
  [15]: https://maven.apache.org/
  [16]: https://spring.io/guides/gs/maven
  [17]: https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin
  [18]: https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
  [19]: https://spring.io/guides/gs/sts/
  [20]: https://spring.io/guides/gs/intellij-idea
  [21]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html#scheduling-annotation-support-scheduled
  [22]: https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html
  [23]: https://spring.io/guides/gs/spring-boot/
  [24]: https://spring.io/guides/gs/batch-processing/
  [25]: http://creativecommons.org/licenses/by-nc-sa/4.0/
