[zhongchubin正在翻译中]
# Spring 定时任务

# Scheduling Tasks
> 原文：[Scheduling Tasks](https://spring.io/guides/gs/scheduling-tasks/)

> 译者：[rhwayfun][1]

> 校对：[codedrinker][2]

## Scheduling Tasks

This guide walks you through the steps for scheduling tasks with Spring.

### What you’ll build
You’ll build an application that prints out the current time every five seconds using Spring’s `@Scheduled` annotation.

### What you’ll need

 - About 15 minutes
 - A favorite text editor or IDE
 - [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
 - [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
 - You can also import the code straight into your IDE:
    - [Spring Tool Suite (STS)][3]
    - [IntelliJ IDEA][4]

### How to complete this guide
Like most [Spring Getting Started guides][5], you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To start from scratch, move on to [Build with Gradle][https://spring.io/guides/gs/scheduling-tasks/#scratch]。

To skip the basics, do the following:

 - [Download][6]and unzip the source repository for this guide, or clone it using[Git][7]：
    `git clone https://github.com/spring-guides/gs-scheduling-tasks.git`
 - cd into `gs-scheduling-tasks/initial`
 - Jump ahead to `Create a scheduled task`

When you’re finished, you can check your results against the code in `gs-scheduling-tasks/complete`.

### Build with Gradle

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Gradle][8] and [Maven][9] is included here. If you’re not familiar with either, refer to[Building Java Projects with Gradle][10]or[Building Java Projects with Maven][11]。

#### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on `*nix` systems:

```
└── src
    └── main
        └── java
            └── hello
```

#### Create a Gradle build file

Below is the[initial Gradle build file][12]

build.gradle

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
    baseName = 'gs-spring-boot'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    // tag::jetty[]
    compile("org.springframework.boot:spring-boot-starter-web") {
        exclude module: "spring-boot-starter-tomcat"
    }
    compile("org.springframework.boot:spring-boot-starter-jetty")
    // end::jetty[]
    // tag::actuator[]
    compile("org.springframework.boot:spring-boot-starter-actuator")
    // end::actuator[]
    testCompile("junit:junit")
}
```

[Spring Boot gradle plugin][13] provides many convenient features:

* It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.

* It searches for the `public static void main()` method to flag as a runnable class.

* It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies][14]. You can override any version you wish, but it will default to Boot’s chosen set of versions.

### Build with Maven

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven][15] is included here. If you’re not familiar with Maven, refer to Building[Building Java Projects with Maven][16].

#### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with mkdir -p `src/main/java/hello` on `*nix` systems:

```
└── src
    └── main
        └── java
            └── hello
```

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-scheduling-tasks</artifactId>
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
            <artifactId>spring-boot-starter</artifactId>
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

[Spring Boot Maven Plugin][17] provides many convenient features:

* It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.

* It searches for the `public static void main()` method to flag as a runnable class.

* It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies][18].You can override any version you wish, but it will default to Boot’s chosen set of versions.

### Build with your IDE

*   [Read how to import this guide straight into][19].

*   [Read how to work with this guide][20].

### Create a scheduled task
Now that you’ve set up your project, you can create a scheduled task.

`src/main/java/hello/ScheduledTasks.java`

```java
package hello;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledTasks {

    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("The time is now {}", dateFormat.format(new Date()));
    }
}
```

The `Scheduled` annotation defines when a particular method runs. NOTE: This example uses `fixedRate`, which specifies the interval between method invocations measured from the start time of each invocation. There are[other options][21], like fixedDelay, which specifies the interval between invocations measured from the completion of the task. You can also[use `@Scheduled(cron=". . .")` expressions for more sophisticated task scheduling.][22]

### Enable Scheduling
Although scheduled tasks can be embedded in web apps and WAR files, the simpler approach demonstrated below creates a standalone application. You package everything in a single, executable JAR file, driven by a good old Java main() method.

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class Application {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class);
    }
}
```
`@SpringBootApplication` is a convenience annotation that adds all of the following:

 - `@Configuration` tags the class as a source of bean definitions for the application context.
 - `@EnableAutoConfiguration` tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
 - Normally you would add `@EnableWebMvc` for a Spring MVC app, but Spring Boot adds it automatically when it sees **spring-webmvc** on the classpath. This flags the application as a web application and activates key behaviors such as setting up a `DispatcherServlet`.
 - `@ComponentScan` tells Spring to look for other components, configurations, and services in the `hello` package, allowing it to find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there wasn’t a single line of XML? No **web.xml** file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

`@EnableScheduling` ensures that a background task executor is created. Without it, nothing gets scheduled.

**Build an executable JAR**

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using `./gradlew bootRun`. Or you can build the JAR file using `./gradlew build`. Then you can run the JAR file:

`java -jar target/gs-scheduling-tasks-0.1.0.jar`

> The procedure above will create a runnable JAR. You can also opt to build a classic WAR file instead.

Logging output is displayed and you can see from the logs that it is on a background thread. You should see your scheduled task fire every 5 seconds:

```
[...]
2016-08-25 13:10:00.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:00
2016-08-25 13:10:05.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:05
2016-08-25 13:10:10.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:10
2016-08-25 13:10:15.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:15
```

### Summary
Congratulations! You created an application with a scheduled task. Heck, the actual code was shorter than the build file! This technique works in any type of application.

### See Also

The following guides may also be helpful:

*   [Building an Application with Spring Boot][23]

*   [Creating a Batch Service][24]

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines][https://github.com/spring-guides/getting-started-guides/wiki].

> All guides are released with an ASLv2 license for the code, and an [Attribution, NoDerivatives creative commons license][https://creativecommons.org/licenses/by-nd/3.0/] for the writing.

--------------------------------------------------------------------------------


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可][25]协议进行许可。


[1]: https://github.com/happyxiaofan
[2]: https://github.com/codedrinker
[3]: https://spring.io/guides/gs/sts
[4]: https://spring.io/guides/gs/intellij-idea/
[5]: https://spring.io/guides
[6]: https://github.com/spring-guides/gs-scheduling-tasks/archive/master.zip
[7]: https://spring.io/understanding/Git
[8]: http://gradle.org/
[9]: https://maven.apache.org/
[10]: https://spring.io/guides/gs/gradle
[11]: https://spring.io/guides/gs/maven
[12]: https://github.com/spring-guides/gs-scheduling-tasks/blob/master/initial/build.gradle
[13]: https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin
[14]: https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
[15]: https://maven.apache.org/
[16]: https://spring.io/guides/gs/maven
[17]: https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin
[18]: https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
[19]: https://spring.io/guides/gs/sts/
[20]: https://spring.io/guides/gs/intellij-idea
[21]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html#scheduling-annotation-support-scheduled
[22]: https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html
[23]: https://spring.io/guides/gs/spring-boot/
[24]: https://spring.io/guides/gs/batch-processing/
[25]: http://creativecommons.org/licenses/by-nc-sa/4.0/