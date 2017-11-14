# 使用 Spring Boot Actuator 构建 RESTful Web 服务

> 原文：[Building a RESTful Web Service with Spring Boot Actuator](https://spring.io/guides/gs/actuator-service/)
>
> 译者：[xyq000](https://github.com/xyq000)
>
> 校对：[strongant](https://github.com/strongant)

[Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready) 是 Spring Boot 的一个子项目。开发者只需少量工作，就可为应用添加若干种生产级服务。在这篇指南中，你将构建一个应用并学会如何添加这些服务。

## 你将构建什么

这篇指南将带领你利用 Spring Boot Actuator 构建一个“hello world”的 [RESTful web 服务](https://spring.io/understanding/REST)。你将构建一个可接收 HTTP GET 请求的服务：

```shell
$ curl http://localhost:9000/hello-world
```

它会以下面的 JSON 响应：

```json
{"id":1,"content":"Hello, World!"}
```

为了在一个生产（或其他）环境中管理服务，也会有许多其他开箱即用的特性被添加到你的应用中。这项服务的业务功能类似于[构建 RESTful Web 服务](https://spring.io/guides/gs/rest-service)。你在使用本指南时，无需参考该指南，尽管比较二者的结果可能会很有趣。

### 你需要什么

- 大约15分钟时间
- 一个喜欢的文本编辑器或者 IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或更高版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到 IDE：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成指南

像大多数 Spring [入门指南](https://spring.io/guides) 一样，你可以从头开始完成每一步, 或者绕过你熟悉的基本步骤。不管通过哪种方式，你最后都会得到一份可用的代码。

**若从基础开始**，查看[使用 Gradle 构建](#scratch)。

**若跳过基础部分**，按如下流程操作：

- [下载](https://github.com/spring-guides/gs-actuator-service/archive/master.zip) 并解压本指南的源码存档，或使用 [Git](https://spring.io/understanding/Git) 克隆：`git clone https://github.com/spring-guides/gs-actuator-service.git`

- 进入 `gs-actuator-service/initial` 目录
- 跳转到[创建表示类](#representtationClass).

**当你完成之后**，你可以在 `gs-actuator-service/complete` 根据代码检查结果。

<h2 id="scratch"> 使用 Gradle 构建 </h2>

首先你得编写一个基础构建脚本。在使用 Spring 构建应用的时候，你可以使用任何你喜欢的构建系统，这里提供一份你可能会用到的用 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你的项目根目录，创建如下的子目录结构；例如，在 *nix 系统上使用 `mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份 [Gradle 初始构建文件](https://github.com/spring-guides/gs-accessing-data-rest/blob/master/initial/build.gradle)

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
    baseName = 'gs-accessing-data-rest'
    version = '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

-  将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，这使得执行和分发你的服务变得更加方便。
- 搜索 `public static void main()` 方法并且将它标记为可执行类。
- 提供了内置的依赖解析器，可以将 [Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 依赖的版本号设置为最新。你可以用你想要的任意版本进行改写，不过这会是 Spring Boot 的默认版本设置。

## 使用 Maven 构建
首先你得编写一个基础构建脚本。在使用 Spring 构建应用的时候，你可以使用任何你喜欢的构建系统，这里提供一份你可能会用到的用 [Maven](https://maven.apache.org/) 构建的代码。 如果你对 Maven 不熟悉, 你可以先去参考[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建目录结构
在你的项目根目录，创建如下的子目录结构；例如，在 *nix 系统上使用 `mkdir -p src/main/java/hello`:

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
    <artifactId>gs-actuator-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
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

</project>
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多非常方便的功能：

-  将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，这使得执行和分发你的服务变得更加方便。
- 搜索 `public static void main()` 方法并且将它标记为可执行类。
- 提供了内置的依赖解析器，可以将 [Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 依赖的版本号设置为最新。你可以用你想要的任意版本进行改写，不过这会是 Spring Boot 的默认版本设置。

## 使用你的 IDE 构建
- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读在 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 中使用本指南。

## 运行空服务
为了照顾初学者，这里有一个空的 Spring MVC 应用。

`src/main/java/hello/HelloWorldConfiguration.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloWorldConfiguration {

    public static void main(String[] args) {
        SpringApplication.run(HelloWorldConfiguration.class, args);
    }

}
```

`@SpringBootApplication` 注解根据类路径上的内容，提供了一系列默认加载（比如嵌入式servlet容器）及其他事物。它也开启了Spring MVC 的 @EnableWebMvc注解来激活 web 端点。

本应用中不会定义任何端点，但已足够启动服务并体会 Actuator的部分特性。`SpringApplication.run()` 命令知道如何启动这个web 应用。你所需做的全部工作仅是运行这个命令。

```
$ ./gradlew clean build && java -jar build/libs/gs-actuator-service-0.1.0.jar
```

你还几乎没有写任何代码，所以会发生什么呢？等待server启动，并打开另一个终端试试看吧：

```
$ curl localhost:8080
{"timestamp":1384788106983,"error":"Not Found","status":404,"message":""}
```

可见 server 正在运行，但你还没有定义任何业务端点。你能看见一条来自 Actuator `/error` 端点的通用 JSON 响应，而不是由容器产生的默认的 HTML 报错响应。你能从 server 启动的控制台日志看见有哪些开箱即用的端点被提供了。试点别的吧，例如

```
$ curl localhost:8080/health
{"status":"UP"}
```

很好，你“上线”了。

更多细节，请参见 Spring Boot 的 [Actuator 项目](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator) 。

 <h2 id="representtationClass"> 创建表示类 </h2>

首先，想想你的 API 将是什么样子。

你希望能处理对 `/hello-world` 的 GET 请求，它可以选择性地附带一个姓名查询参数。你会回送一条表示欢迎的 JSON 作为对该请求的响应，它看起来像是

```
{
    "id": 1,
    "content": "Hello, World!"
}
```

`id` 字段是欢迎信息的唯一标识符，而 `content` 字段是该欢迎信息的文本表示。

创建一个表示类来对该欢迎建模：

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

现在你将创建一个服务于该表示类的端点控制器。

## 创建资源控制器

在Spring 中，REST 端点即为 Spring MVC 的控制器。如下 Spring MVC控制器处理一条发往 /hello-world的GET 请求并返回 `Greeting` 资源：

`src/main/java/hello/HelloWorldController.java`

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/hello-world")
public class HelloWorldController {

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @RequestMapping(method=RequestMethod.GET)
    public @ResponseBody Greeting sayHello(@RequestParam(value="name", required=false, defaultValue="Stranger") String name) {
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }

}
```

在面向人的控制器和 REST 端点控制器之间的关键差别在于响应是如何创建的。不同于在 HTML 中依靠视图（如JSP）渲染模型数据，一个端点控制器简单地返回直接写在响应实体中的数据。

[`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html)  注解告诉Spring MVC 不要在视图中渲染模型，而是把要返回的对象写入响应实体。它利用 Spring 的一种消息转换器来做到这一点。由于 Jackson 2 在类路径下，这意味着如果请求的 Accept 首部指定了应返回 JSON，那么[`MappingJackson2HttpMessageConverter`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html)  将会处理从 Greeting 对象到 JSON 的转换。

> 你是怎么知道 Jackson 2 在类路径下呢？运行 ` mvn dependency:tree` 或 `./gradlew dependencies` ，你会得到一个包含 Jackson 2.x 的详细的依赖树。如你所见它来自于 [spring-boot-starter-web](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-starters/spring-boot-starter-web/pom.xml)。

## 创建可执行主类

你可以从一个自定义的主类启动应用，或者我们可以直接用一个配置类做到这一点。最简单的办法是试用 `SpringApplication` 辅助类：

`src/main/java/hello/HelloWorldConfiguration.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloWorldConfiguration {

	public static void main(String[] args) {
		SpringApplication.run(HelloWorldConfiguration.class, args);
	}

}
```

在一个传统的 Spring MVC 应用中，你可以通过添加  `@EnableWebMvc`  注解来开启包括配置 `DispatcherServlet` 在内的关键行为。在 Spring Boot 中，当它检测到 **spring-webmvc** 在类路径下时，它会自动启用该注解。这使得你可以在后续步骤中构建控制器。

`@SpringBootApplication` 同时引入了 [`@ComponentScan`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html) 注解，它告诉 Spring 扫描 `hello` 包来寻找控制器（以及其他被注解了的组件类）。

## 构建可执行 JAR 包

利用 Gradle 或 Maven，你可以从命令行启动应用。或者你可以构建一个单一的可执行 JAR 文件并运行它，该 JAR 文件中包含了所需的所有依赖，class 文件和资源文件。这使得在整个开发的生命周期中，在不同环境间以应用的形式迁移、改版和部署服务等等都变得更容易。

如果你使用的是Gradle，你可以用 `./gradlew bootRun` 运行应用。或者你可以用 `./gradlew build` 构建一个 JAR 文件，然后运行它：

```
java -jar build/libs/gs-actuator-service-0.1.0.jar
```

如果你使用的是 Maven，你可以用 `./mvnw spring-boot:run` 运行应用。或者你可以用 `./mvnw clean package` 构建一个 JAR 文件，然后运行它：

```
java -jar target/gs-actuator-service-0.1.0.jar
```

> 上述步骤会创建一个可执行的 JAR 文件。你也可以选择[构建传统 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

```
... service comes up ...
```

测试：

```
$ curl localhost:8080/hello-world
{"id":1,"content":"Hello, Stranger!"}
```

## 切换到其他 server 端口

Spring Boot Actuator 默认运行在 8080 端口上。通过添加一个`application.properties` 文件，你可以重写该设置。

`src/main/resources/application.properties`

```
server.port: 9000
management.port: 9001
management.address: 127.0.0.1
```

重启 server：

```
$ ./gradlew clean build && java -jar build/libs/gs-actuator-service-0.1.0.jar

... service comes up on port 9000 ...
```

测试：

```
$ curl localhost:8080/hello-world
curl: (52) Empty reply from server
$ curl localhost:9000/hello-world
{"id":1,"content":"Hello, Stranger!"}
$ curl localhost:9001/health
{"status":"UP"}
```

## 测试你的应用

你应当编写单元/集成测试来检查你的应用是否起作用。你可以在下面找到这样一个测试的例子，它用于检查：

- 你的控制器是否有响应
- 你的管理端点是否有响应

正如你所见，我们在一个随机端口上启动应用来做测试。

`src/test/java/hello/HelloWorldConfigurationTests.java`

```java
/*
 * Copyright 2012-2014 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package hello;

import java.util.Map;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.embedded.LocalServerPort;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.context.TestPropertySource;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.BDDAssertions.then;

/**
 * Basic integration tests for service demo application.
 *
 * @author Dave Syer
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = HelloWorldConfiguration.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(properties = {"management.port=0"})
public class HelloWorldConfigurationTests {

	@LocalServerPort
	private int port;

	@Value("${local.management.port}")
	private int mgt;

	@Autowired
	private TestRestTemplate testRestTemplate;

	@Test
	public void shouldReturn200WhenSendingRequestToController() throws Exception {
		@SuppressWarnings("rawtypes")
		ResponseEntity<Map> entity = this.testRestTemplate.getForEntity(
				"http://localhost:" + this.port + "/hello-world", Map.class);

		then(entity.getStatusCode()).isEqualTo(HttpStatus.OK);
	}

	@Test
	public void shouldReturn200WhenSendingRequestToManagementEndpoint() throws Exception {
		@SuppressWarnings("rawtypes")
		ResponseEntity<Map> entity = this.testRestTemplate.getForEntity(
				"http://localhost:" + this.mgt + "/info", Map.class);

		then(entity.getStatusCode()).isEqualTo(HttpStatus.OK);
	}

}
```

# 总结
恭喜！你刚用Spring开发了一个简单的 RESTful 服务。你还为它添加了一些有用的内置服务，这多亏了Spring Boot Actutor。

## 参见

以下指南可能也有帮助：

- [使用 Spring Boot 构建应用](https://spring.io/guides/gs/spring-boot/)
- [使用 Spring MVC 提供 Web 内容](https://spring.io/guides/gs/serving-web-content/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
