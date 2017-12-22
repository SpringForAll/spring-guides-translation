# 使用REST访问Neo4j数据

> 原文：[Accessing Neo4j Data with REST](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
>
> 译者：[shaoshao721](https://github.com/shaoshao721)
>
> 校对：

本指南将引导您完成创建应用程序的过程，该应用程序通过基于[超媒体](https://spring.io/guides/gs/rest-hateoas/)的[RESTful前端](https://spring.io/understanding/REST)来访问基于图的数据。



## 您将创建什么

您将构建一个Spring应用程序，让您使用Spring Data REST来创建和检索存储在[Neo4j](https://neo4j.com/) [NoSQL](https://spring.io/understanding/NoSQL)数据库中的`Person`对象。Spring Data REST采取了[Spring HATEOAS](https://projects.spring.io/spring-hateoas/)和[Spring Data Neo4j](https://projects.spring.io/spring-data-neo4j/)的特性，并将他们自动组合在一起。

> Spring Data REST还支持[Spring Data JPA](https://spring.io/guides/gs/accessing-neo4j-data-rest/)，[Spring Data Gemfire](https://spring.io/guides/gs/accessing-gemfire-data-rest/)和[Spring Data MongoDB](https://spring.io/guides/gs/accessing-mongodb-data-rest/)作为后端数据存储，但这些不是本指南的一部分。

## 开始之前您需要准备

- 大约15分钟时间
- 一个喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 您也可以直接导入代码到IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成指南？

像大多数 Spring [入门指南](https://spring.io/guides)一样, 您可以从头开始，完成每一步, 或者您也可以绕过您熟悉的基本步骤再开始。 不管通过哪种方式，您最后都会得到一份可执行的代码。

**如果从基础开始**，您可以往下查看[怎样使用 Gradle 构建项目](https://spring.io/guides/gs/accessing-neo4j-data-rest/#scratch)。

**如果已经熟悉跳过一些基本步骤**，您可以：

- [下载](https://codeload.github.com/spring-guides/gs-accessing-neo4j-data-rest/zip/master)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

  `git clone <https://github.com/spring-guides/gs-accessing-neo4j-data-rest.git`

- 进入 `gs-accessing-neo4j-data-rest/initial`目录

- 跳过前面的部分来[创建一个简单的实体](https://spring.io/guides/gs/accessing-neo4j-data-rest/#initial)

**当您完成之后**，您可以在`gs-accessing-neo4j-data-rest/complete`根据代码检查下结果。

##使用Gradle构建

首先您需要编写基础构建脚本。在构建 Spring 应用的时候，您可以使用任何您喜欢的系统来构建， 这里提供一份您可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果您两者都不是很熟悉, 您可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle/)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven/)。

###创建以下目录结构

在您的项目根目录，创建如下的子目录结构; 例如，如果您使用的是\*nix系统，您可以使用

`mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份[初始化Gradle构建文件](https://github.com/spring-guides/gs-accessing-neo4j-data-rest/blob/master/initial/build.gradle)

`build.gradle`

```
buildscript {
    repositories {
        mavenCentral()
        maven { url "https://repo.spring.io/libs-release" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-accessing-neo4j-data-rest'
    version = '0.1.0'
}

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-release" }
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.data:spring-data-neo4j")
    compile("org.hibernate:hibernate-validator")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件]https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

- 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布您的服务变得更加便捷。
- 搜索 `public static void main()` 方法并且将它标记为可执行类。
- 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。您可以根据您的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建

首先，您需要设置一个基本的构建脚本。当使用 Spring 构建应用程序时，您可以使用任何您喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

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
    <artifactId>gs-accessing-neo4j-data-rest</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-rest</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-neo4j</artifactId>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
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

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布您的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)的版本号进行匹配。您可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。

## 使用您的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/) 来构建。

## 创建一个Neo4j服务器

在您开始创建该项目之前，您需要部署一个Neo4j服务。

您可以免费安装一个Neo4j的开源服务器。

对于Mac用户，只需要输入:

```
$ brew install neo4j
```

对于其他的配置选项，可访问<https://neo4j.com/download/community-edition/>

在您安装后，可以通过默认的配置启动：

```
$ neo4j start
```

您将会看到如下的信息：

```
Starting Neo4j.
Started neo4j (pid 96416). By default, it is available at http://localhost:7474/
There may be a short delay until the server is ready.
See /usr/local/Cellar/neo4j/3.0.6/libexec/logs/neo4j.log for current status.
```

默认情况下，Neo4j具有一个为neo4j/neo4j的默认账户/密码。但是，它必须修改为新的账户密码。要做到这一点，请执行以下命令：

```
$ curl -v -u neo4j:neo4j -X POST localhost:7474/user/neo4j/password -H "Content-type:application/json" -d "{\"password\":\"secret\"}"
```

这将会将密码从**neo4j**修改为**secret**（请注意不要在生产环境下这样做！）在完成这些后，您可以正式开始这篇指南了。

## 创建一个领域对象

创建一个新的领域对象来表示一个人。

`src/main/java/hello/Person.java`

```
package hello;


import org.neo4j.ogm.annotation.GraphId;
import org.neo4j.ogm.annotation.NodeEntity;

@NodeEntity
public class Person {

	@GraphId private Long id;

	private String firstName;
	private String lastName;

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
}
```

这个`Person`拥有first name和last name。 它还包含一个被配置为自动生成的id对象，所以您不必处理它。

## Neo4j访问权限

Neo4j社区版需要凭据来访问数据。凭据可以用一些属性进行配置。

```properties
include complete/src/main/resources/application.properties[]
```

这囊括了默认的用户名`neo4j`和一个我们早先新设置的密码`secret`。

> 不要在您的源代码库中存储真实的凭证。相反，在运行时使用[SpringBoot属性覆盖](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config)来进行配置。

这个时候，让我们来把他们组装起来看看。

##创建一个Person存储库

接下来，您需要创建一个简单的存储库。

`src/main/java/hello/PersonRepository.java`

```
package hello;

import java.util.List;

import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel = "people", path = "people")
public interface PersonRepository extends PagingAndSortingRepository<Person, Long> {

    List<Person> findByLastName(@Param("name") String name);

}
```

这个存储库是一个接口，将允许您执行涉及`Person`对象的各种操作。 它通过扩展Spring Data Commons中定义的[PagingAndSortingRepositry](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/PagingAndSortingRepository.html)接口来获得这些操作。

在运行时，Spring Data REST会自动创建这个接口的实现。 然后，它将使用[@RepositoryRestResource](https://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/annotation/RepositoryRestResource.html) 注解来指导Spring MVC在`/people`处创建RESTful端点。

> 存储库不需要导出`@RepositoryRestResource`。 它仅用于更改导出详细信息，例如使用`/people`而不是`/persons`的默认值。

在这里，您还定义了一个自定义查询来检索基于lastName的`Person`对象列表。 您将看到如何在本指南中进一步调用它。

##使应用程序可执行

尽管可以将此服务作为传统[WAR](https://spring.io/understanding/WAR)文件打包以部署到外部应用程序服务器，但下面演示的更简单的方法创建了独立的应用程序。 您把所有东西都封装在一个单独的，可执行的JAR文件中，由一个好的旧的Java `main()`方法驱动。 在这过程中，您使用Spring的支持来将[Tomcat](https://spring.io/understanding/Tomcat) servlet容器作为HTTP运行时嵌入，而不是部署到外部实例。

`src/main/java/hello/Application.java`

```
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.data.neo4j.repository.config.EnableNeo4jRepositories;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@EnableTransactionManagement
@EnableNeo4jRepositories
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

`@SpringBootApplication`是一个方便的注释，它增加了以下所有内容：

- `@Configuration`将类标记为应用程序上下文的bean定义的来源。
- `@EnableAutoConfiguration`通知Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
- 通常情况下，您会为Spring MVC应用程序添加`@EnableWebMvc`，但Spring Boot会在类路径中看到**spring-webmvc**时自动添加它。 这将该应用程序标记为Web应用程序，并激活关键行为，如设置`DispatcherServlet`。
- `@ComponentScan`告诉Spring在`hello`包中查找其他组件，配置和服务，以便找到控制器。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。 您有没有注意到这里没有一行XML代码？ 也没有web.xml文件。 这个Web应用程序是100％纯Java代码编写的，您不必处理任何关于配置管道或基础设施的事情。

`@ EnableNeo4jRepositories` 注解激活Spring Data Neo4j。 Spring Data Neo4j将创建`PersonRepository`的具体实现，并将其配置为使用Cypher查询语言与嵌入的Neo4j数据库对话。

##创建可执行的JAR

您可以在命令行使用 `Gradle` 或 `Maven` 运行应用程序。或者您可以构建一个可执行的 `JAR` 文件,其中包含所有必需的依赖关系,类,和资源,并运行。这使得在整个开发生命周期中非常容易，跨不同的环境，版本等传输和部署服务成为一个应用程序,等等。

如果您使用的是 Gradle，则可以使用该应用程序 `./gradlew bootRun`,或者您可以使用 JAR 文件构建 `./gradlew build` ，然后可以运行 `JAR` 文件：

```shell
java -jar build/libs/gs-accessing-data-neo4j-0.1.0.jar
```

如果您使用 `Maven`，则可以使用该命令 `./mvnw spring-boot:run` ，或者您可以使用 `JAR` 文件来构建 `./mvnw clean package` ，然后可以运行 `JAR` 文件：

```shell
java -jar target/gs-accessing-data-neo4j-0.1.0.jar
```

> 上面的过程将创建一个可运行的 JAR。您也可以选择 [构建一个 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

日志会被显示。 该服务应该在几秒钟内启动并运行。

## 测试应用程序

现在应用程序正在运行，您可以测试它。 您可以使用任何您希望使用的REST客户端。 以下示例使用 *nix工具`curl`。

首先，你需要查看顶级服务。

```
$ curl http://localhost:8080
{
  "_links" : {
    "people" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    }
  }
}
```

在这里你可以第一眼看到这个服务器提供的东西。 **people** 的链接地址 <http://localhost:8080/people>。 它有一些选项，如`?page`，`?size`和`?sort`。

> Spring Data REST使用[HAL格式](http://stateless.co/hal_specification.html)来输出JSON。 它是灵活的，并且提供了一个便捷的方式来提供它所服务的数据附近的链接。

```
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
```

这里目前没有元素也没有页面。 现在去创建一个新的`Person`！

```
$ curl -i -X POST -H "Content-Type:application/json" -d '{  "firstName" : "Frodo",  "lastName" : "Baggins" }' http://localhost:8080/people
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/people/0
Content-Length: 0
Date: Wed, 26 Feb 2014 20:26:55 GMT
```

- `-i` 确保您可以看到包含标题的响应消息。 显示新创建的`Person`的URI
- `-X POST` 表示这是用于创建新条目的`POST`
- `-H"Content-Type：application / json"`设置内容类型，因此应用程序知道有效载荷包含一个JSON对象
- `-d'{"firstName"："Frodo"，"lastName"："Baggins"}'` 是发送的数据

> 请注意，以前的`POST`操作是如何包含`Location`标题的。 这包含新创建的资源的URI。 Spring Data REST在`RepositoryRestConfiguration.setReturnBodyOnCreate(...)`和`setReturnBodyOnCreate(...)`上也有两个方法，您可以使用它们来配置框架以立即返回刚刚创建的资源的表示形式。

在这里你可以查询所有人：

```
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "_embedded" : {
    "people" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/0"
        }
      }
    } ]
  },
  "page" : {
    "size" : 20,
    "totalElements" : 1,
    "totalPages" : 1,
    "number" : 0
  }
}
```

**people**对象包含与Frodo的列表。 注意它是如何包含**自我**链接的。 Spring Data REST还使用[Evo Inflector](http://www.atteo.org/2011/12/12/EvoInflector.html)来为分组复制实体的名称。

您可以直接查询个人记录：

```
$ curl http://localhost:8080/people/0
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/0"
    }
  }
}
```

在本指南中，只有一个领域对象。 在领域对象相互关联的更复杂的系统中，Spring Data REST将提供额外的链接来帮助导航到连接的记录。

找到所有的自定义查询：

```
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

您可以看到查询的URL，包括HTTP查询参数`name`。 你将注意到，这与嵌入在接口中的`@Param("name")`注解相匹配。

要使用`findByLastName`查询，请执行以下操作：

```
$ curl http://localhost:8080/people/search/findByLastName?name=Baggins
{
  "_embedded" : {
    "people" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/0"
        },
        "person" : {
          "href" : "http://localhost:8080/people/0"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/search/findByLastName?name=Baggins"
    }
  }
}
```

因为您将其定义为在代码中返回`List<Person>`，所以它将返回所有结果。 如果你定义了它只返回`Person`，它将会选择一个Person对象返回。 由于这可能是不可预知的，您可能不希望返回多个条目的查询。

您也可以发出`PUT`，`PATCH`和`DELETE` REST调用来替换，更新或删除现有记录。

```
$ curl -X PUT -H "Content-Type:application/json" -d '{ "firstName": "Bilbo", "lastName": "Baggins" }' http://localhost:8080/people/0
$ curl http://localhost:8080/people/0
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/0"
    }
  }
}
```

```
$ curl -X PATCH -H "Content-Type:application/json" -d '{ "firstName": "Bilbo Jr." }' http://localhost:8080/people/0
$ curl http://localhost:8080/people/0
{
  "firstName" : "Bilbo Jr.",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/0"
    }
  }
}
```

> `PUT`替换整个记录。 未提供的字段将被替换为`null`。 `PATCH`可以用来更新项目的一个子集。

您可以删除记录：

```
$ curl -X DELETE http://localhost:8080/people/0
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
```

这个[超媒体驱动接口](https://spring.io/understanding/HATEOAS)的一个非常方便的特点是 如何使用curl（或者您正在使用的任何REST客户端）发现所有RESTful端点。 它没有必要与您的客户交换正式的合同或接口文件。

##总结

恭喜！ 您刚刚开发了[基于超媒体的RESTful](https://spring.io/guides/gs/rest-hateoas/)前端和基于Neo4j的后端的应用程序。

## 了解更多

下面的指南也非常有帮助：

- [使用REST访问JPA数据](https://spring.io/guides/gs/accessing-data-rest/)
- [使用REST访问MongoDB数据](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
- [使用MySQL访问数据](https://spring.io/guides/gs/accessing-data-mysql/)
- [使用REST访问Gemfire数据](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
- [使用REST风格的Web服务](https://spring.io/guides/gs/consuming-rest/)
- [使用AngularJS来使用REST风格的Web服务](https://spring.io/guides/gs/consuming-rest-angularjs/)
- [使用jQuery使用REST风格的Web服务](https://spring.io/guides/gs/consuming-rest-jquery/)
- [使用rest.js来使用RESTful Web服务](https://spring.io/guides/gs/consuming-rest-restjs/)
- [保护Web应用程序](https://spring.io/guides/gs/securing-web/)
- [用Spring构建REST服务](https://spring.io/guides/tutorials/bookmarks/)
- [使用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)
- [使用Restdoc创建API文档](https://spring.io/guides/gs/testing-restdocs/)
- [启用RESTful Web服务的跨源请求](https://spring.io/guides/gs/rest-service-cors/)
- [构建超媒体驱动的RESTful Web服务](https://spring.io/guides/gs/rest-hateoas/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。