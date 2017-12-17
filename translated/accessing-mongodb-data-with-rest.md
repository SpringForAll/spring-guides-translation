# 使用 REST 访问 MongoDB 数据

> 原文：[Accessing MongoDB Data with REST(https://spring.io/guides/gs/accessing-mongodb-data-rest/)
>
> 译者：[麻酱](https://github.com/codedrinker)
>
> 校对：


本指南引导您完成创建应用程序的过程，该应用程序通过基于[超媒体](https://spring.io/guides/gs/rest-hateoas)的 [RESTful](https://spring.io/understanding/REST) 访问基于文档的数据结构。


## 你将创建什么  

您将构建一个 `Spring` 应用程序，让您使用 `Spring Data REST` 创建和检索存储在 `MongoDB NoSQL` 数据库中的 `Person` 对象。 `Spring Data REST` 集成了 [Spring HATEOAS](https://projects.spring.io/spring-hateoas) 和 [Spring Data MongoDB](https://projects.spring.io/spring-data-mongodb)。  

> `Spring Data REST` 同样支持 [Spring Data JPA](https://spring.io/guides/gs/accessing-data-rest) , [Spring Data Gemfire](https://spring.io/guides/gs/accessing-gemfire-data-rest) 和 [Spring Data Neo4j](https://spring.io/guides/gs/accessing-neo4j-data-rest)， 但是不在本文的讨论范围之内。 

## 开始之前你需要准备

大约15分钟时间

一个喜欢的文本编辑器或者IDE

[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

[Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

你也可以直接导入代码到IDE:

[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)


## 如何完成指南？  

像大多数 `Spring` [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#使用Gradle构建)。

**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/gs-accessing-mongodb-data-rest/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/gs-accessing-mongodb-data-rest.git`
* 进入 `gs-accessing-mongodb-data-rest/initial`目录
* 跳过前面的部分[安装并启动MongoDB](#安装并启动MongoDB)

**当你完成之后**，你可以在`gs-accessing-mongodb-data-rest/complete`根据代码检查下结果。

## 使用Gradle构建

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建， 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

在你的项目根目录，创建如下的子目录结构; 例如，如果你使用的是\*nix系统，你可以使用`mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份[初始化Gradle构建文件](https://github.com/spring-guides/gs-accessing-mongodb-data-rest/blob/master/initial/build.gradle)

`build.gradle`

```
buildscript {
    repositories {
        mavenCentral()
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
    baseName = 'gs-accessing-mongodb-data-rest'
    version = '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.boot:spring-boot-starter-data-mongodb")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷。
* 搜索 `public static void main()` 方法并且将它标记为可执行类。
* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建

首先，你需要设置一个基本的构建脚本。当使用 Spring 构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

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
    <artifactId>gs-accessing-mongodb-data-rest</artifactId>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
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

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布你的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。  

## 使用你的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。

## 安装并启动MongoDB

在您的项目设置完成后，您可以安装并启动 `MongoDB` 数据库。

如果你使用的是 `Mac`，你可以使用 `homebrew` 来安装，命令如下：

```sh
$ brew install mongodb
```

如果你使用 MacPorts,像这样:
```sh
$ port install mongodb
```
对于其它包管理系统,比如,Redhat, Ubuntu, Debian, CentOS, Windows系统,请参阅[ 说明 ](http://docs.mongodb.org/manual/installation/)  


在你安装 `MongoDB` 后，你需要通过后台进程启动 `Mongo`。
```sh
$ mongod
```
```
所有信息都输出到这个文件: /usr/local/var/log/mongodb/mongo.log

```
同时你可以使用其他的终端窗口运行 `mongo` 命令启动一个 `Mongo` 的客户端。

## 定义实体对象
创建一个实体对象 `Person` 用于存储 `person`。
```
src/main/java/hello/Person.java
```

```java
package hello;

import org.springframework.data.annotation.Id;

public class Person {

    @Id private String id;

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

`Person` 对象具有三个属性`id`，`firstName` 和 `lastName`，但是 `id`会自动生成，所以这里我们可以不考虑它。  

## 创建 Person Repository
接下来我们需要创建一个简单的 `repository`  
```
src/main/java/hello/PersonRepository.java
```
```java
package hello;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel = "people", path = "people")
public interface PersonRepository extends MongoRepository<Person, String> {

    List<Person> findByLastName(@Param("name") String name);

}
```
这个 `repository` 是一个接口，使得您可以对Person对象进行各种操作。它是通过继承 `MongoRepository`来获得这些操作，而 `MongoRepository` 又实现了 `Spring Data Commons `中定义的 `PagingAndSortingRepository` 接口。  
在运行时，`Spring Data REST` 会自动创建这个接口的实现。 然后，它将使用 `@RepositoryRestResource` 注解让 `Spring MVC` 生成 `/people` 的 `Restful` 映射。  

> `@RepositoryRestResource` 对于 `repository` 不是必须的，它只是为了修改映射，因为默认映射为 `/persons`。  

到这里，您已经实现了通过 `lastName` 检索 `Person` 对象列表的功能，下面我们会进一步对其进行说明。  

> `Spring Boot` 默认尝试连接到本地的 `MongoDB` 实例。如果你想指向其他的 `MongoDB` 实例，可以详细阅读[文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-mongodb)。

## 构建一个可执行的应用程序

尽管可以将此服务作为传统 `WAR` 文件打包以部署到外部应用程序服务器，但下面演示的更简单的方法创建了独立的应用程序。 你把所有东西都封装在一个单独的，可执行的 `JAR` 文件中，由`Java main`方法执行。 当然 `Spring` 同样支持内置的 `Tomcat`容器，这样你也不需要部署到其他的 `Tomcat`。
```
src/main/java/hello/Application.java
```
```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication`作为一个方便使用的注解，提供了如下的功能：

- `@Configuration` 表明使用该注解的类是应用程序上下文(Applicaiton Context)中Bean定义的来源。
- `@EnableAutoConfiguration` 注解根据 `classpath` 的配置、其他bean的定义或者不同的属性设置(property settings)等条件，使 `Spring Boot` 自动加入所需的bean。
- 对于 `Spring MVC` 应用，通常需要加入`@EnableWebMvc`注解，但是当**spring-webmvc** 存在于 `classpath`中时，`Spring Boot` 自动加入该注解。该注解将当前应用标记为web应用，并激活 `web` 应用的关键行为，例如开启 `DispatcherServlet`。
- `@ComponentScan` 注解使 `Spring` 在 `hello` 包(package)中搜索其他的组件、配置(configurations)和服务(service),在本例中，spring会搜索到控制器(controllers)。

`main()` 方法使用 `Spring Boot` 的 `SpringApplication.run()` 方法来启动应用。你有没有注意到本例子中一行 `XML` 代码都没有吗？也没有 `web.xml` 文件。此web应用100%使纯java代码，因此不需花精力处理任何像基础设施或者下水管道一般的配置工作。

## 构建可执行的JAR

您可以在命令行使用 `Gradle` 或 `Maven` 运行应用程序。或者您可以构建一个可执行的 `JAR` 文件,其中包含所有必需的依赖关系,类,和资源,并运行。这使得在整个开发生命周期中非常容易，跨不同的环境，版本等传输和部署服务成为一个应用程序,等等。

如果您使用的是 `Gradle`，则可以使用该应用程序 `./gradlew bootRun`,或者您可以使用 `JAR` 文件构建 `./gradlew build`,然后可以运行 `JAR` 文件：

```
java -jar build/libs/gs-accessing-mongodb-data-rest-0.1.0.jar
```
如果您使用 `Maven`，则可以使用该命令 `./mvnw spring-boot:run`,或者您可以使用 `JAR` 文件来构建 `./mvnw clean package`,然后可以运行 `JAR` 文件：

```
java -jar target/gs-accessing-mongodb-data-rest-0.1.0.jar
```

> 上面的过程将创建一个可运行的 `JAR`。您也可以选择 [构建一个 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

输出日志，该服务应该在几秒钟内启动并运行。

## 测试应用  

现在应用程序正在运行，您可以测试它。您可以使用您喜欢的 `Rest` 客户端访问。 以下示例使用 `*nix` 的 `CURL` 命令。
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
这是你便可以见到服务所提供的接口，`http://localhost:8080/people` 定位到 `people` 资源，同时还提供了 `page`，`size` 和 `sort`等选项。

> `Spring Data REST` 使用 `HAL` 格式来输出 `JSON`，他可以灵活的显示当前链接还能提供的支持。

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
这时数据库还没有数据，所以也没有返回结果，所以我们现在应该创建一个 `Person`。  

> 如果您多次运行本指南，则可能会遗留数据。 如果你需要一个新的开始，请参考 [MongoDB shell](http://docs.mongodb.org/manual/reference/mongo-shell/) 快速参考找到命令来查找和删除数据库。

```
$ curl -i -X POST -H "Content-Type:application/json" -d "{  \"firstName\" : \"Frodo\",  \"lastName\" : \"Baggins\" }" http://localhost:8080/people
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/people/53149b8e3004990b1af9f229
Content-Length: 0
Date: Mon, 03 Mar 2014 15:08:46 GMT
```
  
- -i 确保您可以看到包含标题的响应消息。 显示新创建的 `Person` 的 `URI`  

- -X POST 表示发送一个 `POST` 请求  

- -H “Content-Type：application / json” 设置内容类型，当前请求为 `JSON` 对象  

- -d '{“firstName”：“Frodo”，“lastName”：“Baggins”}“ 是发送的数据   

> 请注意，以前的POST操作是如何包含位置标题的。 这包含新创建的资源的 URI。 Spring Data REST 在 RepositoryRestConfiguration.setReturnBodyOnCreate（...）和setReturnBodyOnUpdate（...）上也有两个方法，您可以使用它们来配置框架以立即返回刚刚创建/更新的资源的表示形式。  

现在你可以查询 `people`了
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
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
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
`persons` 对象列表包含了 `Frodo`，当然你可以发现他哎呦一个 `self` 的 `href`。`Spring Data REST` 使用 [Evo Inflector](http://www.atteo.org/2011/12/12/Evo-Inflector.html) 生成了对象的访问链接。   
这样你就可以直接查询个人记录：
```
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
```
> 这可能看起来纯粹的 `web`服务，但是其实他已经和 `MongoDB` 进行交流了。  
在本指南中，只有一个实体对象。 在实体对象相互关联的更复杂的系统中，`Spring Data REST` 将提供额外的链接来定位相关记录。  
搜索所有 `people`
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
你可以看到 `URL` 里面包含了 `name` 参数，这是用来匹配 接口中定义的 `@Param("name")` 注解的。  
如果您想仅使用 `findByLastName` 进行搜索，可以运行如下命令
```
$ curl http://localhost:8080/people/search/findByLastName?name=Baggins
{
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
        }
      }
    } ]
  }
}
```
因为您将其定义为在代码中返回 `List <Person>`，它将返回所有结果。如果你定义了它只返回 `Perso`n，它将会选择一个 `Person` 对象返回。因为这是不可预期的，所以您可能不希望这样做可以返回多个条目的查询。  

您也可以发出`PUT`，`PATCH`和`DELETE` `REST`请求来实现更新，部分更新和删除现有记录。
```
$ curl -X PUT -H "Content-Type:application/json" -d "{ \"firstName\": \"Bilbo\", \"lastName\": \"Baggins\" }" http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
```
```
$ curl -X PATCH -H "Content-Type:application/json" -d "{ \"firstName\": \"Bilbo Jr.\" }" http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Bilbo Jr.",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
```
> PUT替换整个记录。未提供的字段将被替换为null。 PATCH可以用来更新文档的一个子集。  

你可以删除文档
```
$ curl -X DELETE http://localhost:8080/people/53149b8e3004990b1af9f229
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
这个 [hypermedia-driven interface](https://spring.io/understanding/HATEOAS) 可以让你非常方便的发现 `RESTful` 接口的地址，这样就不需要写一个联调文档或者是接口文档了。

## 总结

恭喜你!实现了一个通过基于[超媒体](https://spring.io/guides/gs/rest-hateoas)的 [RESTful](https://spring.io/understanding/REST)应用程序。

## 了解更多
下面的指南也非常有帮助：
- [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)  
- [Accessing Gemfire Data with REST](https://spring.io/guides/gs/accessing-gemfire-data-rest/)  
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


想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
