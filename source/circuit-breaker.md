【ligang翻译中】

> 原文：[Circuit Breaker](https://spring.io/guides/gs/circuit-breaker/)
>
> 译者：[ligang](http://github.com/holy12345/)
>
> 校对：

# Circuit Breaker

This guide walks you through the process of applying circuit breakers to potentially-failing method calls using the Netflix Hystrix fault tolerance library.

## What you’ll build

You’ll build a microservice application that uses the [Circuit Breaker pattern](http://martinfowler.com/bliki/CircuitBreaker.html) to gracefully degrade functionality when a method call fails. Use of the Circuit Breaker pattern can allow a microservice to continue operating when a related service fails, preventing the failure from cascading and giving the failing service time to recover.

## What you’ll need

- About 15 minutes
- A favorite text editor or IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- You can also import the code straight into your IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## How to complete this guide

Like most Spring [Getting Started guides](https://spring.io/guides), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Build with Gradle](https://spring.io/guides/gs/circuit-breaker/#scratch).

To **skip the basics**, do the following:

- [Download](https://github.com/spring-guides/gs-circuit-breaker/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone https://github.com/spring-guides/gs-circuit-breaker.git`
- cd into `gs-circuit-breaker/initial`
- Jump ahead to [Set up a server microservice application](https://spring.io/guides/gs/circuit-breaker/#initial).

**When you’re finished**, you can check your results against the code in `gs-circuit-breaker/complete`.

## Build with Gradle

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) is included here. If you’re not familiar with either, refer to [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) or [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

```
└── src
    └── main
        └── java
            └── hello
```

### Create a Gradle build file

Below is the [initial Gradle build file](https://github.com/spring-guides/gs-circuit-breaker/blob/master/initial/build.gradle).

`bookstore/build.gradle`

```
buildscript {
	ext {
		springBootVersion = '1.5.2.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'bookstore'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}


eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}
```

`reading/build.gradle`

```
buildscript {
	ext {
		springBootVersion = '1.5.2.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
	baseName = 'reading'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-hystrix')
	compile('org.springframework.boot:spring-boot-starter-web')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:Camden.SR5"
	}
}


eclipse {
	classpath {
		 containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
		 containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
	}
}
```

The [Spring Boot gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) provides many convenient features:

- It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
- It searches for the `public static void main()` method to flag as a runnable class.
- It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.



## Build with Maven

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven](https://maven.apache.org/) is included here. If you’re not familiar with Maven, refer to [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

```
└── src
    └── main
        └── java
            └── hello
```

`bookstore/pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>bookstore</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
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

`reading/pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>reading</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Camden.SR5</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

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

The [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) provides many convenient features:

- It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
- It searches for the `public static void main()` method to flag as a runnable class.
- It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.



## Build with your IDE

- Read how to import this guide straight into [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- Read how to work with this guide in [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea).

## Set up a server microservice application

The Bookstore service will have a single endpoint. It will be accessible at `/recommended`, and will (for simplicity) return a `String` recommended reading list.

Edit our main class, in `BookstoreApplication.java`. It should look like this:

`bookstore/src/main/java/hello/BookstoreApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
@SpringBootApplication
public class BookstoreApplication {

  @RequestMapping(value = "/recommended")
  public String readingList(){
    return "Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)";
  }

  public static void main(String[] args) {
    SpringApplication.run(BookstoreApplication.class, args);
  }
}
```

The `@RestController` annotation marks `BookstoreApplication` as a controller class, like `@Controller` does, and also ensures that `@RequestMapping` methods in this class will behave as though annotated with `@ResponseBody`. That is, the return values of `@RequestMapping`methods in this class will be automatically converted appropriately from their original types and will be written directly to the response body.

We’re going to run this application locally alongside a client service application, so in `src/main/resources/application.properties`, set `server.port` so that the Bookstore service won’t conflict with the client when we get that running.

`bookstore/src/main/resources/application.properties`

```
server.port=8090
```

## Set up a client microservice application

The Reading application will be our front-end (as it were) to the Bookstore application. We’ll be able to view our reading list there at `/to-read`, and that reading list will be retrieved from the Bookstore service application.

`reading/src/main/java/hello/ReadingApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.client.RestTemplate;
import java.net.URI;

@RestController
@SpringBootApplication
public class ReadingApplication {

  @RequestMapping("/to-read")
  public String readingList() {
    RestTemplate restTemplate = new RestTemplate();
    URI uri = URI.create("http://localhost:8090/recommended");

    return restTemplate.getForObject(uri, String.class);
  }

  public static void main(String[] args) {
    SpringApplication.run(ReadingApplication.class, args);
  }
}
```

To get the list from Bookstore, we’re using Spring’s `RestTemplate` template class. `RestTemplate` makes an HTTP GET request to the Bookstore service’s URL as we provide it and then returns the result as a `String`. (For more information on using Spring to consume a RESTful service, see the [Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/) guide.)

Add the `server.port` property to `src/main/resources/application.properties`:

`reading/src/main/resources/application.properties`

```
server.port=8080
```

We now can access, in a browser, the `/to-read` endpoint on our Reading application, and see our reading list. Yet since we rely on the Bookstore application, if anything happens to it, or if Reading is simply unable to access Bookstore, we’ll have no list and our users will get a nasty HTTP 500 error message.

## Apply the Circuit Breaker pattern

Netflix’s Hystrix library provides an implementation of the Circuit Breaker pattern: when we apply a circuit breaker to a method, Hystrix watches for failing calls to that method, and if failures build up to a threshold, Hystrix opens the circuit so that subsequent calls automatically fail. While the circuit is open, Hystrix redirects calls to the method, and they’re passed on to our specified fallback method.

Spring Cloud Netflix Hystrix looks for any method annotated with the `@HystrixCommand`annotation, and wraps that method in a proxy connected to a circuit breaker so that Hystrix can monitor it. This currently only works in a class marked with `@Component` or `@Service`, so in the Reading application, under `src/main/java/hello`, add a new class: `BookService`.

The `RestTemplate` will be injected into the constructor of the `BookService` when it is created. The complete class should look like this:

`reading/src/main/java/hello/BookService.java`

```java
package hello;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.net.URI;

@Service
public class BookService {

  private final RestTemplate restTemplate;

  public BookService(RestTemplate rest) {
    this.restTemplate = rest;
  }

  @HystrixCommand(fallbackMethod = "reliable")
  public String readingList() {
    URI uri = URI.create("http://localhost:8090/recommended");

    return this.restTemplate.getForObject(uri, String.class);
  }

  public String reliable() {
    return "Cloud Native Java (O'Reilly)";
  }

}
```

We’ve applied `@HystrixCommand` to our original `readingList()` method. We also have a new method here: `reliable()`. The `@HystrixCommand` annotation has `reliable` as its `fallbackMethod`, so if, for some reason, Hystrix opens the circuit on `readingList()`, we’ll have an excellent (if short) placeholder reading list ready for our users.

In our main class, `ReadingApplication`, we will create a `RestTemplate` bean, inject the `BookService` and call it for our reading list:

`reading/src/main/java/hello/ReadingApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.web.client.RestTemplate;

@EnableCircuitBreaker
@RestController
@SpringBootApplication
public class ReadingApplication {

  @Autowired
  private BookService bookService;

  @Bean
  public RestTemplate rest(RestTemplateBuilder builder) {
    return builder.build();
  }

  @RequestMapping("/to-read")
  public String toRead() {
    return bookService.readingList();
  }

  public static void main(String[] args) {
    SpringApplication.run(ReadingApplication.class, args);
  }
}
```

Now, to retrieve the list from the Bookstore service, we call `bookService.readingList()`. You’ll also notice that we’ve added one last annotation, `@EnableCircuitBreaker`; that’s necessary to tell Spring Cloud that the Reading application uses circuit breakers and to enable their monitoring, opening, and closing (behavior supplied, in our case, by Hystrix).

## Try it out

Run both the Bookstore service and the Reading service, and then open a browser to the Reading service, at `localhost:8080/to-read`. You should see the complete recommended reading list:

```
Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)
```

Now shut down the Bookstore application. Our list source is gone, but thanks to Hystrix and Spring Cloud Netflix, we have a reliable abbreviated list to stand in the gap; you should see:

```
Cloud Native Java (O'Reilly)
```

## Summary

Congratulations! You’ve just developed a Spring application that uses the Circuit Breaker pattern to protect against cascading failures and to provide fallback behavior for potentially failing calls.

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

