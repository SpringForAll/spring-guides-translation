# 创建异步方法

> 原文：[Creating Asynchronous Methods](http://spring.io/guides/gs/async-method/)
>
> 译者：[shaoshao721](https://github.com/shaoshao721)
>
> 校对：

本指南将引导您完成异步查询GitHub。本文重点关注异步部分，这是在弹性服务时中经常用到的特性。

## 你将要构建什么

你将构建一个查询服务来查询GitHub用户信息，并且通过GitHub Api取回数据。扩展服务的一种方法是在后台运行重量级的工作，并且使用Java中的`CompletableFuture`接口等待结果。Java的`CompletableFuture`是未来发展的趋势。它可以轻松地将多个异步操作合并成单个异步进行计算。

## 你需要什么

- 大约15分钟
- 一个你喜欢的文本编辑器或IDE
- JDK1.8或更高的版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以将代码直接导入你的IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成这篇指南

像大多数的[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，逐步完成，或者你也可以跳过已经熟悉的基本步骤。 无论哪种方式，你最终会得到有用的代码。

**从零开始**，请移步[Build with Gradle](http://spring.io/guides/gs/async-method/#scratch).

**跳过基础步骤**，请执行以下操作：

- [下载](https://github.com/spring-guides/gs-async-method/archive/master.zip)并解压此篇指南的源码，或者使用Git来克隆：`git clone https://github.com/spring-guides/gs-async-method.git`
- 使用cd命令进入`gs-async-method/initial`
- 前往[创建GitHub用户的表示](http://spring.io/guides/gs/async-method/#initial)

**当你完成后**，你可以使用`gs-async-method/complete` 中的代码来和你的结果进行对比.

## 使用Gradle构建

首先你得设置一个基础的构建脚本. 当使用Spring构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 [Gradle](http://gradle.org/) 和[Maven](https://maven.apache.org/) 的代码如下所示.如果你对两者都不熟悉，请参阅[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) 或者 [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle构造文件

如下是一个 [Gradle初始化构造文件](https://github.com/spring-guides/gs-routing-and-filtering/blob/master/initial/build.gradle).

`build.gradle`

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
    baseName = 'gs-async-method'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter")
	compile("org.springframework:spring-web")
	compile("com.fasterxml.jackson.core:jackson-databind")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布你的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类.
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用Maven构建

首先，你需要设置一个基本的构建脚本。当使用Spring构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示.如果你对Maven不熟悉，请参考[Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-async-method</artifactId>
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
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
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

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布你的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类.
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用你的IDE构建

- 阅读如何将这篇指南直接导入进 [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建.

## 构建GitHub User的表示

在你创建GitHub查找服务之前，你需要为通过GitHub API检索到的数据定义一个表示。

为了对用户的表示建模，你需要创建一个资源表示类。使用字段、构造函数和访问器提供一个POJO对象:

`src/main/java/hello/User.java`

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown=true)
public class User {

    private String name;
    private String blog;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getBlog() {
        return blog;
    }

    public void setBlog(String blog) {
        this.blog = blog;
    }

    @Override
    public String toString() {
        return "User [name=" + name + ", blog=" + blog + "]";
    }

}
```

Spring使用[Jackson JSON](http://wiki.fasterxml.com/JacksonHome)库将GitHub的JSON响应转换为一个`User`对象。`@JsonIgnoreProperties`注解表示Spring需要忽略该类中未列出的所有属性。 这使得使用REST调用并生成域对象变得简单。

在本指南中，处于示范目的，我们只获取`name`和`blog`URL对象。

## 创建一个GitHub查找服务

接下来，你需要创建一个查询GitHub的服务来查找用户信息。

`src/main/java/hello/GitHubLookupService.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.util.concurrent.CompletableFuture;

@Service
public class GitHubLookupService {

    private static final Logger logger = LoggerFactory.getLogger(GitHubLookupService.class);

    private final RestTemplate restTemplate;

    public GitHubLookupService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    @Async
    public CompletableFuture<User> findUser(String user) throws InterruptedException {
        logger.info("Looking up " + user);
        String url = String.format("https://api.github.com/users/%s", user);
        User results = restTemplate.getForObject(url, User.class);
        // Artificial delay of 1s for demonstration purposes
        Thread.sleep(1000L);
        return CompletableFuture.completedFuture(results);
    }

}
```

`GitHubLookupService`类使用Spring的`RestTemplate` 类来调用远程REST端点(api.github.com/users/)，然后将响应转换为一个`User`对象。 Spring Boot自动提供一个`RestTemplateBuilder`类，它可以自定义任何自动配置位（即`MessageConverter`）的默认值。

该类标记有`@Service`注解，这使其成为Spring组件扫描的候选项，用于检测它并将其添加到[应用程序上下文](http://spring.io/understanding/application-context)中。

`findUser`方法使用Spring的`@Async`注解，表示它将在单独的线程上运行。所有的异步服务均要求该方法的返回类型是`CompletableFuture <User>`而非`User`。 此代码使用`completedFuture`方法来返回已完成的具有GitHub查询结果的`CompletableFuture`实例。

>创建`GitHubLookupService`类的本地实例不允许`findUser`方法异步运行。 它必须在标记了`@Configuration` 注解的类中被创建或由`@ComponentScan`拾取。

GitHub的API的时间可以有所不同。 为了展示本指南的后续优点，此服务已添加了一秒钟的额外延迟。

## 使应用程序可执行

要运行本示例，你可以创建一个可执行的jar。 Spring的`@Async`注释与Web应用程序配合使用，但你完全不需要额外设置Web容器就能查看它的优点。

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@SpringBootApplication
@EnableAsync
public class Application {

    public static void main(String[] args) {
        // close the application context to shut down the custom ExecutorService
        SpringApplication.run(Application.class, args).close();
    }

    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(2);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("GithubLookup-");
        executor.initialize();
        return executor;
    }


}
```

`@SpringBootApplication`是一个方便的注释，它添加以下所有内容：

- `@Configuration`将该类标记为应用程序上下文的bean定义的源。
- `@EnableAutoConfiguration`指示Spring Boot根据类路径设置、其他bean和各种属性设置添加bean。
- 通常，你将手动为Spring MVC应用程序添加`@EnableWebMvc` 注解，但是当Spring Boot在类路径中看到**spring-webmvc**时，Spring Boot将会自动添加它。 这将应用程序标记为一个Web应用程序，并激活诸如设置`DispatcherServlet`等的关键行为。
- `@ComponentScan`告诉Spring在`hello`包中查找其他组件，配置和服务，并且允许它寻找控制器。

`main()`方法使用Spring Boot中的`SpringApplication.run()`方法来启动应用程序。 你注意到这里没有一行XML代码并且也没有**web.xml**文件吗？ 此Web应用程序是100％纯Java并且你无需处理任何管道或基础设施的配置。

`@EnableAsync` 注解使得Spring拥有在后台线程池中运行`@Async`方法的能力。 这个类还定制了使用的`Executor`。 在我们的例子中，我们希望将并发线程的数量限制为2，并将队列的大小限制为500。这里[还有更多的东西可以调整](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/htmlsingle/#scheduling-task-executor)。 默认情况下，使用`SimpleAsyncTaskExecutor`。

还有一个`CommandLineRunner` 将注入`GitHubLookupService` 中并调用该服务3次以演示该方法是异步执行的。

`src/main/java/hello/AppRunner.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import java.util.concurrent.CompletableFuture;

@Component
public class AppRunner implements CommandLineRunner {

    private static final Logger logger = LoggerFactory.getLogger(AppRunner.class);

    private final GitHubLookupService gitHubLookupService;

    public AppRunner(GitHubLookupService gitHubLookupService) {
        this.gitHubLookupService = gitHubLookupService;
    }

    @Override
    public void run(String... args) throws Exception {
        // Start the clock
        long start = System.currentTimeMillis();

        // Kick of multiple, asynchronous lookups
        CompletableFuture<User> page1 = gitHubLookupService.findUser("PivotalSoftware");
        CompletableFuture<User> page2 = gitHubLookupService.findUser("CloudFoundry");
        CompletableFuture<User> page3 = gitHubLookupService.findUser("Spring-Projects");

        // Wait until they are all done
        CompletableFuture.allOf(page1,page2,page3).join();

        // Print results, including elapsed time
        logger.info("Elapsed time: " + (System.currentTimeMillis() - start));
        logger.info("--> " + page1.get());
        logger.info("--> " + page2.get());
        logger.info("--> " + page3.get());

    }

}
```

## 构建可执行的JAR

你可以借助Gradle或Maven使用命令行运行应用程序。 或者，你可以构建一个包含所有必需依赖项，类和资源的单个可执行JAR文件，并运行该文件。 在整个开发的生命周期、在不同的平台下，他都能像一个应用程序一样被发送、更新和部署。

如果你使用的是Gradle，则可以使用`./gradlew bootRun` 命令运行应用程序。 或者你可以使用`./gradlew build`来构建JAR文件。 然后运行JAR文件：

```
java -jar build/libs/gs-async-method-0.1.0.jar
```

如果你使用的是Maven，可以使用`./mvnw spring-boot` 命令运行应用程序：run。 或者你可以使用`./mvnw clean package`来构建JAR文件。 然后运行JAR文件：

```
java -jar target/gs-async-method-0.1.0.jar
```

> 上面的操作可以创建一个可执行的JAR。你也可以选择[构建一个经典的WAR文件](http://spring.io/guides/gs/convert-jar-to-war/)来代替。

显示记录输出，显示每个查询到GitHub。 在`allOf`工厂方法的帮助下，我们创建一个`CompletableFutures`数组，通过调用`join`方法，可以等待所有的`CompletableFutures`完成。

```
2016-09-01 10:25:21.295  INFO 17893 --- [ GithubLookup-2] hello.GitHubLookupService                : Looking up CloudFoundry
2016-09-01 10:25:21.295  INFO 17893 --- [ GithubLookup-1] hello.GitHubLookupService                : Looking up PivotalSoftware
2016-09-01 10:25:23.142  INFO 17893 --- [ GithubLookup-1] hello.GitHubLookupService                : Looking up Spring-Projects
2016-09-01 10:25:24.281  INFO 17893 --- [           main] hello.AppRunner                          : Elapsed time: 2994
2016-09-01 10:25:24.282  INFO 17893 --- [           main] hello.AppRunner                          : --> User [name=Pivotal Software, Inc., blog=http://pivotal.io]
2016-09-01 10:25:24.282  INFO 17893 --- [           main] hello.AppRunner                          : --> User [name=Cloud Foundry, blog=https://www.cloudfoundry.org/]
2016-09-01 10:25:24.282  INFO 17893 --- [           main] hello.AppRunner                          : --> User [name=Spring, blog=http://spring.io/projects]
```

请注意，前两个调用发生在单独的线程（`GithubLookup-2`，`GithubLookup-1`）中，第三个调用停留，直到两个线程中的一个可用。 要比较没有异步功能的时间长短，请尝试注释掉`@Async` 注解并再次运行该服务。 总的经过时间应该显着增加，因为每个查询需要至少一秒钟。 您还可以调整`Executor`以增加`corePoolSize`属性。

基本上，任务所需的时间越长，同时调用的任务越多，您将看到的异步效果越大。 折衷的是处理`CompletableFuture`接口。 它增加了一层间接过程，因为你不再直接处理结果。

## 总结

恭喜！你刚刚开发了一种异步服务，可让你一次扩展多个调用。

## 发现

以下指南也可能有帮助：

- [使用Spring MVC提供Web内容](https://spring.io/guides/gs/serving-web-content/)
- [用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)

想写一个新的指南或贡献一个现有的？ 查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 所有指南都将发布ASLv2许可证的代码，以及署名，NoDerivatives为写作创作共用许可证。



