#构建一项RESTful风格的web服务(RESTful Web Service)

> 原文：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) 
>
> 译者：冀天宇
>
> 校对：

这篇指南将指导你完成如下过程：使用Spring创建一项"hello world"类型的RESTful web service。
## 你将构建什么
你将构建一项服务(service), 该服务在以下地址接收`HTTP GET`请求：
```
http://localhost:8080/greeting
```
然后返回`JSON`格式的问候语：
```
{"id":1,"content":"Hello, World!"}
```
如果在`HTTP GET`请求的查询字符串中加入一个可选的`name`参数，就可以自定义问候语：
```
http://localhost:8080/greeting?name=User
```
`name`参数重写了默认值`World`,结果反映在http响应（http response）中：
```
{"id":1,"content":"Hello, User!"}
```
## 你需要做哪些工作
- 大概15分钟
- 你钟爱的文本编辑器或者IDE
- JDK1.8或者更高的版本
- 也可以直接将本节代码导入到你的IDE中
 - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
 - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成此指南
就像大部分的Spring Getting Started guides一样， 你可以从零开始，亲自完成每一个步骤。或者跳过你已经熟悉的基本步骤。无论如何，结束此指南的时候你都会实现可以正常工作的代码。
要从零开始，请跳转到[Build with Gradle](https://spring.io/guides/gs/rest-service/#scratch)
要跳过基本的步骤，这样做：
- [下载](https://github.com/spring-guides/gs-rest-service/archive/master.zip)并解压缩此指南的代码，或者使用Git clone:`
`git clone https://github.com/spring-guides/gs-rest-service.git`
- 进入`gs-rest-service/initial`目录
- 直接跳转到[创建一个类用来表示资源](https://spring.io/guides/gs/rest-service/#initial)目录

当完成以上步骤后，可以在`gs-rest-service/complete`目录中再次查看代码。

### 使用Gradle构建
第一步，建立基本的构建脚本(build script)。 当使用Spring构建apps的时候，几乎可以使用任何你喜欢的构建工具， 但是此指南只介绍了如何使用Gradle和Maven来构建目标app。如果这两个工具你都不熟悉，请参考[ Building Java Projects with Gradle ](https://spring.io/guides/gs/gradle)或者[ Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

###创建目录结构
在你选定的工程目录中，创建如下的子目录结构。例如，在*nix系统中使用命令`mkdir -p src/main/java/hello`来创建该目录结构。
```
└── src
    └── main
        └── java
            └── hello
```
### 创建Gradle构建文件(build file)
下面是Gradle的初始构建文件(initial build file)。
`build.gradle`
```
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
[Spring Boot gradle plugin](http://https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多方便的功能：
- 将classpath中的所有jar文件集中起来，构建成单独的可运行的"über-jar", 这使得服务的运行和转移更加便捷。
- 搜索`public static void main()`方法，并对该可执行类进行标记。
- 提供了内置的依赖解析功能，该功能将依赖的版本与[ Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相匹配。用户可以按照需求覆盖依赖(dependency)的任何版本号，但是默认版本号是Spring Boot中已经选择好的版本号的集合。

###使用Maven构建
第一步，建立基本构建脚本(build script)。 当使用Spring构建apps的时候，几乎可以使用任何你喜欢的构建工具， 但是此部分只介绍了如何使用Maven来构建目标app。如果你不熟悉这个工具，请参考[ Building Java Projects with Maven](https://spring.io/guides/gs/maven)。
####创建目录结构
在你选定的工程目录中，创建如下的子目录结构。例如，在*nix系统中使用命令`mkdir -p src/main/java/hello`来创建该目录结构。
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
    <artifactId>gs-rest-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
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
[Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了许多方便的功能：
- 将classpath中的所有jar文件集中起来，构建成单独的可运行的"über-jar", 这使得服务的运行和转移更加便捷。
- 搜索`public static void main()`方法，并对该可执行类进行标记。
- 提供了内置的依赖解析功能，该功能将依赖的版本与[ Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相匹配。用户可以按照需求覆盖依赖(dependency)的任何版本号，但是默认版本号是Spring Boot中已经选择好的版本号的集合。

### 使用IDE构建
- 阅读如何将本指南直接导入到[Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)完成本指南。

### 创建一个表示资源的类(Class)
现在已经建立了工程、配置好了构建系统，可以开始创建Web 服务(Service)了。

现在考虑一下这项服务进行交互的细节，这作为开始。

本服务将处理访问`/greeting`的`GET`请求，并且在查询字符串中可以加入可选的`name`参数。`GET`请求应该返回`200 OK`，在`Response Body`中应该包含表示问候语的`JSON`数据，例如：
```
{
    "id": 1,
    "content": "Hello, World!"
}
```

`id`是表示问候语唯一性的标识符，`content`是问候语的文本内容。

为了将问候语建模便于处理，创建一个表示资源的类。在POJO(plain old java object)的基础上，加入`id`和`content`字段、以及对应的构造函数、访问函数：
`src/main/java/hello/Greeting.java`
```
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

>正如在接下来的步骤中会看到的，Spring使用`Jackson JSON`库，自动将`Greeting`类型的实例转化为`JSON`类型。

接下来创建资源控制器(Resource Controller)来实现问候语功能。

### 创建资源控制器(Resource Controller)
使用Spring构建RESTful Web Services的解决方案中，使用Controller来处理http请求。这些Controller组件由`@RestController`注解修饰，很容易区分。下面的`GreetingController`处理对地址`/greeting`的`GET`请求，返回`Greeting`类的一个新实例：
`src/main/java/hello/GreetingController.java`
```
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
上面的控制器(Controller)看起来简洁、简单，但是在底层做了许多工作。现在让我们将其逐步分解。

`@RequestMapping`注解确保了所有访问`/greeting`的HTTP请求均被映射到`greeting()`方法。

>因为`@RequestMapping`默认处理所有的HTTP请求方法，所以上面的例子没有具体指明`GET`、`PUT`还是`POST`或其他方法。可以使用注解`@RequestMapping(method=GET)`来缩小映射的范围，仅限定于`GET`方法。

`@RequestParam`把查询字符串的参数`name`的值绑定到`greeting()方法`的`name`参数上。此查询字符串的参数被显式指定为可选的(默认情况下`required=true`)：如果查询字符串的`name`参数不存在，那么会使用默认值`defaultValue`的值"World"。

`greeting()`方法创建并返回一个新的`Greeting`对象，这个对象的`id`属性等于`counter`返回的下一个值，`content`属性等于`template`将`name`参数格式化字输出的字符串值。

传统的MVC controller 与 上面的RESTful web service controller 的关键区别在于HTTP响应体的创建方式。前者依赖于[视图技术](https://spring.io/understanding/view-templates)在服务器端将`Greeting`数据生成HTML，而后者仅仅生成并返回`Greeting`对象。这个对象将被按照JSON格式直接写入到HTTP响应中。

这段代码使用Spring 4的新注解：`@RestController`, 该注解将使用此注解的类标记为controller, 并且其中的每个方法都返回一个领域对象(domain object), 而不是返回视图。这个注解是`@Controller`和 `ResponseBody`混合后的简写。

`Greeting`对象必须被转换为JSON格式。依靠Spring的HTTP 消息转换功能的支持，用户不需手动进行转换。由于[Jackson2](http://wiki.fasterxml.com/JacksonHome)位于classpath中， Spring的[`MappingJackson2HttpMessageConverter `](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html)自动执行从`Greeting`实例到JSON类型的转换。

## 使应用程序可以运行
尽管可以将本服务打包为传统的WAR文件，然后部署在外部的应用程序服务器上，下面展示一种更简便的方法，即创建独立的应用程序。这种方法将本服务打包为单独的、可直接运行的JAR文件，这个jar文件由传统的`main()`函数方法驱动。使用这种打包方法，可以使用Spring提供的嵌入式Tomcat servlet容器作为运行时(HTTP runtime)， 而不是直接部署外部实例中。

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
`@SpringBootApplication`作为一个方便使用的注解，提供了如下的功能：
- `@Configuration`表明使用该注解的类是应用程序上下文(Applicaiton Context)中Bean定义的来源。
- `@EnableAutoConfiguration`注解根据classpath的配置、其他bean的定义或者不同的属性设置(property settings)等条件，使Spring Boot自动加入所需的bean。
- 对于Spring MVC应用，通常需要加入`@EnableWebMvc`注解，但是当**spring-webmvc** 存在于classpath中时，Spring Boot自动加入该注解。该注解将当前应用标记为web应用，并激活web应用的关键行为，例如开启`DispatcherServlet`。
- `@ComponentScan`注解使Spring在`hello`包(package)中搜索其他的组件、配置(configurations)和服务(service),在本例中，spring会搜索到控制器(controllers)。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法来加载应用。你有没有注意到本例子中一行XML代码都没有吗？也没有web.xml文件。此web应用100%使纯java代码，因此不需花精力处理任何像基础设施或者下水管道一般的配置工作。

### 构建可执行的JAR文件
可以从Gradle或者Maven的命令行运行此程序，也可以构建一个单独的可执行的JAR文件，此文件包含了应用程序所有必需的依赖、类以及资源。由于应用程序存在不同的开发周期，也会部署于不同的环境，这种方法使应用程序的转移、版本管理、以及发布都变得更加简单。

如果使用Gradle，可以使用`./gradlew bootRun`运行程序。或者使用`./gradlew build`构建JAR文件，然后运行此JAR文件：
```
java -jar build/libs/gs-rest-service-0.1.0.jar
```
如果使用Maven，可以使用`./mvnw spring-boot:run`运行程序。或者使用`./mvnw clean package`构建JAR文件，然后运行此JAR文件：
```
java -jar target/gs-rest-service-0.1.0.jar
```

>上述的过程将会创建可执行的JAR文件。也可以参考[如何构建WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

日志会输出，上述服务应该在几秒钟内准备就绪，开始运行。

###3.8 测试此服务
现在该服务已经准备就绪，访问[http://localhost:8080/greeting](http://localhost:8080/greeting)，将会看到：
```
{"id":1,"content":"Hello, World!"}
```
使用[http://localhost:8080/greeting?name=User](http://localhost:8080/greeting?name=User)访问该地址，提供`name`查询字符串。请注意`content`的值如何从"Hello, World!"变为"Hello User!"的：
```
{"id":2,"content":"Hello, User!"}
```
`content`内容的变化表示`GreetingController`中的`@RequestParam`注解按照预期起作用了。`name`参数具有默认值`World`, 但是可以被查询字符串显式重写。

同时也要注意`id`属性值如何从`1`变为`2`。这表明同一个`GreetingController`实例处理多个HTTP请求，正如预期的那样，其`counter`属性在每次处理完请求后都会自增1。

###总结
恭喜你！你已经成功使用Spring开发了一项RESTful web service。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。