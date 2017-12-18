# 使用Neo4j访问数据

> 原文：[Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/)
>
> 译者：[hh23485](https://github.com/hh23485)
>
> 校对：


本指南将引导您使用[ Spring Data Neo4j ](https://projects.spring.io/spring-data-neo4j/)来构建一个应用程序，该应用程序能够从基于图的[Neo4j](http://www.neo4j.com/)数据库中存储和检索数据。


## 您将创建什么
您将使用[Neo4j](https://spring.io/understanding/NoSQL)的存储图数据的NoSQL来创建一个嵌入式的Neo4j服务器，用于存储实体、关系并制定查询。

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

**如果从基础开始**，您可以往下查看[怎样使用 Gradle 构建项目](#scratch)。

**如果已经熟悉跳过一些基本步骤**，您可以：

* [下载](https://github.com/spring-guides/gs-accessing-data-neo4j/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/gs-accessing-data-neo4j.git`
* 进入 `gs-accessing-data-neo4j/initial`目录
* 跳过前面的部分来[创建一个简单的实体](https://spring.io/guides/gs/accessing-data-neo4j/#initial)

**当您完成之后**，您可以在`gs-accessing-data-neo4j/complete`根据代码检查下结果。

## 使用Gradle构建

首先您需要编写基础构建脚本。在构建 Spring 应用的时候，您可以使用任何您喜欢的系统来构建， 这里提供一份您可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果您两者都不是很熟悉, 您可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

在您的项目根目录，创建如下的子目录结构; 例如，如果您使用的是\*nix系统，您可以使用`mkdir -p src/main/java/hello`:

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
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-accessing-data-neo4j'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-release" }
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-neo4j")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布您的服务变得更加便捷。
* 搜索 `public static void main()` 方法并且将它标记为可执行类。
* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。您可以根据您的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

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
	<artifactId>gs-accessing-data-neo4j</artifactId>
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
			<artifactId>spring-boot-starter-data-neo4j</artifactId>
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
			<name>Spring Releases</name>
			<url>https://repo.spring.io/libs-release</url>
		</repository>
	</repositories>

</project>
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布您的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。您可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。



## 使用您的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。



## 创建一个Neo4j服务器

在您开始创建该项目之前，您需要部署一个Neo4j服务。

您可以免费安装一个Neo4j的开源服务器。

对于Mac用户，只需要输入:

```shell
$ brew install neo4j
```

对于其他的配置选项，可访问[https://neo4j.com/download/community-edition/](https://neo4j.com/download/community-edition/)

在您安装后，可以通过默认的配置启动：

```shell
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

```shell
$ curl -v -u neo4j:neo4j -X POST localhost:7474/user/neo4j/password -H "Content-type:application/json" -d "{\"password\":\"secret\"}"
```

这将会将密码从**neo4j**修改为**secret**（请注意不要在生产环境下这样做！）在完成这些后，您可以正式开始这篇指南了。

## 定义一个实体

Neo4j捕获实体和它们之间的关系，其中有两个同等重要的方面。想象一下，您在搭建一个存储所有人记录的系统。而且，您还需要跟踪某个人的同事关系（在这个例子中是`teammates`）。在Neo4j中，您可以通过简单的注解捕捉所有的这些关系。

`src/main/java/hello/Person.java`

```java
package hello;

import java.util.Collections;
import java.util.HashSet;
import java.util.Optional;
import java.util.Set;
import java.util.stream.Collectors;

import org.neo4j.ogm.annotation.GraphId;
import org.neo4j.ogm.annotation.NodeEntity;
import org.neo4j.ogm.annotation.Relationship;

@NodeEntity
public class Person {

	@GraphId private Long id;

	private String name;

	private Person() {
		// Empty constructor required as of Neo4j API 2.0.5
	};

	public Person(String name) {
		this.name = name;
	}

	/**
	 * Neo4j doesn't REALLY have bi-directional relationships. It just means when querying
	 * to ignore the direction of the relationship.
	 * https://dzone.com/articles/modelling-data-neo4j
	 */
	@Relationship(type = "TEAMMATE", direction = Relationship.UNDIRECTED)
	public Set<Person> teammates;

	public void worksWith(Person person) {
		if (teammates == null) {
			teammates = new HashSet<>();
		}
		teammates.add(person);
	}

	public String toString() {

		return this.name + "'s teammates => "
				+ Optional.ofNullable(this.teammates).orElse(
						Collections.emptySet()).stream().map(
								person -> person.getName()).collect(Collectors.toList());
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

在这里类 `Person`只包含一个属性`name `。

>在本篇指南中，典型的getter和setter方法将不再赘述。

类`Person`被标注了注解`@NodeEntity`。当Neo4j存储它的时候，它会创建一个新的节点。该类还有一个属性`id`被注解`@GraphId`标注。Neo4j使用`@GraphId`注解在内部跟踪数据。

下一个重要的点是一组`teammates`的集合。它使用简单的`Set<Person>`来表示，但是被标注了`@Relationship`注解。这表示该集合所有的成员都被期望作为独立于`Person`的节点。注意，这里的关系方向被设置为`UNDIRECTED`。这表示当您查询`TEAMMATE`关系的时候，Spring Data Neo4j将会忽略关系的方向。

有了`worksWith()`方法，您可以轻松的将人们连接到一起。

最后，您可以方便的使用`toString()`方法来输出所有人的姓名和这个人的同事们。



## 创建简单的查询

Spring Data Neo4j 专注于在Neo4j中存储数据。但它继承了Spring Data Commons 项目的功能，包括了派生查询的能力。本质上来说，您不需要学习如何使用Neo4j的查询语言，您也可以简单的写出易用的查询和方法。

为了展示它是如何工作的，创建一个接口来查询`Person`节点。

``src/main/java/hello/PersonRepository.java``

```java
package hello;

import java.util.List;

import org.springframework.data.neo4j.repository.GraphRepository;
import org.springframework.data.repository.CrudRepository;

public interface PersonRepository extends GraphRepository<Person> {

    Person findByName(String name);
}
```

`PersonRepository`继承自`GraphRepository`接口，并且插入了`Person`作为操作的类型。开箱即用，该接口生来即包括了标准的CURD（创建create-读取read-更新update-delete删除）操作。

您可以按需定义其他的查询，只需要简单的声明方法的签名即可。在这个例子中，您添加了`findByName`，该方法将会查询`Person`节点，并且找到一个`name`字段相匹配的元素。您也可以用`findByTeammatesName`来钻入每一个实体的`teammates`字段查找，直到找到一个匹配了同事名称为`name`的`Person`节点。



## Neo4j访问权限

Neo4j社区版需要凭据来访问数据。凭据可以用如下几个属性进行配置。

```properties
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=secret
```

这囊括了默认的用户名`neo4j`和一个我们早先新设置的密码`secret`。

> 不要在您的源代码库中存储真实的凭证。相反，在运行时使用[SpringBoot属性覆盖](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config)来进行配置。

这个时候，让我们来把他们组装起来看看。



## 创建一个应用程序类

我们使用所有的组件来创建一个应用程序类。

``src/main/java/hello/Application.java``

```java
package hello;

import java.util.Arrays;
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.neo4j.repository.config.EnableNeo4jRepositories;

@SpringBootApplication
@EnableNeo4jRepositories
public class Application {

	private final static Logger log = LoggerFactory.getLogger(Application.class);

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Application.class, args);
	}

	@Bean
	CommandLineRunner demo(PersonRepository personRepository) {
		return args -> {

			personRepository.deleteAll();

			Person greg = new Person("Greg");
			Person roy = new Person("Roy");
			Person craig = new Person("Craig");

			List<Person> team = Arrays.asList(greg, roy, craig);

			log.info("Before linking up with Neo4j...");

			team.stream().forEach(person -> log.info("\t" + person.toString()));

			personRepository.save(greg);
			personRepository.save(roy);
			personRepository.save(craig);

			greg = personRepository.findByName(greg.getName());
			greg.worksWith(roy);
			greg.worksWith(craig);
			personRepository.save(greg);

			roy = personRepository.findByName(roy.getName());
			roy.worksWith(craig);
			// We already know that roy works with greg
			personRepository.save(roy);

			// We already know craig works with roy and greg

			log.info("Lookup each person by name...");
			team.stream().forEach(person -> log.info(
					"\t" + personRepository.findByName(person.getName()).toString()));
		};
	}

}
```

`@SpringBootApplication`是一个简洁的注解来代替下面的所有：

- `@Configuration`标记这个类为定义应用上下文来源的Bean
- `@EnableAutoConfiguration`告诉SpringBoot开始根据类路径设置、其他的beans和各种属性设置来添加beans
- 通常您会为Spring MVC 应用添加`@EnableWebMvc`注解，但是SpringBoot在类路径中发现**spring-webmvc**会自动添加该注解。这表示应用程序作为web应用程序并且激活`DispatcherServlet`的核心功能
- `@ComponentScan`告诉Spring在`hello`包中查找其他的组件、配置和服务，也允许发现控制器。

在`main()`方法中，使用Spring Boot的`SpringApplication.run()`方法来启动一个应用程序。您是不是注意到这里一行XML都没有？也没有**web.xml**文件。这个web应用程序是100%纯Java的，您并不需要为任何管道或基础平台进行配置。

Spring Boot 将会自动处理repositories，只要他们和`@SpringBootAplication`标注的类放在同一个包（或子包）即可。如果需要在注册的时候添加更多的控制，您可以使用`@EnableNeo4jRepositories`注解。

> 默认情况下，`@EnableNeo4jRepositories`将会自动扫描当前包来查找所有继承自Spring Data 的repository的接口。如果您的项目布局中有多个项目，而且不能够找到您的repositories，您可以使用它的`basePackageClasses=MyRepository.class`可以告诉Spring Data Neo4j去扫描一个不同的根包和类型。

日志会被输出。服务会在启动的几秒后开始运行。

您自动装配了一个您之前定义的`PersonRepository`的实例。Spring Data Neo4j 将会动态的实现接口并且插入所需的查询语句以满足您的查询功能。

`public static void main`使用SpringBoot的`SpringApplication.run()`方法来启动一个应用程序并调用`CommandLineRunner`来创建关系。

在这个例子中，您创建了3个本地的`Person`对象，**Greg**，**Roy**和**Craig**。最开始的时候，他们只是存在于内存中。同样需要注意的是，他们现在彼此都不是彼此的同事。

首先，您找到Greg并且推断他与Roy和Craig一起工作，然后存储他。记住，由于同事关系被标注为`UNDIRECTED`，这意味着，同事关系是双向的。说明Roy和Craig也将会被更新。

这就是为什么当您想要更新Roy的时候，您需要先从Neo4j抓取记录。而且在您将**Craig**添加到Roy的同事列表之前，您需要保证他的同事列表状态是最新的。

为什么没有代码获取Craig数据和添加任何关系呢？因为您已经完成了它！**Greg**早先标注了Craig为一个同事，Roy也是。这意味着此时不再需要更新Craig的关系列表了。您可以通过遍历他们的队员并输出到控制台的方式来查看这些关系。

最后，检查您之前看到的查询，然后回答：“谁和谁一起工作呢？”



## 创建可执行的JAR



您可以在命令行使用 `Zradle` 或 `Maven` 运行应用程序。或者您可以构建一个可执行的 `JAR` 文件,其中包含所有必需的依赖关系,类,和资源,并运行。这使得在整个开发生命周期中非常容易，跨不同的环境，版本等传输和部署服务成为一个应用程序,等等。

如果您使用的是 `Gradle`，则可以使用该应用程序 `./gradlew bootRun`,或者您可以使用 `JAR` 文件构建 `./gradlew build`,然后可以运行 `JAR` 文件：

```shell
java -jar build/libs/gs-accessing-data-neo4j-0.1.0.jar
```

如果您使用 `Maven`，则可以使用该命令 `./mvnw spring-boot:run`,或者您可以使用 `JAR` 文件来构建 `./mvnw clean package`,然后可以运行 `JAR` 文件：

```shell
java -jar target/gs-accessing-data-neo4j-0.1.0.jar
```

> 上面的过程将创建一个可运行的 `JAR`。您也可以选择 [构建一个 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

您应该能看到如下的的内容（以及其他查询之类的内容）

```
Before linking up with Neo4j...
	Greg's teammates => []
	Roy's teammates => []
	Craig's teammates => []

Lookup each person by name...
	Greg's teammates => [Roy, Craig]
	Roy's teammates => [Greg, Craig]
	Craig's teammates => [Roy, Greg]
```

您可以从输出中发现。最开始没有人和其他人之间有关联关系。再添加了人之后，他们连接在了一起。最终，您可以看到，基于teammate能够进行方便的查询。

## 总结

恭喜！您创建了一个嵌入式的Neo4j服务，存储了一些简单、存在关联的实体，并且开发了一些快捷的查询。

> 如果您对通过超媒体来从前端访问后端暴露的Neo4j仓库感兴趣，您可以阅读[使用REST访问Neo4j](https://spring.io/guides/gs/accessing-neo4j-data-rest)

## 了解更多
下面的指南也非常有帮助：

*   [访问MySQL数据](https://spring.io/guides/gs/accessing-data-mysql/)
*   [访问JPA数据](https://spring.io/guides/gs/accessing-data-jpa/)
*   [访问MongoDB数据](https://spring.io/guides/gs/accessing-data-mongodb/)
*   [访问Gemfire数据](https://spring.io/guides/gs/accessing-data-gemfire/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
