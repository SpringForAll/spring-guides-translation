# 构建一个超媒体驱动的RESTful Web服务

本指导将引导你使用Spring创建一个超媒体驱动的REST web服务的入门程序。

[超媒体](https://en.wikipedia.org/wiki/HATEOAS)是[REST](https://spring.io/understanding/REST)的一个重要方面。 它允许你可以在很大程度上构建客户端和服务器分离的服务，并允许它们独立构建、发展。 REST资源返回的信息不仅包含数据，还包含指向相关资源的链接。 因此，返回信息的设计对整体服务的设计至关重要。

## 你将要构建什么

您将使用Spring HATEOAS（一个API库）构建一个超媒体驱动的REST服务，您可以使用它来轻松创建指向Spring MVC控制器的链接，构建资源表示，并控制如何将其呈现为支持的超媒体格式，例如HAL。

该服务将接受如下的HTTP GET请求:

```
http://localhost:8080/greeting
```

并用一个[JSON](https://spring.io/understanding/JSON)表示，用最简单的超媒体元素来丰富一个问候语，包含一个指向资源本身的链接：

```
{
  "content":"Hello, World!",
  "_links":{
    "self":{
      "href":"http://localhost:8080/greeting?name=World"
    }
  }
}
```

响应已经表明您可以在查询字符串中使用可选的`name`参数来定制问候语：

```
http://localhost:8080/greeting?name=User
```

“name”参数值覆盖“World”的默认值，并在响应中体现：

```
{
  "content":"Hello, User!",
  "_links":{
    "self":{
      "href":"http://localhost:8080/greeting?name=User"
    }
  }
}
```

## 你需要什么

- 约15分钟
- 一个你喜欢的文本编辑器或IDE
- JDK 1.8+
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以将代码直接导入你的IDE：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成这篇指南

像大多数 Spring [入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以跳过已经熟悉的基本步骤。 无论哪种方式，你都会得到起作用的代码。

**从零开始**，请移步 [Build with Gradle](https://spring.io/guides/gs/rest-hateoas/#scratch).

**跳过基础步骤**，按照下面步骤进行：

- [下载](https://github.com/spring-guides/gs-rest-hateoas/archive/master.zip) 并解压此篇指南的源码，或者使用[Git](https://spring.io/understanding/Git)来克隆 : `git clone https://github.com/spring-guides/gs-rest-hateoas.git`
- 使用cd命令进入 `gs-rest-hateoas/initial`
- 前往 [创建一个资源表示类](https://spring.io/guides/gs/rest-hateoas/#initial).

**当你完成后**，你可以和此处的代码进行对比 `gs-rest-hateoas/complete`.

## Build with Gradle  

首先你得设置基础的构建脚本。你可以使用任意你喜欢的构建系统去构建Spring应用， 你需要使用的代码包含在这儿：[Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) 。如果你对两者都不熟悉，可以先参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) 或者 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如， 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle文件

如下是一个 [Gradle初始化文件。](https://github.com/spring-guides/gs-routing-and-filtering/blob/master/initial/build.gradle)

`build.gradle`

```
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
    baseName = 'gs-rest-hateoas'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-hateoas")
    testCompile("com.jayway.jsonpath:json-path")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的特性：

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类。
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## Build with Maven

首先，你需要设置一个基本的构建脚本。你可以在构建Spring应用程序时使用任意你喜欢的构建系统。但是用于[Maven](https://maven.apache.org/)的代码如下所示。如果你对Maven不熟悉，请参考[Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如， 在Linux/Unix系统中使用如下命令： `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

`pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-rest-hateoas</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-hateoas</artifactId>
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

- [Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性：
  - 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
  - 它寻找`public static void main()` 方法并标记为一个可执行的类.
  - 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用你的IDE构建

- 阅读如何导入这篇指南进入你的IDE [Spring Tool Suite](https://spring.io/guides/gs/sts/)。


- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。

##创建一个资源表示类

现在你已经建立了项目，配置好了构建系统，接下来可以创建你的Web服务了。

在开始之前考虑下服务的交互方式。

该服务将暴露`/greeting`处的资源来处理`GET`请求，在查询字符串中使用可选的 `name`参数。 GET请求应该在表示问候的正文中返回一个带有JSON的`200 OK`响应。

除此之外，资源的JSON将包含“_links”属性对应的超媒体元素。 这个链接指向资源本身。 所以表示应该是这样的：

```
{
  "content":"Hello, World!",
  "_links":{
    "self":{
      "href":"http://localhost:8080/greeting?name=World"
    }
  }
}
```

`content`是问候语的文字表示。 `_links`元素包含一个链接列表，在本例中，只有一个关系类型为`rel`，`href`属性指向刚访问的资源。

为了模拟问候表示，您可以创建一个资源表示类。 由于`_links`属性是表示模型的基本属性，Spring HATEOAS提供了一个基类`ResourceSupport`，它允许你添加`Link`的实例并确保它们如上所示被渲染。

所以你只需创建一个普通的常规java对象来扩展`ResourceSupport`并添加内容的字段和存取器以及一个构造函数：

`src/main/java/hello/Greeting.java`

```
package hello;

import org.springframework.hateoas.ResourceSupport;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;

public class Greeting extends ResourceSupport {

    private final String content;

    @JsonCreator
    public Greeting(@JsonProperty("content") String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```

- @JsonCreator - 标记Json如何反序列化为对象 [POJO](https://spring.io/understanding/POJO)
- @JsonProperty - 清楚地标记Jsckson应该将这个构造方法的参数标记到哪个参数

> 如下所示，Spring将使用 *Jackson* JSON库自动将`Greeting`类型的实例编组为JSON。

接下来，创建将提供这些问候的资源控制器。

## Create a RestController

In Spring’s approach to building RESTful web services, HTTP requests are handled by a controller. The components are easily identified by the [`@RestController`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html) annotation, which combines the [`@Controller`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html) and [`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html) annotations. The `GreetingController` below handles `GET` requests for `/greeting` by returning a new instance of the `Greeting` class:

在Spring构建RESTful Web服务的方法中，HTTP请求由控制器处理。 这些组件可以通过[`@RestController`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html)轻松识别，它结合了[`@Controller`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)和[`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html)注解。 下面的`GreetingController`通过返回`Greeting`类的新实例来处理`/greeting`的`GET`请求：

`src/main/java/hello/GreetingController.java`

```
package hello;

import static org.springframework.hateoas.mvc.ControllerLinkBuilder.*;

import org.springframework.http.HttpEntity;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@RestController
public class GreetingController {

    private static final String TEMPLATE = "Hello, %s!";

    @RequestMapping("/greeting")
    public HttpEntity<Greeting> greeting(
            @RequestParam(value = "name", required = false, defaultValue = "World") String name) {

        Greeting greeting = new Greeting(String.format(TEMPLATE, name));
        greeting.add(linkTo(methodOn(GreetingController.class).greeting(name)).withSelfRel());

        return new ResponseEntity<Greeting>(greeting, HttpStatus.OK);
    }
}
```

这个controller简单明了，但是接下来我们还有很多东西需要操作。让我们继续往下看。

The `@RequestMapping` annotation ensures that HTTP requests to `/greeting` are mapped to the `greeting()` method.

`@RequestMapping注解确保对`/greeting`的HTTP请求被映射到`greeting()`方法。

> 上面的例子没有指定`GET`与`PUT`，`POST`等等，因为`@RequestMapping`默认映射*所有* HTTP操作。 使用`@RequestMapping(path =“/greeting”，method = RequestMethod.GET)`缩小这个映射。 在这种情况下，你也要`导入org.springframework.web.bind.annotation.RequestMethod;`。

`@RequestParam`将查询字符串参数`name`的值绑定到`greeting()`方法的`name`参数中。 这个查询字符串参数不是`required`; 如果请求中不存在name参数，则默认使用“World”作为defaultValue的默认值。

Because the `@RestController` annotation is present on the class, an implicit [`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html) annotation is being added onto the `greeting` method. This causes Spring MVC to render the returned `HttpEntity` and its payload, the `Greeting`, directly to the response.

由于@RestController注解存在于类中，所以隐含的[`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/注释/ResponseBody.html)注解被添加到`greeting`方法。 这样Spring MVC将返回的“HttpEntity”带着“Greeting”信息直接呈现给响应。

方法实现最有趣的部分是如何创建指向控制器方法的链接以及如何将其添加到表示模型。 `linkTo(...)`和`methodOn(...)`是`ControllerLinkBuilder`  里的静态方法，它允许你在控制器上伪造一个方法调用。 返回的`LinkBuilder`将检查控制器方法的映射注解，以精确地建立该方法映射到的URI。

> Spring HATEOAS 支持各种**X-FORWARDED-** headers。如果您将Spring HATEOAS服务放在代理之后，并使用**X-FORWARDED-HOST **标头正确配置它，则生成的链接将被正确格式化。

The call to `withSelfRel()` creates a `Link` instance that you add to the `Greeting` representation model.

对`withSelfRel()`的调用会创建一个表示模型的`Link`实例，它将被添加到`Greeting`表示模型的`Link`实例。

## 让程序运行起来

尽管可以将此服务打包为传统的*web应用程序归档*或[WAR](https://spring.io/understanding/WAR)文件以部署到外部应用程序服务器，下面演示的更简单的方法会创建*独立应用程序*。 你可以把所有的东西都封装在一个单独的，可执行的JAR文件中，由一个好的Java main()方法驱动。执行它，您将使用Spring支持的嵌入 [Tomcat](https://spring.io/understanding/Tomcat) servlet容器作为HTTP运行时，而不是部署到外部实例。

`src/main/java/hello/Application.java`

```
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

`@SpringBootApplication` is a convenience annotation that adds all of the following:

`@SpringBootApplication`是一个方便的注解，涵盖了下面的注解内容：

- ``@Configuration`将该类标记为应用程序中bean定义的源。
- `@EnableAutoConfiguration`告诉Spring Boot根据类路径中的项目配置，其他定义的bean，和其他的配置文件来加载bean。
- 通常，你将为Spring MVC应用程序添加`@EnableWebMvc`注解，但是当Spring Boot在类路径中看到*spring-webmvc*时，Spring Boot会自动添加这个注解。这样应用程序会被标记为Web程序，并激活一系列关键指令比如设置`DispatcherServlet`。
- `@ComponentScan` 告诉Spring在`hello`包中扫描components, configurations,services,也允许其扫描controllers.

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。 你注意到没有？没有一行XML 没有`web.xml`文件。 此Web应用程序是纯Java代码，你无需担心任何其他的配置。

### 构建一个可执行jar

您可以使用Gradle或Maven从命令行运行应用程序。 或者您可以构建一个包含所有必需的依赖项，类和资源的可执行JAR文件，然后运行该文件。 这使得在整个开发生命周期内跨越不同的环境等等，将服务作为应用程序发布，版本化和部署变得容易。

如果你使用Gradle，你可以使用`./gradlew bootRun`来运行应用。你也可以使用` ./gradlew build`来构建一个JAR，然后使用下面语句运行JAR：

```
java -jar build/libs/gs-rest-hateoas-0.1.0.jar
```

如果你使用Gradle，你可以使用`./mvnw spring-boot:run`来运行应用。你也可以使用` ./mvnw clean package`来构建一个JAR，然后使用下面语句运行JAR：

```
java -jar target/gs-rest-hateoas-0.1.0.jar
```

> 上面的操作将创建一个可执行的JAR。 您也可以选择[建立一个传统的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)来替代。

Logging output is displayed. The service should be up and running within a few seconds.

运行jar后，会输出日志记录。服务也将在数秒内启动。

## 测试服务

当服务启动后，访问http://localhost:8080/greeting，你将看到：

```
{
  "content":"Hello, World!",
  "_links":{
    "self":{
      "href":"http://localhost:8080/greeting?name=World"
    }
  }
}
```

Provide a `name` query string parameter with <http://localhost:8080/greeting?name=User>. Notice how the value of the `content` attribute changes from "Hello, World!" to "Hello, User!" and the `href` attribute of the `self` link reflects that change as well:

为<http://localhost:8080/greeting?name=User>提供一个`name`查询参数。 注意`content`属性的值是如何从“Hello, World！ 到“Hello, User!”。而`self`链接的`href`属性也反映了这种变化：

```
{
  "content":"Hello, User!",
  "_links":{
    "self":{
      "href":"http://localhost:8080/greeting?name=User"
    }
  }
}
```

此更改演示了`GreetingController`中的`@RequestParam`布置按预期工作。 `name`参数的默认值是“World”，但可以通过查询字符串显式覆盖。

## 总结

恭喜！你已使用Spring HATEOAS构建了超媒体RESTful web服务。
