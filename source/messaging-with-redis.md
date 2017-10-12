
# Redis消息交互

> 原文：[Messaging With Redis](https://spring.io/guides/gs/messaging-redis/) （这里为示例，译者需根据具体文章修改）
>
> 译者：linzx2015
>
> 校对：

本指南将会引导你完成使用Spring Data Redis 发布和订阅通过redis发送的消息的过程。

## 你将构建什么

你将构建一个使用<font color=blue size=2>StringRedisTemplate</font>发布字符串消息并使用<font color=blue size=2>MessageListenerAdapter</font>进行[POJO](https://spring.io/understanding/POJO)订阅它的应用程序。
使用Spring Data Redis作为发布消息的手段可能听起来会很奇怪，但是你会发现，Redis不仅提供NoSQL数据存储，也提供了消息交互的体系。

## 你需要准备哪些工作

* 大概15分钟

* 一款钟爱的文本编辑器或者IDE

* [JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或者更高的版本

* [Gradle 2.3+](https://gradle.org/install/)或者[Maven 3.0+](https://maven.apache.org/download.cgi)

* 也可以将代码直接导入到你的IDE中

    * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
    
    * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    
       * Redis server (安装说明如下:)

## 如何完成本指南
跟大多数[Spring入门指南](https://spring.io/guides)一样,你可以从头开始然后一步步完成，
也可以直接绕过对你而言已经很熟悉的基本步骤。无论哪种方式,你都可以运行起来代码。

`从头开始的话`,请移步[通过Gradle构建](https://spring.io/guides/gs/messaging-redis/#scratch)。

`跳过基础步骤的话`,请按如下步骤进行:

* [下载](https://github.com/spring-guides/gs-messaging-redis/archive/master.zip)并解压缩本指南的源代码仓库,或者使用
[Git](https://spring.io/understanding/Git)命令:<font color=blue size=2>git clone</font> https://github.com/spring-guides/gs-messaging-redis.git

* 进入目录 <font color=blue size=2>gs-messaging-redis/initial</font>

* 直接跳转到 [Create a Redis message receiver](https://spring.io/guides/gs/messaging-redis/#initial)。

`结束时`，你可以在<font color=blue size=2>gs-messaging-redis/complete</font>里面根据代码检查结果。

## \>通过Gradle构建

## \>通过Maven构建

## \>通过你的IDE构建

## 搭建一个redis服务器

在你建立好可以进行消息传递的应用之前,你需要先搭建好可以接受和发送消息的服务器。

Redis是一个开源的、BSD许可的键值对形式数据存储数据库,还附带消息系统。Redis服务器可以在[http://redis.io/download](http://redis.io/download)上免费获得,
你可以手动下载,或者使用Mac自己搭建:

<table>
  <tr>
   <td bgcolor=#D9D9D9 width=800 height=30>
    &nbsp;&nbsp;&nbsp;brew install redis
   </td>
 </tr>
</table>

一旦你解压完Redis后,你可以使用默认设置来启动它。
<table>
  <tr>
   <td bgcolor=#D9D9D9 width=800 height=30>
    &nbsp;&nbsp;&nbsp;redis-server
   </td>
 </tr>
</table>

你应该会看到如下的信息:
```
<pre>[35142] 01 May 14:36:28.939 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
[35142] 01 May 14:36:28.940 * Max number of open files set to 10032
                _._
              _.-``__ ''-._
        _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 64 bit
    .-`` .-```.  ```\/    _.,_ ''-._
  (    '      ,       .-`  | `,    )     Running in stand alone mode
  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
  |    `-._   `._    /     _.-'    |     PID: 35142
    `-._    `-._  `-./  _.-'    _.-'
  |`-._`-._    `-.__.-'    _.-'_.-'|
  |    `-._`-._        _.-'_.-'    |           http://redis.io
    `-._    `-._`-.__.-'_.-'    _.-'
  |`-._`-._    `-.__.-'    _.-'_.-'|
  |    `-._`-._        _.-'_.-'    |
    `-._    `-._`-.__.-'_.-'    _.-'
        `-._    `-.__.-'    _.-'
            `-._        _.-'
                `-.__.-'

[35142] 01 May 14:36:28.941 # Server started, Redis version 2.6.12
[35142] 01 May 14:36:28.941 * The server is now ready to accept connections on port 6379</pre>
```
## 创建一个Redis消息接收者
在任何基于消息传递的应用程序中,都会有消息发布者和消息接受者。 要创建消息接收者，请实现一个接收方来响应消息：

<font color=#3366ff size=3>&nbsp;&nbsp;&nbsp;src/main/java/hello/Receiver.java</font>

```
package hello;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

public class Receiver {
    private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

    private CountDownLatch latch;

    @Autowired
    public Receiver(CountDownLatch latch) {
        this.latch = latch;
    }

    public void receiveMessage(String message) {
        LOGGER.info("Received <" + message + ">");
        latch.countDown();
    }
}
```
接收着是一个简单的POJO，它定义了一种接收消息的方法。你会看到当你把接受者注册为消息监听器时,
</br>你将可以按你需要的方法命名消息处理的方法。

>为了演示的目的,它会通过同步工具类自动装配好构造函数。这样,它就可以在消息到来时发送信号。

## 注册监听和发送消息
Spring Data Redis 提供了所有你需要Redis发送和接受消息的组件,具体来说，你需要配置：

* 一个连接工厂
* 一个消息监听的容器
* 一个Redis模板

你将使用Redis模板发送消息，你将使用消息侦听器容器注册`Receiver`，以便它接收消息。连接工厂驱动模板和消息侦听器容器,
</br>使其能够连接到Redis服务器。

这个示例使用的是Spring Boot默认的`RedisConnectionFactory`,该对象是基于`Jedis` Redis库的`JedisConnectionFactory`实例。
</br>连接工厂将被注入消息侦听器容器和Redis模板。

<font color=#3366ff size=3>&nbsp;&nbsp;&nbsp;src/main/java/hello/Application.java</font>
```
package hello;

import java.util.concurrent.CountDownLatch;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.listener.PatternTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;

@SpringBootApplication
public class Application {

    private static final Logger LOGGER = LoggerFactory.getLogger(Application.class);

	@Bean
	RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
			MessageListenerAdapter listenerAdapter) {

		RedisMessageListenerContainer container = new RedisMessageListenerContainer();
		container.setConnectionFactory(connectionFactory);
		container.addMessageListener(listenerAdapter, new PatternTopic("chat"));

		return container;
	}

	@Bean
	MessageListenerAdapter listenerAdapter(Receiver receiver) {
		return new MessageListenerAdapter(receiver, "receiveMessage");
	}

	@Bean
	Receiver receiver(CountDownLatch latch) {
		return new Receiver(latch);
	}

	@Bean
	CountDownLatch latch() {
		return new CountDownLatch(1);
	}

	@Bean
	StringRedisTemplate template(RedisConnectionFactory connectionFactory) {
		return new StringRedisTemplate(connectionFactory);
	}

	public static void main(String[] args) throws InterruptedException {

		ApplicationContext ctx = SpringApplication.run(Application.class, args);

		StringRedisTemplate template = ctx.getBean(StringRedisTemplate.class);
		CountDownLatch latch = ctx.getBean(CountDownLatch.class);

		LOGGER.info("Sending message...");
		template.convertAndSend("chat", "Hello from Redis!");

		latch.await();

		System.exit(0);
	}
}
```

在`listenerAdapter`方法中定义的bean在容器中定义的消息侦听器容器中注册为消息侦听器，并且将在“聊天”的主题上收听消息。
因为Receiver类是POJO，它需要被包装在一个消息侦听器适配器中，该适配器实现了addMessageListener()所需的MessageListener接口。
</br> 消息侦听器适配器还配置为当消息到达时在Receiver上调用receiveMessage()方法。

连接工厂和消息侦听器容器都是你需要用来监听消息的。要发送消息，你还需要一个Redis模板。在这里,它是一个配置为
</br>`StringRedisTemplate的Bean`,`RedisTemplate`的实现集中在Redis的常见用途，其中的两个键和值都是“String”。

main()方法通过创建一个绝无仅有的Spring应用程序上下文,然后通过应用程序上下文来启动消息侦听器容器,</br>
并且消息侦听器容器对象开始监听消息。然后main()方法从应用程序上下文中检索StringRedisTemplate对象,</br>
并使用它发送一个“Hello from Redis！”消息到“聊天”主题。最后，它关闭了Spring应用程序上下文，应用程序结束。

## 构建一个可执行的jar包
你可以在Gradle或Maven通过命令行运行应用程序。或者,你可以构建一个包含所有必需依赖项,类和资源文件的单个可执行JAR文件,并运行该文件。
</br>这使得在整个开发生命周期中,在到不同的环境下,更容易将其不同版本的服务部署成为一个应用程序。

如果你正在使用Gradle,你可以通过使用命令 `./gradlew bootRun` 来运行程序。或者你可以使用 `./gradlew build`将其构建成JAR包,然后再执行该JAR文件:

```
java -jar build/libs/gs-messaging-redis-0.1.0.jar
```
如果你正在使用Maven,你可以通过使用命令 `./mvnw spring-boot:run` 来运行程序。
或者你可以使用 `./mvnw clean package`将其构建成JAR包,然后再执行该JAR文件:
```
java -jar target/gs-messaging-redis-0.1.0.jar
```
>上面的过程将创建一个可运行的JAR文件。你也可以选择构建成一个[经典的WAR](https://spring.io/guides/gs/convert-jar-to-war/)文件。

你应该会看到如下的输出:
```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.7.RELEASE)

2014-04-18 08:03:34.032  INFO 47002 --- [           main] hello.Application                        : Starting Application on retina with PID 47002 (/Users/gturnquist/src/spring-guides/gs-messaging-redis/complete/target/classes started by gturnquist)
2014-04-18 08:03:34.062  INFO 47002 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@7a53c84a: startup date [Fri Apr 18 08:03:34 CDT 2014]; root of context hierarchy
2014-04-18 08:03:34.326  INFO 47002 --- [           main] o.s.c.support.DefaultLifecycleProcessor  : Starting beans in phase 2147483647
2014-04-18 08:03:34.357  INFO 47002 --- [           main] hello.Application                        : Started Application in 0.605 seconds (JVM running for 0.899)
2014-04-18 08:03:34.357  INFO 47002 --- [           main] hello.Application                        : Sending message...
2014-04-18 08:03:34.370  INFO 47002 --- [    container-2] hello.Receiver                           : Received <Hello from Redis!>
2014-04-18 08:03:34.379  INFO 47002 --- [       Thread-1] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@7a53c84a: startup date [Fri Apr 18 08:03:34 CDT 2014]; root of context hierarchy
2014-04-18 08:03:34.380  INFO 47002 --- [       Thread-1] o.s.c.support.DefaultLifecycleProcessor  : Stopping beans in phase 2147483647
```

## 总结
恭喜！ 你刚刚使用Spring和Redis开发了一个简单的发布和订阅应用程序。

> 获取[Redis支持](http://gopivotal.com/products/redis)  

## 也可以看看

以下的指南可能也很有帮助的:

* [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)
* [Messaging with JMS](https://spring.io/guides/gs/messaging-jms/)
* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
想写一个新的指南或贡献一个现有的？ 查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



