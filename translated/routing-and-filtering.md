# 路由和过滤

> 原文：[Routing and Filtering](https://spring.io/guides/gs/routing-and-filtering/)
>
> 译者：[hh23485](https://github.com/hh23485)
>
> 校对：[oshare](https://github.com/oshare)

本指南将引导您完成使用Netflix Zuul边缘服务库来路由和过滤微服务应用程序请求。

## 你将要构建什么

你将会写一个简单的微服务应用，接着使用 [Netflix Zuul](https://github.com/spring-cloud/spring-cloud-netflix) 构建一个反向代理应用将请求转发到应用服务。你也会了解如何使用 Zuul 过滤代理服务中的请求。

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

**从零开始**，请移步[Build with Gradle](https://spring.io/guides/gs/routing-and-filtering/#scratch).

**跳过基础步骤**，按照下面步骤进行：

- [下载](https://github.com/spring-guides/gs-routing-and-filtering/archive/master.zip)并解压此篇指南的源码，或者使用Git来克隆：`git clone https://github.com/spring-guides/gs-routing-and-filtering.git`
- 使用cd命令进入`gs-routing-and-filtering/initial`
- 前往[构建一个微服务](https://spring.io/guides/gs/routing-and-filtering/#initial)

**当你完成后**，你可以和此处的代码进行对比`gs-routing-and-filtering/complete`.

## 使用Gradle构建

首先你得设置基础的构建脚本. 你可以使用任意你喜欢的构建系统去构建Spring应用， 你需要使用的代码包含在这儿：[Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) . 如果你对两者都不熟悉，可以先参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) 或者 [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如， 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle文件

如下是一个 [Gradle初始化文件](https://github.com/spring-guides/gs-routing-and-filtering/blob/master/initial/build.gradle).

`book/build.gradle`

```groovy
buildscript {
	ext {
		springBootVersion = '1.4.0.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'book'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}


eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}

task wrapper(type: Wrapper) {
	gradleVersion = '2.9'
}
```

`gateway/build.gradle`

```groovy
buildscript {
	ext {
		springBootVersion = '1.4.0.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'book'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}


eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}

task wrapper(type: Wrapper) {
	gradleVersion = '2.9'
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的特性：

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类.
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用Maven构建

首先，你需要设置一个基本的构建脚本。你可以在构建Spring应用程序时使用任意你喜欢的构建系统。但是用于[Maven](https://maven.apache.org/)的代码如下所示。如果你对Maven不熟悉，请参考[Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如， 在Linux/Unix系统中使用如下命令： `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

`book/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>book</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>book</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
	</properties>

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

`gateway/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.example</groupId>
  <artifactId>gateway</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-zuul</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Brixton.SR5</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

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

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性：

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类.
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用你的IDE构建

- 阅读如何导入这篇指南进入你的IDE [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建.

## 构建一个微服务

订阅服务将会特别简单，编辑`BookApplication.java`使它如下所示：

`book/src/main/java/hello/BookApplication.java`

``` java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class BookApplication {

  @RequestMapping(value = "/available")
  public String available() {
    return "Spring in Action";
  }

  @RequestMapping(value = "/checked-out")
  public String checkedOut() {
    return "Spring Boot in Action";
  }

  public static void main(String[] args) {
    SpringApplication.run(BookApplication.class, args);
  }
}
```

`BookApplication`类是一个Rest Controller。`@RestController`注解表示这是一个控制器类，并且也确保`@RequestMapping`方法的返回值会被自动转换为合适的格式并直接写入到HTTP响应中。

对于`@RequestMapping`方法，我们已经添加两个：`available()`和`checkedOut()`。他们分别处理路径为`/available`和`/checked-out`的请求，每一个都简单地返回`String`格式的书名。

在`src/main/resources/application.properties`文件中设置应用程序的名称(`book`)。

```properties
spring.application.name=book
server.port=8090
```

我们也可以在这里设置`server.port`来防止我们和边缘服务都在本地启动运行发生端口冲突。

## 创建一个边缘服务

Spring Cloud Netflix 包括了一个内嵌的Zuul代理，我们可以使用`@EnableZuulProxy`注解来启用。这能够让网关(Gateway)应用成为一个代理并转发相关的请求到其他服务中，例如我们的Book服务。

打开网关应用的`GatewayApplication`类并添加该注解，如下所示：

`gateway/src/main/java/hello/GatewayApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {

  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }
}
```

为了能够让请求从网关应用中转发出去，我们需要告诉Zuul路由信息来监测和服务并决定向什么路由进行转发。我们在properties的`zuul.routes`前缀下指定路由。每一个微服务可以在`zuul.routes.NAME`下定义一个入口，`NAME`是应用的名称(即在`spring.application.name`属性中存储的值)。



在网关应用的`src/main/resources`文件夹下添加`application.properties`。它应该如下所示：

`gateway/src/main/resources/application.properties`

```properties
zuul.routes.books.url=http://localhost:8090

ribbon.eureka.enabled=false

server.port=8080
```

Spring Cloud Zuul 本会根据应用的名称自动设定路径。在这例子中，因为我们设置了`zuul.routes.books.url`，所以Zuul将会代理请求至`/book`到该 url。

注意上述文件的第二个到最后一个属性：Spring Cloud Netflix 使用 Netflix Ribbon 来实现客户端的负载均衡。默认情况下，Ribbon 将会使用 Netflix Eureka 作为服务发现。对于一个简单的例子来说，我们跳过了服务发现应用，所以我们设置`ribbon.eureka.enabled`为`false`。这样，Ribbon 不会使用`Eureka`来查找服务，我们必须为 Book 服务指定`url`。

## 添加一个过滤器

现在，让我们看看如何在代理服务中过滤请求。Zuul有四个标准的过滤类型：

- **pre** 过滤器能够在请求被路由之前执行
- **route** 过滤器能够处理请求的实际路由
- **post** 过滤器能够在请求被路由之后执行
- **error** 过滤器能够在处理请求发生错误时被执行

让我们来写一个 *pre* 过滤器。Spring Cloud Netflix 认为所有标注了`@Bean`的`com.netflix.zuul.ZuulFilter`子类都是过滤器，并且他们都能够在应用上下文中被获取到。创建一个新的目录，`src/main/java/hello/filters/pre`，并且在其中创建过滤文件`SimpleFilter.java`：

`gateway/src/main/java/hello/filters/pre/SimpleFilter.java`

```java
package hello.filters.pre;

import javax.servlet.http.HttpServletRequest;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.ZuulFilter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class SimpleFilter extends ZuulFilter {

  private static Logger log = LoggerFactory.getLogger(SimpleFilter.class);

  @Override
  public String filterType() {
    return "pre";
  }

  @Override
  public int filterOrder() {
    return 1;
  }

  @Override
  public boolean shouldFilter() {
    return true;
  }

  @Override
  public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    HttpServletRequest request = ctx.getRequest();

    log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));

    return null;
  }

}
```

过滤器类将实现4个方法：

- `filterType()`返回一个`String`。这代表了过滤器的类型，在该例中，`pre`或者可以是`route`来作为一个过滤器的路由。
- `filterOrder()`给出过滤器执行的顺序，或者相对于其他过滤器的顺序。
- `shouldFilter()`包含决定是否执行过滤器的逻辑，在该例中，它将会*始终*执行。
- `run()`包含了过滤的功能。

Zuul过滤器在`RequestContext`中存储（或分享）请求的信息。我们可以使用它来获取`HttpServletRequest`，然后我们可以在转发前记录( log )HTTP请求的方法 (HTTP method) 和URL。

`GatewayApplication`类标注了`@SpringBootApplication`注解，这等效于（包括）`@Configuration`注解。该注解告诉Spring在给定的类中查找@Bean定义。为我们的`SimpleFilter`在这里添加一个Bean定义。

`gateway/src/main/java/hello/GatewayApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.context.annotation.Bean;
import hello.filters.pre.SimpleFilter;

@EnableZuulProxy
@SpringBootApplication
public class GatewayApplication {

  public static void main(String[] args) {
    SpringApplication.run(GatewayApplication.class, args);
  }

  @Bean
  public SimpleFilter simpleFilter() {
    return new SimpleFilter();
  }

}
```

## 试试看

确保两个应用程序都在运行。在浏览器中，通过网关应用访问Book应用。如果你使用本文前面展示的配置，你可以直接通过`localhost：8090`访问Book服务，并且通过`localhost:8080/books`访问基于网关的Book服务。

访问Book服务的一个终端，例如`localhost:8080/books/available`，并且你可以看到在访问Book应用之前，你的请求被记录在网关应用的控制台中。

```
2016-01-19 16:51:14.672  INFO 58807 --- [nio-8080-exec-6] hello.filters.pre.SimpleFilter           : GET request to http://localhost:8080/books/available
2016-01-19 16:51:14.672  INFO 58807 --- [nio-8080-exec-6] o.s.c.n.zuul.filters.ProxyRouteLocator   : Finding route for path: /books/available
```

## 总结

恭喜！你已经使用了Spring开发了一个可以代理和过滤微服务请求的边缘服务应用程序。

## 参考阅读

以下指南也可能有帮助：

- [使用SpringBoot构建应用程序](https://spring.io/guides/gs/spring-boot/)
- [构建一个RESTful Web服务](https://spring.io/guides/gs/rest-service/)


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/)协议进行许可。
