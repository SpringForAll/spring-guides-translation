【wkk52586翻译中】

原文：[Accessing Facebook Data](https://spring.io/guides/gs/accessing-facebook/)

译者：[wkk52586](https://github.com/wkk52586)

校对：

# 访问Facebook数据

本指南将引导你创建一个简单的web应用来访问Facebook数据。

## 你会创建什么

你将会创建一个web应用访问Facebook的一个用户信息，和这个用户的信息流（feed）的一些帖子。

## 你需要的准备

- 大约15分钟时间
- 一个喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或更高版本
- [Gradle 2.3+](http://www.gradle.org/downloads)或 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接把代码导入到IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    - 一个在Facebook[注册应用程序](https://spring.io/guides/gs/register-facebook-app)时获得的ID和密钥 。

## 怎样完成本指南
像大多数Spring [入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，或者跳过那些你熟悉的基本步骤。不管那一种方式，最终会得到可以正常工作的代码。

**从头开始**，移步到[使用Gradle]
(https://spring.io/guides/gs/accessing-facebook/#scratch)构建项目。

**略过基本步骤**，从这里开始：

- [下载](https://github.com/spring-guides/gs-accessing-facebook/archive/master.zip) 并解压缩本指南的代码，或者使用[Git](https://spring.io/understanding/Git)克隆: `git clone https://github.com/spring-guides/gs-accessing-facebook.git`
- cd 到 `gs-accessing-facebook/initial`
- 跳转到[启用Facebook](https://spring.io/guides/gs/accessing-facebook/#initial)。

**当你完成之后**，你可以根据代码在`gs-accessing-facebook/complete`目录下检查结果。

## 使用Gradle构建

首先创建一个基础构建脚本。你可以使用任何你喜欢的构建系统构建Spring应用， 但是 [Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/)的代码已经包含在这里。如果这两者都不熟悉，请参考[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle)或者 [使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录里，创建如下的子目录结构；例如，在 *nix系统上使用命令`mkdir -p src/main/java/hello` 

```java
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle构建文件

下面是[初始化Gradle构建文件](https://github.com/spring-guides/gs-accessing-facebook/blob/master/initial/build.gradle)。

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

 [Spring Boot gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供许多方便有用的特性:

- 它能收集所有classpath中的jar包构建一个可运行的"über-jar"，这个jar可以很方便地被执行和传输。
- 它能搜索到`public static void main()` 方法并把所在的类标记为可运行类。
- 它提供了将内部依赖的版本都去匹配 [Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)的版本.你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建
首先，你需要设置一个基本的构建脚本。当使用Spring构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 Maven 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在 类unix 系统中使用如下命令: mkdir -p src/main/java/hello


└── src

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

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布你的服务。
- 它寻找public static void main() 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择了一组版本。

## 使用IDE构建

- 阅读如何将本指南直接导入[Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何在[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)使用本指南。

## 启用Facebook

在从Facebook获取一个用户数据之前，必须通过设置`spring.social.facebook.appId` 和 `spring.social.facebook.appSecret`来指定你的应用程序id和secret。你可以设置这些通过任何Spring Boot支持的方式，其中也包括在`application.properties`属性文件中设置：

`src/main/resources/application.properties`

```shell
spring.social.facebook.appId=233668646673605
spring.social.facebook.appSecret=33b17e044ee6a4fa383f46ec6e28ea1d
```

上述显示的属性是用于示例的虚拟值。。当你[注册Facebook应用程序](https://spring.io/guides/gs/register-facebook-app)的时候，可以得到对应的真正的应用程序的消费者key和密钥。

上述属性和Spring Social Facebook出现在classpath中，将触发自动配置Spring Social的`ConnectController`， `FacebookConnectionFactory`，和连接其它Spring Social’s的组件框架。

## 创建连接状态视图

尽管`ConnectController`所做的很多工作，比如重定向到Facebook和处理一个来自Facebook的重定向，但是当一个GET请求来到/connect，它也能显示连接状态。当没有连接可用的时候，它显示到一个叫做connect/{providerId}Connect的视图，当有连接存在的时候，它显示到provider对应的connect/{providerId}Connected，在这个例子中， **provider ID** 指的是 "facebook"。

`ConnectController`并不定义它自己的连接视图，因此需要创建它们。首先，这里是一个Thymeleaf视图，当没有Facebook连接的时候它会被显示：

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

这个视图中表单将使用POST提交到/connect/facebook，`ConnectController`将会处理请求，从而触发OAuth授权的代码流程。

下面是连接存在时将要显示的视图：

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

## 获取Facebook数据

在你的应用程序里配置Facebook后，现在可以写一个Spring MVC controller获取用户授权的应用程序的数据并显示在浏览器中，`HelloController`就是下面这个样子：

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

创建`HelloController`的时候，在构造方法里注入一个`Facebook`对象。`Facebook`对象是一个Spring Social’s Facebook API绑定的引用。

在`helloFacebook()` 这个方法上加上注解 `@RequestMapping`，表示它会处理根路径(/)的GET请求。这个方法的第一件事情是检查用户是否已授权应用程序访问用户的Facebook数据。如果没有，用户会被重定向到 `ConnectController`并带有开始授权的选项。

如果用户已经授权应用程序访问Facebook数据，应用程序可以获取用户的概要数据同时获取用户最近的几个输入。数据放到model中，在"hello"这个view中用来显示。

说到“hello”视图，这里是一个Thymeleaf模板：

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

## 让应用程序可执行

虽然可以可以像传统的方式那样打成一个*web application archive*或[WAR](https://spring.io/understanding/WAR)发布到应用程序服务器，但是下面更简便的方法是创建一个*独立应用程序*。把所有内容打包成一个可执行的jar文件，有一个友好的`main()` 方法来驱动。而且这样使用内嵌的 [Tomcat](https://spring.io/understanding/Tomcat) servlet容器作为HTTP运行环境，而不是部署到一个外部的容器实例。

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

`@SpringBootApplication` 是一个注解添加下列所有：

- `@Configuration`标记一个类成为应用程序上下文中bean的定义。
- `@EnableAutoConfiguration` 告诉Spring Boot开始添加基于classpath设置的bean，其它bean和各种属性设置。
- 正常来说你应该添加`@EnableWebMvc` 给一个Spring MVC应用，但是Spring Boot 自动增加了当它在classpath看到**spring-webmvc**的时候。这个能标记应用程序是一个web应用而且激活一些关键行为比如建立`DispatcherServlet`等。
- `@ComponentScan` 告诉Spring查找`hello`包中其它组件，配置，和服务，允许查找controllers.

`main()` 方法使用Spring Boot中的`SpringApplication.run()`方法来启动应用程序。你是不是注意到没有一行XML代码呢？也没有**web.xml**。这个应用程序是一个100%的纯Java程序，你不需要处理任何基础配置。

### 创建一个可执行JAR

你可以使用Gradle或者Maven在命令行里运行应用程序。或者创建一个单一的可执行jar文件，里面包含必要的依赖，类和资源，然后运行它。这样就很容易地发送，打版本，和部署服务作为一个应用在整个开发生命周期中，在不同的环境中，等等。

如果你使用Gradle，你可以使用`./gradlew bootRun`运行应用程序。或者使用`./gradlew build`来创建JAR文件，然后运行这个JAR文件：

```shell
java -jar build/libs/gs-accessing-facebook-0.1.0.jar
```

如果你使用Maven，你可以使用`./mvnw spring-boot:run`运行应用程序。或者使用`./mvnw clean package`来创建JAR文件，然后运行这个JAR文件：

```shell
java -jar target/gs-accessing-facebook-0.1.0.jar
```

| **   | 上述过程会创建一个可运行的JAR。 你也可以取而代之选择 [创建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。 |
| ---- | ---------------------------------------- |
|      |                                          |


```shell
... app starts up ...
```

一旦应用程序启动，在浏览器中输入 [http://localhost:8080](http://localhost:8080/) 。连接还没有被创建，因此屏幕上会提示了连接到Facebook:

![No connection to Facebook exists yet.](https://spring.io/guides/gs/accessing-facebook/images/connect.png)

当你点击 **Connect to Facebook**按钮，浏览器重定向到Facebook来获取授权：

![Facebook needs your permission to allow the application to access your data.](https://spring.io/guides/gs/accessing-facebook/images/fbauth.png)

点击 "Okay"按钮授权示例应用来访问你的公开概要信息和feed。

一旦被授予权限，Facebook重定向浏览器回到应用程序。一个连接已被创建而且保存在连接库中。在页面上会看到一个连接成功的提示：

![A connection with Facebook has been created.](https://spring.io/guides/gs/accessing-facebook/images/connected.png)

在连接状态页面上点击链接，就会来到主页。现在连接以及被创建，能看到你在Facebook上所使用的名字和一个你最近feed的列表。里面包含每一个帖子的发送者姓名，发送的信息和发送的图片（如果有的话）。

## 总结

恭喜你，开发了一个简单的web应用，获取用户授权从而从Facebook取到数据。这个应用程序通过Spring Social把用户连接到Facebook，并获取用户的Facebook基本数据，而且也能获取用户好友的一些基本数据。

## 你也可以看

下列的指南也非常有用：

- [在Facebook上注册一个应用程序](https://spring.io/guides/gs/register-facebook-app/)
- [访问Twitter数据](https://spring.io/guides/gs/accessing-twitter/)

你也想写一篇新的指南或者对已有指南做出贡献吗？请查看[贡献指导方针](https://github.com/spring-guides/getting-started-guides/wiki).

本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。 
