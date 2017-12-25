# 表单提交的处理

> 原文：[Handling Form Submission](https://spring.io/guides/gs/handling-form-submission/)
>
> 译者：[Mr.lzc](https://github.com/cleverlzc)
>
> 校对：

本指南将引导你完成使用Srping创建并提交web表单的过程。

## 你将构建什么

在本指南中，你将构建一个web表单，并以下面的这个URL进行访问：
```
http://localhost:8080/greeting
```

在浏览器中查看这个网页，将会显示一个表单。你可以通过向表单域中填充`id`和`content`提交一个问候语。当表单提交后，将会显示一个结果页面。

## 你需要准备什么

- 大约15分钟
- 一个自己喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或以上版本
- [Gradle 2.3+](https://gradle.org/install/) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到本地的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本指南

像大多数[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。不管采用哪种方式，最终都可以得到能够运行的代码。

如果是**从零开始**，则可以从[使用Gradle构建项目](https://github.com/SpringForAll/spring-guides-translation/translated/handling-form-submission#scratch)小节开始。

如果想**跳过基本的设置步骤**，可以按照以下步骤执行：

- [下载](https://github.com/spring-guides/gs-handling-form-submission/archive/master.zip) 并解压本指南相关的源文件，或者直接通过[Git](https://spring.io/understanding/Git)命令克隆到本地：

```bash
git clone https://github.com/spring-guides/gs-handling-form-submission.git
```

- 进入到 `gs-handling-form-submission/initial` 目录
- 直接跳到[创建web控制器](#创建web控制器)小节。

**当完成所有的编码以后**，可以将你的代码与`gs-handling-form-submission/complete`目录下的示例代码进行对比，以检查结果是否正确。

## 使用Gradle构建项目

首先你需要设置一个基本的构建脚本。当使用Spring构建应用时，你可以使用任何你喜欢的构建系统，但是你需要使用[Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/)才能得到可工作的代码。如果两者你都不熟悉，可参考[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle)或者[使用Mavem构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

选择需要创建项目的目录，参照照以下目录结构创建子目录；如果使用的是UNIX/LINUX系统，可以使用`mkdir -p src/main/java/hello`命令创建：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle 的构建文件

下面是一个[原始的Gradle build文件](https://github.com/spring-guides/gs-handling-form-submission/blob/master/initial/build.gradle)。

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
    baseName = 'gs-handling-form-submission'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    testCompile("junit:junit")
}
```

 [Spring Boot gradle 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，把它作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用Maven构建项目

首先，设置基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，在这里为你提供了使用[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Maven不熟悉，可以参照[使用maven构建JAVA项目工程](https://spring.io/guides/gs/maven) 。

### 创建目录结构

选择需要创建项目的目录，参照以下目录结构创建子目录；如果使用的是UNIX/LINUX系统，可以使用`mkdir -p src/main/java/hello` 命令创建：

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
    <artifactId>gs-handling-form-submission</artifactId>
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

[Spring Boot maven 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用IDE构建项目

- 在Spring Tool Suite中构建项目，请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。 
- 在IntelliJ IDEA中构建项目，请参照[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)。 

## 创建web控制器

在Spring创建web站点的方法中，HTTP请求被一个控制器处理。这些组件很容易的被`@Controller`注解来表示。下面的GreetingController处理GET请求，会返回一个`View`的名称，在这个例子中，即“greeting”。`View`负责渲染HTML内容：

`src/main/java/hello/GreetingController.java`

```java
package hello;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class GreetingController {

    @GetMapping("/greeting")
    public String greetingForm(Model model) {
        model.addAttribute("greeting", new Greeting());
        return "greeting";
    }

    @PostMapping("/greeting")
    public String greetingSubmit(@ModelAttribute Greeting greeting) {
        return "result";
    }

}
```

这个控制器时简单明了，但是做了很多事情。让我们一步一步的分析一下。

映射注解允许你将一个HTTP请求映射到一个指定的控制器方法上。该控制器内的两个方法均映射到`／greeting`。你能够使用默认的`@RequestMapping`映射所有的HTTP操作，比如`GET`，`POST`等等。但是在这里例子中，`greetingForm()`方法使用`@GetMapping`专门映射为`GET`，`greetingSubmit()`使用`@PostMapping`专门映射为`POST`。这种映射使得控制器能够区分到`／greeting`端点的请求。

`greetingForm()` 方法使用`Modle`对象暴露一个新的`Greeting`给显示模版。`Greeting`对象（如下）包含`id`和`content`属性，对`greeting`显示应表单域中的属性，将用来捕捉从表单传来的信息。

`src/main/java/hello/Greeting.java`

```java
package hello;

public class Greeting {

    private long id;
    private String content;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

}
```

方法体的实现依赖于[视图技术](https://spring.io/understanding/view-templates)，通过将视图名称“greeting”转换为要呈现的模板来执行HTML的服务器端呈现。在本例中，我们使用[Thymeleaf](http://www.thymeleaf.org/doc/html/Thymeleaf-Spring3.html)，它解析下面的`greeting.html`模板并评估各种模板表达式来呈现表单。

`src/main/resources/templates/greeting.html`

```xml
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
	<h1>Form</h1>
    <form action="#" th:action="@{/greeting}" th:object="${greeting}" method="post">
    	<p>Id: <input type="text" th:field="*{id}" /></p>
        <p>Message: <input type="text" th:field="*{content}" /></p>
        <p><input type="submit" value="Submit" /> <input type="reset" value="Reset" /></p>
    </form>
</body>
</html>
```

`th：action =“@ {/ greeting}”`表达式将表单指向POST到`/greeting`端点，而`th：object =“$ {greeting}”`表达式声明模型对象用于收集表单数据。 以`th：field =“{id}”`和`th：field =“{content}”`表示的两个表单字段对应于上述`Greeting`对象中的字段。

这涵盖了用于呈现表单的控制器，模型和视图。 现在让我们回顾一下提交表单的过程。 如上所述，表单使用POST提交到`/greeting`端点。 `greetingSubmit()`方法接收由窗体填充的`Greeting`对象。 `Greeting`是一个`@ModelAttribute`，所以它被绑定到传入的表单内容，并且提交的数据也可以在结果视图中通过名称来引用（缺省情况下方法参数的名称），对应这里的`“greeting”` 例子）。 该`id`呈现在`<pth：text =“'id：'+ $ {greeting.id}”/>`表达式中。 同样，内容呈现在`<pth：text =“'content：'+ $ {greeting.content}”/>`表达式中。

`src/main/resources/templates/result.html`

```xml
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Getting Started: Handling Form Submission</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
	<h1>Result</h1>
    <p th:text="'id: ' + ${greeting.id}" />
    <p th:text="'content: ' + ${greeting.content}" />
    <a href="/greeting">Submit another message</a>
</body>
</html>
```

为了清楚起见，本例使用两个单独的视图模板来呈现表单并显示提交的数据; 但是，你也可以为了两个目的使用单个视图。

## 使应用程序可以执行

虽然批处理可以嵌入到Web应用程序和WAR文件中，但下面演示的更简单的方法创建了一个独立的应用程序。 你将所有内容都包装在一个可执行的JAR文件中，由一个好的旧的Java `main()`方法驱动。

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication` 是一个方便的注解，它会自动添加以下所有内容：

- `@Configuration`将该类标记为应用程序上下文中bean定义的源。
- `@EnableAutoConfiguration`指示Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
- 通常，你将为Spring MVC 应用程序添加`@EnableWebMvc`注解，但是当Spring Boot在类路径中发现**spring-webmvc**时会自动添加该注解。通过添加该注解将应用程序标记为Web应用程序，并进行一些关键操作，比如设置`DispatcherServlet`。
- `@ComponentScan`通知Spring在`hello`包中查找其他组件，配置和服务，允许Spring扫描到控制器。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。你注意到我们没有写过一行XML代码吗？而且也没有**web.xml**配置文件。此Web应用程序是100%纯Java编写的，无需再配置其他基础设施。

### 构建可执行的JAR

程序创建好以后，可以使用Gradle或Maven从命令行运行。或者，也可以将所有必需的依赖项，类和资源打包成一个可执行的JAR文件，并运行该文件。这种方式使得在整个开发生命周期中，应用程序可以轻松地发布，更新版本和部署服务。

如果你使用的是Gradle，则可以使用`./gradlew bootRun`运行应用程序。或者使用`./gradlew build`来构建JAR文件。然后运行这个JAR文件：

```
java -jar build/libs/gs-handling-form-submission-0.1.0.jar
```

如果你使用的是Maven，可以使用`./mvnw spring-boot:run`运行应用程序，或者使用`./mvnw clean package`来构建JAR文件。然后运行这个JAR文件：

```
java -jar target/gs-handling-form-submission-0.1.0.jar
```

> 上面的过程创建了一个可运行的JAR。也可以选择[构建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

上述操作完成后，将会看到有日志信息输出，服务程序将会在几秒内启动并运行。


## 测试服务

现在该网站正在运行，请访问`http//localhost8080/greeting`，你将看到以下表单：
![Alt text](https://spring.io/guides/gs/handling-form-submission/images/form.png
)
<!-- https://spring.io/guides/gs/handling-form-submission/images/form.png -->

提交id和message来查看一下结果：
![Alt text](https://spring.io/guides/gs/handling-form-submission/images/result.png
)


## 总结

恭喜！你刚刚用Spring创建并提交了一个表单。

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。


