【ligang翻译中】

> 原文：[Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
>
> 译者：[ligang](http://github.com/holy12345/)
>
> 校对：

# Accessing Data with MongoDB

This guide walks you through the process of using [Spring Data MongoDB](https://projects.spring.io/spring-data-mongodb/) to build an application that stores data in and retrieves it from [MongoDB](https://www.mongodb.org/), a document-based database.

## What you’ll build

You will store `Customer` [POJOs](https://spring.io/understanding/POJO) in a MongoDB database using Spring Data MongoDB.

## What you’ll need

- About 15 minutes
- A favorite text editor or IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- You can also import the code straight into your IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## How to complete this guide

Like most Spring [Getting Started guides](https://spring.io/guides), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Build with Gradle](https://spring.io/guides/gs/accessing-data-mongodb/#scratch).

To **skip the basics**, do the following:

- [Download](https://github.com/spring-guides/gs-accessing-data-mongodb/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone https://github.com/spring-guides/gs-accessing-data-mongodb.git`
- cd into `gs-accessing-data-mongodb/initial`
- Jump ahead to [Install and launch MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/#initial).

**When you’re finished**, you can check your results against the code in `gs-accessing-data-mongodb/complete`.

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
###Create a Gradle build file

Below is the [initial Gradle build file](https://github.com/spring-guides/gs-accessing-data-mongodb/blob/master/initial/build.gradle).

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
    baseName = 'gs-accessing-data-mongodb'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-mongodb")
    testCompile("org.springframework.boot:spring-boot-starter-test")
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
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>org.springframework</groupId>
    <artifactId>gs-accessing-data-mongodb</artifactId>
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
    
    <repositories>
        <repository>
            <id>spring-releases</id>
            <name>Spring Releases</name>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <name>Spring Releases</name>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```
The [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) provides many convenient features:

- It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
- It searches for the `public static void main()` method to flag as a runnable class.
- It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.

## Build with your IDE
- Read how to import this guide straight into [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- Read how to work with this guide in [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea).

## Install and launch MongoDB

With your project set up, you can install and launch the MongoDB database.

If you are using a Mac with homebrew, this is as simple as:

```
$ brew install mongodb
```

With MacPorts:

```
$ port install mongodb
```

For other systems with package management, such as Redhat, Ubuntu, Debian, CentOS, and Windows, see instructions at [http://docs.mongodb.org/manual/installation/](http://docs.mongodb.org/manual/installation/).

After you install MongoDB, launch it in a console window. This command also starts up a server process.

```
$ mongod
```

You probably won’t see much more than this:

```
all output going to: /usr/local/var/log/mongodb/mongo.log
```

## Define a simple entity

MongoDB is a NoSQL document store. In this example, you store `Customer` objects.

`src/main/java/hello/Customer.java`

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

Here you have a `Customer` class with three attributes, `id`, `firstName`, and `lastName`. The `id` is mostly for internal use by MongoDB. You also have a single constructor to populate the entities when creating a new instance.

| **   | In this guide, the typical getters and setters have been left out for brevity. |
| ---- | ---------------------------------------- |
|      |                                          |

`id` fits the standard name for a MongoDB id so it doesn’t require any special annotation to tag it for Spring Data MongoDB.

The other two properties, `firstName` and `lastName`, are left unannotated. It is assumed that they’ll be mapped to fields that share the same name as the properties themselves.

The convenient `toString()` method will print out the details about a customer.

| **   | MongoDB stores data in collections. Spring Data MongoDB will map the class `Customer` into a collection called *customer*. If you want to change the name of the collection, you can use Spring Data MongoDB’s [`@Document`](https://docs.spring.io/spring-data/data-mongodb/docs/current/api/org/springframework/data/mongodb/core/mapping/Document.html) annotation on the class. |
| ---- | ---------------------------------------- |
|      |                                          |

## Create simple queries

Spring Data MongoDB focuses on storing data in MongoDB. It also inherits functionality from the Spring Data Commons project, such as the ability to derive queries. Essentially, you don’t have to learn the query language of MongoDB; you can simply write a handful of methods and the queries are written for you.

To see how this works, create a repository interface that queries `Customer` documents.

`src/main/java/hello/CustomerRepository.java`

```java
package hello;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface CustomerRepository extends MongoRepository<Customer, String> {

    public Customer findByFirstName(String firstName);
    public List<Customer> findByLastName(String lastName);

}
```

`CustomerRepository` extends the `MongoRepository` interface and plugs in the type of values and id it works with: `Customer` and `String`. Out-of-the-box, this interface comes with many operations, including standard CRUD operations (create-read-update-delete).

You can define other queries as needed by simply declaring their method signature. In this case, you add `findByFirstName`, which essentially seeks documents of type `Customer` and finds the one that matches on `firstName`.

You also have `findByLastName` to find a list of people by last name.

In a typical Java application, you write a class that implements `CustomerRepository` and craft the queries yourself. What makes Spring Data MongoDB so useful is the fact that you don’t have to create this implementation. Spring Data MongoDB creates it on the fly when you run the application.

Let’s wire this up and see what it looks like!

## Create an Application class

Here you create an Application class with all the components.

`src/main/java/hello/Application.java`

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

`@SpringBootApplication` is a convenience annotation that adds all of the following:

- `@Configuration` tags the class as a source of bean definitions for the application context.
- `@EnableAutoConfiguration` tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
- Normally you would add `@EnableWebMvc` for a Spring MVC app, but Spring Boot adds it automatically when it sees **spring-webmvc** on the classpath. This flags the application as a web application and activates key behaviors such as setting up a `DispatcherServlet`.
- `@ComponentScan` tells Spring to look for other components, configurations, and services in the `hello` package, allowing it to find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there wasn’t a single line of XML? No **web.xml** file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

Spring Boot will handle those repositories automatically as long as they are included in the same package (or a sub-package) of your `@SpringBootApplication` class. For more control over the registration process, you can use the `@EnableMongoRepositories` annotation.

| **   | By default, `@EnableMongoRepositories` will scan the current package for any interfaces that extend one of Spring Data’s repository interfaces. Use it’s `basePackageClasses=MyRepository.class` to safely tell Spring Data MongoDB to scan a different root package by type if your project layout has multiple projects and its not finding your repositories. |
| ---- | ---------------------------------------- |
|      |                                          |

Spring Data MongoDB uses the `MongoTemplate` to execute the queries behind your `find*`methods. You can use the template yourself for more complex queries, but this guide doesn’t cover that.

`Application` includes a `main()` method that autowires an instance of `CustomerRepository`: Spring Data MongoDB dynamically creates a proxy and injects it there. We use the `CustomerRepository` through a few tests. First, it saves a handful of `Customer` objects, demonstrating the `save()` method and setting up some data to work with. Next, it calls `findAll()` to fetch all `Customer` objects from the database. Then it calls `findByFirstName()`to fetch a single `Customer` by her first name. Finally, it calls `findByLastName()` to find all customers whose last name is "Smith".

| **   | Spring Boot by default attempts to connect to a locally hosted instance of MongoDB. Read the [reference docs](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-mongodb) for details on pointing your app to an instance of MongoDB hosted elsewhere. |
| ---- | ---------------------------------------- |
|      |                                          |

## Build an executable JAR

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using `./gradlew bootRun`. Or you can build the JAR file using `./gradlew build`. Then you can run the JAR file:

```shell
java -jar build/libs/gs-accessing-data-mongodb-0.1.0.jar
```

If you are using Maven, you can run the application using `./mvnw spring-boot:run`. Or you can build the JAR file with `./mvnw clean package`. Then you can run the JAR file:

```shell
java -jar target/gs-accessing-data-mongodb-0.1.0.jar
```

| **   | The procedure above will create a runnable JAR. You can also opt to [build a classic WAR file](https://spring.io/guides/gs/convert-jar-to-war/)instead. |
| ---- | ---------------------------------------- |
|      |                                          |

As our `Application` implements `CommandLineRunner`, the `run` method is invoked automatically when boot starts. You should see something like this (with other stuff like queries as well):

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

## Summary

Congratulations! You set up a MongoDB server and wrote a simple application that uses Spring Data MongoDB to save objects to and fetch them from a database — all without writing a concrete repository implementation.

| **   | If you’re interesting in exposing MongoDB repositories with a hypermedia-based RESTful front end with little effort, you might want to read [Accessing MongoDB Data with REST](https://spring.io/guides/gs/accessing-mongodb-data-rest). |
| ---- | ---------------------------------------- |
|      |                                          |

## See Also

The following guides may also be helpful:

- [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Accessing Data with Gemfire](https://spring.io/guides/gs/accessing-data-gemfire/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/)

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines](https://github.com/spring-guides/getting-started-guides/wiki).

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

