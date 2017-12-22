# 使用GemFire访问数据

> 原文：[Accessing Data with GemFire](https://spring.io/guides/gs/accessing-data-gemfire/)
>
> 译者：[dejunyu](https://github.com/dejunyu)
>
> 校对：


本指南将指导您完成使用GemFire的数据结构构建应用程序的过程。

## 你会建立什么

您将使用功能强大的Spring Data GemFire库来存储和检索[POJO](https://spring.io/understanding/POJO)。

## 你需要什么

+ 约 15 分钟
+ 最喜欢的文本编辑器或IDE
+ [JDK 1.8 ](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或更高版本
+ [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
+ 你还可以将代码直接导入到IDE中：
  + [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  + [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

##  如何完成本指南

与大多数“ [入门指南](https://spring.io/guides) ”一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。无论哪种方式，你都会得到可以成功运行的代码。

要**从头开始**，请跳转到使用 [Gradle构建](https://spring.io/guides/gs/testing-restdocs/#scratch)。

要**跳过基本操作**，请执行以下操作：

+ [下载](https://github.com/spring-guides/gs-accessing-data-gemfire/archive/master.zip) 并解压缩本指南的源代码库，或使用 [Git](https://spring.io/understanding/Git) 克隆它：`git clone https://github.com/spring-guides/gs-accessing-data-gemfire.git`
+ cd进入 `gs-accessing-data-gemfire/initial`
+ 跳转到[定义的一个简单的实体](https://spring.io/guides/gs/accessing-data-gemfire/#initial)。

**完成后**，你可以根据代码检查结果 `gs-accessing-data-gemfire/complete`。

## 使用 Gradle 构建

第一步，建立基本的构建脚本（build script）。 当使用 Spring 构建 apps 的时候，几乎可以使用任何你喜欢的构建工具， 但是此指南只介绍了如何使用 Gradle 和 Maven 来构建目标 app。如果这两个工具你都不熟悉，请参考 [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) 或 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构。例如，在 *nix 系统中使用命令 `mkdir -p src/main/java/hello`  来创建该目录结构。

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle构建文件

以下是 [初始化的Gradle构建文件](https://github.com/spring-guides/gs-accessing-data-gemfire/blob/master/initial/build.gradle)。

`build.gradle`

```gradle
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

jar {
    baseName = 'gs-accessing-data-gemfire'
    version = '0.1.0'
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
    compile("org.springframework.boot:spring-boot-starter")
    compile("org.springframework.data:spring-data-gemfire")
    compile("org.projectlombok:lombok:1.16.18")
    runtime("org.springframework.shell:spring-shell:1.2.0.RELEASE")
}
```

在 [Spring Boot Gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多方便的功能：
+ 它收集类路径上的所有jar，并构建一个可运行的“über-jar”，这使得执行和传输服务更加方便。
+ 搜索 `public static void main()` ，并对该可执行类进行标记。
+ 提供了一个内置的依赖解析功能，该功能将依赖的版本与 [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 相匹配。用户可以按照需求覆盖依赖（dependency）的任何版本号，但是默认版本号是 Spring Boot中已经选择好的版本号的集合。

## 使用 Maven 构建应用程序

第一步，建立基本构建脚本(build script)。 当使用Spring构建apps的时候，几乎可以使用任何你喜欢的构建工具， 但是此部分只介绍了如何使用 [Maven](https://maven.apache.org/) 来构建目标app。如果你不熟悉这个工具，请参考 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选定的工程目录中，创建如下的子目录结构。 例如，在 *nix 中使用命令  `mkdir -p src/main/java/hello` 来创建该目录结构。

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

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <groupId>org.springframework</groupId>
    <artifactId>gs-accessing-data-gemfire</artifactId>
    <version>0.1.0</version>

    <properties>
        <java.version>1.8</java.version>
        <spring.version>5.0.1.RELEASE</spring.version>
        <spring-data-releasetrain.version>Kay-SR1</spring-data-releasetrain.version>
        <spring-shell.version>1.2.0.RELEASE</spring-shell.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-gemfire</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.shell</groupId>
            <artifactId>spring-shell</artifactId>
            <version>${spring-shell.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
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

</project>
```

+ 在 [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多方便的功能：
  + 将 classpath 中的所有 jar 文件集中起来，构建成单独的可运行的 "über-jar", 这使得服务的运行和转移更加便捷。
  + 搜索 `public static void main()` ，并对该可执行类进行标记。
  + 提供了一个内置的依赖解析功能，该功能将依赖的版本与 [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 相匹配。用户可以按照需求覆盖依赖（dependency）的任何版本号，但是默认版本号是 Spring Boot中已经选择好的版本号的集合。

## 使用IDE构建

- 阅读如何将本指南直接导入到 [Spring Tool Suite](https://spring.io/guides/gs/sts/) 中。
- 阅读如何在 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 中使用的指南。

## 定义一个简单的实体

Pivotal GemFire是一个将数据映射到区域的内存数据网格（IMDG）。 可以配置分布式区域，用于跨群集中的多个节点分区和复制数据。 但是，在本指南中，您将使用`本地`区域，因此您不必另外设置任何内容，例如整个服务器群集。

GemFire是一个键/值存储区，一个Region实现了`java.util.concurrent.ConcurrentMap`接口。 虽然你可以像一个`java.util.Map`那样对待一个Region，但它比一个简单的Java `Map`要复杂得多，因为数据在这个Region内被分配，复制和管理。

在这个例子中，你只用几个注释就可以把`Person`对象存储在GemFire（一个Region）中。

`src/main/java/hello/Person.java`

```java
package hello;

import java.io.Serializable;

import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.PersistenceConstructor;
import org.springframework.data.gemfire.mapping.annotation.Region;

import lombok.Getter;

@Region(value = "People")
public class Person implements Serializable {

    @Id
    @Getter
    private final String name;

    @Getter
    private final int age;

    @PersistenceConstructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return String.format("%s is %d year(s) old", name, age);
    }
}
```

在这里你有一个`Person`类，有两个字段，`name`和`age`。 在创建新实例时，您也有一个持久的构造函数来填充实体。 该类使用[Project Lombok ](https://projectlombok.org/)来简化实施。

注意这个类是用`@Region（“People”）`注解的。 当GemFire存储这个类的实例时，会在“People”区域内创建一个新条目。 这个类也用`@Id`标记`name`字段。 这表示用于识别和跟踪GemFire中的`Person`数据的标识符。 本质上，`@Id`带注释的字段（例如`name`）是键，`Person`实例是键/值条目中的值。 在GemFire中没有自动生成密钥，因此您必须在将实体持久化到GemFire之前设置id（即`name`）。

下一个重要的部分是这个人的年龄。 在本指南的后面，您将使用它来形成一些查询。

重写的`toString（）`方法将打印出人的姓名和年龄。

## 创建简单的查询

*Spring Data GemFire*专注于存储和访问GemFire中的数据。 它还继承了*Spring Data Commons*项目的强大功能，如派生查询的能力。 本质上，您不必学习GemFire（OQL）的查询语言; 您可以简单地编写一些方法，查询是由框架为您编写的。

要查看这是如何工作的，请创建一个查询存储在GemFire中的`Person`对象的接口。

`src/main/java/hello/PersonRepository.java`

```java
package hello;

import org.springframework.data.gemfire.repository.query.annotation.Trace;
import org.springframework.data.repository.CrudRepository;

public interface PersonRepository extends CrudRepository<Person, String> {

    @Trace
    Person findByName(String name);

    @Trace
    Iterable<Person> findByAgeGreaterThan(int age);

    @Trace
    Iterable<Person> findByAgeLessThan(int age);

    @Trace
    Iterable<Person> findByAgeGreaterThanAndAgeLessThan(int greaterThanAge, int lessThanAge);

}
```

`PersonRepository`扩展了`Spring Data Commons`中的CrudRepository接口，并分别为存储库使用的值和id（键）指定泛型类型参数的类型，即`Person`和`String`。 开箱即用，这个接口带有许多操作，包括基本的CRUD（CREATE，READ UPDATE，DELETE）和简单的查询（例如`findById（..）`）数据访问操作。

您可以通过简单地声明其方法签名来定义其他查询。 在这种情况下，您添加`findByName`，它本质上搜索Person类型的对象并找到与`name`匹配的对象。

你也有：

- `findByAgeGreaterThan`找到超过一定年龄的人
- `findByAgeLessThan`找到一定年龄以下的人
- `findByAgeGreaterThanAndAgeLessThan`找到某个年龄段的人

让我们把它连接起来，看看它是什么样的！

## 创建一个应用程序类

在这里你创建一个包含所有组件的应用程序类。

`src/main/java/hello/Application.java`

```java
package hello;

import static java.util.Arrays.asList;
import static java.util.stream.StreamSupport.stream;

import java.io.IOException;

import org.apache.geode.cache.GemFireCache;
import org.apache.geode.cache.client.ClientRegionShortcut;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.gemfire.client.ClientRegionFactoryBean;
import org.springframework.data.gemfire.config.annotation.ClientCacheApplication;
import org.springframework.data.gemfire.repository.config.EnableGemfireRepositories;

@ClientCacheApplication(name = "DataGemFireApplication", logLevel = "error")
@EnableGemfireRepositories
public class Application {

    public static void main(String[] args) throws IOException {
        SpringApplication.run(Application.class, args);
    }

    @Bean("People")
    public ClientRegionFactoryBean<String, Person> peopleRegion(GemFireCache gemfireCache) {

        ClientRegionFactoryBean<String, Person> peopleRegion = new ClientRegionFactoryBean<>();

        peopleRegion.setCache(gemfireCache);
        peopleRegion.setClose(false);
        peopleRegion.setShortcut(ClientRegionShortcut.LOCAL);

        return peopleRegion;
    }

    @Bean
    ApplicationRunner run(PersonRepository personRepository) {

        return args -> {

            Person alice = new Person("Alice", 40);
            Person bob = new Person("Baby Bob", 1);
            Person carol = new Person("Teen Carol", 13);

            System.out.println("Before accessing data in GemFire...");

            asList(alice, bob, carol).forEach(person -> System.out.println("\t" + person));

            System.out.println("Save Alice, Bob and Carol to GemFire...");

            personRepository.save(alice);
            personRepository.save(bob);
            personRepository.save(carol);

            System.out.println("Lookup each person by name...");

            asList(alice.getName(), bob.getName(), carol.getName())
              .forEach(name -> System.out.println("\t" + personRepository.findByName(name)));

            System.out.println("Query adults (over 18):");

            stream(personRepository.findByAgeGreaterThan(18).spliterator(), false)
              .forEach(person -> System.out.println("\t" + person));

            System.out.println("Query babies (less than 5):");

            stream(personRepository.findByAgeLessThan(5).spliterator(), false)
              .forEach(person -> System.out.println("\t" + person));

            System.out.println("Query teens (between 12 and 20):");

            stream(personRepository.findByAgeGreaterThanAndAgeLessThan(12, 20).spliterator(), false)
              .forEach(person -> System.out.println("\t" + person));
        };
    }
}
```

在配置中，您需要添加`@EnableGemfireRepositories`注释。

- 默认情况下，`@EnableGemfireRepositories`将会扫描当前包中的任何扩展Spring Data接口的接口。使用它的`basePackageClasses = MyRepository.class`安全地告诉Spring Data GemFire通过类型来扫描不同的根包，以获取特定于应用程序的Repository扩展。

包含一个或多个区域的GemFire缓存需要存储所有的数据。为此，您使用Spring Data GemFire的基于配置的便捷注释：`@ClientCacheApplication`，`@PeerCacheApplication`或`@CacheServerApplication`。

GemFire支持不同的缓存拓扑结构，如客户端/服务器，点对点（p2p）甚至WAN安排。在p2p中，对等缓存实例嵌入在应用程序中，您的应用程序将有能力作为对等缓存成员加入群集。但是，您的应用程序受限于在集群中作为对等成员的所有约束，所以这不像通常所说的客户端/服务器拓扑那样。

在我们的例子中，我们将使用`@ClientCacheApplication`来创建一个“客户端”缓存实例，它能够连接到一组服务器并与之通信。但是，为了简单起见，客户端只需使用`LOCAL`客户端区域在本地存储数据，而不需要设置或运行任何服务器。

> Spring建议生产[Pivotal GemFire](https://pivotal.io/pivotal-gemfire)的企业版，您可以在群集中的多个节点上创建分布式缓存和区域。 或者，您也可以使用开源版本[Apache Geode](https://geode.apache.org/)，并从Apache Geode社区获得支持。

现在，请记住如何使用SDG映射注释`@Region（“People”）`将`Person`标记为存储在名为“People”的Region中？ 您可以在这里使用`ClientRegionFactoryBean <String，Person>` bean定义来定义该区域。 你需要注入一个你刚定义的缓存实例，同时命名为“People”。

> GemFire缓存实例（无论是对等体还是客户机）只是存储数据的区域的容器。 您可以将RDBMS和Regions中的缓存视为表中的模式。 但是，缓存还执行其他管理功能来控制和管理您的所有区域。

> 类型是`<String，Person>`，将键类型（`String`）与值类型（`Person`）进行匹配。

`public static void main`方法使用Spring Boot的`SpringApplication.run（）`来启动应用程序并调用`ApplicationRunner`（另一个bean定义），该应用程序使用应用程序的Spring Data Repository在GemFire上执行数据访问操作。

应用程序自动装载您刚刚定义的`PersonRepository`实例。 Spring Data GemFire将动态创建一个实现此接口的具体类，并插入所需的查询代码以满足接口的义务。 这个Repository实例是`run（）`方法用来演示这个功能的。

## 存储和获取数据

在本指南中，您将创建三个本地`Person`对象，*Alice，Baby Bob*和*Teen Carol*。 最初，他们只存在于记忆中。 创建它们之后，你必须将它们保存到GemFire。

现在你运行几个查询。 第一个按名字查找每个人。 然后执行一些查询来查找成年人，婴儿和青少年，所有这些都使用年龄属性。 随着日志的出现，你可以看到Spring Data GemFire为你写的查询。

将`@ClientCacheApplication`注释`logLevel`属性更改为“config”以查看由SDG生成的GemFire OQL查询。 由于查询方法（例如`findByName`）是使用SDG的`@Trace`注释进行注释的，因此会启用GemFire的OQL查询跟踪（查询级别日志记录），该查询会显示生成的OQL，执行时间以及查询是否使用了任何GemFire索引 收集结果以及查询返回的行数。

## 构建一个可执行的JAR

您可以使用Gradle或Maven从命令行运行应用程序。 或者，您可以构建一个包含所有必需的依赖项，类和资源的可执行JAR文件，然后运行该文件。 这使得在整个开发生命周期内跨越不同的环境等等，将服务作为应用程序发布，版本化和部署变得容易。

如果您正在使用Gradle，则可以使用`./gradlew bootRun`运行该应用程序。 或者您可以使用`./gradlew build`生成JAR文件。 然后你可以运行JAR文件：

`java -jar build/libs/gs-accessing-data-gemfire-0.1.0.jar`

如果您正在使用Maven，则可以使用`./mvnw spring-boot：run`来运行该应用程序。 或者你也可以使用`./mvnw clean package`建立JAR文件。 然后你可以运行JAR文件：

`java -jar target/gs-accessing-data-gemfire-0.1.0.jar`

> 上面的过程将创建一个可运行的JAR。 您也可以选择[构建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。
> 你应该看到像这样的东西（以及其他的东西，如查询）：

```text
Before linking up with GemFire...
	Alice is 40 years old.
	Baby Bob is 1 years old.
	Teen Carol is 13 years old.
Lookup each person by name...
	Alice is 40 years old.
	Baby Bob is 1 years old.
	Teen Carol is 13 years old.
Adults (over 18):
	Alice is 40 years old.
Babies (less than 5):
	Baby Bob is 1 years old.
Teens (between 12 and 20):
	Teen Carol is 13 years old.
```

## 概要

恭喜！ 您设置了一个GemFire缓存客户端，存储简单的实体，并开发了快速查询。

## 也可以看看

以下指南也可能有所帮助：

- [使用JPA访问数据](https://spring.io/guides/gs/accessing-data-jpa/)
- [使用MongoDB访问数据](https://spring.io/guides/gs/accessing-data-mongodb/)
- [使用MySQL访问数据](https://spring.io/guides/gs/accessing-data-mysql/)
- [使用Neo4j访问数据](https://spring.io/guides/gs/accessing-data-neo4j/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。