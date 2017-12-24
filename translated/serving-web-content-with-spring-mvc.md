# 使用Spring MVC提供Web内容的服务

> 原文：[Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
>
> 译者：[zhangweiwen](https://github.com/carlzhangweiwen)
>
> 校对：


本指南将引导您完成使用Spring创建“hello world”网站的过程。

## 你会建立什么

您将构建一个具有静态主页的应用程序，并且还将在以下位置接受HTTP GET请求：

`http://localhost:8080/greeting`

并用显示HTML的网页进行响应。 HTML的主体包含一个问候语：

`"Hello, World!"`

您可以使用查询字符串中的可选`name`参数来自定义问候语：

`http://localhost:8080/greeting?name=User`

`name`参数值覆盖“World”的默认值，并反映在响应中：

`"Hello, User!"`

## 开始之前您需要准备

- 大约15分钟时间
- 一个喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 您也可以直接导入代码到IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)


## 如何完成指南？
像大多数 Spring [入门指南](https://spring.io/guides)一样, 您可以从头开始，完成每一步, 或者您也可以绕过您熟悉的基本步骤再开始。 不管通过哪种方式，您最后都会得到一份可执行的代码。

**如果从基础开始**，您可以往下查看[怎样使用 Gradle 构建项目](#scratch)。

**如果已经熟悉跳过一些基本步骤**，您可以：

* [下载](https://github.com/spring-guides/gs-serving-web-content/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/gs-serving-web-content.git`
* 进入 `gs-serving-web-content/initial`目录
* 跳过前面的部分来[创建一个简单的实体](https://spring.io/guides/gs/serving-web-content/#initial)

**当您完成之后**，您可以在`gs-serving-web-content/complete`根据代码检查下结果。

## 使用Gradle构建

首先您需要编写基础构建脚本。在构建 Spring 应用的时候，您可以使用任何您喜欢的系统来构建， 这里提供一份您可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果您两者都不是很熟悉, 您可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

在您的项目根目录，创建如下的子目录结构; 例如，如果您使用的是\*nix系统，您可以使用`mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份[初始化Gradle构建文件](https://github.com/spring-guides/gs-serving-web-content/blob/master/initial/build.gradle)

`build.gradle`

```gradle
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
    baseName = 'gs-serving-web-content'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    compile("org.springframework.boot:spring-boot-devtools")
    testCompile("junit:junit")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布您的服务变得更加便捷。
* 搜索 `public static void main()` 方法并且将它标记为可执行类。
* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。您可以根据您的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建

首先，您需要设置一个基本的构建脚本。当使用 Spring 构建应用程序时，您可以使用任何您喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

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
    <artifactId>gs-serving-web-content</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
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

</project>
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布您的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。您可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。



## 使用您的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。

## 创建一个web controller

在Spring构建网站的方法中，HTTP请求由控制器(controller)处理。 您可以通过`@Controller`注解轻松识别这些请求。 在以下示例中，GreetingController通过返回`View`（在本例中为“greeting”）来处理`/greeting`的GET请求。 `View`负责呈现HTML内容：
`src/main/java/hello/GreetingController.java`

```java
package hello;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class GreetingController {

    @RequestMapping("/greeting")
    public String greeting(@RequestParam(value="name", required=false, defaultValue="World") String name, Model model) {
        model.addAttribute("name", name);
        return "greeting";
    }

}
```
这个控制器简洁明了，但还有很多。 让我们一步一步分解。

`@RequestMapping`注释可以确保对`/greeting`的HTTP请求映射到`greeting()`方法。

>上面的例子没有指定`GET`与`PUT`，`POST`等等，因为`@RequestMapping`默认映射所有的HTTP操作。 使用`@RequestMapping(method=GET)`缩小这个映射。

`@RequestParam`将查询字符串参数`name`的值绑定到`greeting()`方法的`name`参数中。 这个查询字符串参数是非必须的(required); 如果请求中缺失，则使用“World”的缺省值(defaultValue)。 `name`参数的值被添加到`Model`对象，最终使其可以被视图模板访问。

方法体的实现依赖于一种[视图技术](https://spring.io/understanding/view-templates)，在这种情况下是[Thymeleaf](http://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html)，执行HTML的服务器端呈现。Thymeleaf分析下面的`greeting.html`模板并计算`th:text`表达式来呈现在控制器中设置的`${name}`参数的值。

`src/main/resources/templates/greeting.htm`

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Serving Web Content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```

## 开发web应用

开发网络应用程序的一个共同特点是编码改变，重新启动你的应用程序，并刷新浏览器来查看更改。 整个过程可能会花费很多时间。 为了加速事物的循环，Spring Boot附带了一个称为[spring-boot-devtools](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools)的方便模块。

* 启用[hot swapping](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-hotswapping)
* 切换模板引擎以禁用缓存
* 使LiveReload能够自动刷新浏览器
* 其他基于发展而不是生产的合理的违约

## 使应用程序可执行

尽管可以将此服务作为传统[WAR](https://spring.io/understanding/WAR)文件打包以部署到外部应用程序服务器，但下面演示的更简单的方法创建了独立的应用程序。你把所有东西都封装在一个单独的，可执行的JAR文件中，由一个好的旧的Java ·main()`方法驱动。 一路上，您使用Spring的支持来将[Tomcat](https://spring.io/understanding/Tomcat) servlet容器作为HTTP运行时嵌入，而不是部署到外部实例。

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

`@SpringBootApplication`是一个方便的注解，增加了以下所有内容：


* `@Configuration`将类标记为应用程序上下文的bean定义的来源。
* `@EnableAutoConfiguration`通知Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
* 通常情况下，您会为Spring MVC应用程序添加`@EnableWebMvc`，但Spring Boot会在类路径中看到**spring-webmvc**时自动添加它。 这将该应用程序标记为Web应用程序，并激活关键行为，如设置DispatcherServlet。
* `@ComponentScan`告诉Spring在`hello`包中查找其他组件，配置和服务，以便找到控制器。

main()方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。 你有没有注意到没有一行XML？ 没有**web.xml**文件。 这个Web应用程序是100％纯Java，您不必处理配置任何管道或基础设施。

## 构建一个可执行的JAR

您可以使用Gradle或Maven从命令行运行应用程序。 或者，您可以构建一个包含所有必需的依赖项，类和资源的可执行JAR文件，然后运行该文件。 这使得在整个开发生命周期内跨越不同的环境等等，将服务作为应用程序发布，版本化和部署变得容易。

如果您正在使用Gradle，则可以使用`./gradlew bootRun`运行该应用程序。 或者您可以使用`./gradlew build`生成JAR文件。 然后你可以运行JAR文件：

```shell
java -jar build/libs/gs-serving-web-content-0.1.0.jar
```

如果您正在使用Maven，则可以使用`./mvnw spring-boot：run`来运行该应用程序。 或者你也可以使用`./mvnw clean package`构建JAR文件。 然后你可以运行JAR文件：

```shell
java -jar target/gs-serving-web-content-0.1.0.jar
```

> 上面的过程将创建一个可运行的JAR。 您也可以选择构建一个经典的WAR文件。

可以看到日志的输出。 该应用程序应该在几秒钟内启动并运行。

## 测试应用

现在该网站正在运行，请访问 [http://localhost:8080/greeting](http://localhost:8080/greeting),在这里您可以看到：

`"Hello, World!"`

使用[http://localhost:8080/greeting?name=User](http://localhost:8080/greeting?name=User)提供名称查询字符串参数。注意消息是如何从 "Hello, World!"到 "Hello, User!"：

`"Hello, User!"`

此更改演示了GreetingController中的`@RequestParam`排列按预期工作。 `name`参数的默认值是“World”，但可以通过查询字符串显式覆盖。


## 添加一个主页

静态资源，如HTML或JavaScript或CSS，可以很容易地从你的Spring Boot应用程序中提供，只需把它们放在源代码的正确位置即可。 默认情况下，Spring Boot以“/static”（或“/public”）方式从类路径中的资源提供静态内容。 index.html资源是特殊的，因为它被用作“欢迎页面”（如果存在的话），这意味着它将作为根资源提供，即在我们的示例中为`http://localhost:8080/`。 所以创建这个文件：

`src/main/resources/static/index.html`

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Getting Started: Serving Web Content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p>Get your greeting <a href="/greeting">here</a></p>
</body>
</html>
```
当你重新启动应用程序，你会看到在[http://localhost:8080/](http://localhost:8080/)的HTML。

## 总结

恭喜！你刚刚用Spring开发了一个网页。

## 也可以浏览

以下指南也可能有所帮助：

* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
* [Accessing Data with GemFire](https://spring.io/guides/gs/accessing-data-gemfire/)
* [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
* [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
* [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
* [Testing the Web Layer](https://spring.io/guides/gs/testing-web/)
* [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。




