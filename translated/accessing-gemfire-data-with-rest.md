# 通过REST风格的方式读取GemFire数据

> 原文：[Accessing GemFire Data with REST](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
>
> 译者：冀天宇
>
> 校对：

这篇指南将指导你构建一个应用程序，这个应用程序使用[基于超媒体的(hypermedia-based)](https://spring.io/guides/gs/rest-hateoas/)、具有 [REST风格的前端](https://spring.io/understanding/REST)，来获取GemFire数据库中的数据。

## 你将构建什么

你将构建一个基于Spring Web的应用程序，这个应用程序使用Spring Data REST创建并读取存储在[Pivotal GemFire](https://pivotal.io/big-data/pivotal-gemfire)内存数据网格(In-Memory Data Grid) 中的`Person`对象。Spring Data REST采用了[Spring HATEOAS](https://projects.spring.io/spring-hateoas)和[Spring Data GemFire](https://projects.spring.io/spring-data-gemfire)的特性，并将它们自动组合在一起。

> Spring Data REST还支持[Spring Data JPA](https://spring.io/guides/gs/accessing-data-rest)，[Spring Data MongoDB](https://spring.io/guides/gs/accessing-mongodb-data-rest)和[Spring Data Neo4j](https://spring.io/guides/gs/accessing-neo4j-data-rest)作为后端数据存储技术，但这些没有包含在本指南中。

> 如果想要获得有关Pivotal GemFire的概念以及从GemFire访问数据的更多内容，请阅读指南“[使用GemFire访问数据](https://spring.io/guides/gs/accessing-data-gemfire/)”。

## 你需要做哪些工作

- 大概15分钟
- 你钟爱的文本编辑器或者IDE
- JDK1.8或者更高的版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或者[Maven 3.0+](https://maven.apache.org/download.cgi)
- 也可以直接将本节代码导入到你的IDE中
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成此指南

就像大部分的Spring Getting Started guides一样， 你可以从零开始，亲自完成每一个步骤，或者跳过你已经熟悉的基本步骤。无论如何，结束此指南的时候你都会实现可以正常工作的代码。

要从零开始，请跳转到[Build with Gradle](#gradle)。

要跳过基本的步骤，这样做：

- [下载](https://github.com/spring-guides/gs-rest-service/archive/master.zip)并解压缩此指南的代码，或者使用[Git](https://spring.io/understanding/Git) 命令:
  `git clone https://github.com/spring-guides/gs-accessing-gemfire-data-rest.git`
- 进入`gs-accessing-gemfire-data-rest/initial`目录
- 直接跳转到[创建领域对象(domain object)](https://spring.io/guides/gs/rest-service/#initial)

当完成以上步骤后，可以在`gs-accessing-gemfire-data-rest/complete`目录中再次查看代码的结果。

<h2 id="gradle">Gradle</h2>

第一步，建立基本的构建脚本(build script)。 当使用Spring构建apps的时候，几乎可以使用任何你喜欢的构建工具， 但是此指南只介绍了如何使用[Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/)来构建目标应用。如果这两个工具你都不熟悉，请参考[ Building Java Projects with Gradle ](https://spring.io/guides/gs/gradle)或者[ Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选定的工程目录中，创建如下的子目录结构。例如，在*nix系统中使用命令`mkdir -p src/main/java/hello`来创建该目录结构。

```shell
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件(build file)

下面是Gradle的[初始构建文件](https://github.com/spring-guides/gs-accessing-gemfire-data-rest/blob/master/initial/build.gradle)(initial build file)。
`build.gradle`

```groovy
buildscript {
    repositories {
        mavenCentral()
        maven { url "https://repo.spring.io/libs-release" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}

plugins {
    id "io.spring.dependency-management" version "1.0.3.RELEASE"
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8
targetCompatibility = 1.8

ext['jackson.version'] = '2.9.2';

jar {
    baseName = 'gs-accessing-gemfire-data-rest'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-release" }
}

dependencyManagement {
    imports {
        mavenBom 'org.springframework:spring-framework-bom:5.0.1.RELEASE'
        mavenBom 'org.springframework.data:spring-data-releasetrain:Kay-SR1'
    }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.data:spring-data-gemfire")
    compile("org.projectlombok:lombok:1.16.18")
    runtime("org.springframework.shell:spring-shell:1.2.0.RELEASE")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle plugin](http://https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多方便的功能：

- 将classpath中的所有jar文件集中起来，构建成单独的可运行的"über-jar", 这使得服务的运行和转移更加便捷。
- 搜索`public static void main()`方法，并对该可执行类进行标记。
- 提供了内置的依赖解析功能，该功能将依赖的版本与[ Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相匹配。用户可以按照需求覆盖依赖(dependency)的任何版本号，但是默认版本号是Spring Boot中已经选择好的版本号的集合。

## 使用Maven构建

第一步，建立基本的构建脚本(build script)。 当使用Spring构建apps的时候，几乎可以使用任何你喜欢的构建工具， 但是此处只介绍了如何使用[Maven](https://maven.apache.org/)来构建目标应用。如果这你不熟悉，请参考[ Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

#### 创建目录结构

在你选定的工程目录中，创建如下的子目录结构。例如，在*nix系统中使用命令`mkdir -p src/main/java/hello`来创建该目录结构。

```
└── src
    └── main
        └── java
            └── hello
```

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <groupId>org.springframework</groupId>
    <artifactId>gs-accessing-gemfire-data-rest</artifactId>
    <version>0.1.0</version>

    <properties>
        <java.version>1.8</java.version>
        <jackson.version>2.9.2</jackson.version>
        <spring.version>5.0.1.RELEASE</spring.version>
        <spring-data-releasetrain.version>Kay-SR1</spring-data-releasetrain.version>
        <spring-shell.version>1.2.0.RELEASE</spring-shell.version>
    </properties>

    <repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-gemfire</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.shell</groupId>
            <artifactId>spring-shell</artifactId>
            <version>${spring-shell.version}</version>
            <scope>runtime</scope>
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

[Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了许多方便的功能：

- 将classpath中的所有jar文件集中起来，构建成单独的可运行的"über-jar", 这使得服务的运行和转移更加便捷。
- 搜索`public static void main()`方法，并对该可执行类进行标记。
- 提供了内置的依赖解析功能，该功能将依赖的版本与[ Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相匹配。用户可以按照需求覆盖依赖(dependency)的任何版本号，但是默认版本号是Spring Boot中已经选择好的版本号的集合。

## 使用IDE构建

- 阅读如何将本指南直接导入到[Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)完成本指南。




## 创建领域对象

创建一个新的领域对象来表示一个人。

`src/main/java/hello/Person.java`

```java
package hello;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.PersistenceConstructor;
import org.springframework.data.gemfire.mapping.annotation.Region;

import lombok.Data;

@Data
@Region("People")
public class Person {

	private static AtomicLong COUNTER = new AtomicLong(0L);

	@Id
	private Long id;

	private String firstName;
	private String lastName;

	@PersistenceConstructor
	public Person() {
		this.id = COUNTER.incrementAndGet();
	}
}
```

`Person`对象拥有姓氏和名字。 Pivotal GemFire中的领域对象需要有id属性，所以随着`Person`对象的创建, `AtomicLong`类型的属性值也随之增加。

## 创建 `Person` Repository

接下来，你需要创建一个简单的存储库(Repository)来持久化或者访问存储在Pivotal GemFire中的Person对象。

`src/main/java/hello/PersonRepository.java`

```java
package hello;

import java.util.List;

import org.springframework.data.repository.CrudRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel = "people", path = "people")
public interface PersonRepository extends CrudRepository<Person, Long> {

	List<Person> findByLastName(@Param("name") String name);

}
```

这个Repository是一个接口，它允许你执行涉及Person对象的各种数据访问操作（例如，基本的CRUD和简单的查询）。 它通过继承`CrudRepository` 来实现这些方法。

程序运行的时候，Spring Data GemFire会自动创建这个接口的实现。 然后，Spring Data REST将使用[`@RepositoryRestResource`](https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/annotation/RepositoryRestResource.html)注解来引导Spring MVC在`/people`路径上创建REST风格的访问点(endpoint)。

>使用Repository读取数据的时候，`@RepositoryRestResource` 注解并不是必需的。 它仅用于更改访问数据的详细信息，例如使用`/people` 路径访问数据，而不是使用默认路径`/persons`。

在这里，您还定义了一个自定义查询，这个查询基于`lastName`字段， 查找`Person`对象列表。 在本指南中你将看到如何调用这个自定义查询。

## 使应用程序可以运行

尽管可以将本服务打包为传统的WAR文件，然后部署在外部的应用程序服务器上，下面展示一种更简便的方法，即创建独立的应用程序。这种方法将本服务打包为单独的、可直接运行的JAR文件，这个jar文件由传统的`main()`函数方法驱动。使用这种打包方法，可以使用Spring提供的嵌入式Tomcat servlet容器作为运行时(HTTP runtime)， 而不是直接部署外部serlvet 容器中。

`src/main/java/hello/Application.java`

```java
package hello;

import org.apache.geode.cache.GemFireCache;
import org.apache.geode.cache.client.ClientRegionShortcut;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.gemfire.client.ClientRegionFactoryBean;
import org.springframework.data.gemfire.config.annotation.ClientCacheApplication;
import org.springframework.data.gemfire.repository.config.EnableGemfireRepositories;

@SpringBootApplication
@ClientCacheApplication(name = "DataGemFireRestApplication", logLevel = "error")
@EnableGemfireRepositories
@SuppressWarnings("unused")
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Bean("People")
	public ClientRegionFactoryBean<Object, Object> peopleRegion(GemFireCache gemfireCache) {

		ClientRegionFactoryBean<Object, Object> peopleRegion = new ClientRegionFactoryBean<>();

		peopleRegion.setCache(gemfireCache);
		peopleRegion.setClose(false);
		peopleRegion.setShortcut(ClientRegionShortcut.LOCAL);

		return peopleRegion;
	}
}
```

`@SpringBootApplication`作为一个方便使用的注解，提供了如下的功能：

- `@Configuration`表明使用该注解的类是应用程序上下文(Applicaiton Context)中Bean定义的来源。
- `@EnableAutoConfiguration`注解根据classpath的配置、其他bean的定义或者不同的属性设置(property settings)等条件，使Spring Boot自动加入所需的bean。
- 对于Spring MVC应用，通常需要加入`@EnableWebMvc`注解，但是当**spring-webmvc** 存在于classpath中时，Spring Boot自动加入该注解。该注解将当前应用标记为web应用，并激活web应用的关键行为，例如开启`DispatcherServlet`。
- `@ComponentScan`注解使Spring在`hello`包(package)中搜索其他的组件、配置(configurations)和服务(service),在本例中，spring会搜索到控制器(controllers)。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法来加载应用。你有没有注意到本例子中一行XML代码都没有吗？也没有web.xml文件。此web应用100%使纯java代码，因此不需花精力处理任何像基础设施或者下水管道一般的配置工作。

`@EnableGemfireRepositories` 注解激活了Spring Data GemFire Repository。 Spring Data GemFire将创建`PersonRepository`接口的具体实现，并配置该接口与Pivotal GemFire的嵌入式实例进行会话。

### 构建可执行的JAR文件

可以从Gradle或者Maven的命令行运行此程序，也可以构建一个单独的可执行的JAR文件，此文件包含了应用程序所有必需的依赖、类以及资源。由于应用程序存在不同的开发周期，也会部署于不同的环境，这种方法使应用程序的转移、版本管理、以及发布都变得更加简单。

如果使用Gradle，可以使用`./gradlew bootRun`运行程序。或者使用`./gradlew build`构建JAR文件，然后运行此JAR文件：

`java -jar build/libs/gs-accessing-gemfire-data-rest-0.1.0.jar`

如果使用Maven，可以使用`./mvnw spring-boot:run`运行程序。或者使用`./mvnw clean package`构建JAR文件，然后运行此JAR文件：

`java -jar target/gs-accessing-gemfire-data-rest-0.1.0.jar`

> 上述的过程将会创建可执行的JAR文件。也可以参考[如何构建WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

## 测试此服务

现在应用程序正在运行，你可以测试它。 你可以使用任何你喜欢的REST客户端。 以下示例使用* nix工具`curl` 。

首先，你想看到如下顶层服务。

```shell
$ curl http://localhost:8080
{
  "_links" : {
    "people" : {
      "href" : "http://localhost:8080/people"
    }
  }
}
```

在这里你可以看到服务器提供的内容。 `People` 数据的路径位于[http:// localhost:8080/people](http://localhost:8080/people) 。 Spring Data GemFire不像其他Spring Data REST指南那样支持页码导航，所以没有额外的导航链接。

> Spring Data REST使用[HAL格式](http://stateless.co/hal_specification.html)来输出JSON。 它很灵活，并提供了一个便捷的方式来提供所服务的数据的临近数据的链接。

```shell
$ curl http://localhost:8080/people
{
  "_links" : {
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  }
}
```

是时候创建新的`Person` 对象了！

```shell
$ curl -i -X POST -H "Content-Type:application/json" -d '{  "firstName" : "Frodo",  "lastName" : "Baggins" }' http://localhost:8080/people
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/people/1
Content-Length: 0
Date: Wed, 05 Mar 2014 20:16:11 GMT
```

- `-i`确保你可以看到包含标题的响应消息，新创建的`Person` 对象的URI显示出来了
- `-X POST` 表示发出一个HTTP POST请求来创建一个新实体
- `-H "Content-Type:application/json"` 设置内容类型，以便应用程序知道有效内容包含JSON对象
- `-d '{ "firstName" : "Frodo", "lastName" : "Baggins" }' `是正在被发送的数据

> 注意前面的HTTP POST请求是如何包含Location字段的。 Location字段包含了新创建的资源的URI。Spring Data REST也提供了`RepositoryRestConfiguration.setReturnBodyOnCreate（...）`和`setReturnBodyOnCreate（...）` 这两个方法，可以使用它们来配置框架，用来立即返回刚刚创建的资源的表现形式。

从这里你可以查询所有人：

```shell
$ curl http://localhost:8080/people
{
  "_links" : {
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/1"
        }
      }
    } ]
  }
}
```

People集合资源包含一个含有Frodo的列表。 注意它是如何包含自我链接的。 Spring Data REST还使用Evo Inflector来将分组实体的名称变为复数。

你可以直接查询个人记录：

```shell
$ curl http://localhost:8080/people/1
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/1"
    }
  }
}
```

> 这似乎是纯粹基于网络的，但在幕后，`curl`正在与嵌入式Pivotal GemFire数据库进行会话。

在本指南中，只有一个领域对象。 在领域对象相互关联的更复杂的系统中，Spring Data REST将提供额外的链接来简化相互关联的记录之间的导航。

找到所有的自定义查询：

```shell
$ curl http://localhost:8080/people/search
{
  "_links" : {
    "findByLastName" : {
      "href" : "http://localhost:8080/people/search/findByLastName{?name}",
      "templated" : true
    }
  }
}
```

你可以看到查询的URL，包括HTTP请求的参数名称。 你是否注意到了，这与在接口中使用的`@Param（“name”）`注解相匹配。

要使用`findByLastName`查询，请执行以下操作：

```shell
$ curl http://localhost:8080/people/search/findByLastName?name=Baggins
{
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/1"
        }
      }
    } ]
  }
}
```

因为在代码中定义了要返回`List<Person>` 类型，这个方法就会返回所有的结果。如果代码中定义返回`Person` 类型， 这个方法就会返回任意一个`Person` 对象。由于这样返回的结果不可预测，对于返回多个条目的查询，你可能不希望这样做。

也可以发出`PUT`，`PATCH`和`DELETE` REST调用来替换，更新或删除现有记录。

```shell
$ curl -X PUT -H "Content-Type:application/json" -d '{ "firstName": "Bilbo", "lastName": "Baggins" }' http://localhost:8080/people/1
$ curl http://localhost:8080/people/1
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/1"
    }
  }
}
```

```shell
$ curl -X PATCH -H "Content-Type:application/json" -d '{ "firstName": "Bilbo Jr." }' http://localhost:8080/people/1
$ curl http://localhost:8080/people/1
{
  "firstName" : "Bilbo Jr.",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/1"
    }
  }
}
```

> `PUT`方法会替换整个记录，未提供值的字段将被替换为null。 `PATCH` 方法可以用来更新一系列记录的一个子集。

可以删除记录：

```shell
$ curl -X DELETE http://localhost:8080/people/1
$ curl http://localhost:8080/people
{
  "_links" : {
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  }
}
```

这个[超媒体驱动的接口](https://spring.io/understanding/HATEOAS)的一个非常方便的特点是可以使用`curl`（或者您正在使用的任何REST客户端）来发现所有REST风格的端点(endpoints)。 没有必要与客户端交换正式的合同化的或接口化的文档。

## 总结

恭喜！ 您刚刚开发了一个基于超媒体的REST风格的前端和一个基于GemFire的后端的应用程序。

## 也可以看看

下面的指南可能也会有帮助:

- [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)
- [Accessing MongoDB Data with REST](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Accessing Neo4j Data with REST](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
- [Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/)
- [Consuming a RESTful Web Service with AngularJS](https://spring.io/guides/gs/consuming-rest-angularjs/)
- [Consuming a RESTful Web Service with jQuery](https://spring.io/guides/gs/consuming-rest-jquery/)
- [Consuming a RESTful Web Service with rest.js](https://spring.io/guides/gs/consuming-rest-restjs/)
- [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
- [Building REST services with Spring](https://spring.io/guides/tutorials/bookmarks/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
- [Creating API Documentation with Restdocs](https://spring.io/guides/gs/testing-restdocs/)
- [Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)
- [Building a Hypermedia-Driven RESTful Web Service](https://spring.io/guides/gs/rest-hateoas/)

想写一个新的指南或为现有的指南提意见？ 请查看我们的[贡献准则](https://github.com/spring-guides/getting-started-guides/wiki)。



> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



