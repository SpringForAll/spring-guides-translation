【chenzhijun 正在翻译.......】

## Messaging with RabbitMQ
============================================================

> 原文：[Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)
>
> 译者：[chenzhijun](https://github.com/chenzhijun)
>
> 校对：[校对者id-1](https://github.com/校对者id-1)，[校对者id-2](https://github.com/校对者id-2)，[校对者id-3](https://github.com/校对者id-3)

This guide walks you through the process of setting up a RabbitMQ AMQP server that publishes and subscribes to messages.

### What you’ll build

You’ll build an application that publishes a message using Spring AMQP’s RabbitTemplate and subscribes to the message on a [POJO][18] using MessageListenerAdapter.

### What you’ll need

*   About 15 minutes 大概15分钟时间

*   A favorite text editor or IDE

*   [JDK 1.8][4] or later  jdk版本1.6

*   [Gradle 2.3+][5] or [Maven 3.0+][6]

*   You can also import the code straight into your IDE:[Spring Tool Suite (STS)][7][IntelliJ IDEA][8]RabbitMQ server (installation instructions below)

### How to complete this guide

Like most Spring [Getting Started guides][19], you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To start from scratch, move on to [Build with Gradle][20].

To skip the basics, do the following:

*   [Download][9] and unzip the source repository for this guide, or clone it using [Git][10]: git clone [https://github.com/spring-guides/gs-messaging-rabbitmq.git][11]

*   cd into gs-messaging-rabbitmq/initial

*   Jump ahead to [Create a RabbitMQ message receiver][12].

When you’re finished, you can check your results against the code in gs-messaging-rabbitmq/complete.

### Build with Gradle

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Gradle][30] and [Maven][31] is included here. If you’re not familiar with either, refer to [Building Java Projects with Gradle][32] or [Building Java Projects with Maven][33].

#### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with mkdir -p src/main/java/hello on *nix systems:

```
└── src
    └── main
        └── java
            └── hello
```

#### Create a Gradle build file

Below is the [initial Gradle build file][34].

build.gradle

```
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

The [Spring Boot gradle plugin][35] provides many convenient features:

*   It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.

*   It searches for the public static void main() method to flag as a runnable class.

*   It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies][29]. You can override any version you wish, but it will default to Boot’s chosen set of versions.

### Build with Maven

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven][26] is included here. If you’re not familiar with Maven, refer to [Building Java Projects with Maven][27].

#### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with mkdir -p src/main/java/hello on *nix systems:

```
└── src
    └── main
        └── java
            └── hello
```

pom.xml

```
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

The [Spring Boot Maven plugin][28] provides many convenient features:

*   It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.

*   It searches for the public static void main() method to flag as a runnable class.

*   It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies][25]. You can override any version you wish, but it will default to Boot’s chosen set of versions.

### Build with your IDE

*   Read how to import this guide straight into [Spring Tool Suite][13].

*   Read how to work with this guide in [IntelliJ IDEA][14].

### Set up RabbitMQ broker

Before you can build your messaging application, you need to set up the server that will handle receiving and sending messages.

RabbitMQ is an AMQP server. The server is freely available at [http://www.rabbitmq.com/download.html][21]. You can download it manually, or if you are using a Mac with homebrew:

```
brew install rabbitmq
```

Unpack the server and launch it with default settings.

```
rabbitmq-server
```

You should see something like this:

```
            RabbitMQ 3.1.3\. Copyright (C) 2007-2013 VMware, Inc.
##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
##  ##
##########  Logs: /usr/local/var/log/rabbitmq/rabbit@localhost.log
######  ##        /usr/local/var/log/rabbitmq/rabbit@localhost-sasl.log
##########
            Starting broker... completed with 6 plugins.
```

You can also use [Docker Compose][22] to quickly launch a RabbitMQ server if you have docker running locally. There is a docker-compose.yml in the root of the "complete" project in Github. It is very simple:

docker-compose.yml

```
rabbitmq:
  image: rabbitmq:management
  ports:
    - "5672:5672"
    - "15672:15672"
```

With this file in the current directory you can run docker-compose up to get RabbitMQ running in a container.

### Create a RabbitMQ message receiver

With any messaging-based application, you need to create a receiver that will respond to published messages.

src/main/java/hello/Receiver.java

```
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

The Receiver is a simple POJO that defines a method for receiving messages. When you register it to receive messages, you can name it anything you want.

|  | For convenience, this POJO also has a `CountDownLatch`. This allows it to signal that the message is received. This is something you are not likely to implement in a production application. |

### Register the listener and send a message

Spring AMQP’s RabbitTemplate provides everything you need to send and receive messages with RabbitMQ. Specifically, you need to configure:

*   A message listener container

*   Declare the queue, the exchange, and the binding between them

*   A component to send some messages to test the listener

|  | Spring Boot automatically creates a connection factory and a RabbitTemplate, reducing the amount of code you have to write. |

You’ll use RabbitTemplate to send messages, and you will register a Receiver with the message listener container to receive messages. The connection factory drives both, allowing them to connect to the RabbitMQ server.

src/main/java/hello/Application.java

```
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

@SpringBootApplication is a convenience annotation that adds all of the following:

*   @Configuration tags the class as a source of bean definitions for the application context.

*   @EnableAutoConfiguration tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.

*   Normally you would add @EnableWebMvc for a Spring MVC app, but Spring Boot adds it automatically when it sees spring-webmvc on the classpath. This flags the application as a web application and activates key behaviors such as setting up a DispatcherServlet.

*   @ComponentScan tells Spring to look for other components, configurations, and services in the hello package, allowing it to find the controllers.

The main() method uses Spring Boot’s SpringApplication.run() method to launch an application. Did you notice that there wasn’t a single line of XML? No web.xml file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

The bean defined in the listenerAdapter() method is registered as a message listener in the container defined in container(). It will listen for messages on the "spring-boot" queue. Because the Receiver class is a POJO, it needs to be wrapped in the MessageListenerAdapter, where you specify it to invoke receiveMessage.

|  | JMS queues and AMQP queues have different semantics. For example, JMS sends queued messages to only one consumer. While AMQP queues do the same thing, AMQP producers don’t send messages directly to queues. Instead, a message is sent to an exchange, which can go to a single queue, or fanout to multiple queues, emulating the concept of JMS topics. For more, see [Understanding AMQP][1]. |

The message listener container and receiver beans are all you need to listen for messages. To send a message, you also need a Rabbit template.

The queue() method creates an AMQP queue. The exchange() method creates a topic exchange. The binding() method binds these two together, defining the behavior that occurs when RabbitTemplate publishes to an exchange.

|  | Spring AMQP requires that the `Queue`, the `TopicExchange`, and the `Binding` be declared as top level Spring beans in order to be set up properly. |

### Send a Test Message

Test messages are sent by a CommandLineRunner, which also waits for the latch in the receiver and closes the application context:

src/main/java/hello/Runner.java

```
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

The runner can be mocked out in tests, so that the receiver can be tested in isolation.

### Run the Application

The main() method starts that process by creating a Spring application context. This starts the message listener container, which will start listening for messages. There is a Runner bean which is then automatically executed: it retrieves the RabbitTemplate from the application context and sends a "Hello from RabbitMQ!" message on the "spring-boot" queue. Finally, it closes the Spring application context and the application ends.

### Build an executable JAR

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using ./gradlew bootRun. Or you can build the JAR file using ./gradlew build. Then you can run the JAR file:

```
java -jar build/libs/gs-messaging-rabbitmq-0.1.0.jar
```

If you are using Maven, you can run the application using ./mvnw spring-boot:run. Or you can build the JAR file with ./mvnw clean package. Then you can run the JAR file:

```
java -jar target/gs-messaging-rabbitmq-0.1.0.jar
```

|  | The procedure above will create a runnable JAR. You can also opt to [build a classic WAR file][2]instead. |

You should see the following output:

```
Sending message...
Received 
```

### Summary

Congratulations! You’ve just developed a simple publish-and-subscribe application with Spring and RabbitMQ. There’s [more you can do with Spring and RabbitMQ][23] than what is covered here, but this should provide a good start.

### See Also

The following guides may also be helpful:

*   [Messaging with Redis][15]

*   [Messaging with JMS][16]

*   [Building an Application with Spring Boot][17]

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines][24].

|  | All guides are released with an ASLv2 license for the code, and an [Attribution, NoDerivatives creative commons license][3] for the writing. |

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
[29]:https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml
[30]:http://gradle.org/
[31]:https://maven.apache.org/
[32]:https://spring.io/guides/gs/gradle
[33]:https://spring.io/guides/gs/maven
[34]:https://github.com/spring-guides/gs-spring-boot/blob/master/initial/build.gradle
[35]:https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin

这是我的测试
