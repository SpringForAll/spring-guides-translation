# 通过CORS（跨域资源共享）来访问一个RESTful风格的Web服务

> 原文：[Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)
>
> 译者：[helloworldtang](https://github.com/helloworldtang)
>
> 校对：


本入门教程将向您介绍使用Spring创建一个“hello world”的[RESTful风格的web 服务](https://spring.io/understanding/REST) ，这个服务的响应头信息包含了 [Cross-Origin Resource Sharing (CORS)](https://spring.io/understanding/CORS)。您还可以在博客[blog post](https://spring.io/blog/2015/06/08/cors-support-in-spring-framework)中找到更多关于Spring CORS的信息。

## 我们将完成一个什么样的服务

您将构建一个接受HTTP GET请求的服务:
```
http://localhost:8080/greeting
```

然后这个GET请求的返回值是一个表示问候的 [JSON](https://spring.io/understanding/JSON):
```json
{"id":1,"content":"Hello, World!"}
```

您可以在查询字符串中使用一个可选参数`name`来自定义问候信息
```
http://localhost:8080/greeting?name=User
```

这个`name` 参数对应的值会覆盖"World"的默认值，并且这个GET请求的返回值也会随之发生变化：
```json
{"id":1,"content":"Hello, User!"}
```

该服务与[构建一个RESTful风格的Web服务 ](https://spring.io/guides/gs/rest-service/) 中所描述的略有不同，因为它将使用Spring Framework CORS的相关特性来添加需要的CORS响应头。

## 你将需要什么

- 大约需要15分钟
- 最喜欢的文本编辑器或IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或者更新的版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 您也可以直接将代码导入IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成这个入门教程

就像大多数[Spring入门教程](https://spring.io/guides)一样，您可以从头开始，完成每一步，或者您可以绕过您已经熟悉的基本设置步骤。无论采用哪种方式，最终都将得到是可以是可以正常运行的代码。
**从头开始**，继续[使用Gradle构建](https://spring.io/guides/gs/rest-service-cors/#scratch)
**跳过基本内容**，请执行以下操作
- [下载](https://github.com/spring-guides/gs-rest-service-cors/archive/master.zip) 并解压缩该入门教程的源代码库，或者使用 [Git](https://spring.io/understanding/Git)克隆: `git clone https://github.com/spring-guides/gs-rest-service-cors.git` 
- 使用cd命令进入目录`gs-rest-service-cors/initial`
- 直接跳到[创建一个资源表示类](https://spring.io/guides/gs/rest-service-cors/#initial)部分

  **当您完成后**，您可以在`gs-rest-service-cors/complete`目录中检查您的结果。

## 使用Gradle构建

首先，您需要创建一个构建脚本。您可以使用任何你擅长的构建工具来编译Spring应用程序，但是您使用 [Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) 构建的代码在这里都是一样的。如果您不熟悉这两个构建工具，请参考[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle) ，或者 [使用Maven构建 Java项目](https://spring.io/guides/gs/maven).。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构;例如，在*nix系统使用可以使用命令 `mkdir -p src/main/java/hello`:
```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle构建脚本文件

下面是 [初始 Gradle 构建文件](https://github.com/spring-guides/gs-rest-service-cors/blob/master/initial/build.gradle)
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
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-rest-service-cors'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
}
```

[Spring Boot gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 插件提供了许多方便的特性:
- 收集类路径上的所有jar，并构建一个单一的、可运行的"über-jar"，这使得执行和传输您的服务更加方便。
- 查找 `public static void main()`方法并标记为一个可运行的类。
- 提供了一个内置的依赖解析器，它可以设置版本号来匹配 [Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)。您可以覆盖任何您希望的版本，但是它将默认引导所选择的版本集。

## 使用Maven构建

首先，您需要创建一个构建脚本。您可以使用任何你擅长的构建工具来编译Spring应用程序，但是您使用 [Maven](https://maven.apache.org/) 构建的代码在这里都是一样的。如果您不熟悉Maven，请参阅 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构;例如，在*nix系统使用可以使用命令`mkdir -p src/main/java/hello`：

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
    <artifactId>gs-rest-service-cors</artifactId>
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

[Spring Boot Maven plugin]提供许多方便的特性：
- 收集类路径上的所有jar，并构建一个单一的、可运行的"über-jar"，这使得执行和传输您的服务更加方便。
- 它查找 `public static void main()`方法并标记为一个可运行的类。
- 它提供了一个内置的依赖解析器，它可以设置版本号来匹配[Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)。您可以覆盖任何您希望的版本，但是它将默认引导所选择的版本集。

## 使用IDE构建

- 阅读如何将这个入门教程涉及的内容在[Spring Tool Suite](https://spring.io/guides/gs/sts/)中使用。
- 阅读如何将这个入门教程涉及的内容[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)中使用。

## 创建一个代表一个资源的类

现在您已经建立了项目和构建系统，您可以创建您的web服务。
通过考虑服务交互来开始这个过程。
该服务将处理 `GET`请求`/greeting`，并可选地使用查询字符串中的 `name`参数。 `GET`请求应该返回一个`200 OK`响应，该响应是表示问候的正文中的JSON。应该是这样的:
```json
{
    "id": 1,
    "content": "Hello, World!"
}
```

`id`字段是问候的唯一标识符，`content`是问候的文本表示。
要创建greeting模型，您需要创建一个资源表示类。为`id`和`content`数据提供一个普通的java POJO对象，其中包含字段、构造函数和访问器
`src/main/java/hello/Greeting.java`

```java
package hello;

public class Greeting {

    private final long id;
    private final String content;

    public Greeting() {
        this.id = -1;
        this.content = "";
    }

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


> 正如您在下面的步骤中看到的，Spring使用[Jackson JSON](http://wiki.fasterxml.com/JacksonHome)库将类型`Greeting`的实例自动编组为JSON。

接下来，您将创建资源控制器来提供这些问候信息

## 创建一个资源控制器

在使用Spring构建RESTful风格 web服务的方法中，HTTP请求由控制器处理。这些组件很容易被 [`@Controller`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)注解标识，下面的`GreetingController`通过处理`/greeting`接口的 `GET` 请求来返回一个新的`Greeting`类实例:
`src/main/java/hello/GreetingController.java`

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(required=false, defaultValue="World") String name) {
        System.out.println("==== in greeting ====");
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }

}
```

这个控制器简洁、简单，但是在底层有很多东西。让我们一步一步地把它分解。
`@RequestMapping`注解确保将到`/greeting`的HTTP请求映射到 `greeting()` 方法。
>上面的例子使用了`@GetMapping`注解，它充当了`@RequestMapping(method = RequestMethod.GET)`的快捷方式。


`@RequestParam` 将参数名为`name`的查询字符串绑定`greeting()`方法的 `name`参数。这个查询字符串参数不是 `required`;如果请求中没有，则使用默认值"World" 。
方法提供了创建并返回一个带有`id`和 `content`属性的新的 `Greeting`对象的功能，该对象基于`counter`的下一个值，并通过使用问候`template`来格式化给定的`name`属性 。
传统的MVC控制器和上面的RESTful风格的web服务控制器之间的一个关键区别是HTTP响应主体的创建方式。这个rest式的web服务控制器并不依赖于 [view technology](https://spring.io/understanding/view-templates)来执行向HTML的问候数据的服务器端呈现，而是简单地填充并返回一个Greeting对象。对象数据将以JSON形式直接写入HTTP响应。
要做到这一点， [`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html) 注解在`greeting()`方法告诉Spring MVC，它不需要通过服务器端视图层来呈现问候对象，而是返回的Greeting对象*就是* 响应体，应该直接输出到客户端。
`Greeting`对象必须转换为JSON。由于Spring提供了HTTP消息转换器，您不需要手工进行这种转换。因为 [Jackson](http://wiki.fasterxml.com/JacksonHome)在类路径中,Spring会自动使用[`MappingJackson2HttpMessageConverter`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html)将`Greeting` 实例转换为JSON。

## 启用CORS 

### 在控制器方法上配置CORS 

因此，基于RESTful风格的web服务将在其响应中包含CORS的访问控制头信息，您只需在处理请求的方法上添加一个`@CrossOrigin`注解:
`src/main/java/hello/GreetingController.java`

```java
    @CrossOrigin(origins = "http://localhost:9000")
    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(required=false, defaultValue="World") String name) {
        System.out.println("==== in greeting ====");
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }
```

这个`@CrossOrigin`注解只支持这个特定方法的跨域请求。默认情况下，它允许使用`@RequestMapping`注释中指定的所有的域，所有的头信息、HTTP方法以及30分钟的maxAge。您可以通过指定一个注释属性的值来定制这个行为:`origins`、`methods`、`allowedHeaders`、`exposedHeaders`、`allowCredentials`或 `maxAge`。在本例中，我们只允许 `http://localhost:9000`发送跨域请求。

>还可以在控制器类级别上添加这个注释，以便在这个类的所有处理程序方法上启用CORS。

### 全局CORS配置

作为细粒度的基于注释的配置的另一种选择，您还可以定义一些全局的CORS配置。这类似于使用基于`Filter`的解决方案，但可以在Spring MVC中声明，并结合细粒度的`@CrossOrigin`配置。默认情况下，所有域和`GET`, `HEAD`和 `POST`方法都是允许的。
`src/main/java/hello/GreetingController.java`

```java
    @GetMapping("/greeting-javaconfig")
    public Greeting greetingWithJavaconfig(@RequestParam(required=false, defaultValue="World") String name) {
        System.out.println("==== in greeting ====");
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }
```

`src/main/java/hello/Application.java`

```java
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/greeting-javaconfig").allowedOrigins("http://localhost:9000");
            }
        };
    }
```

您可以很容易地更改任何属性(例如示例中的`allowedOrigins`)，并且只将这个CORS配置应用于特定的路径模式。全局和控制器级别的CORS配置也可以组合在一起。

## 使用应用程序可以执行

虽然可以将此服务打包为用于部署到外部应用服务器的传统[WAR](https://spring.io/understanding/WAR) 包，但下面演示的更简单的方法创建了一个独立的应用程序。您可以将所有内容打包到一个单独的、可执行的JAR文件中，由一个古老但好用的Java `main()`方法驱动。在此过程中，您使用Spring的支持来将 [Tomcat](https://spring.io/understanding/Tomcat)  servlet容器嵌入到HTTP运行时中，而不是将其部署到外部实例中。
`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

`@SpringBootApplication`是一个方便的注解，它添加了以下所有内容:
- `@Configuration`将类标记为应用程序上下文的bean定义的来源。
- `@EnableAutoConfiguration` 告诉Spring Boot 开始添加基于类路径设置、其他bean和各种属性设置的bean。
- 通常情况下，你会为一个Spring MVC应用添加 `@EnableWebMvc`注解，但是当Spring Boot在类路径上看到 **spring-webmvc** 时，它会自动添加它。这将应用程序标记为web应用程序，并激活诸如设置 `DispatcherServlet`之类的关键行为。
- `@ComponentScan`告诉Spring在`hello` 包中寻找其他组件、配置和服务，以便它能够找到所有的控制器。
`main()`方法使用Spring Boot的 `SpringApplication.run()`来启动应用程序。您注意到没有一行XML吗?也没有**web.xml**文件。这个web应用程序是100%纯Java的，您不需要处理任何管道或基础设施的配置。

### 构建一个可执行JAR

您可以在命令行中使用Gradle或Maven来运行该应用程序。或者您可以构建一个单一的可执行JAR文件，该文件包含所有必需的依赖项、类和资源，并运行该文件。这使得在整个开发生命周期中，跨不同的环境，以及在不同的环境中，将服务作为一个应用程序进行部署、版本和部署是很容易的。
如果您在使用Gradle，您可以使用这个应用程序`./gradlew bootRun`。或者您可以使用`./gradlew build`来构建JAR文件。然后您可以使用以下命令运行JAR文件:
```bash
java -jar build/libs/gs-rest-service-cors-0.1.0.jar
```

如果使用Maven，则可以使用`./mvnw spring-boot:run`运行这个应用程序。或者您可以用`./mvnw clean package`构建JAR文件 。然后您可以使用以下命令运行JAR文件:
```bash
java -jar target/gs-rest-service-cors-0.1.0.jar
```

>上面的过程将创建一个可运行的JAR。您也可以选择[构建一个典型的WAR包](https://spring.io/guides/gs/convert-jar-to-war/)。


显示日志输出。该服务应该在几秒钟内启动并运行。

## 测试上面的service


现在服务已经启动，访问`http://localhost:8080/greeting`接口,你将看到如下返回值:
```json
{"id":1,"content":"Hello, World!"}
```

`http://localhost:8080/greeting?name=User`提供一个`name`查询字符串参数。注意， `content`属性的值已经从“Hello，World!”变为 "Hello User!":
```json
{"id":2,"content":"Hello, User!"}
```

这一变化表明，`GreetingController`中的`@RequestParam` 安排是按预期工作的。`name`参数被赋予了默认值“World”，但是可以通过查询字符串中的值显式地覆盖它。
还要注意`id`属性是如何从`1`变为`2`的。这就证明了您在多个请求中对同一个`GreetingController` 实例进行了操作，并且它的`counter`字段在每次调用时都是按照预期的那样递增的。
现在要测试CORS头文件的位置，并允许来自另一个域的Javascript客户端访问该服务，您需要创建一个Javascript客户端来使用该服务
首先，创建一个简单的Javascript文件 `hello.js` ，包含以下内容:
`public/hello.js`

```js
$(document).ready(function() {
    $.ajax({
        url: "http://localhost:8080/greeting"
    }).then(function(data, status, jqxhr) {
       $('.greeting-id').append(data.id);
       $('.greeting-content').append(data.content);
       console.log(jqxhr);
    });
});
```

这个脚本使用jQuery调用这个REST服务`http://localhost:8080/greeting`。这个js脚本由下面这个`index.html` 加载:
`public/index.html`

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Hello CORS</title>
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
        <script src="hello.js"></script>
    </head>

    <body>
        <div>
            <p class="greeting-id">The ID is </p>
            <p class="greeting-content">The content is </p>
        </div>
    </body>
</html>
```

>这本质上是[使用jQuery调用RESTful风格的Web服务](https://spring.io/guides/gs/consuming-rest-jquery/)中创建的REST客户端，稍微修改一下，以便调用在本机8080端口上运行的服务。有关如何开发该客户端的详细信息，请参阅该入门教程。

因为REST服务已经在本地主机8080端口上运行，所以您需要确保从另一个服务器和/或端口启动客户机。这不仅避免了这两个应用程序之间的冲突，而且还将确保客户端代码所在的服务与上面完成的服务不同。在本地主机上启动使用9000端口的客户端:
```bash
mvn spring-boot:run -Dserver.port=9000
```

客户端所依赖的服务启动后，在浏览器中打开[http://localhost:9000](http://localhost:9000/)，您将看到:

![如果正确的CORS头部在响应中，则从REST服务中检索到的模型数据被呈现到DOM中。](https://spring.io/guides/gs/rest-service-cors/images/hello.png)


如果服务的响应信息中包含了CORS头信息，那么ID和content将被呈现到页面中。
但是如果缺少了CORS的头信息(或者对客户端不够定义)，那么浏览器将会导致请求失败，而这些值将不会被呈现到DOM中:

![如果响应中缺少CORS标头，则浏览器将会失败。没有数据将被呈现到DOM中。](https://spring.io/guides/gs/rest-service-cors/images/hello_fail.png)
。
## 小结

恭喜!您刚刚开发了一个基于Spring并且支持跨域资源共享的RESTful风格web服务。

## 参阅

以下的入门教程可能也有帮助:
- [构建一个RESTful风格的Web服务](https://spring.io/guides/gs/rest-service/)
- [构建一个超文本驱动的RESTful风格的Web服务](https://spring.io/guides/gs/rest-hateoas/)
- [使用Restdocs创建一个API文档](https://spring.io/guides/gs/testing-restdocs/)
- [基于REST方式访问GemFire Data](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
- [基于REST方式访问 MongoDB Data](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
- [基于MySQL的数据访问](https://spring.io/guides/gs/accessing-data-mysql/)
- [基于REST方式的JPA 数据访问](https://spring.io/guides/gs/accessing-data-rest/)
- [基于REST方式的 Neo4j 数据访问](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
- [调用RESTful风格的Web服务](https://spring.io/guides/gs/consuming-rest/)
- [基于AngularJS访问一个 RESTful风格Web服务](https://spring.io/guides/gs/consuming-rest-angularjs/)
- [基于 jQuery调用一个RESTful风格的Web服务](https://spring.io/guides/gs/consuming-rest-jquery/)
- [基于rest.js调用一个RESTful风格的Web服务](https://spring.io/guides/gs/consuming-rest-restjs/)
- [保护Web应用程序](https://spring.io/guides/gs/securing-web/)
- [使用Spring构建REST服务](https://spring.io/guides/tutorials/bookmarks/)
- [React.js和Spring Data REST](https://spring.io/guides/tutorials/react-and-spring-data-rest/)
- [使用Spring Boo构建应用程序](https://spring.io/guides/gs/spring-boot/)

想要写一本新的入门教程或对现有的入门教程做出贡献?看看我们的 [共创相关的入门教程](https://github.com/spring-guides/getting-started-guides/wiki)。
> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。