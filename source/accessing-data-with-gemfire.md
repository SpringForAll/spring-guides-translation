【chenzhijun翻译中】

> 原文：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) （这里为示例，译者需根据具体文章修改）
>
> 译者：[chenzhijun](http://github.com/chenzhijun)
>
> 校对：

# Accessing Data with GemFire

This guide walks you through the process of building an application with GemFire’s data fabric.

## What you’ll build

You will use the powerful Spring Data GemFire library to store and retrieve [POJOs](https://spring.io/understanding/POJO).

## What you’ll need

- About 15 minutes
- A favorite text editor or IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- You can also import the code straight into your IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## How to complete this guide

Like most Spring [Getting Started guides](https://spring.io/guides), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Build with Gradle](https://spring.io/guides/gs/accessing-data-gemfire/#scratch).

To **skip the basics**, do the following:

- [Download](https://github.com/spring-guides/gs-accessing-data-gemfire/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone https://github.com/spring-guides/gs-accessing-data-gemfire.git`
- cd into `gs-accessing-data-gemfire/initial`
- Jump ahead to [Define a simple entity](https://spring.io/guides/gs/accessing-data-gemfire/#initial).

**When you’re finished**, you can check your results against the code in `gs-accessing-data-gemfire/complete`.

## Build with Gradle

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) is included here. If you’re not familiar with either, refer to [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) or [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

```
└── src
    └── main
        └── java
            └── hello
```

### Create a Gradle build file

Below is the [initial Gradle build file](https://github.com/spring-guides/gs-accessing-data-gemfire/blob/master/initial/build.gradle).

`build.gradle`

```
buildscript {
    repositories {
        mavenCentral()
        maven { url "https://repo.spring.io/libs-release" }

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
    baseName = 'gs-accessing-data-gemfire'
    version = '0.1.0'
}

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-release" }
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-gemfire")
    runtime("org.springframework.shell:spring-shell:1.0.0.RELEASE")
}
```

The [Spring Boot gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) provides many convenient features:

- It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
- It searches for the `public static void main()` method to flag as a runnable class.
- It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.

## Build with Maven

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven](https://maven.apache.org/) is included here. If you’re not familiar with Maven, refer to [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

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
    <artifactId>gs-accessing-data-gemfire</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
        <spring-shell.version>1.0.0.RELEASE</spring-shell.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-gemfire</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.shell</groupId>
            <artifactId>spring-shell</artifactId>
            <version>${spring-shell.version}</version>
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

    <repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>

</project>
```

The [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) provides many convenient features:

- It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
- It searches for the `public static void main()` method to flag as a runnable class.
- It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.

## Build with your IDE

- Read how to import this guide straight into [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- Read how to work with this guide in [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea).

## Define a simple entity

GemFire is a data fabric. It maps data into regions, and it’s possible to configure distributed regions across multiple nodes. However, for this guide you use a local region so you don’t have to set up anything extra.

In this example, you store Person objects with a few annotations.

`src/main/java/hello/Person.java`

```java
package hello;

import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.PersistenceConstructor;
import org.springframework.data.gemfire.mapping.Region;

@Region("hello")
public class Person {

    @Id
    public String name;
    public int age;

    @PersistenceConstructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return name + " is " + age + " years old.";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

Here you have a `Person` class with two attributes, the `name` and the `age`. You also have a single constructor to populate the entities when creating a new instance.

Notice that this class is annotated `@Region("hello")`. When GemFire stores the class, a new entry is created inside that specific region. This class also has `name` marked with `@Id`. This is for internal usage to help GemFire track the data.

The next important piece is the person’s age. Later in this guide, you will use it to fashion some queries.

The convenient `toString()` method will print out the person’s name and age.

## Create simple queries

Spring Data GemFire focuses on storing data in GemFire. It also inherits powerful functionality from the Spring Data Commons project, such as the ability to derive queries. Essentially, you don’t have to learn the query language of GemFire; you can simply write a handful of methods and the queries are written for you.

To see how this works, create an interface that queries `Person` nodes.

`src/main/java/hello/PersonRepository.java`

```java
package hello;

import org.springframework.data.repository.CrudRepository;

public interface PersonRepository extends CrudRepository<Person, String> {

    Person findByName(String name);

    Iterable<Person> findByAgeGreaterThan(int age);

    Iterable<Person> findByAgeLessThan(int age);

    Iterable<Person> findByAgeGreaterThanAndAgeLessThan(int age1, int age2);
}
```

`PersonRepository` extends the `CrudRepository` interface and plugs in the type of values and keys it works with: `Person` and `String`. Out-of-the-box, this interface comes with many operations, including standard CRUD (create-read-update-delete).

You can define other queries as needed by simply declaring their method signature. In this case, you add `findByName`, which essentially seeks nodes of type `Person` and find the one that matches on `name`.

You also have:

- `findByAgeGreaterThan` to find people above a certain age
- `findByAgeLessThan` to find people below a certain age
- `findByAgeGreaterThanAndAgeLessThan` to find people in a certain range

Let’s wire this up and see what it looks like!

## Create an application class

Here you create an Application class with all the components.

`src/main/java/hello/Application.java`

```java
package hello;

import java.io.IOException;
import java.util.Properties;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.gemfire.CacheFactoryBean;
import org.springframework.data.gemfire.LocalRegionFactoryBean;
import org.springframework.data.gemfire.repository.config.EnableGemfireRepositories;

import com.gemstone.gemfire.cache.GemFireCache;

@Configuration
@EnableGemfireRepositories
@SuppressWarnings("unused")
public class Application implements CommandLineRunner {

    @Bean
    Properties gemfireProperties() {
        Properties gemfireProperties = new Properties();
        gemfireProperties.setProperty("name", "DataGemFireApplication");
        gemfireProperties.setProperty("mcast-port", "0");
        gemfireProperties.setProperty("log-level", "config");
        return gemfireProperties;
    }

    @Bean
    CacheFactoryBean gemfireCache() {
        CacheFactoryBean gemfireCache = new CacheFactoryBean();
        gemfireCache.setClose(true);
        gemfireCache.setProperties(gemfireProperties());
        return gemfireCache;
    }

    @Bean
    LocalRegionFactoryBean<String, Person> helloRegion(final GemFireCache cache) {
        LocalRegionFactoryBean<String, Person> helloRegion = new LocalRegionFactoryBean<>();
        helloRegion.setCache(cache);
        helloRegion.setClose(false);
        helloRegion.setName("hello");
        helloRegion.setPersistent(false);
        return helloRegion;
    }

    @Autowired
    PersonRepository personRepository;

    @Override
    public void run(String... strings) throws Exception {
        Person alice = new Person("Alice", 40);
        Person bob = new Person("Baby Bob", 1);
        Person carol = new Person("Teen Carol", 13);

        System.out.println("Before linking up with Gemfire...");
        for (Person person : new Person[] { alice, bob, carol }) {
            System.out.println("\t" + person);
        }

        personRepository.save(alice);
        personRepository.save(bob);
        personRepository.save(carol);

        System.out.println("Lookup each person by name...");
        for (String name : new String[] { alice.name, bob.name, carol.name }) {
            System.out.println("\t" + personRepository.findByName(name));
        }

        System.out.println("Adults (over 18):");
        for (Person person : personRepository.findByAgeGreaterThan(18)) {
            System.out.println("\t" + person);
        }

        System.out.println("Babies (less than 5):");
        for (Person person : personRepository.findByAgeLessThan(5)) {
            System.out.println("\t" + person);
        }

        System.out.println("Teens (between 12 and 20):");
        for (Person person : personRepository.findByAgeGreaterThanAndAgeLessThan(12, 20)) {
            System.out.println("\t" + person);
        }
    }

    public static void main(String[] args) throws IOException {
        SpringApplication application = new SpringApplication(Application.class);
        application.setWebEnvironment(false);
        application.run(args);
    }

}
```

In the configuration, you need to add the `@EnableGemFireRepositories` annotation.

- By default, `@EnableGemfireRepostories` will scan the current package for any interfaces that extend one of Spring Data’s repository interfaces. Use it’s `basePackageClasses=MyRepository.class` to safely tell Spring Data GemFire to scan a different root package by type.

A GemFire cache is required, to store all data. For that, you have Spring Data GemFire’s convenient `CacheFactoryBean`.

| **   | In this guide, the cache is created locally using built-in components and an evaluation license. For a production solution, Spring recommends the production version of GemFire, where you can create distributed caches and regions across multiple nodes. |
| ---- | ---------------------------------------- |
|      |                                          |

Remember how you tagged `Person` to be stored in `@Region("hello")`? You define that region here with `LocalRegionFactoryBean<String, Person>`. You need to inject an instance of the cache you just defined while also naming it `hello`.

| **   | The types are `<String, Person>`, matching the key type (`String`) with the value type (`Person`). |
| ---- | ---------------------------------------- |
|      |                                          |

The `public static void main` uses Spring Boot’s `SpringApplication.run()` to launch the application and invoke the `CommandLineRunner` that builds the relationships.

The application autowires an instance of `PersonRepository` that you just defined. Spring Data GemFire will dynamically create a concrete class that implements that interface and will plug in the needed query code to meet the interface’s obligations. This repository instance is the used by the `run()` method to demonstrate the functionality.

## Store and fetch data

In this guide, you are creating three local `Person` s, **Alice**, **Baby Bob**, and **Teen Carol**. Initially, they only exist in memory. After creating them, you have to save them to GemFire.

Now you run several queries. The first looks up everyone by name. Then you execute a handful of queries to find adults, babies, and teens, all using the age attribute. With the logging turned up, you can see the queries Spring Data GemFire writes on your behalf.

## Build an executable JAR

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using `./gradlew bootRun`. Or you can build the JAR file using `./gradlew build`. Then you can run the JAR file:

```shell
java -jar build/libs/gs-accessing-data-gemfire-0.1.0.jar
```

If you are using Maven, you can run the application using `./mvnw spring-boot:run`. Or you can build the JAR file with `./mvnw clean package`. Then you can run the JAR file:

```shell
java -jar target/gs-accessing-data-gemfire-0.1.0.jar
```

| **   | The procedure above will create a runnable JAR. You can also opt to [build a classic WAR file](https://spring.io/guides/gs/convert-jar-to-war/)instead. |
| ---- | ---------------------------------------- |
|      |                                          |

You should see something like this (with other stuff like queries as well):

```
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

## Summary

Congratulations! You set up an embedded GemFire server, stored simple entities, and developed quick queries.

## See Also

The following guides may also be helpful:

- [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
