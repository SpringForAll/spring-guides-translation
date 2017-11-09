【wkk52586翻译中】

原文：[Accessing Facebook Data](https://spring.io/guides/gs/accessing-facebook/)

译者：

校对：

# 访问Facebook数据

This guide walks you through the process of creating a simple web application that accesses Facebook data.

## What you’ll build

You’ll build a web application that accesses data from a Facebook user profile, as well as posts from that user’s Facebook feed.

## What you’ll need

- About 15 minutes
- A favorite text editor or IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- You can also import the code straight into your IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    - An application ID and secret obtained from [registering an application with Facebook](https://spring.io/guides/gs/register-facebook-app).

## How to complete this guide

Like most Spring [Getting Started guides](https://spring.io/guides), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Build with Gradle](https://spring.io/guides/gs/accessing-facebook/#scratch).

To **skip the basics**, do the following:

- [Download](https://github.com/spring-guides/gs-accessing-facebook/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone https://github.com/spring-guides/gs-accessing-facebook.git`
- cd into `gs-accessing-facebook/initial`
- Jump ahead to [Enable Facebook](https://spring.io/guides/gs/accessing-facebook/#initial).

**When you’re finished**, you can check your results against the code in `gs-accessing-facebook/complete`.

## Build with Gradle

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) is included here. If you’re not familiar with either, refer to [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) or [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

```java
└── src
    └── main
        └── java
            └── hello
```

### Create a Gradle build file

Below is the [initial Gradle build file](https://github.com/spring-guides/gs-accessing-facebook/blob/master/initial/build.gradle).

`build.gradle`

```shell
buildscript {
    repositories {
        maven { url "https://repo.spring.io/libs-milestone" }
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
    baseName = 'gs-accessing-facebook'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/libs-milestone" }
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    compile("org.springframework.social:spring-social-facebook")
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

```java
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
    <artifactId>gs-accessing-facebook</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.social</groupId>
            <artifactId>spring-social-facebook</artifactId>
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

    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
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

## Enable Facebook

Before you can fetch a user’s data from Facebook, you must specify your application’s ID and secret by setting the `spring.social.facebook.appId` and `spring.social.facebook.appSecret`properties. You can set these via any means supported by Spring Boot, including setting them in an `application.properties` file:

`src/main/resources/application.properties`

```shell
spring.social.facebook.appId=233668646673605
spring.social.facebook.appSecret=33b17e044ee6a4fa383f46ec6e28ea1d
```

As shown here, the properties have fake values. The values given to these properties correspond to your application’s consumer key and secret you obtain when you [register the application with Facebook](https://spring.io/guides/gs/register-facebook-app). For the code to work, substitute the real values given to you by Facebook in place of the fake values.

The presence of these properties and Spring Social Facebook in the classpath will trigger automatic configuration of Spring Social’s `ConnectController`, `FacebookConnectionFactory`, and other components of Spring Social’s connection framework.

## Create connection status views

Although much of what `ConnectController` does involves redirecting to Facebook and handling a redirect from Facebook, it also shows connection status when a GET request to /connect is made. It defers to a view named connect/{providerId}Connect when no existing connection is available and to connect/{providerId}Connected when a connection exists for the provider. In this case, **provider ID** is "facebook".

`ConnectController` does not define its own connection views, so you need to create them. First, here’s a Thymeleaf view to be shown when no connection to Facebook exists:

`src/main/resources/templates/connect/facebookConnect.html`

```html
<html>
	<head>
		<title>Hello Facebook</title>
	</head>
	<body>
		<h3>Connect to Facebook</h3>

		<form action="/connect/facebook" method="POST">
			<input type="hidden" name="scope" value="user_posts" />
			<div class="formInfo">
				<p>You aren't connected to Facebook yet. Click the button to connect this application with your Facebook account.</p>
			</div>
			<p><button type="submit">Connect to Facebook</button></p>
		</form>
	</body>
</html>
```

The form on this view will POST to /connect/facebook, which is handled by `ConnectController` and will kick off the OAuth authorization code flow.

Here’s the view to be displayed when a connection exists:

`src/main/resources/templates/connect/facebookConnected.html`

```html
<html>
	<head>
		<title>Hello Facebook</title>
	</head>
	<body>
		<h3>Connected to Facebook</h3>

		<p>
			You are now connected to your Facebook account.
			Click <a href="/">here</a> to see some entries from your Facebook feed.
		</p>
	</body>
</html>
```

## Fetch Facebook data

With Facebook configured in your application, you now can write a Spring MVC controller that fetches data for the user who authorized the application and presents it in the browser. `HelloController` is just such a controller:

`src/main/java/hello/HelloController.java`

```java
package hello;

import org.springframework.social.connect.ConnectionRepository;
import org.springframework.social.facebook.api.Facebook;
import org.springframework.social.facebook.api.PagedList;
import org.springframework.social.facebook.api.Post;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/")
public class HelloController {

    private Facebook facebook;
    private ConnectionRepository connectionRepository;

    public HelloController(Facebook facebook, ConnectionRepository connectionRepository) {
        this.facebook = facebook;
        this.connectionRepository = connectionRepository;
    }

    @GetMapping
    public String helloFacebook(Model model) {
        if (connectionRepository.findPrimaryConnection(Facebook.class) == null) {
            return "redirect:/connect/facebook";
        }

        model.addAttribute("facebookProfile", facebook.userOperations().getUserProfile());
        PagedList<Post> feed = facebook.feedOperations().getFeed();
        model.addAttribute("feed", feed);
        return "hello";
    }

}
```

`HelloController` is created by injecting a `Facebook` object into its constructor. The `Facebook` object is a reference to Spring Social’s Facebook API binding.

The `helloFacebook()` method is annotated with `@RequestMapping` to indicate that it should handle GET requests for the root path (/). The first thing the method does is check whether the user has authorized the application to access the user’s Facebook data. If not, the user is redirected to `ConnectController` with the option to kick off the authorization process.

If the user has authorized the application to access Facebook data, the application fetches the user’s profile as well as several of the most recent entries in the user’s feed. The data is placed into the model to be displayed by the view identified as "hello".

Speaking of the "hello" view, here it is as a Thymeleaf template:

`src/main/resources/templates/hello.html`

```html
<html>
	<head>
		<title>Hello Facebook</title>
	</head>
	<body>
		<h3>Hello, <span th:text="${facebookProfile.name}">Some User</span>!</h3>

		<h4>Here is your feed:</h4>

		<div th:each="post:${feed}">
			<b th:text="${post.from.name}">from</b> wrote:
			<p th:text="${post.message}">message text</p>
			<img th:if="${post.picture}" th:src="${post.picture}"/>
			<hr/>
		</div>
	</body>
</html>
```

## Make the application executable

Although it is possible to package this service as a traditional *web application archive* or [WAR](https://spring.io/understanding/WAR)file for deployment to an external application server, the simpler approach demonstrated below creates a *standalone application*. You package everything in a single, executable JAR file, driven by a good old Java `main()` method. And along the way, you use Spring’s support for embedding the [Tomcat](https://spring.io/understanding/Tomcat) servlet container as the HTTP runtime, instead of deploying to an external instance.

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

`@SpringBootApplication` is a convenience annotation that adds all of the following:

- `@Configuration` tags the class as a source of bean definitions for the application context.
- `@EnableAutoConfiguration` tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
- Normally you would add `@EnableWebMvc` for a Spring MVC app, but Spring Boot adds it automatically when it sees **spring-webmvc** on the classpath. This flags the application as a web application and activates key behaviors such as setting up a `DispatcherServlet`.
- `@ComponentScan` tells Spring to look for other components, configurations, and services in the `hello` package, allowing it to find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there wasn’t a single line of XML? No **web.xml** file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

### Build an executable JAR

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using `./gradlew bootRun`. Or you can build the JAR file using `./gradlew build`. Then you can run the JAR file:

```shell
java -jar build/libs/gs-accessing-facebook-0.1.0.jar
```

If you are using Maven, you can run the application using `./mvnw spring-boot:run`. Or you can build the JAR file with `./mvnw clean package`. Then you can run the JAR file:

```shell
java -jar target/gs-accessing-facebook-0.1.0.jar
```

| **   | The procedure above will create a runnable JAR. You can also opt to [build a classic WAR file](https://spring.io/guides/gs/convert-jar-to-war/)instead. |
| ---- | ---------------------------------------- |
|      |                                          |

```shell
... app starts up ...
```

Once the application starts up, point your web browser to [http://localhost:8080](http://localhost:8080/). No connection is established yet, so this screen prompts you to connect with Facebook:

![No connection to Facebook exists yet.](https://spring.io/guides/gs/accessing-facebook/images/connect.png)

When you click the **Connect to Facebook** button, the browser is redirected to Facebook for authorization:

![Facebook needs your permission to allow the application to access your data.](https://spring.io/guides/gs/accessing-facebook/images/fbauth.png)

Click "Okay" to grant permission for the sample application to access your public profile and your feed.

Once permission is granted, Facebook redirects the browser back to the application. A connection is created and stored in the connection repository. You should see this page indicating that a connection was successful:

![A connection with Facebook has been created.](https://spring.io/guides/gs/accessing-facebook/images/connected.png)

Click the link on the connection status page, and you are taken to the home page. This time, now that a connection has been created, you see your name on Facebook and a list of some recent entries in your home feed. Included for each post in the feed list is the name of the sender, the post’s message, and the post’s picture (if available).

## Summary

Congratulations! You have developed a simple web application that obtains user authorization to fetch data from Facebook. The application connects the user to Facebook through Spring Social, retrieves data from the user’s Facebook profile, and also fetches profile data from the user’s Facebook friends.

## See Also

The following guides may also be helpful:

- [Registering an Application with Facebook](https://spring.io/guides/gs/register-facebook-app/)
- [Accessing Twitter Data](https://spring.io/guides/gs/accessing-twitter/)

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines](https://github.com/spring-guides/getting-started-guides/wiki).

本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
