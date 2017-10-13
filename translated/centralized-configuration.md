> 原文：[Centralized Configuration](https://spring.io/guides/gs/centralized-configuration/)
>
> 译者：[zaixiandemiao](https://github.com/zaixiandemiao)
>
> 校对：[hh23485](https://github.com/hh23485)

# 配置中心

本指南将引导你使用[Spring Cloud Config Server](https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html)来创建和使用配置中心。

## 你将要构建什么

你首先将构建一个配置服务[Config Server](https://cloud.spring.io/spring-cloud-config/spring-cloud-config.html)，之后将创建一个客户端来在启动时从该服务中读取配置信息并在不重启客户端的情况下刷新配置信息。

## 你需要什么

- 约15分钟
- 一个你喜欢的文本编辑器或IDE
- [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)≥1.8
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以将代码直接导入你的IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成这篇指南

像大多数的Spring指南 [Getting Started guides](https://spring.io/guides),你可以从零开始，完成每一步，也可以跳过已经熟悉的基本设置。 无论哪种方式，你都会得到起作用的代码。

 **从零开始**, 移步到[Build with Gradle](#gradle).

**跳过基础部分**, 执行以下操作:

- [下载](https://github.com/spring-guides/gs-securing-web/archive/master.zip) 并解压本仓库的源码, 或者使用 [Git](https://spring.io/understanding/Git): 克隆 `git clone https://github.com/spring-guides/gs-centralized-configuration.git`
- `cd`进入 `gs-centralized-configuration/initial`  
- 跳到 [Set up Spring Security](https://spring.io/guides/gs/securing-web/#initial)这一节.

**当你完成后**, 你可以和此处的代码进行对比 `gs-centralized-configuration/complete`.

<h2 id="gradle"> 使用Gradle构建</h2>

首先你得安装基础的构建脚本. 你可以使用任意你喜欢的构建系统去构建Spring应用, 但你使用 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/)构建需要的代码在本文提供. 如果你对两者都不熟悉,可以先参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) 或者 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle文件

如下是一个 [Gradle初始化文件](https://github.com/spring-guides/gs-securing-web/blob/master/initial/build.gradle).

`configuration-service/build.gradle`

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
		classpath('io.spring.gradle:dependency-management-plugin:0.5.4.RELEASE')
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'configuration-service'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.cloud:spring-cloud-config-server')
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

`configuration-client/build.gradle`

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
		classpath('io.spring.gradle:dependency-management-plugin:0.5.4.RELEASE')
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'configuration-client'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-config')
	compile('org.springframework.boot:spring-boot-starter-actuator')
	compile('org.springframework.boot:spring-boot-starter-web')
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

 [Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个单个的、可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类.
- 它提供了一个内置的依赖解析器，将应用与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用Maven构建

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，但你用 [Maven](https://maven.apache.org) 构建时的代码在本章节提供。如果你对 Maven 还不是很熟悉，你可以先去看下[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven/).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

为了让你快速地开始构建项目，这里提供了server和client两个应用的完整配置。

`configuration-service/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>configuration-service</artifactId>
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
			<artifactId>spring-cloud-config-server</artifactId>
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

`configuration-client/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>configuration-client</artifactId>
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
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
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

 [Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类.
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用你的IDE构建

- 阅读如何导入这篇指南进入你的IDE [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建.

## 创建一个配置服务

你首先需要一个配置服务来充当你的Spring应用和一个存放配置文件的典型的版本控制仓库之间的媒介。你可以使用Spring Cloud的`@EnableConfigServer`注解来创建一个配置服务器以供其他应用访问。这是一个普通的Spring boot应用，你需要做的仅仅是添加一个注解来使得配置服务可用。

`configuration-service/src/main/java/hello/ConfigServiceApplication.java`  

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@EnableConfigServer
@SpringBootApplication
public class ConfigServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServiceApplication.class, args);
    }
}
```

这个配置服务需要知道要管理的仓库的位置。这里有几种可选项，但我们将使用一个基于Git的文件系统来作为仓库。你也可以简单地把配置服务指向Github或者Gitlab上的一个repository。在这个文件系统上，创建一个新目录并`git init`该目录。之后添加一个名为`a-bootiful-client.properties`的文件到该Git repository中。确保正确的执行了`git commit`步骤之后，你将通过一个Spring Boot应用来连接该配置服务器。这个Spring Boot应用的`spring.application.name`属性被设置为`a-bootiful-client`,这也是配置服务器中对于该Spring Boot应用的唯一标识。这就是配置服务器如何知道发送哪些配置信息到特定客户端的方式。配置服务器也会发送Git repository中名为`application.properties`或`application.yml`的文件中的全部配置信息。更加具体命名的文件（如`a-bootiful-client.properties`）中的配置信息会覆盖掉
`application.properties`或`application.yml`文件中重复的属性值。

添加一个简单的属性和值`message = Hello world`,将它加入`a-bootiful-client.properties`文件中并`git commit`该修改后的文件。

配置服务器中`configuration-service/src/main/resources/application.properties`文件内的`spring.cloud.config.server.git.uri`属性指定了配置文件所在的Git repository。在同一台电脑上运行config server和client的时候，确保`server.port`属性被指定为不同的值来防止端口冲突。

`configuration-service/src/main/resources/application.properties`

```
server.port=8888

spring.cloud.config.server.git.uri=${HOME}/Desktop/config
```


## 使用client读取配置服务器中的配置信息

现在我们已经创建了一个配置服务器了，让我们创建一个新的Spring Boot应用来使用该配置服务器加载客户端自己的配置信息并且在不重启JVM的情况下，刷新配置服务器中修改过的属性值。连接Config Server时，需要添加`org.springframework.cloud:spring-cloud-starter-config`依赖。Spring从服务端读取的配置信息与从本地项目中的`application.properties`或`application.yml`文件或其他配置源中读取到的相同，没有差别。

在启动(bootstrap)阶段，Config Client的本地配置信息必须优先于远程从Config Server中读取的配置。在`configuration-client/src/main/resources/bootstrap.properties`文件中指定Client的`spring.application.name`为`a-bootiful-client`,并指定配置服务器的位置`spring.cloud.config.uri`.该文件在Client启动时优先于任何其他的配置文件。

`configuration-client/src/main/resources/bootstrap.properties`

```
spring.application.name=a-bootiful-client
# N.B. this is the default:
spring.cloud.config.uri=http://localhost:8888
management.security.enabled=false
```

使用传统的方法（如`@ConfigurationProperties, @Value("${…​}")`，或者使用`Environment `抽象类），客户端可以访问Config Server中的任何值。创建一个Spring MVC REST controller来返回读取到的`message`属性值。你可以查看[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)指南来学习如何使用Spring MVC和Spring Boot来创建一个REST 服务。

默认情况下，客户端启动时读取配置信息，并不会再次读取配置。通过给`MessageRestController`添加Spring Cloud Config的`@RefreshScope`注解，你可以强制一个bean对象在配置服务端触发了*refresh*事件之后，通过拉取更新值的方式，刷新客户端已经读取的配置信息。

`configuration-client/src/main/java/hello/ConfigClientApplication.java`

```java
package hello;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }
}

@RefreshScope
@RestController
class MessageRestController {

    @Value("${message:Hello default}")
    private String message;

    @RequestMapping("/message")
    String getMessage() {
        return this.message;
    }
}
```

## 测试这个应用

为完成端对端结果的测试，你需要首先启动Config Service，当启动完毕后，启动客户端。在浏览器中输入`http://localhost:8080/message`访问client应用，你将得到`Hello world`字符串作为返回值。

在Git repository中，将`a-bootiful-client.properties`文件中的`message`值修改为一个不同的值(如`Hello Spring!`)。你可以通过访问`http://localhost:8888/a-bootiful-client/default`来确保Config Server中该值发生了变化。你需要触发一下Spring Boot Actuator的`refresh` endpoint，来强制client刷新自身并将修改后的属性值读入。Spring Boot Actuator暴露了一些操作端口，如应用的健康检查和环境信息。为了使用Actuator，你必须将`org.springframework.boot:spring-boot-starter-actuator`添加到client应用的CLASSPATH中。你可以通过向client的`refresh` endpoint（`http://localhost:8080/refresh`）发送一个空的HTTP `POST`请求来调用Actuator的`refresh` endpoint。之后，通过浏览器输入`http://localhost:8080/message`来查看client是否刷新了`message`的属性值。

> 我们设置`management.security.enabled=false`来使得client的测试变得容易（Spring Boot 1.5之后Actuator的endpoint默认是加密访问的）。如果你不设置该标记为false，默认情况下你仍可以通过JMX来访问Actuator的endpoint

## 总结

恭喜！你刚刚使用Spring为你所有的服务完成了一个启动时读取和动态更新的配置中心。

## 了解更多

下面的指南也非常有帮助。

* [使用Spring Boot构建一个应用](https://spring.io/guides/gs/spring-boot/)
* [创建一个多模块的项目](https://spring.io/guides/gs/multi-module/)

--------------------------------------------------------------------------------


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/)协议进行许可。
