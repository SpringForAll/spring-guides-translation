# 服务注册和发现

> 原文：[Service Registration and Discovery](https://spring.io/guides/gs/service-registration-and-discovery/)
>
> 译者：[lovedboy2012](https://github.com/lovedboy2012)
>
> 校对：[chenzhijun](https://github.com/chenzhijun)

本指南将引导您了解Netflix Eureka服务注册中心的创建和消费过程

## 你将要构建什么
您将创建一个[Netflix Eureka服务注册中心][1]，然后构建一个能实现去Eureka服务器自注册的客户端，并向Eureka服务器提交自己主机的服务信息。
服务注册中心是有很用的，因为它不仅使得客户端负载均衡，而且服务提供者与消费者分离，而不需要DNS地址解析。

## 你需要什么
 - 大约需要15分钟
 - 一个您喜爱的文本编辑器或者IDE（集成开发工具）
 - [JDK 1.8][2]或更高版本
 - [Gradle 2.3+][3] 或者 [Maven 3.0+][4]
 - 您也可以直接导入代码到IDE中：
    - [Spring Tool Suite (STS)][5]
    - [IntelliJ IDEA][6]
    
## 如何完成这份指南
与大多数Spring[入门指南][7]一样，您可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。无论哪种方式，你都会得到工作代码。

如果从零开始，请继续使用Gradle构建。

如果打算跳过基本步骤，按照如下操作：

  - [下载][8]并解压这份指南的源码包，或者使用[Git][9]克隆：
     `git clone https://github.com/spring-guides/gs-scheduling-tasks.git`
  - 进入目录 `gs-service-registration-and-discovery/initial`
  - 跳转到 [Eureka服务注册和发现章节][10]。
  
完成这些后，您可以依据`gs-service-registration-and-discovery/complete`中的代码检查你的结果。

## 使用 Gradle 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Gradle][11] 或者 [Maven][12] 构建的代码。
如果你对两者都不是很熟悉，你可以先去看下[如何使用 Gradle 构建 Java 项目][13]或者[如何使用 Maven 构建 Java 项目][14]。

### 创建 Gradle 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

### 创建 Gradle 构建文件

下面是一份[初始化Gradle构建文件][15]

`eureka-service/build.gradle`

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
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'eureka-service'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencyManagement {
	imports {
		mavenBom 'org.springframework.cloud:spring-cloud-dependencies:Camden.SR5'
	}
}

dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-eureka-server')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}


eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}
```

`eureka-client/build.gradle`

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
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'eureka-client'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencyManagement {
	imports {
		mavenBom 'org.springframework.cloud:spring-cloud-dependencies:Camden.SR5'
	}
}

dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-eureka')
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
	testCompile('org.springframework.cloud:spring-cloud-starter-eureka-server')
}

eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}
```

[Spring Boot gradle 插件][16] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][17].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用 Maven 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Maven][18] 构建的代码。如果你对 Maven 还不是很熟悉，你可以先去看下[如何使用 Maven 构建 Java 项目][19].

### 创建 Maven 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

`eureka-service/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>eureka-service</artifactId>
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
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
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

`eureka-client/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>eureka-client</artifactId>
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
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
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

[Spring Boot Maven 插件][20] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][21].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用你的 IDE 进行构建

*   [如何在Spring Tool Suite中构建][22].

*   [如何在IntelliJ IDEA中构建][23].

## 开启Eureka服务注册
首先需要一个Eureka注册中心。你可以使用Spring Clound的注解`@EnabledEurekaServer`开启其他应用程序可以对话的注册服务器。
以下是一个通过注解方式开启了服务注册的普通Spring Boot 应用程序。

`eureka-service/src/main/java/hello/EurekaServiceApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }
}
```

当Eureka服务器启动后，若不记录服务注册连接信息的堆栈日志是会存在问题的，况且在生产环境中，还将会有多个服务实例存在。但是这里为了简单起见，不作相关日志的处理。

默认情况下，注册服务也会尝试自注册，所以你也需要关闭它。

本地使用时，习惯上推荐将每个注册服务使用单独的端口。

为了满足需要，添加一些属性配置到`eureka-service/src/main/resources/application.properties`

`eureka-service/src/main/resources/application.properties`

```
server.port=8761

eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false

logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF

```
## 与服务注册中心进行对话

现在我们已经建立了一个Eureka server服务器，现在让我们使用Spring Cloud 的注解`DiscoveryClient`来发现所有注册服务的主机和端口，从而创建一个实现自注册的客户端。在Spring Cloud中，注解`@EnableDiscoveryClient`不仅使Eureka作为服务注册组件实现了`DiscoveryClient`，同时为 [Hashicorp Consul][24] 和 [Apache Zookeeper][25]也都提供了支持
   
`eureka-client/src/main/java/hello/EurekaClientApplication.java`                             

```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@EnableDiscoveryClient
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}

@RestController
class ServiceInstanceRestController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @RequestMapping("/service-instances/{applicationName}")
    public List<ServiceInstance> serviceInstancesByApplicationName(
            @PathVariable String applicationName) {
        return this.discoveryClient.getInstances(applicationName);
    }
}

```

无论你选择什么组件实现，你都可以通过指定spring.application.name属性任何名称来注册eureka客户端，并且可以在server端看到该名称。
这个属性在Spring Cloud 中使用很多，通常是配置服务时最先配的。这个属性作为服务的系统配置，习惯上放在`eureka-client/src/main/resources/bootstrap.properties`这个属性文件，此配置文件在`src/main/resources/application.properties`之前加载。

`eureka-client/src/main/resources/bootstrap.properties`

```
spring.application.name=a-bootiful-client
```
`eureka-client`定义了一个名为`ServiceInstanceRestController`的Spring MVC REST服务站点，
通过访问[http：//localhost：8080/service-instances/a-bootiful-client][26]，将返回所有
`ServiceInstance`服务实例的注册列表。 请参阅[Building a RESTful Web Service][27]，
了解更多关于使用Spring MVC 和Spring Boot构建REST服务。

## 测试你的应用程序

首先启动`eureka-service`服务器，然后一旦`eureka-client`加载并启动成功，即可完成客户端到服务器的注册测试。
`eureka-client`需要大约一分钟注册中心去注册自己，并刷新从服务器获取的注册表信息，所有的这些阈值都是可配置的。
在浏览器中访问`eureka-client`，通过[http://localhost:8080/service-instances/a-bootiful-client][28]，可以看到该响应的`eureka-client`服务在注册中心的`ServiceInstance`。

## 总结

恭喜！ 您刚刚使用Spring来启动了Netflix Eureka服务注册中心，并在客户端应用程序中实现了服务的注册。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

[1]: https://github.com/spring-cloud/spring-cloud-netflix
[2]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
[3]: http://www.gradle.org/downloads
[4]: https://maven.apache.org/download.cgi
[5]: https://spring.io/guides/gs/sts
[6]: https://spring.io/guides/gs/intellij-idea/
[7]: https://spring.io/guides
[8]: https://github.com/spring-guides/gs-service-registration-and-discovery/archive/master.zip
[9]: https://spring.io/understanding/Git
[10]: https://spring.io/guides/gs/service-registration-and-discovery/#initial
[11]: http://gradle.org/
[12]: https://maven.apache.org/
[13]: https://spring.io/guides/gs/gradle
[14]: https://spring.io/guides/gs/maven
[15]: https://github.com/spring-guides/gs-service-registration-and-discovery/blob/master/initial/build.gradle
[16]: https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin
[17]: https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
[18]: https://maven.apache.org/
[19]: https://spring.io/guides/gs/maven
[20]: https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin
[21]: https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
[22]: https://spring.io/guides/gs/sts/
[23]: https://spring.io/guides/gs/intellij-idea
[24]: https://www.consul.io/
[25]: https://zookeeper.apache.org/
[26]:http://localhost:8080/service-instances/a-bootiful-client
[27]:https://spring.io/guides/gs/rest-service/
[28]:http://localhost:8080/service-instances/a-bootiful-client


