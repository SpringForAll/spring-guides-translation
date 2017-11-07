# 使用Spring JDBC访问关系型数据库

> 原文：[Accessing Relational Data using JDBC with Spring](https://spring.io/guides/gs/relational-data-access/)
>
> 译者：[王春磊](https://github.com/codedrinker)
>
> 校对：

本指南将引导你使用`Spring`访问关系型数据库。

## 你将构建什么应用 

你会通过`Spring`的`JdbcTemplate`搭建一个应用来访问关系型数据库。

## 你需要准备什么？

- 大约15分钟时间
- 一个喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接导入代码到IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 你如何完成这个入门指南

像其他Spring入门指南一样，你可以从头开始逐步完成每一步，也可以跳过一些你熟悉的步骤。不管怎样，最后你都将得到一份可执行的代码。

**从零开始**，可以点击[怎样使用 Gradle 构建项目](#scratch).

**跳过基础步骤**，请继续往下看

* [下载](https://github.com/spring-guides/gs-relational-data-access/archive/master.zip)并解压这个引导。或者直接使用 [Git](https://spring.io/understanding/Git):**git clone**:`git clone https://github.com/spring-guides/gs-relational-data-access.git`
* 进入`gs-relational-data-access/initial`目录
* 跳到[创建Customer对象](https://spring.io/guides/gs/relational-data-access/#initial).

完成上述步骤，你可以查看结果，并与`gs-relational-data-access/complete`对比。

## 使用Gradle构建

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建, 如果你想使用[Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建那么请阅读如下代码。如果你对两者都不熟悉,可以先参考[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle) 或者 [使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录下，创建以下子目录;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle文件

如下是一个 [Gradle初始化文件](https://github.com/spring-guides/gs-relational-data-access/blob/master/initial/build.gradle).

`build.gradle`

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.8.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-relational-data-access'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter")
    compile("org.springframework:spring-jdbc")
    compile("com.h2database:h2")
    testCompile("junit:junit")
}
```

 [Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的特性:

- 它收集`classpath`上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和传递你的应用。
- 它寻找`public static void main()` 方法并将该类作为可执行类。
- 它提供了一个内置的依赖解析器，使得项目可以自动匹配`Spring Boot`的依赖版本。当然你可以修改成任意的版本，它只是默认设置为`Boot`所选择的一组版本。

## 使用Maven构建

首先你需要编写基础构建脚本。在使用`Spring`构建应用程序时，你可以使用任何构建系统。如果你想使用[`Maven`](https://maven.apache.org/)构建那么请阅读如下代码。如果你不熟悉`Maven`，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

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
    <artifactId>gs-relational-data-access</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
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

- 它收集`classpath`上的所有jar包，并构建一个可运行的`jar`包，这样可以更方便地执行和传递你的应用。
- 它寻找`public static void main()` 方法并将该类作为可执行类。
- 它提供了一个内置的依赖解析器，使得项目可以自动匹配`Spring Boot`的依赖版本。当然你可以修改成任意的版本，它只是默认设置为`Boot`所选择的一组版本。

## 使用你的IDE构建

- 如何在`Spring Tool Suite`中构建项目 [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- 如何在`IntelliJ IDEA`构建项目 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建.

## 创建一个Customer类

下面是一个简单访问数据库的逻辑，我们需要创建`Customer`类然后实现对`fisrtName`和`lastName`属性的操作。

`src/main/java/hello/Customer.java`

```java
package hello;

public class Customer {
    private long id;
    private String firstName, lastName;

    public Customer(long id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%d, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

    // getters & setters omitted for brevity
}
```
## 存储和读取数据
`Spring`提供了一个叫做`JdbcTemplate`的类，可以轻松的使用`JDBC`访问关系型数据库。大多数`JDBC`的代码需要你处理资源，连接管理，异常处理和错误检查等相关性不是很强的问题，然而`JdbcTemplate`可以只让你关注业务逻辑。

`src/main/java/hello/Application.java`
```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jdbc.core.JdbcTemplate;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

@SpringBootApplication
public class Application implements CommandLineRunner {

    private static final Logger log = LoggerFactory.getLogger(Application.class);

    public static void main(String args[]) {
        SpringApplication.run(Application.class, args);
    }

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(String... strings) throws Exception {

        log.info("Creating tables");

        jdbcTemplate.execute("DROP TABLE customers IF EXISTS");
        jdbcTemplate.execute("CREATE TABLE customers(" +
                "id SERIAL, first_name VARCHAR(255), last_name VARCHAR(255))");

        // Split up the array of whole names into an array of first/last names
        List<Object[]> splitUpNames = Arrays.asList("John Woo", "Jeff Dean", "Josh Bloch", "Josh Long").stream()
                .map(name -> name.split(" "))
                .collect(Collectors.toList());

        // Use a Java 8 stream to print out each tuple of the list
        splitUpNames.forEach(name -> log.info(String.format("Inserting customer record for %s %s", name[0], name[1])));

        // Uses JdbcTemplate's batchUpdate operation to bulk load data
        jdbcTemplate.batchUpdate("INSERT INTO customers(first_name, last_name) VALUES (?,?)", splitUpNames);

        log.info("Querying for customer records where first_name = 'Josh':");
        jdbcTemplate.query(
                "SELECT id, first_name, last_name FROM customers WHERE first_name = ?", new Object[] { "Josh" },
                (rs, rowNum) -> new Customer(rs.getLong("id"), rs.getString("first_name"), rs.getString("last_name"))
        ).forEach(customer -> log.info(customer.toString()));
    }
}
```

`@SpringBootApplication`作为一个方便使用的注解，提供了如下的功能：  

- `@Configuration`表明使用该注解的类是应用程序上下文(Applicaiton Context)中Bean定义的来源。
- `@EnableAutoConfiguration`注解根据classpath的配置、其他bean的定义或者不同的属性设置(property settings)等条件，使Spring Boot自动加入所需的bean。  
- 对于Spring MVC应用，通常需要加入`@EnableWebMvc`注解，但是当**spring-webmvc** 存在于classpath中时，Spring Boot自动加入该注解。该注解将当前应用标记为web应用，并激活web应用的关键行为，例如开启`DispatcherServlet`。
- `@ComponentScan`注解使Spring在`hello`包(package)中搜索其他的组件、配置(configurations)和服务(service),在本例中，spring会搜索到控制器(controllers)。  

`main()`方法使用Spring Boot的`SpringApplication.run()`方法来加载应用。你有没有注意到本例子中一行XML代码都没有吗？也没有web.xml文件。此web应用100%使纯java代码，因此不需花精力处理任何像基础设施或者下水管道一般的配置工作。  

`Spring Boot`同样支持关系型内存数据库`H2`，并且可以自动获取`Connection`。因为我们使用的是`spring-jdbc`它可以自动帮我们创建`JdbcTemplate`。然后使用`@Autowired JdbcTemplate`注解就可以加载和使用了。 

我们这个`Application`实现了`CommandLineRunner`接口，这样在程序上下文加载完成的时候我们就可以执行其`run()`方法来运行程序。  

那么首先让我们运行`JdbcTemplate`的`execute`方法生成DDL。  
然后定义一个包含`firstName`和`lastName`的`String list`，使用`Java 8`的`Stream`处理成`firstName`和`lastName`组合的`list`。  
接着使用`JdbcTemplate`的`batchUpdate`方法把解析好的数据插入到表中。`batchUpdate`的第一个参数是`sql`语句，第二个参数是一个对象数组，它循环的把对应的值放到`sql`的占位符`?`里面。

> 插入一条数据的使用使用`JdbcTemplate`的`insert`方法，如果是多条建议使用`batchUpdate`方法。
当接受到消息的时候，获取到name,服务端服务将会发出一个回应，这个回应消息就和客户端传递的那样，也是一个JSON对象，就像下面这样。  

> 通过`?`占位符绑定变量的方式来避免`SQL`注入的攻击。

最后使用`query`方法根据条件查询结果，当然这时候也可以是用`?`占位符，等到真正执行的时候再赋值即可。最后的参数是使用了`Java 8`的`lambda`语法，循环把每一个结果转成成了`Customer`对象。

> `Java 8 lambdas`很好地映射到单个方法接口上，比如`Spring`的`RowMapper`。 如果您使用的是`Java 7`或更早版本，则可以使用匿名实现接口的方式替代，里面的内容和`Lambda`的一样就可以了，`Spring`也会正常的解析。

## 构建可执行的JAR

你可以从命令行运行`Gradle`或`Maven`应用程序。或者，你可以构建一个包含所有必需依赖项，类和资源的单个可执行JAR文件，并运行该文件。这使得在整个开发生命周期中，可以非常轻松的传递，版本控制和部署，当然跨平台也是可以的。
如果你使用的是`Gradle`，则可以使用`./gradlew bootRun`运行应用程序。或者你可以使用`./gradlew build构建来构建JAR文件。然后可以运行JAR文件：

```shell
java -jar build/libs/gs-relational-data-access-0.1.0.jar
```

如果使用Maven，可以使用`./mvnw spring-boot:run`运行应用程序，或者你可以使用`./mvnw clean package`来构建JAR文件，然后可以运行JAR文件

```shell
java -jar target/gs-relational-data-access-0.1.0.jar
```

> 上面的过程将创建一个可运行的`jar`，你也可以选择构建一个典型的[`war`](https://spring.io/guides/gs/convert-jar-to-war/)文件。
 
下面是运行的输出结果
```
2015-06-19 10:58:31.152  INFO 67731 --- [           main] hello.Application                        : Creating tables
2015-06-19 10:58:31.219  INFO 67731 --- [           main] hello.Application                        : Inserting customer record for John Woo
2015-06-19 10:58:31.220  INFO 67731 --- [           main] hello.Application                        : Inserting customer record for Jeff Dean
2015-06-19 10:58:31.220  INFO 67731 --- [           main] hello.Application                        : Inserting customer record for Josh Bloch
2015-06-19 10:58:31.220  INFO 67731 --- [           main] hello.Application                        : Inserting customer record for Josh Long
2015-06-19 10:58:31.230  INFO 67731 --- [           main] hello.Application                        : Querying for customer records where first_name = 'Josh':
2015-06-19 10:58:31.242  INFO 67731 --- [           main] hello.Application                        : Customer[id=3, firstName='Josh', lastName='Bloch']
2015-06-19 10:58:31.242  INFO 67731 --- [           main] hello.Application                        : Customer[id=4, firstName='Josh', lastName='Long']
2015-06-19 10:58:31.244  INFO 67731 --- [           main] hello.Application  
```

## 总结

恭喜！你已经使用`Spring`成功构建了一个`JDBC`程序。

## 了解更多

以下指南也可能有帮助：

- [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
- [Accessing Data with GemFire](https://spring.io/guides/gs/accessing-data-gemfire/)
- [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
