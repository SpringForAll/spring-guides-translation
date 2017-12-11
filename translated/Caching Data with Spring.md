# 使用 Spring 缓存数据

> 原文：[Caching Data with Spring](https://spring.io/guides/gs/caching/)
>
> 译者：[feilangrenM](https://github.com/feilangrenM)
>
> 校对：

本指南将帮助你了解怎样对一个Spring管理的bean进行缓存。

## 你将创建什么

你将构建一个应用程序，在程序里对一个简单的书库进行缓存。

## 你需要准备什么

- 大约15分钟
- 一个自己喜欢的编辑器或者IDE


- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 及以上版本


- [Gradle 2.3+](https://gradle.org/install/) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)


- 你也可以直接将代码导入到本地的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 怎样使用本指南

与大多数[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。不管采用哪种方式，最终都可以获得能够运行的代码。

如果是**从零开始**，则可以从[使用Gradle构建项目](#使用gradle构建项目)小节开始。

如果想**跳过基本的设置步骤**，可以按照以下步骤执行：

- [下载](https://github.com/spring-guides/gs-soap-service/archive/master.zip) 并解压本指南相关的源文件，或者直接通过[Git](https://spring.io/understanding/Git)命令克隆到本地：

  ```
  git clone https://github.com/spring-guides/gs-caching.git
  ```


- 进入 `gs-caching/initial` 目录
- 直接跳到[创建一个书库](#创建一个书库)小节。

**当完成所有的编码以后**，可以将你的代码与`gs-caching/complete`目录下的示例代码进行对比，以检查结果是否正确。

## 使用Gradle构建项目

首先需要设置一个基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，这里准备了在使用[Gradle](https://gradle.org/)和[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Gradle和Maven都不熟悉，可以参照[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle/)或[使用Maven构建Java项目](https://spring.io/guides/gs/gradle/)。

### 创建目录结构

选择需要创建项目的目录，参照照以下目录结构创建子目录；如果使用的是UNIX系统，可以使用`mkdir -p src/main/java/hello`命令创建：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle 的构建文件

下面是一个[原始的Gradle build文件](https://github.com/spring-guides/gs-producing-web-service/blob/master/initial/build.gradle)。

`build.gradle`

```groovy
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
    baseName = 'gs-producing-web-service'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
}
```

 [Spring Boot gradle 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，把它作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用Maven构建项目

首先，设置基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，在这里为你提供了使用[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Maven不熟悉，可以参照[使用maven构建JAVA项目工程](https://spring.io/guides/gs/maven) 。

### 创建目录结构

选择需要创建项目的目录，参照以下目录结构创建子目录；如果使用的是UNIX系统，可以使用`mkdir -p src/main/java/hello` 命令创建：

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
    <artifactId>gs-producting-web-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>

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

## 创建一个书库

首先，我们将书抽象为一个简单的模型。

`src/main/java/hello/Book.java`

```java
package hello;

public class Book {

    private String isbn;
    private String title;

    public Book(String isbn, String title) {
        this.isbn = isbn;
        this.title = title;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public String toString() {
        return "Book{" + "isbn='" + isbn + '\'' + ", title='" + title + '\'' + '}';
    }

}
```

然后，为上述模型创建一个书库：

`src/main/java/hello/BookRepository.java`

```java
package hello;

public interface BookRepository {

    Book getByIsbn(String isbn);

}
```

你可以使用[Spring Data](https://projects.spring.io/spring-data/)在不同的SQL或者NoSQL存储系统中创建你的书库，但是在本指南中，我们将使用一个简单的实现来模拟数据访问的延迟（网络服务，缓慢的延迟...等等）。

`src/main/java/hello/SimpleBookRepository.java`

```java
package hello;

import org.springframework.stereotype.Component;

@Component
public class SimpleBookRepository implements BookRepository {

    @Override
    public Book getByIsbn(String isbn) {
        simulateSlowService();
        return new Book(isbn, "Some book");
    }

    // Don't do this at home
    private void simulateSlowService() {
        try {
            long time = 3000L;
            Thread.sleep(time);
        } catch (InterruptedException e) {
            throw new IllegalStateException(e);
        }
    }

}
```

`simulateSlowService`故意在每个`getByIsbn`调用中增加三秒的延迟。在后面的例子中，你可以使用缓存进行加速。

## 使用书库

接下来，连接书库，并使用它来访问一些书籍。

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

`@SpringBootApplication` 是一个方便的注解，它会自动添加以下所有内容：

- `@Configuration`将该类标记为应用程序上下文中bean定义的源。


- `@EnableAutoConfiguration`指示Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。


- 通常，Spring MVC 应用程序会添加`@EnableWebMvc`注解，但是当Spring Boot在类路径中发现**spring-webmvc**时会自动添加该注解。通过添加该注解将应用程序标记为Web应用程序，并进行一些关键操作，比如设置`DispatcherServlet`。
- `@ComponentScan`通知Spring在hello包中查找其他组件，配置和服务，允许Spring扫描到控制器。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。 你注意到我们没有写过一行XML代码吗？ 而且也没有**web.xml**配置文件。 此Web应用程序是100%纯Java编写的，无需再配置其他基础设施。

此外，还需要创建一个`CommandLineRunner`来注入`BookRepository`并用不同的参数对它多次调用。

`src/main/java/hello/AppRunner.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class AppRunner implements CommandLineRunner {

    private static final Logger logger = LoggerFactory.getLogger(AppRunner.class);

    private final BookRepository bookRepository;

    public AppRunner(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    @Override
    public void run(String... args) throws Exception {
        logger.info(".... Fetching books");
        logger.info("isbn-1234 -->" + bookRepository.getByIsbn("isbn-1234"));
        logger.info("isbn-4567 -->" + bookRepository.getByIsbn("isbn-4567"));
        logger.info("isbn-1234 -->" + bookRepository.getByIsbn("isbn-1234"));
        logger.info("isbn-4567 -->" + bookRepository.getByIsbn("isbn-4567"));
        logger.info("isbn-1234 -->" + bookRepository.getByIsbn("isbn-1234"));
        logger.info("isbn-1234 -->" + bookRepository.getByIsbn("isbn-1234"));
    }

}
```

如果此时你尝试运行程序，你会发现即使多次检索同一本书，速度也非常慢。

```
2014-06-05 12:15:35.783  ... : .... Fetching books
2014-06-05 12:15:40.783  ... : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
2014-06-05 12:15:43.784  ... : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
2014-06-05 12:15:46.786  ... : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
```

从时间戳中可以看出，即使是反复查找同一个标题的书籍，每本书仍需要大概三秒钟才能找出。

## 开启缓存

接下来，让我们在`SimpleBookRepository`上启用缓存，以便查找的书籍可以被加入到`books`缓存中。

`src/main/java/hello/SimpleBookRepository.java`

```java
package hello;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Component;

@Component
public class SimpleBookRepository implements BookRepository {

    @Override
    @Cacheable("books")
    public Book getByIsbn(String isbn) {
        simulateSlowService();
        return new Book(isbn, "Some book");
    }

    // Don't do this at home
    private void simulateSlowService() {
        try {
            long time = 3000L;
            Thread.sleep(time);
        } catch (InterruptedException e) {
            throw new IllegalStateException(e);
        }
    }

}
```

现在，需要在启动类中开启缓存注解。

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

`@EnableCaching`注解会触发一个后置处理器，它会检查每个Spring bean是否存在public方法上添加了缓存注解。 如果找到了缓存注解，则自动创建一个代理来拦截方法调用，并进行相应的缓存处理。

后置处理器管理的注解有`Cacheable`，`CachePut`和`CacheEvict`。想了解更多细节， 你可以参考javadocs和[相关文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#cache)。

Spring Boot会自动配置一个合适的`CacheManager`作为相关缓存的提供者。有关更多详细信息，请参阅[Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-caching.html)。

在上述示例中没有使用特定的缓存库，因此缓存存储是使用`ConcurrentHashMap`简单实现的。 Spring的缓存抽象可以很好地支持不同的缓存库，完全符合JSR-107（JCache）规范。

## 构建可执行的JAR

程序创建好以后，可以使用Gradle或Maven从命令行运行。或者，也可以将所有必需的依赖项，类和资源打包成一个可执行的JAR文件，并运行该文件。这种方式使得在整个开发生命周期中，应用程序可以轻松地发布，更新版本和部署服务。

如果你使用的是Gradle，则可以使用`./gradlew bootRun`运行应用程序。或者使用`./gradlew build`来构建JAR文件。然后运行：

```
java -jar build/libs/gs-caching-0.1.0.jar
```

如果你使用的是Maven，可以使用./mvnw spring-boot:run运行应用程序，或者使用./mvnw clean package来构建JAR文件。然后运行JAR文件：

```
java -jar target/gs-caching-0.1.0.jar
```

> 上面的过程创建了一个可运行的JAR。也可以选择[构建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

## 测试应用程序

现在缓存已经开启，你可以再次执行程序，通过设置相同或者不同的`isbn`进行多次尝试。 你会发现开启缓存以后，程序的执行结果会有很大的不同。

```
2016-09-01 11:12:47.033  .. : .... Fetching books
2016-09-01 11:12:50.039  .. : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
2016-09-01 11:12:53.044  .. : isbn-4567 -->Book{isbn='isbn-4567', title='Some book'}
2016-09-01 11:12:53.045  .. : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
2016-09-01 11:12:53.045  .. : isbn-4567 -->Book{isbn='isbn-4567', title='Some book'}
2016-09-01 11:12:53.045  .. : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
2016-09-01 11:12:53.045  .. : isbn-1234 -->Book{isbn='isbn-1234', title='Some book'}
```

控制台的摘录显示，第一次获取书籍，每个标题花了三秒钟，但是随后的调用几乎是一瞬间就完成了。

## 小结

恭喜！你刚刚在一个Spring管理的bean上开启了缓存。

想要写一个新的指南或者为现有的一个贡献力量吗？请查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。