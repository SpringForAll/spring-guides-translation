# 构建一个RESTful Web服务

> 原文: [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
> 译者: [马勇斌](https://https://github.com/stormmaybin)
>
> 校对: 

本教程将引导你使用Spring构建一个"Hello World"的[RESTful的web服务](https://spring.io/understanding/REST)。

## 你将构建什么
你将创建一个能接收Http Get请求的服务在:

```bash
http://localhost:8080/greeting
```
并且能响应[JSON](https://spring.io/understanding/JSON)数据表示问候:

```json
{"id":1,"content":"Hello, World!"}
```

你可以用一个可选的自定义问候`name`在查询字符串参数:

```bash
http://localhost:8080/greeting?name=User
```
`name` 参数值会重写掉上面的`content`字段的"World"，如下:

```json
{"id":1,"content":"Hello, User!"}
```

## 你要准备什么

- 大约十五分钟的时间
- 一个你最喜欢的文本编辑器或者IDE
- JDK1.8+
- Gradle2.3+或者Maven3.0+
- 你也可以直接将代码导入你的IDE
	- [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  	- [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本教程
像大多数的Spring系列教程 [Getting Started guides](https://spring.io/guides)，你可以从头开始，完成每一步，也可以跳过已经熟悉的步骤。 无论哪种方式，你都可以成功。

**从头开始**, 参阅[使用Gradle构建](https://spring.io/guides/gs/spring-boot-docker/#scratch).

**跳过基础部分**, 执行以下操作:

- [下载](https://github.com/spring-guides/gs-rest-service/archive/master.zip)并解压本仓库代码，或者使用`git clone https://github.com/spring-guides/gs-rest-service.git`克隆代码库。
- 进入`gs-rest-service/initial`目录
- 前往[创建一个资源表示类](https://spring.io/guides/gs/rest-service/#initial)

**完成以上操作之后**，你可以和`gs-rest-service/complete`代码进行比对。

## 使用Gradle构建
首先你得安装构建脚本. 你可以使用你喜欢的构建系统去构建Spring应用, 你需要的工具在下面都可以找到: [Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/) . 如果你对[Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/)都不熟悉，请参阅[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle)或者[使用Maven构建Java项目](https://spring.io/guides/gs/maven).

### 创建目录结构
在你当前项目工作目录中，创建如下子目录:

```text
└── src
    └── main
        └── java
            └── hello
```
如在Linux系统下使用`mkdir -p src/main/java/hello`来创建。

### 创建Gradle项目文件(build.gradle)

`build.gradle`

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-rest-service'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```
[Spring Boot gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的功能:

- 它集中了`classpath`下的所有jar，并构建一个可运行的jar，这样可以更方便地执行和发布服务。
- 它找到`public static void main()`方法并标记该类为可执行。
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成你希望的版本，但它默认为Spring Boot选择的版本。

## 使用maven构建
首先你得安装构建脚本. 你可以使用你喜欢的构建系统去构建Spring应用，你可以前往[Maven官网](https://maven.apache.org/)来指导你安装。 如果你对[Maven](https://maven.apache.org/)不熟悉，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构
在你当前项目工作目录中，创建如下子目录:

```text
└── src
    └── main
        └── java
            └── hello
```

如在Linux系统下使用`mkdir -p src/main/java/hello`来创建。

`pom.xml`


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-rest-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

[Spring Boot maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的功能:

- 它集中了`classpath`下的所有jar，并构建一个可运行的jar，这样可以更方便地执行和发布服务。
- 它找到`public static void main()`方法并标记该类为可执行。
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成你希望的版本，但它默认为Spring Boot选择的版本。

## 使用你的IDE构建
- 参阅[Spring Tool Suite](https://spring.io/guides/gs/sts/)如何直接导入到你的IDE。
- 参阅[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)如何来构建。

## 创建一个资源表示类
万事具备，你现在可以创建您的web服务。

开始思考服务交互的过程。

服务将处理`GET`请求`/greeting`,可以选择使用一个`name`字符串参数。`GET`请求应该返回一个`200 OK`响应与JSON的响应体。它应该是这样的:

```json
{
    "id": 1,
    "content": "Hello, World!"
}
```
`id`字段表示一个惟一的标识符, `content`字段表示问候文本。

创建一个资源表示类(Model)，此类应该包含`id`和`content`字段。

`src/main/java/hello/Greeting.java`

```java
package hello;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```

> 当你看到下面的步骤，Spring使用[Jacksom JSON](http://wiki.fasterxml.com/JacksonHome)库自动格式化好了资源表示类到JSON格式。

接下来，创建控制器(Controller):

## 创建一个资源控制器

使用Spring构建一个RESTful的web服务，资源控制器用来处理HTTP请求。使用`@RestController`注解来标识一个资源控制器。下面的`GreetingController`处理`/greeting`的`GET`请求，并返回一个`Greeting`的类实例。

`src/main/java/hello/GreetingController.java`

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping("/greeting")
    public Greeting greeting(@RequestParam(value="name", defaultValue="World") String name) {
        return new Greeting(counter.incrementAndGet(),
                            String.format(template, name));
    }
}
```

这个控制器看起来很简洁，但是完成的功能有很多。接下来我们一步一步分开解释。

`@RequestMapping`注解确保HTTP请求`/greeting`映射到`greeting()`方法下。

> 上面的例子没有指定请求方法，`GET`，`PUT`或者是`POST`。因为`@RequestMapping`默认映射所有的HTTP请求方法。使用`@RequestMapping(method = GET)来缩小映射范围。`

`@RequestParam`注解绑定了`name`的字符串参数到`greeting()`方法的`name`参数上。这种query的参数默认是必需的(`required=true`)。如果请求中没有带指定的query参数，那么`defaultValue`将会被使用。


方法体的实现基于来自`Counter`的下一个值创建并返回具有`id`和`content`属性的新的`Greeting`对象，并使用greeting模板格式化给定名称。

传统的MVC控制器和上面的REST风格的Web服务控制器之间的主要区别在于HTTP响应体的创建方式。这个RESTful Web服务控制器并不依赖于视图技术来执行将问候数据的服务器端呈现给HTML，而是简单地填充并返回一个`Greeting`对象。对象数据将作为JSON直接写入HTTP响应。

这段代码使用了Spring 4的新的`@RestController`注解，它将类标记为一个控制器，其中每个方法都返回一个域对象而不是一个视图。这是`@Controller`和`@ResponseBody`的缩写。

`Greeting`对象必须转换为JSON。由于`Spring`的HTTP消息转换器支持，您不需要手动执行此转换。由于[Jackson 2](http://wiki.fasterxml.com/JacksonHome)在类路径上，所以自动选择Spring的`MappingJackson2HttpMessageConverter`将`Greeting`实例转换为JSON。

## 使应用程序可执行
尽管可以将此服务作为传统`WAR`文件打包以部署到外部应用程序服务器，但下面演示的更简单的方法创建了独立的应用程序。你把所有东西都封装在一个单独的，可执行的`JAR`文件中，由一个`Java main（）`方法驱动。一路上，您使用Spring的支持来将`Tomcat` servlet容器作为HTTP运行时嵌入，而不是部署到外部实例。

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
`@SpringBootApplication`是一个方便的注解，它增加了以下所有内容：

- `@Configuration`将类标记为应用程序上下文的bean定义的来源。
- `@EnableAutoConfiguration`通知Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
- 通常情况下，您会为Spring MVC应用程序添加`@EnableWebMvc`，但Spring Boot会在类路径中看到spring-webmvc时自动添加它。这将该应用程序标记为Web应用程序，并激活关键行为，如设置`DispatcherServlet`。
- `@ComponentScan`告诉Spring在`hello`包中查找其他组件，配置和服务，以便找到控制器。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。你有没有注意到没有一行XML？没有**web.xml**文件。这个Web应用程序是100％纯Java，您不必处理配置任何管道或基础设施。

## 构建一个可执行的JAR包
您可以使用Gradle或Maven从命令行运行应用程序。或者，您可以构建一个包含所有必需的依赖项，类和资源的可执行JAR文件，然后运行该文件。这使得在整个开发生命周期内跨越不同的环境等等，将服务作为应用程序发布，版本化和部署变得容易。

如果您正在使用`Gradle`，则可以使用`./gradlew bootRun`运行该应用程序。或者您可以使用`./gradlew`构建生成JAR文件。然后你可以运行JAR文件:

```bash
java -jar build/libs/gs-rest-service-0.1.0.jar
```
如果您正在使用Maven，则可以使用`./mvnw spring-boot:run`来运行该应用程序。或者你也可以使用`./mvnw clean package`建立JAR文件。然后你可以运行JAR文件：

```bash
java -jar build/libs/gs-rest-service-0.1.0.jar
```

> 上面的过程将创建一个可运行的JAR。您也可以选择构建一个[经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。


记录输出显示。该服务应该在几秒钟内启动并运行。

## 服务测试


现在服务已经启动，请访问`http://localhost:8080/greeting`，在这里您可以看到：

```json
{"id":1, "content": "Hello, World"}
```
使用`http://localhost:8080/greeting?name=User`提供名称查询字符串参数。注意`content`属性的值是如何从"Hello，World!"到"Hello, User!"：

```json
{"id":2,"content":"Hello, User!"}
```

此更改演示了`GreetingController`中的`@RequestParam`排列按预期工作。 `name`参数的默认值是"World"，但可以通过查询字符串显式覆盖。

还要注意`id`属性是如何从1改变为2.这证明你正在针对多个请求中的同一个`GreetingController`实例工作，并且它的`counter`字段在每次调用时都按照预期递增。

## 概要
恭喜！您刚刚用Spring开发了一个RESTful Web服务。

## 也可以看看
以下指南也可能有所帮助：

- [Accessing GemFire Data with REST](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
- [Accessing MongoDB Data with REST](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)
- [Accessing Neo4j Data with REST](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
- [Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/)
- [Consuming a RESTful Web Service with AngularJS](https://spring.io/guides/gs/consuming-rest-angularjs/)
- [Consuming a RESTful Web Service with jQuery](https://spring.io/guides/gs/consuming-rest-jquery/)
- [Consuming a RESTful Web Service with rest.js](https://spring.io/guides/gs/consuming-rest-restjs/)
- [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
- [Building REST services with Spring](https://spring.io/guides/tutorials/bookmarks/)
- [React.js and Spring Data REST](https://spring.io/guides/tutorials/react-and-spring-data-rest/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
- [Creating API Documentation with Restdocs](https://spring.io/guides/gs/testing-restdocs/)
- [Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)
- [Building a Hypermedia-Driven RESTful Web Service](https://spring.io/guides/gs/rest-hateoas/)
- [Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/)


想写一个新的指南或贡献一个现有的？查看我们的[贡献准则](https://github.com/spring-guides/getting-started-guides/wiki)。

> 所有指南均以代码的ASLv2许可证发布，以及用于撰写的归属，[NoDerivatives创意公共许可证](https://creativecommons.org/licenses/by-nd/3.0/)。
