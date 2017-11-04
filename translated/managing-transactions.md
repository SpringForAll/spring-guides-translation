# 事务管理

> 原文：[Managing Transactions](https://spring.io/guides/gs/managing-transactions/) 
>
> 译者：[Mr.lzc](https://github.com/cleverlzc)
>
> 校对：

本指南将引导你完成使用非侵入事务封装数据库操作的过程。

## 你将创建什么

你将构建一个简单的JDBC应用程序，其中你可以使数据库操作事务化，而无需编写[专门的JDBC代码](http://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html#commit_transactions)。

## 你需要准备什么

- 大约15分钟
- 一个自己喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 及以上版本
- [Gradle 2.3+](https://gradle.org/install/) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到本地的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本指南

与大多数[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。不管采用哪种方式，最终都可以写出能够运行的代码。

如果是**从零开始**，则可以从[使用Gradle构建项目](https://spring.io/guides/gs/managing-transactions/#scratch)小节开始。

如果想**跳过基本的设置步骤**，可以按照以下步骤执行：

- [下载](https://github.com/spring-guides/gs-soap-service/archive/master.zip) 并解压本指南相关的源文件，或者直接通过[Git](https://spring.io/understanding/Git)命令克隆到本地：

```bash
git clone https://github.com/spring-guides/gs-managing-transactions.git
```

- 进入 `gs-managing-transactions/initial` 目录
- 直接跳到[创建预定服务](https://spring.io/guides/gs/managing-transactions/#initial)小节。

**当完成所有的编码以后**，可以将你的代码与`gs-managing-transactions/complete`目录下的示例代码进行对比，以检查结果是否正确。

## 使用Gradle构建项目

首先需要设置一个基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，这里准备了在使用[Gradle](https://gradle.org/)和[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Gradle和Maven都不熟悉，可以参照[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle/)或[使用Maven构建Java项目](https://spring.io/guides/gs/gradle/)。

### 创建目录结构

选择需要创建项目的目录，参照照以下目录结构创建子目录；如果使用的是UNIX/LINUX系统，可以使用`mkdir -p src/main/java/hello`命令创建：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle 的构建文件

下面是一个[原始的Gradle build文件](https://github.com/spring-guides/gs-managing-transactions/blob/master/initial/build.gradle)。

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
    baseName = 'gs-managing-transactions'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-jdbc")
    runtime("com.h2database:h2")
}
```

 [Spring Boot gradle 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，把它作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用Maven构建项目

首先，设置基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，在这里为你提供了使用[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Maven不熟悉，可以参照[使用maven构建JAVA项目工程](https://spring.io/guides/gs/maven) 。

### 创建目录结构

选择需要创建项目的目录，参照以下目录结构创建子目录；如果使用的是UNIX/LINUX系统，可以使用`mkdir -p src/main/java/hello` 命令创建：

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
    <artifactId>gs-managing-transactions</artifactId>
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
            <scope>runtime</scope>
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

[Spring Boot maven 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用IDE构建项目

- 在Spring Tool Suite中构建项目，请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。 
- 在IntelliJ IDEA中构建项目，请参照[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)。 

## 创建录入服务

首先，使用`BookingService`类创建一个基于JDBC的服务，将人员按名字录入到系统中。

`src/main/java/hello/BookingService.java`

```java
package hello;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

@Component
public class BookingService {

    private final static Logger logger = LoggerFactory.getLogger(BookingService.class);

    private final JdbcTemplate jdbcTemplate;

    public BookingService(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Transactional
    public void book(String... persons) {
        for (String person : persons) {
            logger.info("Booking " + person + " in a seat...");
            jdbcTemplate.update("insert into BOOKINGS(FIRST_NAME) values (?)", person);
        }
    }

    public List<String> findAllBookings() {
        return jdbcTemplate.query("select FIRST_NAME from BOOKINGS",
                (rs, rowNum) -> rs.getString("FIRST_NAME"));
    }

}
```

该代码有一个自动装配的`JdbcTemplate`类，它是一个方便的模板类，完成下面的代码所需的所有数据库交互。

你拥有一个`book`方法，旨在将多个人的信息录入数据库。它循环遍历人们的列表，并且对于每个人的信息，使用`JdbcTemplate`模板类将它们插入到`BOOKINGS`数据库表中。这个方法被标记为`@Transactional`，意味着任何失败都会导致整个操作回滚到之前的状态，并重新抛出原来的异常。这意味着如果一个人的信息添加失败，那么任何人的信息都不会被添加到`BOOKINGS`数据库表中。

你还有一个`findAllBookings`方法来查询数据库。从数据库中获取的每一行都被转换成一个`String`，然后组装成一个`List`。

## 构建应用

`src/main/java/hello/Application.java`

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

`@SpringBootApplication`是一个方便的注解，它会自动添加以下所有内容：

- `@Configuration`将该类标记为应用程序上下文中bean定义的源。
- `@EnableAutoConfiguration`指示Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
- 通常，Spring MVC 应用程序会添加`@EnableWebMvc`注解，但是当Spring Boot在类路径中发现**spring-webmvc**时会自动添加该注解。通过添加该注解将应用程序标记为Web应用程序，并进行一些关键操作，比如设置`DispatcherServlet`。
- `@ComponentScan`通知Spring在`hello`包中查找其他组件，配置和服务，允许Spring扫描到控制器。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。你注意到我们没有写过一行XML代码吗？ 而且也没有**web.xml**配置文件。此Web应用程序是100%纯Java编写的，无需再配置其他基础设施。

你的应用程序实际上是零配置。Spring Boot会检测类路径和`h2`上的`spring-jdbc`，并自动为你创建一个`DataSource`和一个`JdbcTemplate`。由于这样的基础设施现在是可用的，而且你并没有专门的配置，所以也会为你创建一个`DataSourceTransactionManager`：这是侦听`@Transactional`注解方法的组件（比如`BookingService`类中的`book`方法）。`BookingService`类通过类路径扫描进行检测。

本指南中演示的另一个Spring Boot功能，[在启动时初始化模式的能力](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-initialize-a-database-using-spring-jdbc)：

`src/main/resources/schema.sql`

```sql
drop table BOOKINGS if exists;
create table BOOKINGS(ID serial, FIRST_NAME varchar(5) NOT NULL);
``

还有一个`CommandLineRunner`接口注入`BookingService`类，并展示各种事务用例。

`src/main/java/hello/AppRunner.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;

@Component
class AppRunner implements CommandLineRunner {

    private final static Logger logger = LoggerFactory.getLogger(AppRunner.class);

    private final BookingService bookingService;

    public AppRunner(BookingService bookingService) {
        this.bookingService = bookingService;
    }

    @Override
    public void run(String... args) throws Exception {
        bookingService.book("Alice", "Bob", "Carol");
        Assert.isTrue(bookingService.findAllBookings().size() == 3,
                "First booking should work with no problem");
        logger.info("Alice, Bob and Carol have been booked");
        try {
            bookingService.book("Chris", "Samuel");
        } catch (RuntimeException e) {
            logger.info("v--- The following exception is expect because 'Samuel' is too " +
                    "big for the DB ---v");
            logger.error(e.getMessage());
        }

        for (String person : bookingService.findAllBookings()) {
            logger.info("So far, " + person + " is booked.");
        }
        logger.info("You shouldn't see Chris or Samuel. Samuel violated DB constraints, " +
                "and Chris was rolled back in the same TX");
        Assert.isTrue(bookingService.findAllBookings().size() == 3,
                "'Samuel' should have triggered a rollback");

        try {
            bookingService.book("Buddy", null);
        } catch (RuntimeException e) {
            logger.info("v--- The following exception is expect because null is not " +
                    "valid for the DB ---v");
            logger.error(e.getMessage());
        }

        for (String person : bookingService.findAllBookings()) {
            logger.info("So far, " + person + " is booked.");
        }
        logger.info("You shouldn't see Buddy or null. null violated DB constraints, and " +
                "Buddy was rolled back in the same TX");
        Assert.isTrue(bookingService.findAllBookings().size() == 3,
                "'null' should have triggered a rollback");
    }

}
```

你可以使用Gradle或Maven从命令行运行应用程序。或者，你可以构建一个包含所有必需的依赖项，类和资源的可执行JAR文件，然后运行该文件。将服务作为应用程序，这使得在整个开发生命周期内即使跨越不同的环境，发布、版本化和部署等等都将变得容易。

如果你正在使用Gradle，则可以使用`./gradlew bootRun`命令运行该应用程序。或者你可以使用`./gradlew build`生成JAR文件。接下来你可以运行这个JAR文件：

```bash
java -jar build/libs/gs-managing-transactions-0.1.0.jar
```

如果你正在使用Maven，则可以使用`./mvnw spring-boot:run`命令运行该应用程序。或者你可以使用`./mvnw clean package`生成JAR文件。接下来你可以运行这个JAR文件：

```bash
java -jar target/gs-managing-transactions-0.1.0.jar
```

上面的过程将创建一个可运行的JAR文件。你也可以选择[构建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

你应该看到如下的输出：

```log
2016-09-01 09:47:55.031  INFO 16911 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executing SQL script from URL [file:/Users/foo/workspace/gs-managing-transactions/target/classes/schema.sql]
2016-09-01 09:47:55.052  INFO 16911 --- [           main] o.s.jdbc.datasource.init.ScriptUtils     : Executed SQL script from URL [file:/Users/foo/workspace/gs-managing-transactions/target/classes/schema.sql] in 21 ms.
2016-09-01 09:47:55.259  INFO 16911 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2016-09-01 09:47:55.282  INFO 16911 --- [           main] hello.BookingService                     : Booking Alice in a seat...
2016-09-01 09:47:55.292  INFO 16911 --- [           main] hello.BookingService                     : Booking Bob in a seat...
2016-09-01 09:47:55.292  INFO 16911 --- [           main] hello.BookingService                     : Booking Carol in a seat...
2016-09-01 09:47:55.306  INFO 16911 --- [           main] hello.AppRunner                          : Alice, Bob and Carol have been booked
2016-09-01 09:47:55.306  INFO 16911 --- [           main] hello.BookingService                     : Booking Chris in a seat...
2016-09-01 09:47:55.306  INFO 16911 --- [           main] hello.BookingService                     : Booking Samuel in a seat...
2016-09-01 09:47:55.312  INFO 16911 --- [           main] o.s.b.f.xml.XmlBeanDefinitionReader      : Loading XML bean definitions from class path resource [org/springframework/jdbc/support/sql-error-codes.xml]
2016-09-01 09:47:55.419  INFO 16911 --- [           main] o.s.jdbc.support.SQLErrorCodesFactory    : SQLErrorCodes loaded: [DB2, Derby, H2, HSQL, Informix, MS-SQL, MySQL, Oracle, PostgreSQL, Sybase, Hana]
2016-09-01 09:47:55.424  INFO 16911 --- [           main] hello.AppRunner                          : v--- The following exception is expect because 'Samuel' is too big for the DB ---v
2016-09-01 09:47:55.424 ERROR 16911 --- [           main] hello.AppRunner                          : PreparedStatementCallback; SQL [insert into BOOKINGS(FIRST_NAME) values (?)]; Value too long for column "FIRST_NAME VARCHAR(5) NOT NULL": "'Samuel' (6)"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [22001-192]; nested exception is org.h2.jdbc.JdbcSQLException: Value too long for column "FIRST_NAME VARCHAR(5) NOT NULL": "'Samuel' (6)"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [22001-192]
2016-09-01 09:47:55.424  INFO 16911 --- [           main] hello.AppRunner                          : So far, Alice is booked.
2016-09-01 09:47:55.424  INFO 16911 --- [           main] hello.AppRunner                          : So far, Bob is booked.
2016-09-01 09:47:55.424  INFO 16911 --- [           main] hello.AppRunner                          : So far, Carol is booked.
2016-09-01 09:47:55.424  INFO 16911 --- [           main] hello.AppRunner                          : You shouldn't see Chris or Samuel. Samuel violated DB constraints, and Chris was rolled back in the same TX
2016-09-01 09:47:55.425  INFO 16911 --- [           main] hello.BookingService                     : Booking Buddy in a seat...
2016-09-01 09:47:55.425  INFO 16911 --- [           main] hello.BookingService                     : Booking null in a seat...
2016-09-01 09:47:55.427  INFO 16911 --- [           main] hello.AppRunner                          : v--- The following exception is expect because null is not valid for the DB ---v
2016-09-01 09:47:55.427 ERROR 16911 --- [           main] hello.AppRunner                          : PreparedStatementCallback; SQL [insert into BOOKINGS(FIRST_NAME) values (?)]; NULL not allowed for column "FIRST_NAME"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [23502-192]; nested exception is org.h2.jdbc.JdbcSQLException: NULL not allowed for column "FIRST_NAME"; SQL statement:
insert into BOOKINGS(FIRST_NAME) values (?) [23502-192]
2016-09-01 09:47:55.427  INFO 16911 --- [           main] hello.AppRunner                          : So far, Alice is booked.
2016-09-01 09:47:55.427  INFO 16911 --- [           main] hello.AppRunner                          : So far, Bob is booked.
2016-09-01 09:47:55.427  INFO 16911 --- [           main] hello.AppRunner                          : So far, Carol is booked.
2016-09-01 09:47:55.427  INFO 16911 --- [           main] hello.AppRunner 
```

`BOOKINGS`表在`first_name`列上有两个约束：

- 名字不能超过五个字符。
- 名字不能为空。

前三个名字是**Alice**, **Bob**, 和**Carol**。应用程序断言有三个人被添加到该表中。如果断言没有成功，应用程序将会提前退出。

接下来，是对**Chris**和**Samuel**的录入。Samuel的名字故意太长，强制遇到插入错误。根据事务行为的规定，Chris和Samuel都不能被插入到数据库表中，也就是说，这个事务应该被回滚。因此，正如这个断言所展示的，这个数据库表中应该只有三个人的信息。

最后，**Buddy**和**null**被录入。如输出所示，null也会导致回滚，因此数据库表中只留下前三个人的信息。

## 总结

恭喜！你已使用Spring开发了一个包含非侵入事务的简单JDBC应用程序。

## 了解更多

以下指南也可能有所帮助：

- [使用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)
- [使用JPA访问数据](https://spring.io/guides/gs/accessing-data-jpa/)
- [使用MongoDB访问数据](https://spring.io/guides/gs/accessing-data-mongodb/)
- [使用MySQL访问数据](https://spring.io/guides/gs/accessing-data-mysql/)

想写一个新的指南或贡献一个已有的？ 请查看我们的[贡献准则](https://github.com/spring-guides/getting-started-guides/wiki)。


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
