【ligang翻译完成】

> 原文：[Client Side Load Balancing with Ribbon and Spring Cloud](https://spring.io/guides/gs/client-side-load-balancing/)
>
> 译者：[ligang](http://github.com/holy12345/)
>
> 校对：

# 通过Ribbon和Spring Cloud完成客户端负载均衡

本指南将指导你使用Netflix Ribbon为微服务应用程序提供客户端负载均衡。

## 你将得到什么

你将构建一个使用Netflix Ribbon和Spring Cloud Netflix的微服务应用程序，以便在对另一个微服务的调用中提供客户端负载均衡。

## 你需要准备什么

- 大约15分钟
- 一个你喜欢的文本编辑器或者IDE开发工具
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或者更高的版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到本地开发工具中:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 怎样完成指南

像大多数Spring [入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。无论哪种方式，你最后都会得到一份可执行的代码。

 **如果从基础开始**，你可以从[使用Gradle构建](https://spring.io/guides/gs/client-side-load-balancing/#scratch)小节开始。

**如果已经熟悉跳过一些基本步骤**，可以按照以下步骤执行:

- [下载](https://github.com/spring-guides/gs-client-side-load-balancing/archive/master.zip) 并解压源码库， 或者通过 [Git](https://spring.io/understanding/Git)克隆： `git clone https://github.com/spring-guides/gs-client-side-load-balancing.git`
- 进入  `gs-client-side-load-balancing/initial`目录
- 直接跳到 [Write a server service](https://spring.io/guides/gs/client-side-load-balancing/#initial)小节。

**当你完成之后**，你可以根据代码检查结果 `gs-client-side-load-balancing/complete`.

## 使用Gradle构建

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建,，这里提供一份你可能需要用 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/) 构建的代码.。如果你两者都不是很熟悉,，你可以先去参考 [如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle) 或 [如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

你的项目根目录,创建如下的子目录结构；例如，如果你使用的是Unix/Linux系统，你可以使用  `mkdir -p src/main/java/hello` ：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份 [初始化Gradle构建文件](https://github.com/spring-guides/gs-client-side-load-balancing/blob/master/initial/build.gradle).

`say-hello/build.gradle`

```groovy
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
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'say-hello'
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

`user/build.gradle`

```groovy
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
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'user'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
	maven { url "https://repo.spring.io/snapshot" }
	maven { url "https://repo.spring.io/milestone" }
}


dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-ribbon')
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

- 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷。
- 搜索 `public static void main()`方法并且将它标记为可执行类。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用Maven构建

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Maven](https://maven.apache.org/) 构建的代码。如果您不熟悉Maven，请参阅 [使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构。例如， 如果你使用的是Unix/Linux系统: `mkdir -p src/main/java/hello` ：

```
└── src
    └── main
        └── java
            └── hello
```

`say-hello/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>say-hello</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>say-hello</name>
	<description>Demo project for Spring Boot</description>

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

`user/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>user</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>user</name>
	<description>Demo project for Spring Boot</description>

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
			<artifactId>spring-cloud-starter-ribbon</artifactId>
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

	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性:

- 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷。
- 搜索 `public static void main()`方法并且将它标记为可执行类。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 通过IDE构建项目

- 如何使用Spring Tool Suite构建项目请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 如何使用IntelliJ IDEA构建项目请参照 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)。

## 编写一个服务器服务

我们的服务器服务被称为Say Hello。 它将从可访问的端点`/greeting`返回一个随机问候语（从三个静态列表中选出）。

在`src/main/java/hello`中， 创建文件 `SayHelloApplication.java`。它应该看起来像这样:

`say-hello/src/main/java/hello/SayHelloApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;

@RestController
@SpringBootApplication
public class SayHelloApplication {

  private static Logger log = LoggerFactory.getLogger(SayHelloApplication.class);

  @RequestMapping(value = "/greeting")
  public String greet() {
    log.info("Access /greeting");

    List<String> greetings = Arrays.asList("Hi there", "Greetings", "Salutations");
    Random rand = new Random();

    int randomNum = rand.nextInt(greetings.size());
    return greetings.get(randomNum);
  }

  @RequestMapping(value = "/")
  public String home() {
    log.info("Access /");
    return "Hi!";
  }

  public static void main(String[] args) {
    SpringApplication.run(SayHelloApplication.class, args);
  }
}
```

`@RestController`注解的效果与我们一起使用`@Controller`和`@ResponseBody`效果相同。它将`SayHelloApplication`标记为控制器类（这是`@ Controller`的作用），并确保来自类的`@RequestMapping`方法的返回值将自动从其原始类型转换为适当的值，并直接写入响应体 (这是`@ResponseBody`的作用）。我们有一个用于`/ greeting`的`@ RequestMapping`方法，另一个用于根路径`/`。(当我们使用Ribbon的时候，我们需要第二种方法。)

我们将在客户端服务应用程序本地运行此应用程序的多个实例，因此创建目录`src/main/ resources`，在其中创建`application.yml`文件，然后在该文件中为`server.port`设置默认值。(我们将指示应用程序的其他实例在其他端口上运行，以便在运行时，Say Hello实例不会与客户端发生冲突。)当我们在这个文件中时， 我们也会为我们的服务设置`spring.application.name`。

`say-hello/src/main/resources/application.yml`

```
spring:
  application:
    name: say-hello

server:
  port: 8090
```

## 从客户端服务访问

用户应用程序将是我们的用户看到的。 它会调用Say Hello应用程序来获取问候语，然后在用户以 `/hi`访问端点时将其发送给我们的用户。

在用户应用程序目录的`src/main/java/hello`下，添加`UserApplication.java`文件：

`user/src/main/java/hello/UserApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@RestController
public class UserApplication {

  @Bean
  RestTemplate restTemplate(){
    return new RestTemplate();
  }

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/hi")
  public String hi(@RequestParam(value="name", defaultValue="Artaban") String name) {
    String greeting = this.restTemplate.getForObject("http://localhost:8090/greeting", String.class);
    return String.format("%s, %s!", greeting, name);
  }

  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
```

为了得到Say Hello的问候，我们使用了Spring的 `RestTemplate `模板类。 `RestTemplate `会向我们提供的Say Hello服务的URL发出一个HTTP GET请求，并将结果作为字符串提供给我们。 (有关使用Spring来使用RESTful服务的更多信息，请参阅使用RESTful Web服务指南)。 [使用REST风格的Web服务](https://spring.io/guides/gs/consuming-rest/) 。)

将`spring.application.name`和`server.port`属性添加到`src/main/resources/ application.properties`或`src/main/resources/application.yml`中：

`user/src/main/resources/application.yml`

```
spring:
  application:
    name: user

server:
  port: 8888
```

## 负载均衡服务器实例

现在，我们可以访问用户服务`/hi`，看到一个友好的问候语：

```shell
$ curl http://localhost:8888/hi
Greetings, Artaban!

$ curl http://localhost:8888/hi?name=Orontes
Salutations, Orontes!
```

从一个通过硬编码URL访问服务器转到一个负载均衡的解决方案，让我们通过Ribbon进行设置。 在`user/src/main/resources/`下的`application.yml`文件中添加如下属性：

`user/src/main/resources/application.yml`

```
say-hello:
  ribbon:
    eureka:
      enabled: false
    listOfServers: localhost:8090,localhost:9092,localhost:9999
    ServerListRefreshInterval: 15000
```

在Ribbon 客户端上配置属性。 Spring Cloud Netflix为我们的应用程序中每一个的Ribbon 客户端创建一个`ApplicationContext`。并且为客户端提供一组用于Ribbon组件实例的bean，其中包括：

- `IClientConfig`，存储客户端或负载均衡器的客户端配置，
- `ILoadBalancer`，代表软件负载均衡器，
- `ServerList`，定义了如何获取可供选择的服务器列表，
- `IRule`，描述了一个负载均衡策略，
- `IPing`，说明了如何执行服务器的周期性ping操作。 

在我们上面的例子中，客户端名为`say-hello`。 我们设置的属性是`eureka.enabled`（我们设置为false），`listOfServers`和`ServerListRefreshInterval`。 Ribbon中的负载均衡器通常从Netflix Eureka服务注册表获取其服务器列表。(有关在Spring Cloud中使用Eureka服务注册表的信息，请参阅[服务注册和发现](https://spring.io/guides/gs/service-registration-and-discovery/) 。)对于我们的简单目的，我们正在跳过Eureka，因此我们将`ribbon.eureka.enabled`属性设置为`false`，而将Ribbon设置为静态`listOfServers`。 `ServerListRefreshInterval`是Ribbon服务列表的刷新间隔（以毫秒为单位）。

在我们的`UserApplication`类中，切换`RestTemplate`以使用Ribbon客户端获取Say Hello的服务器地址：

`user/src/main/java/hello/UserApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.ribbon.RibbonClient;

@SpringBootApplication
@RestController
@RibbonClient(name = "say-hello", configuration = SayHelloConfiguration.class)
public class UserApplication {

  @LoadBalanced
  @Bean
  RestTemplate restTemplate(){
    return new RestTemplate();
  }

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/hi")
  public String hi(@RequestParam(value="name", defaultValue="Artaban") String name) {
    String greeting = this.restTemplate.getForObject("http://say-hello/greeting", String.class);
    return String.format("%s, %s!", greeting, name);
  }

  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
```

我们已经对`UserApplication`类做了一些其他的相关修改。 我们的`RestTemplate`现在也被标记为`LoadBalanced`；这告诉Spring Cloud我们想要利用其负载均衡的支持（在这种情况下，由Ribbon提供）。 在`say-hello`客户端服务中我们用`@RibbonClient`注解的`name`属性去关联另外一个配置类`configuration`，它包含了该客户端的额外配置。

我们需要创建这个类。 在`user/src/main/java/hello`目录中添加一个新文件`SayHelloConfiguration.java`：

`user/src/main/java/hello/SayHelloConfiguration.java`

```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.IPing;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.PingUrl;
import com.netflix.loadbalancer.AvailabilityFilteringRule;

public class SayHelloConfiguration {

  @Autowired
  IClientConfig ribbonClientConfig;

  @Bean
  public IPing ribbonPing(IClientConfig config) {
    return new PingUrl();
  }

  @Bean
  public IRule ribbonRule(IClientConfig config) {
    return new AvailabilityFilteringRule();
  }

}
```

我们可以通过使用相同名称创建自己的bean来覆盖Spring Cloud Netflix为我们提供的与Ribbon相关的所有Bean。 在这里，我们覆盖默认负载均衡器使用的`IPing`和`IRule`。 默认`IPing`是`NoOpPing`（实际上并不ping服务器实例，而是总是报告它们是稳定的），默认的`IRule`是`ZoneAvoidanceRule`（这样可以避免服务器故障最多的亚马逊 EC2区域， 在我们当地的环境中尝试有点困难）。

我们的`IPing`是一个`PingUrl`，它将ping一个URL来检查每个服务器的状态。 调用Say Hello服务，如果服务正常运行访问`/`路径你将得到正确的应答；这意味着Ribbon会在运行Say Hello服务器时获得HTTP 200响应。 我们设置的`IRule`，`AvailabilityFilteringRule`，将使用功能区的内置断路器功能来过滤处于“开路”状态的任何服务器：如果ping无法连接到给定的服务器，或者如果读取失败 对于服务器，Ribbon将认为该服务器是挂掉的，直到它开始正常响应。

>`UserApplication`类的`@SpringBootApplication`注解等同于（其中）`@Configuration`注解，它将类标记为bean定义的来源。 这就是为什么我们不需要使用`@Configuration`注解在`SayHelloConfiguration`类上：因为它与`UserApplication`在同一个包中，所以它已经被扫描了bean的方法。这种方法的确意味着我们的Ribbon配置将成为主应用程序上下文的一部分 并因此被用户应用程序中的所有Ribbon客户端共享。 在正常的应用程序中，可以通过将Ribbon相关的Bean保留在主应用程序上下文中来避免这种情况（例如，在此示例中，可以将`SayHelloConfiguration`置于与`UserApplication`不同的包中）。

## 试着运行下

使用Gradle运行Say Hello服务：

```shell
$ ./gradlew bootRun
```

或者Maven:

```shell
$ mvn spring-boot:run
```

再次使用Gradle在端口9092和9999上运行其他实例:

```shell
$ SERVER_PORT=9092 ./gradlew bootRun
```

或者 Maven:

```shell
$ SERVER_PORT=9999 mvn spring-boot:run
```

然后启动用户服务。 访问`localhost:8888/hi`，然后观察Say Hello服务实例。 你可以看到Ribbon每15秒钟进行一次`ping`操作：

```shell
2016-03-09 21:13:22.115  INFO 90046 --- [nio-8090-exec-1] hello.SayHelloApplication                : Access /
2016-03-09 21:13:22.629  INFO 90046 --- [nio-8090-exec-3] hello.SayHelloApplication                : Access /
```

你对User服务的请求应该导致对Say Hello的调用以循环的方式传播到正在运行的实例上：

```shell
2016-03-09 21:15:28.915  INFO 90046 --- [nio-8090-exec-7] hello.SayHelloApplication                : Access /greeting
```

现在关闭一个Say Hello服务器实例。 一旦Ribbon`ping`到了关闭的实例并且认为是关闭，你应该可以看到请求在剩余实例中通过负载均衡选择一个。

## 总结

恭喜！你刚刚开发了一个Spring应用程序，该应用程序使用客户端负载均衡对另一个应用程序完成了调用。

想写一个新的指南或贡献一个现有的？ 看看我们的 [贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

