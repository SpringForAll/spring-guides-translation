# 使用WebSocket来构建交互式Web应用
 ============================================================

> 原文：[Using WebSocket to build an interactive web application](https://spring.io/guides/gs/messaging-stomp-websocket/)
>
> 译者：[汪志峰](https://github.com/maskwang520)
>
> 校对：[白文辉](https://github.com/strongant)

本指南将引导你完成创建“hello world”应用程序的过程，该应用程序在浏览器和服务器之间来回传送消息。 WebSocket是一个轻量级的建立在TCP之上的。它使得非常适合使用subprotocols"来嵌入消息。在本指南中，我们将使用Spring进行[STOMP](https://en.wikipedia.org/wiki/Streaming_Text_Oriented_Messaging_Protocol)消息传递，并创建一个交互式Web应用程序。

## 你将构建什么应用 
你通过建立服务端来接收带有用户姓名的信息。之后给由客户端构成的队列来问候一个问候信息。
## 你需要准备什么？
 
 大约15分钟时间
 
 一个喜欢的文本编辑器或者IDE
 
 [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本
 
 [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
 
 你也可以直接导入代码到IDE:
 
 [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
 
 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
 
 
## 你如何完成这个入门指南
像其他Spring入门指南一样，你可以逐渐的完成每一步，也可以跳过一些你熟悉的步骤。不管怎样，最后你都将得到一份可执行的代码。
* 你可以往下查看[怎样使用 Gradle 构建项目](#scratch).
* 跳过一些基础的，你可以像下面这样做
[下载](https://github.com/spring-guides/gs-messaging-stomp-websocket/archive/master.zip) 并解压这个引导。或者直接使用 [Git](https://spring.io/understanding/Git): git clone [https://github.com/spring-guides/gs-messaging-stomp-websocket.git](https://github.com/spring-guides/gs-messaging-stomp-websocket.git)

完成上述步骤，你可以查看结果，并与gs-messaging-stomp-websocket/complete对比。

## 使用Gradle构建

首先你得安装基础的构建脚本. 你可以使用任意你喜欢的构建系统去构建Spring应用, 你需要使用的代码包含在这儿: [Gradle](http://gradle.org/) and [Maven](https://maven.apache.org/) . 如果你对两者都不熟悉,可以先参考[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle) 或者 [使用Maven构建Java项目](https://spring.io/guides/gs/maven).

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle文件

如下是一个 [Gradle初始化文件](https://github.com/spring-guides/gs-securing-web/blob/master/initial/build.gradle).

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
    baseName = 'gs-securing-web'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    testCompile("junit:junit")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    testCompile("org.springframework.security:spring-security-test")

}
```

 [Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类.
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。

## 使用Maven构建

首先你设置一个基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何构建系统，但你需要使用与Maven一起使用的代码。如果你不熟悉Maven，请参阅使用Maven构建Java项目。

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
    <artifactId>gs-messaging-stomp-websocket</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>

        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>webjars-locator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>sockjs-client</artifactId>
            <version>1.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>stomp-websocket</artifactId>
            <version>2.3.3</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>bootstrap</artifactId>
            <version>3.3.7</version>
        </dependency>
        <dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

 [Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:
- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布服务。
- 它寻找`public static void main()` 方法并标记为一个可执行的类.
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为Boot所选择的一组版本。
## 使用你的IDE构建

- 阅读如何导入这篇指南到你的IDE [Spring Tool Suite](https://spring.io/guides/gs/sts/).
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建.
## 创建资源代表类
现在你已经建立你的项目，你就可以创建你的 STOMP服务。
开始想下服务交互的过程。
服务将会接受包含在 STOMP消息中的name,它的消息体是[JSON](https://spring.io/understanding/JSON) 对象，如何给name指定Fred，那么消息体就像下面这样。
```json
{
    "name": "Fred"
}
```
为了把携带name的消息模型化，你可以创建一个带有name属性的普通Java类和相应的getName()方法。
```java
src/main/java/hello/HelloMessage.java
```
```java
package hello;

public class HelloMessage {

    private String name;

    public HelloMessage() {
    }

    public HelloMessage(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
当接受到消息的时候，获取到name,服务端服务将会发出一个回应，这个回应消息就和客户端传递的那样，也是一个JSON对象，就像下面这样。
```java
{
    "content": "Hello, Fred!"
}
```
为了把携带问候的消息模型化，你可以创建一个带有content属性的普通Java类和相应的getContent()方法。
```java
src/main/java/hello/Greeting.java
```
```java
package hello;

public class Greeting {

    private String content;

    public Greeting() {
    }

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }

}
```
Spring将会用 [Jackson JSON](http://wiki.fasterxml.com/JacksonHome)把Greeting的实例解析成JSON。
接下来，你可以创建一个controller来发送hello 消息和回应greeting消息
## 创建一个消息处理controller
在Spring访问STOMP消息时候，STOMP可以被注解有[@Controller
](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)的类路由到。例如，GreetingController可以映射到目的地”/hello“。
```java
src/main/java/hello/GreetingController.java
```
```javapackage hello;

import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class GreetingController {


    @MessageMapping("/hello")
    @SendTo("/topic/greetings")
    public Greeting greeting(HelloMessage message) throws Exception {
        Thread.sleep(1000); // simulated delay
        return new Greeting("Hello, " + message.getName() + "!");
    }

}
```

这个控制器简洁而简单，但有很多事情。让我们一步一步地剖析它。
[@MessageMapping
](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/handler/annotation/MessageMapping.html)注解确保如果将消息发送到目标“/ hello”，则会调用greeting（）方法。
消息的有效内容绑定到一个HelloMessage对象，该对象被传递到greeting（）。
在内部，该方法的实现通过使线程睡眠1秒来模拟处理延迟。这是为了说明在客户端发送消息之后，服务器可以采取与需要异步处理消息一样长的时间。客户可以继续其需要做的任何工作，而不必等待响应。
在延迟1秒后，greeting（）方法创建一个Greeting对象并返回它。返回值将广播给所有订阅者，如[@SendTo 
](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/handler/annotation/SendTo.html)注释中指定的“/ topic / greetings”。   
## 用Spring配置STOMP 消息
现在服务最重要的一步已经建立起来啦，你可以用Spring来配置WebSocket 和STOMP消息。
创建一个Java类WebSocketConfig，像下面这样。
```java
src/main/java/hello/WebSocketConfig.java
```
```java
package hello;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.AbstractWebSocketMessageBrokerConfigurer;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/gs-guide-websocket").withSockJS();
    }

}
```
WebSocketConfig用@Configuration进行注解，表示它是一个Spring配置类。它也注释为[@EnableWebSocketMessageBroker
](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/simp/config/EnableWebSocketMessageBroker.html)。顾名思义，@EnableWebSocketMessageBroker启用WebSocket消息处理，由消息代理支持。
configureMessageBroker（）方法覆盖WebSocketMessageBrokerConfigurer中的默认方法来配置消息代理。它通过调用enableSimpleBroker（）来启用一个简单的基于内存的消息代理，将问候语消息返回给客户端，前缀为“/ topic”。它还为绑定为@MessageMapping注解方法的邮件指定“/app”前缀。该前缀将用于定义所有消息映射;例如，“/app/hello”是GreetingController.greeting（）方法映射到的端点。
registerStompEndpoints（）方法注册“/ gs-guide-websocket”端点，启用SockJS后备选项，以便在WebSocket不可用时可以使用备用传输。 SockJS客户端将尝试连接到“/ gs-guide-websocket”并使用最好的传输（websocket，xhr-streaming，xhr-polling等）。
## 创建一个浏览器客户端
使用服务器端部件，现在让我们将注意力从服务器端转移到发送消息并能够接受消息的avaScript客户端。像下面这样建立一个index.html 的页面
```java
src/main/resources/static/index.html
```
```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello WebSocket</title>
    <link href="/webjars/bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <link href="/main.css" rel="stylesheet">
    <script src="/webjars/jquery/jquery.min.js"></script>
    <script src="/webjars/sockjs-client/sockjs.min.js"></script>
    <script src="/webjars/stomp-websocket/stomp.min.js"></script>
    <script src="/app.js"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>
<div id="main-content" class="container">
    <div class="row">
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="connect">WebSocket connection:</label>
                    <button id="connect" class="btn btn-default" type="submit">Connect</button>
                    <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
                    </button>
                </div>
            </form>
        </div>
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="name">What is your name?</label>
                    <input type="text" id="name" class="form-control" placeholder="Your name here...">
                </div>
                <button id="send" class="btn btn-default" type="submit">Send</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <table id="conversation" class="table table-striped">
                <thead>
                <tr>
                    <th>Greetings</th>
                </tr>
                </thead>
                <tbody id="greetings">
                </tbody>
            </table>
        </div>
    </div>
</div>
</body>
</html>
```
此HTML文件导入SockJS和STOMP JavaScript库，这些库将用于通过websocket使用STOMP与我们的服务器进行通信。我们还在这里导入一个包含我们客户端应用程序逻辑的app.js。

来创建一个文件
```java
src/main/resources/static/app.js
```
```java
var stompClient = null;

function setConnected(connected) {
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#conversation").show();
    }
    else {
        $("#conversation").hide();
    }
    $("#greetings").html("");
}

function connect() {
    var socket = new SockJS('/gs-guide-websocket');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function (frame) {
        setConnected(true);
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/greetings', function (greeting) {
            showGreeting(JSON.parse(greeting.body).content);
        });
    });
}

function disconnect() {
    if (stompClient !== null) {
        stompClient.disconnect();
    }
    setConnected(false);
    console.log("Disconnected");
}

function sendName() {
    stompClient.send("/app/hello", {}, JSON.stringify({'name': $("#name").val()}));
}

function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}

$(function () {
    $("form").on('submit', function (e) {
        e.preventDefault();
    });
    $( "#connect" ).click(function() { connect(); });
    $( "#disconnect" ).click(function() { disconnect(); });
    $( "#send" ).click(function() { sendName(); });
});
```

这个JavaScript文件的主要部分是connect（）和sendName（）函数。

connect（）函数使用SockJS和stomp.js来打开与“/ gs-guide-websocket”的连接，这是SockJS服务器正在等待连接的地方。在成功连接之后，客户端订阅“/ topic / greetings”目的地，服务器将发布问候消息。当在该目的地收到一个问候语时，它将附加一个段落元素到DOM以显示问候留言。
sendName（）函数检索用户输入的名称，并使用STOMP客户端将其发送到“/ app / hello”目的地（GreetingController.greeting（）将接收到）。

## 使得应用可执行
虽然可以将此服务打包为传统WAR文件以部署到外部应用程序服务器，但下面演示的简单方法创建了独立应用程序。你将所有内容都包装在一个可执行的JAR文件中，由一个Java main（）方法驱动。在这过程中，你使用Spring的支持将Tomcat servlet容器嵌入到HTTP运行时，而不是部署到外部实例。
```java
src/main/java/hello/Application.java
```
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

@SpringBootApplication是一个方便的注解，它添加以下所有内容：
@Configuration将该类标记为应用程序上下文的bean定义的源。
@EnableAutoConfiguration指示Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
通常，你将为Spring MVC应用程序添加@EnableWebMvc，但是当Spring类在类路径中看到spring-webmvc时，Spring Boot会自动添加它。这将应用程序标记为Web应用程序，并激活诸如设置DispatcherServlet等关键行为。
@ComponentScan告诉Spring在hello包中查找其他组件，配置和服务，允许它找到控制器。
main（）方法使用Spring Boot的SpringApplication.run（）方法启动应用程序。你注意到没有一行XML吗？没有web.xml文件。此Web应用程序是100％纯Java，你无需处理配置其他基础构件。
## 构建可执行的JAR
你可以从命令行运行Gradle或Maven应用程序。或者，你可以构建一个包含所有必需依赖项，类和资源的单个可执行JAR文件，并运行该文件。这使得在整个开发生命周期中，跨不同的环境等运输，版本和部署服务成为一个应用程序。
如果你使用的是Gradle，则可以使用./gradlew bootRun运行应用程序。或者你可以使用./gradlew构建来构建JAR文件。然后可以运行JAR文件：
```shell
java -jar build/libs/gs-messaging-stomp-websocket-0.1.0.jar
```
如果使用Maven，可以使用./mvnw spring-boot运行应用程序：run。或者你可以使用./mvnw clean包来构建JAR文件。然后可以运行JAR文件
```shell
java -jar target/gs-messaging-stomp-websocket-0.1.0.jar
```
>上面的过程将创建一个可运行的JAR。你也可以选择构建一个典型的WAR文件。
显示记录输出。该服务应该在几秒钟内启动并运行。
## 测试服务
现在服务正在运行，请将浏览器指向http://localhost：8080，然后单击“连接”按钮。

打开连接后，系统会询问你的姓名。输入你的姓名，然后点击“发送”。你的姓名通过STOMP作为JSON消息发送到服务器。经过1秒钟的模拟延迟后，服务器将返回一条消息，并显示在页面上显示的“Hello”问候语。此时，你可以发送另一个名称，也可以单击“断开连接”按钮关闭连接。

## 总结

恭喜！你刚刚使用Spring开发了基于STOMP的消息传递服务。
## 你也可以看如下的
下面的指导也可能是有用的
* [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/)协议进行许可。