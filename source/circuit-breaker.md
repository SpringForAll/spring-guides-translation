【翻译完成】

> 原文：[Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/)
>
> 译者：[ligang](http://github.com/holy12345/)
>
> 校对：

# 断路器

本指南将引导你了解使用Netflix Hystrix容错库将断路器应用于潜在故障方法调用的过程。

## 你会得到什么？

你将构建一个微服务应用程序,当方法调用失败时,使用 [断路器模式](http://martinfowler.com/bliki/CircuitBreaker.html) 完成降级功能. 使用断路器模式可以允许微服务程序在相关服务发生故障时继续运行,防止故障级联并给出故障恢复服务时间.

## 你需要准备什么？

- 大约15分钟
- 一个你喜欢的文本编辑器或者IDE开发工具
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或者更高的版本
- [Gradle 2.3+](http://www.gradle.org/downloads)或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到本地开发工具中:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 怎样完成指南？

像大多数Spring [入门指南](https://spring.io/guides)一样,你可以从头开始,完成每一步,也可以绕过已经熟悉的基本设置步骤. 无论哪种方式,你最后都会得到一份可执行的代码.

 **如果从基础开始**,你可以往下查看 [怎样使用 Gradle 构建项目](https://spring.io/guides/gs/circuit-breaker/#scratch).

 **如果已经熟悉跳过一些基本步骤**,你可以:

- [下载](https://github.com/spring-guides/gs-circuit-breaker/archive/master.zip) 并解压源码库, 或者通过 [Git](https://spring.io/understanding/Git) 克隆: `git clone https://github.com/spring-guides/gs-circuit-breaker.git`
- 进入 `gs-circuit-breaker/initial`目录
- 跳过前面的部分 [设置服务端微服务应用程序](https://spring.io/guides/gs/circuit-breaker/#initial).

**当你完成之后**, 你可以根据代码检查结果 `gs-circuit-breaker/complete`.

## 使用Gradle构建

首先你需要编写基础构建脚本.在构建 Spring 应用的时候,你可以使用任何你喜欢的系统来构建, 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/) 构建的代码. 如果你两者都不是很熟悉, 你可以先去参考 [如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle) 或 [如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven).

### 创建目录结构

在你的项目根目录,创建如下的子目录结构; 例如,如果你使用的是*nix系统,你可以使用 `mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份 [初始化Gradle构建文件](https://github.com/spring-guides/gs-circuit-breaker/blob/master/initial/build.gradle).

`bookstore/build.gradle`

```
buildscript {
	ext {
		springBootVersion = '1.5.2.RELEASE'
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
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'bookstore'
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
```

`reading/build.gradle`

```
buildscript {
	ext {
		springBootVersion = '1.5.2.RELEASE'
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
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'reading'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-hystrix')
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:Camden.SR5"
	}
}


eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)  提供了很多非常方便的功能:

- 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件,使得运行和发布你的服务变得更加便捷.
- 搜索 `public static void main()`方法并且将它标记为可执行类.
- 提供了将内部依赖的版本都去匹配[Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). 你可以根据你的需要来重写版本, 但是它默认提供给了 Spring Boot 依赖的版本.



## 使用Maven构建

首先你需要编写基础构建脚本.在构建 Spring 应用的时候,你可以使用任何你喜欢的系统来构建,这里提供一份你可能需要用 [Maven](https://maven.apache.org/) 构建的代码.如果您不熟悉Maven,请参阅 [使用Maven构建Java项目](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中,创建以下子目录结构.例如, 如果你使用的是*nix系统:`mkdir -p src/main/java/hello` :

```
└── src
    └── main
        └── java
            └── hello
```

`bookstore/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>bookstore</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
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

`reading/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>reading</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
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
				<version>Camden.SR5</version>
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

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性:

- 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件,使得运行和发布你的服务变得更加便捷.
- 搜索 `public static void main()`方法并且将它标记为可执行类.
- 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). 你可以根据你的需要来重写版本, 但是它默认提供给了 Spring Boot 依赖的版本.



## 通过IDE构建项目

- 如何使用Spring Tool Suite构建项目请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- 如何使用IntelliJ IDEA构建项目请参照 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea).

## 设置服务端微服务应用程序

书店服务将有一个端点. 它可以访问`/recommended`,并（为了简单）返回一个`String`推荐的阅读列表。

在`BookstoreApplication.java`中编辑你的主类。应该看起来像这样:

`bookstore/src/main/java/hello/BookstoreApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
@SpringBootApplication
public class BookstoreApplication {

  @RequestMapping(value = "/recommended")
  public String readingList(){
    return "Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)";
  }

  public static void main(String[] args) {
    SpringApplication.run(BookstoreApplication.class, args);
  }
}
```

`@RestController`注释将`BookstoreApplication`标记为控制器类，如`@Controller`,并确保此类中的`@RequestMapping`方法的行为尽可能用`@ResponseBody`进行注释. 也就是说,此类中的`@RequestMapping`方法的返回值将自动从其原始类型转换并直接写入响应体.

我们将在本地与客户端服务应用程序一起运行此应用程序,因此在`src/main/resources/ application.properties`中设置`server.port`,以便当我们运行时，Bookstore服务不会与客户端冲突,

`bookstore/src/main/resources/application.properties`

```
server.port=8090
```

## 设置客户端微服务应用程序

阅读应用程序将是我们的书店应用服务的前端（我们可以看到）我们将能够通过`/to-read`查看我们的阅读列表,并将从书店服务应用程序检索该阅读列表.

`reading/src/main/java/hello/ReadingApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.client.RestTemplate;
import java.net.URI;

@RestController
@SpringBootApplication
public class ReadingApplication {

  @RequestMapping("/to-read")
  public String readingList() {
    RestTemplate restTemplate = new RestTemplate();
    URI uri = URI.create("http://localhost:8090/recommended");

    return restTemplate.getForObject(uri, String.class);
  }

  public static void main(String[] args) {
    SpringApplication.run(ReadingApplication.class, args);
  }
}
```

要从书店服务获取列表,我们使用Spring的`RestTemplate`模板类.当我们使用它时,`RestTemplate`会向书店服务的URL发出HTTP GET请求,然后以`String`形式返回结果.（有关使用Spring来使用RESTful服务的更多信息,请参阅）[使用RESTful Web Service指南](https://spring.io/guides/gs/consuming-rest/) 指南.)

将 `server.port` 属性添加到 `src/main/resources/application.properties`:
`reading/src/main/resources/application.properties`
```
server.port=8080
```

我们现在可以在浏览器中访问我们的阅读应用程序的`to-read`端点,并查看我们的阅读列表. 然而,由于我们依靠书店服务`BookService`,如果发生任何事情,或者如果阅读完全无法访问书店服务`BookService`,我们将没有列表,我们的用户将收到一个讨厌的HTTP 500错误消息.

## 应用断路器模式

Netflix的Hystrix库提供了断路器模式的实现:当我们将一个断路器应用于一个方法时,Hystrix会监控该方法的失败情况,如果故障达到阈值,Hystrix将打开断路器,以便后续访问该方法自动失败. 当断路器打开时,Hystrix会重新调用该方法,并将它们传递到我们指定的回退方法.

Spring Cloud Netflix Hystrix查找使用`@HystrixCommand`注释的任何方法,并将该方法包装在连接到断路器的代理中,以便Hystrix可以监控它. 这当前只能在标有`@Component`或`@Service`的类中工作,所以在阅读应用程序中,在`src/main/java/hello`下,添加一个新类：`BookService`.

在创建`BookService`时,`RestTemplate`将被注入到`BookService`的构造函数中.完整的类应该是这样的:

`reading/src/main/java/hello/BookService.java`

```java
package hello;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.net.URI;

@Service
public class BookService {

  private final RestTemplate restTemplate;

  public BookService(RestTemplate rest) {
    this.restTemplate = rest;
  }

  @HystrixCommand(fallbackMethod = "reliable")
  public String readingList() {
    URI uri = URI.create("http://localhost:8090/recommended");

    return this.restTemplate.getForObject(uri, String.class);
  }

  public String reliable() {
    return "Cloud Native Java (O'Reilly)";
  }

}
```

我们已经将`@HystrixCommand`应用于我们原来的`readingList()`方法. 我们也有一个新的方法：`reliable()`. `@HystrixCommand`注释作为它的回退方法是可靠的,所以如果由于某些原因`Hystrix`在`readingList()`上打开断路器,我们将为我们的用户准备一个非常好的（如果是短的）占位符阅读列表.

在我们的主类`ReadingApplication`中,我们将创建一个`RestTemplate bean`,注入`BookService`并将其称为阅读列表：

`reading/src/main/java/hello/ReadingApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.web.client.RestTemplate;

@EnableCircuitBreaker
@RestController
@SpringBootApplication
public class ReadingApplication {

  @Autowired
  private BookService bookService;

  @Bean
  public RestTemplate rest(RestTemplateBuilder builder) {
    return builder.build();
  }

  @RequestMapping("/to-read")
  public String toRead() {
    return bookService.readingList();
  }

  public static void main(String[] args) {
    SpringApplication.run(ReadingApplication.class, args);
  }
}
```

现在,要从书店服务检索列表,我们调用`bookService.readingList()`.你还会注意到,我们添加了最后一个注释,`@ EnableCircuitBreaker`; 告诉Spring Cloud有必要在使用阅读应用程序中使用断路器并启用它的监控功能,打开和关闭(在我们的例子中由Hystrix提供的行为).

## 试着运行下

运行书店服务和阅读服务,然后打开一个浏览器到阅读服务,在`localhost:8080/to-read`. 你应该看到完整的推荐阅读列表:

```
Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)
```

现在关闭书店应用程序.我们的清单来源已经不见了,但是由于Hystrix和Spring Cloud Netflix,我们有一个可靠的缩写列表来弥补; 你应该看到:

```
Cloud Native Java (O'Reilly)
```

## 总结

祝贺你!您刚刚开发了一个Spring应用程序,它使用断路器模式来防止级联故障,并为潜在的失败调用提供回退行为.

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
