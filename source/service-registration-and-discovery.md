# 服务注册和发现

> 原文：[Service Registration and Discovery](https://spring.io/guides/gs/service-registration-and-discovery/)
>
> 译者：[lovedboy2012](https://github.com/lovedboy2012)
>
> 校对：

本指南将引导您了解Netflix Eureka服务注册中心的创建和消费过程

## 你将要构建什么
您将创建一个Netflix Eureka服务注册中心，然后构建一个能实现去注册中心自注册的客户端，并使用Eureka服务端来解析自己主机地址。服务注册中心是有很用的，因为它不仅使得客户端负载均衡，而且服务提供者与消费者分离，而不需要DNS地址解析。

## 你需要什么
 - 大约需要15分钟
 - 一个您喜爱的文本编辑器或者IDE（集成开发工具）
 - [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或更高版本
 - [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
 - 您也可以直接导入代码到IDE中：
    - [Spring Tool Suite (STS)][3]
    - [IntelliJ IDEA][4]
    
## 如何完成这份指南
与大多数“入门指南”指南一样，您可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。无论哪种方式，你都会得到工作代码。

如果从零开始，请继续使用Gradle构建。

如果打算跳过基本步骤，按照如下操作：

  - [下载][6]并解压这份指南的源码包，或者使用[Git][7]克隆：
     `git clone https://github.com/spring-guides/gs-scheduling-tasks.git`
  - 进入目录 `gs-service-registration-and-discovery/initial`
  - 跳转到 `开始Eureka服务注册`。
  
完成这些后，您可以依据gs-service-registration-and-discovery/complete中的代码检查你的结果。

## 使用 Gradle 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Gradle][8] 或者 [Maven][9] 构建的代码。如果你对两者都不是很熟悉，你可以先去看下[如何使用 Gradle 构建 Java 项目][10]或者[如何使用 Maven 构建 Java 项目][11]。

### 创建 Gradle 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

### 创建 Gradle 构建文件

下面是一份[初始化Gradle构建文件][12]

eureka-service/build.gradle

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
eureka-client/build.gradle

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

[Spring Boot gradle 插件][13] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][14].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用 Maven 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Maven][15] 构建的代码。如果你对 Maven 还不是很熟悉，你可以先去看下[如何使用 Maven 构建 Java 项目][16].

### 创建 Maven 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

eureka-service/pom.xml

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
eureka-client/pom.xml

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

[Spring Boot Maven 插件][17] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][18].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用你的 IDE 进行构建

## 开启Eureka服务注册
首先需要一个Eureka注册中心。你可以使用Spring Clound的注解@EnabledEurekaServer开启其他应用程序可以通信的服务端。以下是一个通过注解方式开启了服务注册的常规Spring Boot 应用程序。

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

当注册服务启动后，若不记录服务注册连接信息的堆栈日志是会存在问题的，况且在生产环境中，还将会有多个服务实例存在。但是这里为了简单起见，不作相关日志的处理。

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
##将服务注册到Eureka Server
现在我们已经建立了一个Eureka server服务端，现在让我们使用Spring Cloud 的注解`DiscoveryClient`来发现所有注册服务的主机和端口，从而创建一个实现自注册的客户端。在Spring Cloud中，注解`EnableDiscoveryClient`不仅使Eureka作为服务注册组件实现了`DiscoveryClient`，同时为 Hashicorp Consul 和 Apache Zookeeper也都提供了支持
   
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

无论你选择什么组件实现，你都可以通过指定spring.application.name属性任何名称来注册eureka客户端，并且可以在server端看到该名称。这个属性在Spring Cloud 中使用很多，通常是配置服务时最先配的。这个属性作为服务的系统配置，习惯上放在`eureka-client/src/main/resources/bootstrap.properties`这个属性文件，此配置文件在`src/main/resources/application.properties`之前加载。

`eureka-client/src/main/resources/bootstrap.properties`

```
spring.application.name=a-bootiful-client
```
eureka-client定义了一个名为ServiceInstanceRestController的Spring MVC REST服务站点，
通过访问http：//localhost：8080/service-instances/a-bootiful-client，将返回所有
`ServiceInstance`服务实例的注册列表。 请参阅[Building a RESTful Web Service]，
了解更多关于使用Spring MVC 和Spring Boot构建REST服务。

##测试你的应用程序

首先启动Eureka server服务，然后一旦eureka-client加载并启动成功，即可完成客户端到服务端的注册测试。eureka-client需要大约一分钟注册中心去注册自己，并刷新注册中心的注册服务列表，所有的这些参数都是可配置的。在浏览器中访问eureka-client，通过http://localhost:8080/service-instances/a-bootiful-client，应该可以看到该响应的`eureka-client`服务在注册中心的`ServiceInstance`。

##总结

恭喜！ 您刚刚使用Spring来启动了Netflix Eureka服务注册中心，并在客户端应用程序中实现了服务的注册。

想写一个新的指南或贡献一个现有的？ 查看我们的[贡献指南]。

```
所有指南的代码都将发布ASLv2许可证，以及署名，NoDerivatives创作共用许可证的写作。

```

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



