# 中文标题

> 原文：[Messaging with JMS](https://spring.io/guides/gs/messaging-jms/)
>
> 译者：[zaixiandemiao](https://github.com/zaixiandemiao)
>
> 校对：

本指南将帮助你了解如何使用JMS的broker发布和订阅消息。

## 你将构建什么

你将构建一个应用来使用Spring的`JmsTemplate`发布一条简单的消息，并使用`@JmsListener`注解来修饰一个受管理的bean的方法来订阅这条消息。

## 开始前你需要准备

- 约15分钟
- 一个你喜欢的文本编辑器或IDE
- [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)≥1.8
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以将代码直接导入你的IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成这篇指南

像大多数的Spring指南 [Getting Started guides](https://spring.io/guides),你可以从零开始，完成每一步，也可以跳过已经熟悉的基本设置。 无论哪种方式，你都会得到一份可用的代码。

**若从基础开始**，查看[使用 Gradle 构建](#scratch)。

**若跳过基础部分**，按如下流程操作：

- [下载](https://github.com/spring-guides/gs-actuator-service/archive/master.zip) 并解压本指南的源码存档，或使用 [Git](https://spring.io/understanding/Git) 克隆：`git clone https://github.com/spring-guides/gs-actuator-service.git`

- 进入 `gs-messaging-jms/initial` 目录
- 跳转到[创建一个消息接收者](#messageReceiver).

**当你完成之后**，你可以和 `gs-messaging-jms/complete` 的代码对比检查结果。

<h2 id="scratch"> 使用 Gradle 构建 </h2>

首先你得编写一个基础构建脚本。在使用 Spring 构建应用的时候，你可以使用任何你喜欢的构建系统，这里提供一份你可能会用到的用 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你的项目根目录，创建如下的子目录结构；例如，在 \*nix 系统上使用 `mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```


### 创建Gradle构建文件

下面是一份 [Gradle 初始构建文件](https://github.com/spring-guides/gs-accessing-data-rest/blob/master/initial/build.gradle)

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
    baseName = 'gs-accessing-data-rest'
    version = '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

-  将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，这使得执行和分发你的服务变得更加方便。
- 搜索 `public static void main()` 方法并且将它标记为可执行类。
- 提供了内置的依赖解析器，可以将 [Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 依赖的版本号设置为最新。你可以用你想要的任意版本进行改写，不过这会是 Spring Boot 的默认版本设置。


## 使用 Maven 构建
首先你得编写一个基础构建脚本。在使用 Spring 构建应用的时候，你可以使用任何你喜欢的构建系统，这里提供一份你可能会用到的用 [Maven](https://maven.apache.org/) 构建的代码。 如果你对 Maven 不熟悉, 你可以先去参考[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建目录结构
在你的项目根目录，创建如下的子目录结构；例如，在 \*nix 系统上使用 `mkdir -p src/main/java/hello`:

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
    <artifactId>gs-actuator-service</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多非常方便的功能：

-  将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，这使得执行和分发你的服务变得更加方便。
- 搜索 `public static void main()` 方法并且将它标记为可执行类。
- 提供了内置的依赖解析器，可以将 [Spring Boot](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 依赖的版本号设置为最新。你可以用你想要的任意版本进行改写，不过这会是 Spring Boot 的默认版本设置。

## 使用你的 IDE 构建
- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读在 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 中使用本指南。

<h2 id="messageReceiver">创建一个消息接收者</h2>

Spring提供了将消息发布为任意[POJO](https://spring.io/understanding/POJO)的方法。

在本指南中，你将学会如何将一条消息发送到一个JMS message broker。首先，让我们创建一个非常简单的POJO来表示一条email消息的详细信息。注意，我们并不是发送一封email消息。我们只是简单的发送从一个地方到另一个地方所传递消息的内容详情。

`src/main/java/hello/Email.java`

```java
package hello;

public class Email {

    private String to;
    private String body;

    public Email() {
    }

    public Email(String to, String body) {
        this.to = to;
        this.body = body;
    }

    public String getTo() {
        return to;
    }

    public void setTo(String to) {
        this.to = to;
    }

    public String getBody() {
        return body;
    }

    public void setBody(String body) {
        this.body = body;
    }

    @Override
    public String toString() {
        return String.format("Email{to=%s, body=%s}", getTo(), getBody());
    }

}
```

这个POJO类非常简单，包含了两个属性，to和body以及相应的getter、setter方法。

从这里起，你可以定义一个消息接受者。

`src/main/java/hello/Receiver.java`

```java
package hello;

import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

    @JmsListener(destination = "mailbox", containerFactory = "myFactory")
    public void receiveMessage(Email email) {
        System.out.println("Received <" + email + ">");
    }

}
```

`Receiver`也可被称为一个 **消息驱动POJO**。正如你在上面代码中看到的，不需要实现任何特定的接口或者重写某个具有特定签名的方法。另外，这个方法具有[]非常灵活的签名](https://docs.spring.io/spring/docs/current/spring-framework-reference/#jms-annotated-method-signature)。尤其需要注意的是，这个类不需要import任何JMS的API工具类。

`JmsListener`注解定义了这个方法应当监听的`目标地址`的名称和用于创建位于消息监听器下的容器的`JmsListenerContainerFactory`的引用。严格来说，除非你需要自定义container构建的方法，否则最后一个属性是不需要的，Spring Boot会注册一个默认的工厂类。

[参考文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/#jms-annotated)包含了更多的使用细节。

## 使用Spring发送和接收JMS消息

下一步，连通发送者和接受者。

`src/main/java/hello/Application.java`

```Java
package hello;

import javax.jms.ConnectionFactory;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jms.DefaultJmsListenerContainerFactoryConfigurer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.config.DefaultJmsListenerContainerFactory;
import org.springframework.jms.config.JmsListenerContainerFactory;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.support.converter.MappingJackson2MessageConverter;
import org.springframework.jms.support.converter.MessageConverter;
import org.springframework.jms.support.converter.MessageType;

@SpringBootApplication
@EnableJms
public class Application {

    @Bean
    public JmsListenerContainerFactory<?> myFactory(ConnectionFactory connectionFactory,
                                                    DefaultJmsListenerContainerFactoryConfigurer configurer) {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        // This provides all boot's default to this factory, including the message converter
        configurer.configure(factory, connectionFactory);
        // You could still override some of Boot's default if necessary.
        return factory;
    }

    @Bean // Serialize message content to json using TextMessage
    public MessageConverter jacksonJmsMessageConverter() {
        MappingJackson2MessageConverter converter = new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        return converter;
    }

    public static void main(String[] args) {
        // Launch the application
        ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);

        JmsTemplate jmsTemplate = context.getBean(JmsTemplate.class);

        // Send a message with a POJO - the template reuse the message converter
        System.out.println("Sending an email message.");
        jmsTemplate.convertAndSend("mailbox", new Email("info@example.com", "Hello"));
    }

}
```

`@SpringBootApplication`可以方便的添加下列所有的注解:
- `@Configuration`标记这个类为一个定义了应用上下文的bean
- `@EnableAutoConfiguration`告诉Spring Boot根据设置的classpath、其他的bean和多样的属性设置来添加beans。
- 通常你会为Spring MVC项目添加`@EnableWebMvc`注解，但是Spring Boot在看到classpath中的spring-webmvc后会自动添加它。这将应用标志为一个web应用并启用了关键的特性，如设置一个`DispatcherServlet`
- `@ComponentScan`告诉Spring去寻找`hello` package中其他的components， configurations，services，也允许它去寻找controllers。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法来启动一个应用。你注意到这里没有任何一行xml配置吗?也没有web.xml文件。这个web应用是100%纯Java的，且你不需要处理任何的管道或者基础设置的配置。

`@EnableJms`触发了对使用了`@JmsListener`注解修饰过的方法的自动发现，并根据该方法来创建消息监听容器。

为了清晰起见，我们也定义了一个叫做`myFactory`的bean，它被上面使用`JmsListener`注解修饰的receiver所引用。因为我们使用了`DefaultJmsListenerContainerFactoryConfigurer`来作为Spring Boot底层基础设施的支持，`JmsMessageListenerContainer`将会被唯一的表示为Spring Boot默认创建的容器。

默认的`MessageConverter`只能转换基本的类型(如`String`,`Map`,`Serializable`)，我们的`Email`没有特意的实现`Serializable`接口。我们希望使用Jackson来将内容序列化为文本形式的json信息(比如说`TextMessage`)。Spring Boot会检测一个`MessageConverter`的存在，并把它和默认的`JmsTemplate`以及任何`DefaultJmsListenerContainerFactoryConfigurer`创建的`JmsListenerContainerFactory`配合使用。

`JmsTemplate`使得发送一条消息到一个JMS目的地变得非常简单。在`main` runner方法中，在基本的启动设置之后，你可以仅仅使用`jmsTemplate`来发送一个`Email` POJO.因为我们自定义的`MessageConverter`已经被自动的用于消息的转换，一个json 文档只会被生成为一个`TextMessage`.

`JmsTemplate`和`ConnectionFactory`是两个你没有看到的bean,它们会被Spring Boot自动创建。在本例中，ActiveMQ broker在应用中内置地运行着。

默认情况下，当设置**pubSubDomain**为false时，Spring Boot会创建一个配置为[发送消息到队列](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#jms-destinations)的`JmsTemplate`。`JmsMessageListenerContainer`也会被同样的配置。要修改它，通过Spring Boot的属性设置(`application.properties`配置文件或环境变量)，设置`spring.jms.isPubSubDomain=true`.之后，确保接收容器也有相同的设置。

> Spring的`JmsTemplate`可以直接通过它的`receive`接收消息，但是这将是一个同步的过程，也就是说会进行阻塞。这也是为什么我们推荐你使用一个像`DefaultMessageListenerContainer`这样的监听容器，并配合一个基于缓存的连接工厂。这样你就可以异步的消费消息，而它的效率取决于你的最大连接数。

## 构建一个可执行的JAR包

你可以在命令行使用Gradle或者Maven来运行应用。或者你可以构建一个单独的包含了所有必要的依赖、classes文件和resource文件的可执行jar文件，并运行它。这将使得你的整个开发流程中的搬运、版本控制、发布为service更加容易，且可以达到不同环境的兼容运行。

如果你使用的是Gradle,你可以使用`./gradlew bootRun`来运行这个应用。或者你可以使用`./gradlew build`来构建一个可执行的JAR文件。之后，你可以通过下面的命令来运行这个JAR文件。

`java -jar build/libs/gs-messaging-jms-0.1.0.jar`

如果你使用的是Maven,你可以使用`./mvnw spring-boot:run`来运行这个应用。或者你可以使用`./mvnw clean package`来构建可执行JAR文件，并使用下面的命令来运行JAR文件。

`java -jar target/gs-messaging-jms-0.1.0.jar`

> 上面的步骤将创建一个可运行的JAR文件。你可以选择参考[构建一个经典的WAR包](https://spring.io/guides/gs/convert-jar-to-war/)来进行构建。

当它运行的时候，除去所有的logging输出，你将看到下面的打印结果。

```
Sending an email message.
Received <Email{to=info@example.com, body=Hello}>
```

## 总结

恭喜！你刚刚开发了基于JMS的消息发布者和订阅者。

如果你想写一个新的教程或者对我们的的教程有任何建议，可以看我们的[贡献流程](https://github.com/SpringForAll/spring-guides-translation)


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
