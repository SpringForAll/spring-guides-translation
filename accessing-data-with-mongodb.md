# 使用MongoDB访问数据

> 原文：[Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
>
> 译者：[cholf](https://github.com/cholf)
>
> 校对：


本指南将引导你使用[ Spring Data MongoDB ](https://projects.spring.io/spring-data-mongodb/)来构建一个应用程序,该应用程序将数据存储到[ MongoDB ](https://www.mongodb.com/)(一个基于文档的数据库)并从中获取数据。


## 你将创建什么
您将使用Spring Data MongoDB存储 `Customer` [ POJOS ](https://spring.io/understanding/POJO)到MongoDB数据库中。

## 开始之前你需要准备

大约15分钟时间

一个喜欢的文本编辑器或者IDE

[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

[Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

你也可以直接导入代码到IDE:

[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)


## 如何完成指南？
像大多数 Spring [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#scratch)。

**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/gs-accessing-data-mongodb/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/gs-accessing-data-mongodb.git`
* 进入 `gs-accessing-data-mongodb/initial`目录
* 跳过前面的部分[安装并启动MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/#initial)

**当你完成之后**，你可以在`gs-accessing-data-mongodb/complete`根据代码检查下结果。

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

下面是一份[初始化Gradle构建文件](https://github.com/spring-guides/gs-accessing-data-rest/blob/master/initial/build.gradle)

`build.gradle`

```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.7.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-accessing-data-rest'
    version = '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
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

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-accessing-data-rest</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
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
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
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

    <repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
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

在您的项目设置完成后，您可以安装并启动 MongoDB 数据库。

如果你使用的是 Mac，你可以使用 homebrew 来安装,像这样：

```sh
$ brew install mongodb
```

如果你使用 MacPorts,像这样:
```sh
$ port install mongodb
```
对于其它包管理系统,比如,Redhat, Ubuntu, Debian, CentOS, Windows系统,请参阅[ 说明 ](http://docs.mongodb.org/manual/installation/)


在你安装MongoDB后,在控制台窗口启动它,命令还启动了一个服务器进程。
```sh
$ mongod
```
这里有更详细的启动信息:
```
所有信息都输出到这个文件: /usr/local/var/log/mongodb/mongo.log

```

## 定义一个简单的实体
MongoDB是一个NoSQL文档存储。例子里,你将存储 `Customer` 对象。
`
src/main/java/hello/Customer.java
`

```java
package hello;

import org.springframework.data.annotation.Id;


public class Customer {

    @Id
    public String id;

    public String firstName;
    public String lastName;

    public Customer() {}

    public Customer(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%s, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

}
```

这里，您有一个具有三个属性`id`，`firstName` 和 `lastName` 的 `Customer` 类。该 ID 主要用于 MongoDB 内部使用。创建新实例时，您还可以使用单个构造函数来填充实体。



> 在本指南中，为了简洁起见，典型的 setter 和 getter 省略了

`id` 符合 MongoDB 标准的标准名称，因此不需要任何特殊的注释来为 Spring Data MongoDB 标记它。

另外的两个属性,`firstName`, 和 `lastName ``没有注解,假设它们将映射到与属性本身具有相同名称的字段。

方便的 `toString()`` 将会打印关于 customer 的详细信息。

> MongoDB存储数据集合, Spring Data MongoDB 将映射 `Customer`类到一个名为 customer 的集合。如果你要改变集合的名字,你可以在类上使用Spring Data MongoDB 的 [ @Document ](https://docs.spring.io/spring-data/data-mongodb/docs/current/api/org/springframework/data/mongodb/core/mapping/Document.html) 注解。

## 创建简单查询
Spring Data MongoDB专注于存储数据到 MongoDB。
它还继承了Spring Data Commons项目的功能，例如查询的功能。
本质上，您不必学习 MongoDB 数据库的查询语言;
您可以简单地编写一些方法完成查询，数据库的查询已经为您编写好。

要了解它是如何工作的，请创建一个查询 `Customer` 文档的存储库接口。

`src/main/java/hello/CustomerRepository.java`

```Java
package hello;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface CustomerRepository extends MongoRepository<Customer, String> {

    public Customer findByFirstName(String firstName);
    public List<Customer> findByLastName(String lastName);

}
```
`CustomerRepository` 继承 `MongoRepository`接口,开箱即用,接口有许多可使用操作的方法,包括标准的CRUD操作(create-read-update-delete)。

通过简单定义方法签名,你可以定义你需要的查询方法,在这种情况下，添加 `findByFirstName` ,通过 `firstName` 搜索 `Customer` 文档,找到匹配的一个结果。

你也可以通过 `findByLastName` 方法找到匹配姓氏的人员列表。

在典型的Java应用程序中，您编写一个实现`CustomerRepository`并自行编写查询的类。 Spring Data MongoDB如此有用的是,您不需要自己创建实现,当您运行应用程序的时候MongoDB会动态创建它。

让我们来看看它的样子吧！

## 创建一个应用程序类
在这里，您创建一个包含所有组件的 Application 类。
`
src/main/java/hello/Application.java
`
```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application implements CommandLineRunner {

	@Autowired
	private CustomerRepository repository;

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Override
	public void run(String... args) throws Exception {

		repository.deleteAll();

		// save a couple of customers
		repository.save(new Customer("Alice", "Smith"));
		repository.save(new Customer("Bob", "Smith"));

		// fetch all customers
		System.out.println("Customers found with findAll():");
		System.out.println("-------------------------------");
		for (Customer customer : repository.findAll()) {
			System.out.println(customer);
		}
		System.out.println();

		// fetch an individual customer
		System.out.println("Customer found with findByFirstName('Alice'):");
		System.out.println("--------------------------------");
		System.out.println(repository.findByFirstName("Alice"));

		System.out.println("Customers found with findByLastName('Smith'):");
		System.out.println("--------------------------------");
		for (Customer customer : repository.findByLastName("Smith")) {
			System.out.println(customer);
		}

	}

}
```


`@SpringBootApplication`作为一个方便使用的注解，提供了如下的功能：

- `@Configuration`表明使用该注解的类是应用程序上下文(Applicaiton Context)中Bean定义的来源。
- `@EnableAutoConfiguration`注解根据classpath的配置、其他bean的定义或者不同的属性设置(property settings)等条件，使Spring Boot自动加入所需的bean。
- 对于Spring MVC应用，通常需要加入`@EnableWebMvc`注解，但是当**spring-webmvc** 存在于classpath中时，Spring Boot自动加入该注解。该注解将当前应用标记为web应用，并激活web应用的关键行为，例如开启`DispatcherServlet`。
- `@ComponentScan`注解使Spring在`hello`包(package)中搜索其他的组件、配置(configurations)和服务(service),在本例中，spring会搜索到控制器(controllers)。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法来启动应用。你有没有注意到本例子中一行XML代码都没有吗？也没有web.xml文件。此web应用100%使纯java代码，因此不需花精力处理任何像基础设施或者下水管道一般的配置工作。

只要它们包含在`@SpringBootApplication`类的相同包（或子包）中，Spring Boot将自动处理这些存储库。
为了更好地控制注册过程，您可以使用 `@EnableMongoRepositories` 注解。

> 默认情况, `@EnableMongoRepositories`注解会扫描当前包的任何实现了Spring Data的存储库接口的接口。你的项目布局多个项目,没有找到你需要的存储库,使用 `basePackageClasses=MyRepository.class`,它会根据不同类型安全告诉Spring Data MongoDB 扫描不同的包。


Spring Data MongoDB使用 MongoTemplate 来执行 `find*` 方法后面的查询。您可以自己使用该模板进行更复杂的查询，但本指南不包括在内。

`Application`:包括一个自动装配实例的 `main()`方法。

CustomerRepository:Spring Data MongoDB 动态创建一个代理并注入。我们通过几个测试使用 `CustomerRepository`,首先保存少量 `Customer`对象,演示了`save（）`方法，并设置了一些可以使用的数据。下一步,它调用 `findAll()` 方法从数据库取出所有 Customer对象,然后调用 `findByFirstName()`方法,根据 firstName 找到一个 Customer对象。最后,调用`findByLastName()`, 找出所有lashtName 是"Smith"的所有 customers。

> Spring Boot 默认启动尝试连接到一个本地托管MongoDB的实例。阅读[ 参考文档  ](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-mongodb),详细知道应用程序如何关联MongoDB托管在别处的实例。

## 构建可执行的JAR

您可以在命令行使用Gradle或Maven运行应用程序。或者您可以构建一个可执行的JAR文件,其中包含所有必需的依赖关系,类,和资源,并运行。这使得在整个开发生命周期中非常容易，跨不同的环境，版本等传输和部署服务成为一个应用程序,等等。

如果您使用的是 Gradle，则可以使用该应用程序 `./gradlew bootRun`,或者您可以使用JAR文件构建 `./gradlew build`,然后可以运行 JAR 文件：

```
java -jar build/libs/gs-accessing-data-mongodb-0.1.0.jar
```
如果您使用 Maven，则可以使用该命令 `./mvnw spring-boot:run`,或者您可以使用 JAR 文件来构建 `./mvnw clean package`,然后可以运行 JAR 文件：

```
java -jar target/gs-accessing-data-mongodb-0.1.0.jar
```


> 上面的过程将创建一个可运行的JAR。您也可以选择 [构建一个WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

当我们的应用程序实现了 `CommandLineRunner`,引导时启动时会调用`run`方法。你应该看到这样的东西(和查询一样的其他东西):

```
== Customers found with findAll():
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']

== Customer found with findByFirstName('Alice'):
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
== Customers found with findByLastName('Smith'):
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']
```

## 总结

恭喜你!你设置了一个 MongoDB 服务器,编写了一个简单的应用程序,使用Spring Data MongoDB保存数据对象并从数据库中获取数据,而无需编写具体的存储库实现。

> 如果您有兴趣在MongoDB存储库中使用基于超媒体的RESTful前端进行了很少的工作, 你可能会想读[使用REST访问JPA数据](https://spring.io/guides/gs/accessing-mongodb-data-rest/)

## 了解更多
下面的指南也非常有帮助：
*   [使用JPA访问数据](https://spring.io/guides/gs/accessing-data-jpa/)
*   [使用Gemfire访问数据](https://spring.io/guides/gs/accessing-data-gemfire/)
*   [使用MySQL访问数据](https://spring.io/guides/gs/accessing-data-mysql/)
*   [使用Neo4j访问数据](https://spring.io/guides/gs/accessing-data-neo4j/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
