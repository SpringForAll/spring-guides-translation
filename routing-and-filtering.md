# 路由和过滤器

> 原文：[Routing and Filtering](https://spring.io/guides/gs/routing-and-filtering/)
>
> 译者：[RealDeanZhao](https://github.com/RealDeanZhao)
>
> 校对：

本指南会带你走一遍在微服务应用中使用Netflix Zuul边缘服务（edge service）来进行请求的路由以及过滤的流程。

## 你将要构建什么

你将编写一个简单的微服务应用，然后使用[Netflix Zuul](https://github.com/spring-cloud/spring-cloud-netflix)来构建一个反向代理应用从而帮助转发请求到服务应用。你将会明白怎么用Zuul从代理服务中过滤请求。

## 你需要准备什么

- 大约15分钟
- 最喜欢的编辑器或者IDE
- [JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或者更新版本
- [Gradle2.3+](https://gradle.org/install/)或者[Maven3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接把代码拷贝进IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts/)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成这篇指南

正如大多数Spring[入门指南](https://spring.io/guides), 你可以从头开始，完成每一步, 也可以跳过你已经熟悉的基本配置步奏。无论使用哪种方式，你最终会得到可用的代码。

**若想从头开始**，请移步到[使用Gradle构建](https://spring.io/guides/gs/routing-and-filtering/#scratch)。

**若想跳过基础部分**，执行以下操作：

- [下载](https://github.com/spring-guides/gs-routing-and-filtering/archive/master.zip)并解压本指南的源码，或者使用[Git](https://spring.io/understanding/Git)：`git clone https://github.com/spring-guides/gs-routing-and-filtering.git`
- 进入`gs-routing-and-filtering/initial`目录
- 前往[构建微服务章节](#initial)。

**当你完成后**，可以对比`gs-routing-and-filtering/complete`目录的代码来检查结果。

## 使用Gradle构建

首先你需要安装基础构建脚本。当然，你也可以使用任何你喜欢的构建系统来构建Spring应用。你需要的关于使用[Gradle](https://gradle.org/)以及[Maven](https://maven.apache.org/)的代码已经在本指南中包含。如果你对两者都不熟悉，请参考[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle/)或者[使用Maven构建Java项目](https://spring.io/guides/gs/maven/)。

### 创建目录结构

在你选择的项目目录中，创建如下子目录结构；比如在*nix系统中使用命令`mkdir -p src/main/hava/hello`。

```text
└── src
   └── main
        └── java
            └── hello
```

### 创建Gradle构建文件。

如下是[初始化的Gradle构建文件](https://github.com/spring-guides/gs-routing-and-filtering/blob/master/initial/build.gradle)。

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
	baseName = 'gateway'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-zuul')
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:Brixton.SR5"
	}
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

[Spring Boot gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了许多方便的功能：

- 将classpath中所有jar包收集起来，构建成一个单独的可以执行的“über-jar”，这样可以更方便的执行以及交付你所提供的服务。
- 它会搜索`public static void main()`方法，并且将它标记成一个可执行的类。
- 它提供内置的依赖解析器，此解析器会设置一个与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)所匹配的版本号。你也可以改成任何版本，但默认设置为Spring Boot所选择的版本集。

## 使用Maven构建

首先你需要安装基础构建脚本。当然，你也可以使用任何你喜欢的构建系统来构建Spring应用。你需要的关于使用[Maven](https://maven.apache.org/)的代码已经在本指南中包含。如果你对Maven不熟悉，请参考[使用Maven构建Java项目](https://spring.io/guides/gs/maven/)。

### 创建目录结构

在你选择的项目目录中，创建如下子目录结构；比如在*nix系统中使用命令`mkdir -p src/main/hava/hello`。

```text
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

[Spring Boot Maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了许多方便的功能：

- 将classpath中所有jar包收集起来，构建成一个单独的可以执行的“über-jar”，这样可以更方便的执行以及交付你所提供的服务。
- 它会搜索`public static void main()`方法，并且将它标记成一个可执行的类。
- 它提供内置的依赖解析器，此解析器会设置一个与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)所匹配的版本号。你也可以改成任何版本，但默认设置为Spring Boot所选择的版本集。

## 使用IDE构建

- 阅读如何将本指南直接导入[Spring Tool Suite](https://spring.io/guides/gs/sts/)
- 阅读如何结合本指南使用[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 建立微服务

Book Service将会非常简单。参照如下代码编辑`BookApplication.java`：

`book/src/main/java/hello/BookApplication.java`

```java
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

`BookApplication`是一个REST控制器。`@RestController`将一个类标记成一个REST控制器，同时确保`@RequestMapping`所标记的方法会自动将返回值做合适的转换以及直接写入HTTP响应。

我们添加了两个`@RequestMapping`所标记的方法，一个是`available()`，另一个是`checkedOut()`。他们处理来自`/available`以及`/checked-out`路径的请求，两者都简单的返回一个`String`类型的书名。

在配置文件`src/main/resources/application.properties`，设置应用名称(`book`)。

`book/src/main/resources/application.properties`

```text
spring.application.name=book

server.port=8090
```

与此同时，我们也设置了`server.port`以确保当所有服务在本地运行时，不会与边缘服务（edge service）冲突。

## 创建边缘服务（edge service）

Spring Cloud Netflix包含内置的Zuul代理，它可以通过`@EnableZuulProxy`注解启用。这样网关（Gateway）应用就会变成一个反向代理，并且转发相应的请求到其他服务，比如Book Service。

打开网关（Gateway）应用`GatewayApplication`的类并且参考下边代码加入注解：

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

为了让网关（Gateway）应用能够转发请求，我们需要告诉Zuul它所需要监视的路由以及这些路由对应的服务。我们使用`zuul.routes`属性来指定routes。每一个服务为都可以通过`zuul.routes.NAME`作为入口，`NAME`是应用程序名称，它被设置在`spring.application.name`属性当中。

在网关（Gateway）应用中新增目录：`src/main/resources`，然后在此目录添加文件：`application.properties`，参照如下内容编辑：

`gateway/src/main/resources/application.propertie`

```text
zuul.routes.books.url=http://localhost:8090

ribbon.eureka.enabled=false

server.port=8080
```

Spring Cloud Zuul会自动将应用程序名称设置为路径。本例中，因为我们设置了`zuul.routes.books.url`，因此Zuul会将发送到/books路径的请求转发到此URL。

注意倒数第二个属性：Spring Cloud Netflix Zuul使用Netflix的Ribbon作客户端负载均衡，同时Ribbon会使用Netflix Eureka作服务发现（Service Discovery）。在这个简单例子中，我们将会跳过服务发现（Service Discovery），所以设置`ribbon.eureka.enabled`为`false`。由于Ribbon现在不能使用Eureka寻找服务，我们必须为Book Service指定URL。

## 添加过滤器

现在让我们看看怎么通过代理服务来过滤请求。Zuul有如下标准的过滤器类型：

- `pre` 过滤器在请求被路由前执行。
- `route` 过滤器处理请求的实际路由。
- `post` 过滤器在请求被路由后执行。
- `error` 过滤器在处理请求中发生错误时执行。

 我们会写一个pre过滤器。Spring Cloud Netflix会选择所有的在当前应用上下文（Application Context）可用的并且继承ZuulFilter的`@Bean`作为过滤器。创建一个新的目录，`src/main/java/hello/filters/pre`，并且在此目录创建过滤器文件，`SimpleFilter.java`：

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

此过滤器类实现了4个方法：

- `filterType()` 返回一个表示过滤器类型的字符串。在本例中，它可以是`pre`或者`route`。
- `filterOrder()` 指定了当前过滤器相对于其他过滤器的执行顺序。
- `shouldFilter()` 包含了决定什么时候执行此过滤器的逻辑（本例中的过滤器会永远执行）。
- `run()` 包含了此过滤器所包含的功能。

Zuul过滤器会将请求以及声明信息在`ReqestContext`中保存并共享。我们可以使用它来获取`HttpServletRequest`，并在当前请求被转发之前，记录它的HTTP method以及URL。

`GatewayApplication`类被`@SpringBootApplication`注解，正如其他的`@Configuration`注解，它会告诉Spring查看指定类的`@Bean`定义。在此类中再添加一个如下的`SimpleFilter`：

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

## 测试一下

确保两个应有都被跑起来了。在浏览器中，通过网关（Gateway）应用访问Book应用的一个端点。如果你使用了本指南的配置，你可以通过直接访问URL`localhost:8090`或者通过网关（Gateway）应用的URL`localhost:8080/books`来访问Book Service。

访问Book Service的一个端点，比如`localhost:8080/books/available`，你将会看到网关（Gateway）应用在请求被发送到Book应用之前记录了它的method。

```text
2016-01-19 16:51:14.672  INFO 58807 --- [nio-8080-exec-6] hello.filters.pre.SimpleFilter           : GET request to http://localhost:8080/books/available
2016-01-19 16:51:14.672  INFO 58807 --- [nio-8080-exec-6] o.s.c.n.zuul.filters.ProxyRouteLocator   : Finding route for path: /books/available
```

## 总结

恭喜！你已经学会使用Spring开发一个边缘服务（edge service）应用， 并且用它来为你的微服务代理以及过滤请求。

## 同样参考

如下指南也可能会很有帮助：

- [使用Spring Boot构建应用](https://spring.io/guides/gs/spring-boot/)
- [构建Restful Web服务](https://spring.io/guides/gs/rest-service/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
