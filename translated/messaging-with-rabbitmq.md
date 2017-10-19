# Spring Boot 整合消息中间件 RabbitMQ

> 原文：[Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)
>
> 译者：[chenzhijun](https://github.com/chenzhijun)
>
> 校对：[程序猿DD](https://github.com/dyc87112)

本篇指南会告诉您如何使用构建一个基于`AMQP协议`的 RabbitMQ 服务，并且教您如何实现发布和订阅消息。

## 你会得到什么？

你会创建一个应用，它能够使用 Spring AMQP 的`RabbitTemplate`发布消息，并且通过使用`MessageListenerAdapter`包装一个 [POJO][18] 来接受消息。

## 你需要准备什么？

*   大概15分钟时间

*   一个喜欢的文本编辑器或者IDE

*   [JDK 1.8][4] or 更高版本

*   [Gradle 2.3+][5] or [Maven 3.0+][6] 

*   你可以直接导入 RabbitMQ 服务的代码到 IDE : [Spring Tool Suite (STS)][7] 或者 [IntelliJ IDEA][8] (点击进入安装步骤)

## 怎样完成指南？

像大多数 Spring [教程指南][19]一样，你可以选择从最基础开始，一步步的完成 Demo ，或者你也可以绕过你熟悉的步骤再开始。不管哪种方式，你最后都会得到一份可执行的代码。

如果从基础开始，你可以往下查看[怎样使用 Gradle 构建项目][20]。

如果已经熟悉一些基本步骤，你可以：

*   [下载并解压源码库][9]，或者通过 Git 工具克隆一份代码：[了解Git][10] : `git clone [https://github.com/spring-guides/gs-messaging-rabbitmq.git][11]`

*   cd into gs-messaging-rabbitmq/initial

*   往下看 [创建RabbitMQ 消息接收者][12]

当你完成之后，你可以在`gs-messaging-rabbitmq/complete`检查下结果。

## 使用 Gradle 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Gradle][30] 或者 [Maven][31] 构建的代码。如果你对两者都不是很熟悉，你可以先去看下[如何使用 Gradle 构建 Java 项目][32]或者[如何使用 Maven 构建 Java 项目][33]。

### 创建 Gradle 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```

### 创建 Gradle 构建文件

下面是一份[初始化Gradle构建文件][34]

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

[Spring Boot gradle 插件][35] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][25].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用 Maven 构建项目

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建，这里提供一份你可能需要用 [Maven][26] 构建的代码。如果你对 Maven 还不是很熟悉，你可以先去看下[如何使用 Maven 构建 Java 项目][27].

### 创建 Maven 目录结构

在你的项目根目录，创建如下的子目录结构；例如，如果你使用的是`*nix`系统，你可以使用`mkdir -p src/main/java/hello` 

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
    <artifactId>gs-messaging-rabbitmq</artifactId>
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
            <artifactId>spring-boot-starter-amqp</artifactId>
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

[Spring Boot Maven 插件][28] 提供了非常多方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷

* 搜索`public static void main()`方法并且将它标记为可执行类

* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖的版本][25].你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用你的 IDE 进行构建

*   [如何在Spring Tool Suite中构建][13].

*   [如何在IntelliJ IDEA中构建][14].

## 安装 RabbitMQ

在构建消息应用之前，需要先安装 RabbitMQ 消息中间件服务，中间件服务器会处理发送和接受消息。

RabbitMQ 是一个基于`AMQP协议`的消息中间件。它完全开源，你可以在这里[http://www.rabbitmq.com/download.html][21]去下载它，如果你使用的是 Mac 电脑，你可以使用 homebrew 来安装：
```
brew install rabbitmq
```

解压下载文件，并且使用默认配置启动服务.

```
rabbitmq-server
```

启动后，你可以看到下面的提示：

```
            RabbitMQ 3.1.3\. Copyright (C) 2007-2013 VMware, Inc.
##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
##  ##
##########  Logs: /usr/local/var/log/rabbitmq/rabbit@localhost.log
######  ##        /usr/local/var/log/rabbitmq/rabbit@localhost-sasl.log
##########
            Starting broker... completed with 6 plugins.
```

 如果你本地安装了 Docker，你也可以通过 [Docker Compose][22] 的方式来快速启动一个 RabbitMQ 服务。下面是 Github 上面一个建立 RabbitMQ 服务的`docker-compse.yml`，它非常简单：

docker-compose.yml

```yaml
rabbitmq:
  image: rabbitmq:management
  ports:
    - "5672:5672"
    - "15672:15672"
```

将这个文件放到你当前的目录，并且使用`docker-compose`命令，就可以有一个运行 RabbitMQ 的容器了。

## 创建 RabbitMQ 消息接收者

对于一些使用消息的应用，你通常都需要创建一个消息接收者来响应已经发布的消息

src/main/java/hello/Receiver.java

```java
package hello;

import java.util.concurrent.CountDownLatch;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

    private CountDownLatch latch = new CountDownLatch(1);

    public void receiveMessage(String message) {
        System.out.println("Received <" + message + ">");
        latch.countDown();
    }

    public CountDownLatch getLatch() {
        return latch;
    }

}
```

这个接收者是一个非常简单的 POJO ，它定义了一个用来接收消息的方法。如果你自己实现，你也可以给相应的接收方法取任何你喜欢的名字。

> 为了方便，POJO里面定义了一个`CountDownLatch`。它能够在接收到消息的时候给我们一些提示信号。在正式生产的环境里，你是不太可能像这样操作的。

## 注册监听器并且发送消息

Spring AMQP 的 RabbitTemplate 提供了任何你想要通过 RabbitMQ 发送和接受消息的任何功能。当然，你需要先做一些配置：

* 一个消息监听容器

* 声明队列，交换机，并且将它们两者绑定

* 一个发送消息来测试监听器的组件类

> Spring Boot 自动创建了一个连接工厂(译者注:RabbitMQ中的Connection Factory)和一个 RabbitTemplate 用来减少你需要写的大量代码。

你会使用`RabbitTemplate`来发送消息，并且你会注册一个消息监听器的接收者来接收消息。连接工厂已经在底层做了一些实现，来允许他们连接到 RabbitMQ 服务器。

src/main/java/hello/Application.java

```java
package hello;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.listener.adapter.MessageListenerAdapter;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

    final static String queueName = "spring-boot";

    @Bean
    Queue queue() {
        return new Queue(queueName, false);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("spring-boot-exchange");
    }

    @Bean
    Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(queueName);
    }

    @Bean
    SimpleMessageListenerContainer container(ConnectionFactory connectionFactory,
            MessageListenerAdapter listenerAdapter) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames(queueName);
        container.setMessageListener(listenerAdapter);
        return container;
    }

    @Bean
    MessageListenerAdapter listenerAdapter(Receiver receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    public static void main(String[] args) throws InterruptedException {
        SpringApplication.run(Application.class, args);
    }

}
```

`@SpringBootApplication` 是一个非常方便的注解，它增加了以下注解的所有功能

* `@Configuration` 标记这个类是应用上下文 Bean 的源定义.

* `@EnableAutoConfiguration` 告诉 SpringBoot 启动的时候在 classpath 设置、其它已经装载的 Bean 以及其它配置文件的基础上自动进行配置 Bean.

* 通常你会在SpringMVC应用上使用`@EnableMvc`，但是Spring Boot 在看到spring-webmvc 在它的classpath目录下的时候，它会自动加载该注解。这个注解标记了这个应用是一个web应用，并且会激活一些关键功能，比如说加载`DispatcherServlet`.

* `@ComponetScan` 告诉 Spring 在 hello 包下扫描其它的注解，如组件(componets)，配置(configurations)，或者服务(services),Spring 也会通过它找到控制器(controllers)

`main()` 方法里面通过调用 Spring Boot 的`SpringApplication.run()`方法来启动应用。你有没有注意到到现在还没有写过一行 XML ？甚至也没有`web.xml`文件。这个 web 应用完全 100% 都是使用的 Java，并且你还不需要对任何应用的基础设置进行配置。

通过`listenerAdapter()`来定义的`Bean`，用来在`container()`方法里面注册称为一个消息监听器。它会监听来自"spring-boot"队列的消息。因为接受者类是一个`POJO`，它需要在`MessageListenAdapter`里面进行一层包装，然后在这里我们再调用它的`receiveMessage`方法。


> 基于 JMS 的队列和 基于 AMQP 的队列有些不同。比如，JMS 只发送消息给一个消费者，而 AMQP 也可以做到同样的事，并且 AMQP 的生产者不是直接发送消息给队列，它将消息发送一个交换机，交换机可以将消息发送给一个队列，也可以发送给多个队列,就像 JMS 的 topics。[了解更多AMQP][1]

消息监听容器和接收消息的 Bean ，你都应该监听。如果要发送消息，你需要使用 `RabbitTemplate`。

`queue()`方法创建了一个AMQP队列。`exchange()`方法创建了一个`topic`交换机。当 `RabbitTemplate` 发布消息到一个交换机的时候，`binding()`方法将队列和交换机两者进行了绑定。

> Spring AMQP 要求`Queue`,`TopicExchange`,`Binding`被声明成了 Spring 的高级 Beans，之后才能正确的执行。

## 发送文本消息

文本消息通过一个`CommandLineRunner`类来发送，它会等待接收方锁并且关闭应用的上下文：

src/main/java/hello/Runner.java

```java
package hello;

import java.util.concurrent.TimeUnit;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class Runner implements CommandLineRunner {

    private final RabbitTemplate rabbitTemplate;
    private final Receiver receiver;
    private final ConfigurableApplicationContext context;

    public Runner(Receiver receiver, RabbitTemplate rabbitTemplate,
            ConfigurableApplicationContext context) {
        this.receiver = receiver;
        this.rabbitTemplate = rabbitTemplate;
        this.context = context;
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Sending message...");
        rabbitTemplate.convertAndSend(Application.queueName, "Hello from RabbitMQ!");
        receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
        context.close();
    }

}
```

在测试中runner会被mock掉，目的是为了接受者能够在隔离的环境下测试。

## 运行应用

`main()`方法通过创建一个 Spring 应用上下文来启动了程序。它会启动消息监听者容器，容器启动后会监听消息。这里自动执行了一个`Runner`类：它会从应用上下文中检查`RabbitTemplate`，之后会在"spring-boot"队列上发送"Hello from RabbitMQ"消息。最后它会关闭 Spring 应用上下文，然后程序就停止了。

### 构建一个可执行的JAR

你可以通过使用 Gradle 或者 Maven 命令行来运行一个应用。或者你可以先构建一个包含了所有依赖、类、和配置的可执行 JAR 文件，然后运行它。这使得在整个开发生命周期中，以及在不同的环境中，都很容易将应用程序进行部署、版本控制和服务发布。

如果你使用 Gradle ，你可以使用`./gradlew`来启动应用，或者你可以使用`./gradlew build`来构建一个 JAR 文件。之后，你可以通过运行 JAR 文件：

```shell
java -jar build/libs/gs-messaging-rabbitmq-0.1.0.jar
```

如果你使用 Maven ，你可以使用`./mvnw spring-boot:run`来运行应用。或者你可以使用`./mvnw clean package`来构建一个 JAR 包。之后你可以运行JAR文件：

```shell
java -jar target/gs-messaging-rabbitmq-0.1.0.jar
```

> 上面的过程是创建一个可以运行的JAR，如果你需要构建成一个WAR：[怎样构建WAR文件][2]

你应该可以看到下面的输出：

```
Sending message...
Received 
```

## 总结

恭喜！你已经使用 Spring 和 RabbitMQ 开发了一个简单的`发布-订阅应用`。你也可以使用 Spring 和 RabbitMQ 来做[更多的操作][23],上面的例子只是一个好的开始。

## 了解更多

下面的指南也非常有帮助：

*   [Messaging with Redis][15]

*   [Messaging with JMS][16]

*   [使用Spring Boot构建应用][17]

--------------------------------------------------------------------------------

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

[a]:
[1]:https://spring.io/understanding/AMQP
[2]:https://spring.io/guides/gs/convert-jar-to-war/
[3]:https://creativecommons.org/licenses/by-nd/3.0/
[4]:http://www.oracle.com/technetwork/java/javase/downloads/index.html
[5]:http://www.gradle.org/downloads
[6]:https://maven.apache.org/download.cgi
[7]:https://spring.io/guides/gs/sts
[8]:https://spring.io/guides/gs/intellij-idea/
[9]:https://github.com/spring-guides/gs-messaging-rabbitmq/archive/master.zip
[10]:https://spring.io/understanding/Git
[11]:https://github.com/spring-guides/gs-messaging-rabbitmq.git
[12]:https://spring.io/guides/gs/messaging-rabbitmq/#initial
[13]:https://spring.io/guides/gs/sts/
[14]:https://spring.io/guides/gs/intellij-idea
[15]:https://spring.io/guides/gs/messaging-redis/
[16]:https://spring.io/guides/gs/messaging-jms/
[17]:https://spring.io/guides/gs/spring-boot/
[18]:https://spring.io/understanding/POJO
[19]:https://spring.io/guides
[20]:https://spring.io/guides/gs/messaging-rabbitmq/#scratch
[21]:https://www.rabbitmq.com/download.html
[22]:https://docs.docker.com/compose/
[23]:https://docs.spring.io/spring-amqp/reference/html/_introduction.html#quick-tour
[24]:https://github.com/spring-guides/getting-started-guides/wiki
[25]:https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
[26]:https://maven.apache.org/
[27]:https://spring.io/guides/gs/maven
[28]:https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin
