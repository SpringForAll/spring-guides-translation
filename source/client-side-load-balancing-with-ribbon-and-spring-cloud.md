【ligang翻译中】

> 原文：[Client Side Load Balancing with Ribbon and Spring Cloud](https://spring.io/guides/gs/client-side-load-balancing/)
>
> 译者：[ligang](http://github.com/holy12345/)
>
> 校对：
# Client Side Load Balancing with Ribbon and Spring Cloud

This guide walks you through the process of providing client-side load balancing for a microservice application using Netflix Ribbon.

## What you’ll build

You’ll build a microservice application that uses Netflix Ribbon and Spring Cloud Netflix to provide client-side load balancing in calls to another microservice.

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

To **start from scratch**, move on to [Build with Gradle](https://spring.io/guides/gs/client-side-load-balancing/#scratch).

To **skip the basics**, do the following:

- [Download](https://github.com/spring-guides/gs-client-side-load-balancing/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone https://github.com/spring-guides/gs-client-side-load-balancing.git`
- cd into `gs-client-side-load-balancing/initial`
- Jump ahead to [Write a server service](https://spring.io/guides/gs/client-side-load-balancing/#initial).

**When you’re finished**, you can check your results against the code in `gs-client-side-load-balancing/complete`.

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

Below is the [initial Gradle build file](https://github.com/spring-guides/gs-client-side-load-balancing/blob/master/initial/build.gradle).

`say-hello/build.gradle`

```groovy
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
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'say-hello'
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

`user/build.gradle`

```groovy
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
apply plugin: 'org.springframework.boot'

jar {
	baseName = 'user'
	version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
	mavenCentral()
	maven { url "https://repo.spring.io/snapshot" }
	maven { url "https://repo.spring.io/milestone" }
}


dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-ribbon')
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

`say-hello/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>say-hello</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>say-hello</name>
	<description>Demo project for Spring Boot</description>

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

`user/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>hello</groupId>
	<artifactId>user</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>user</name>
	<description>Demo project for Spring Boot</description>

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
			<artifactId>spring-cloud-starter-ribbon</artifactId>
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

	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```

The [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) provides many convenient features:

- It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
- It searches for the `public static void main()` method to flag as a runnable class.
- It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.

## Build with your IDE

- Read how to import this guide straight into [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- Read how to work with this guide in [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea).

## Write a server service

Our “server” service is called Say Hello. It will return a random greeting (picked out of a static list of three) from an endpoint accessible at `/greeting`.

In `src/main/java/hello`, create the file `SayHelloApplication.java`. It should look like this:

`say-hello/src/main/java/hello/SayHelloApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.*;

@RestController
@SpringBootApplication
public class SayHelloApplication {

  private static Logger log = LoggerFactory.getLogger(SayHelloApplication.class);

  @RequestMapping(value = "/greeting")
  public String greet() {
    log.info("Access /greeting");

    List<String> greetings = Arrays.asList("Hi there", "Greetings", "Salutations");
    Random rand = new Random();

    int randomNum = rand.nextInt(greetings.size());
    return greetings.get(randomNum);
  }

  @RequestMapping(value = "/")
  public String home() {
    log.info("Access /");
    return "Hi!";
  }

  public static void main(String[] args) {
    SpringApplication.run(SayHelloApplication.class, args);
  }
}
```

The `@RestController` annotation gives the same effect as if we were using `@Controller` and `@ResponseBody` together. It marks `SayHelloApplication` as a controller class (which is what `@Controller` does) and ensures that return values from the class’s `@RequestMapping` methods will be automatically converted appropriately from their original types and written directly to the response body (which is what `@ResponseBody` does). We have one `@RequestMapping`method for `/greeting` and then another for the root path `/`. (We’ll want that second method when we get to working with Ribbon in just a bit.)

We’re going to run multiple instances of this application locally alongside a client service application, so create the directory `src/main/resources`, create the file `application.yml`within it, and then in that file, set a default value for `server.port`. (We’ll instruct the other instances of the application to run on other ports, as well, so that none of the Say Hello instances will conflict with the client when we get that running.) While we’re in this file, we’ll set the `spring.application.name` for our service too.

`say-hello/src/main/resources/application.yml`

```
spring:
  application:
    name: say-hello

server:
  port: 8090
```

## Access from a client service

The User application will be what our user sees. It will make a call to the Say Hello application to get a greeting and then send that to our user when the user visits the endpoint at `/hi`.

In the User application directory, under `src/main/java/hello`, add the file `UserApplication.java`:

`user/src/main/java/hello/UserApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@RestController
public class UserApplication {

  @Bean
  RestTemplate restTemplate(){
    return new RestTemplate();
  }

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/hi")
  public String hi(@RequestParam(value="name", defaultValue="Artaban") String name) {
    String greeting = this.restTemplate.getForObject("http://localhost:8090/greeting", String.class);
    return String.format("%s, %s!", greeting, name);
  }

  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
```

To get a greeting from Say Hello, we’re using Spring’s `RestTemplate` template class. `RestTemplate` makes an HTTP GET request to the Say Hello service’s URL as we provide it and gives us the result as a `String`. (For more information on using Spring to consume a RESTful service, see the [Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/) guide.)

Add the `spring.application.name` and `server.port` properties to `src/main/resources/application.properties` or `src/main/resources/application.yml`:

`user/src/main/resources/application.yml`

```
spring:
  application:
    name: user

server:
  port: 8888
```

## Load balance across server instances

Now we can access `/hi` on the User service and see a friendly greeting:

```shell
$ curl http://localhost:8888/hi
Greetings, Artaban!

$ curl http://localhost:8888/hi?name=Orontes
Salutations, Orontes!
```

To move beyond a single hard-coded server URL to a load-balanced solution, let’s set up Ribbon. In the `application.yml` file under `user/src/main/resources/`, add the following properties:

`user/src/main/resources/application.yml`

```
say-hello:
  ribbon:
    eureka:
      enabled: false
    listOfServers: localhost:8090,localhost:9092,localhost:9999
    ServerListRefreshInterval: 15000
```

This configures properties on a Ribbon *client*. Spring Cloud Netflix creates an `ApplicationContext` for each Ribbon client name in our application. This is used to give the client a set of beans for instances of Ribbon components, including:

- an `IClientConfig`, which stores client configuration for a client or load balancer,
- an `ILoadBalancer`, which represents a software load balancer,
- a `ServerList`, which defines how to get a list of servers to choose from,
- an `IRule`, which describes a load balancing strategy, and
- an `IPing`, which says how periodic pings of a server are performed.

In our case above, the client is named `say-hello`. The properties we set are `eureka.enabled`(which we set to `false`), `listOfServers`, and `ServerListRefreshInterval`. Load balancers in Ribbon normally get their server lists from a Netflix Eureka service registry. (See the [Service Registration and Discovery](https://spring.io/guides/gs/service-registration-and-discovery/) guide for information on using a Eureka service registry with Spring Cloud.) For our simple purposes here, we’re skipping Eureka, so we set the `ribbon.eureka.enabled` property to `false` and instead give Ribbon a static `listOfServers`. `ServerListRefreshInterval` is the interval, in milliseconds, between refreshes of Ribbon’s service list.

In our `UserApplication` class, switch the `RestTemplate` to use the Ribbon client to get the server address for Say Hello:

`user/src/main/java/hello/UserApplication.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.ribbon.RibbonClient;

@SpringBootApplication
@RestController
@RibbonClient(name = "say-hello", configuration = SayHelloConfiguration.class)
public class UserApplication {

  @LoadBalanced
  @Bean
  RestTemplate restTemplate(){
    return new RestTemplate();
  }

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/hi")
  public String hi(@RequestParam(value="name", defaultValue="Artaban") String name) {
    String greeting = this.restTemplate.getForObject("http://say-hello/greeting", String.class);
    return String.format("%s, %s!", greeting, name);
  }

  public static void main(String[] args) {
    SpringApplication.run(UserApplication.class, args);
  }
}
```

We’ve made a couple of other related changes to the `UserApplication` class. Our `RestTemplate` is now also marked as `LoadBalanced`; this tells Spring Cloud that we want to take advantage of its load balancing support (provided, in this case, by Ribbon). The class is annotated with `@RibbonClient`, which we give the `name` of our client (`say-hello`) and then another class, which contains extra `configuration` for that client.

We’ll need to create that class. Add a new file, `SayHelloConfiguration.java`, in the `user/src/main/java/hello` directory:

`user/src/main/java/hello/SayHelloConfiguration.java`

```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.IPing;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.PingUrl;
import com.netflix.loadbalancer.AvailabilityFilteringRule;

public class SayHelloConfiguration {

  @Autowired
  IClientConfig ribbonClientConfig;

  @Bean
  public IPing ribbonPing(IClientConfig config) {
    return new PingUrl();
  }

  @Bean
  public IRule ribbonRule(IClientConfig config) {
    return new AvailabilityFilteringRule();
  }

}
```

We can override any Ribbon-related bean that Spring Cloud Netflix gives us by creating our own bean with the same name. Here, we override the `IPing` and `IRule` used by the default load balancer. The default `IPing` is a `NoOpPing` (which doesn’t actually ping server instances, instead always reporting that they’re stable), and the default `IRule` is a `ZoneAvoidanceRule`(which avoids the Amazon EC2 zone that has the most malfunctioning servers, and might thus be a bit difficult to try out in our local environment).

Our `IPing` is a `PingUrl`, which will ping a URL to check the status of each server. Say Hello has, as you’ll recall, a method mapped to the `/` path; that means that Ribbon will get an HTTP 200 response when it pings a running Say Hello server. The `IRule` we set up, the `AvailabilityFilteringRule`, will use Ribbon’s built-in circuit breaker functionality to filter out any servers in an “open-circuit” state: if a ping fails to connect to a given server, or if it gets a read failure for the server, Ribbon will consider that server “dead” until it begins to respond normally.

The `@SpringBootApplication` annotation on the `UserApplication` class is equivalent to (among others) the `@Configuration` annotation that marks a class as a source of bean definitions. This is why we don’t need to annotate the `SayHelloConfiguration`class with `@Configuration`: since it’s in the same package as `UserApplication`, it is already being scanned for bean methods.This approach does mean that our Ribbon configuration will be part of the main application context and therefore shared by *all* Ribbon clients in the User application. In a normal application, you can avoid this by keeping Ribbon beans out of the main application context (e.g., in this example, you could put `SayHelloConfiguration` in a different package from `UserApplication`). 

## Trying it out

Run the Say Hello service, using either Gradle:

```shell
$ ./gradlew bootRun
```

or Maven:

```shell
$ mvn spring-boot:run
```

Run other instances on ports 9092 and 9999, again using either Gradle:

```shell
$ SERVER_PORT=9092 ./gradlew bootRun
```

or Maven:

```shell
$ SERVER_PORT=9999 mvn spring-boot:run
```

Then start up the User service. Access `localhost:8888/hi` and then watch the Say Hello service instances. You can see Ribbon’s pings arriving every 15 seconds:

```shell
2016-03-09 21:13:22.115  INFO 90046 --- [nio-8090-exec-1] hello.SayHelloApplication                : Access /
2016-03-09 21:13:22.629  INFO 90046 --- [nio-8090-exec-3] hello.SayHelloApplication                : Access /
```

And your requests to the User service should result in calls to Say Hello being spread across the running instances in round-robin form:

```shell
2016-03-09 21:15:28.915  INFO 90046 --- [nio-8090-exec-7] hello.SayHelloApplication                : Access /greeting
```

Now shut down a Say Hello server instance. Once Ribbon has pinged the down instance and considers it down, you should see requests begin to be balanced across the remaining instances.

## Summary

Congratulations! You’ve just developed a Spring application that performs client-side load balancing for calls to another application.

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines](https://github.com/spring-guides/getting-started-guides/wiki).

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
