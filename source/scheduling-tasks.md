## Scheduling Tasks

标签（空格分隔）： Spring

---

> 原文：[Scheduling Tasks](https://spring.io/guides/gs/scheduling-tasks/)

> 译者：[rhwayfun][1]

> 校对：[codedrinker][2]

## 开始使用定时任务

本指南将一步步引导您如何在Spring中使用定时任务。

### 完成什么
构建一个应用，实现的功能为每5秒打印出当前时间。这点可以通过Spring注解`@Scheduled`完成。

### 准备什么

 - 大约需要15分钟
 - 一个您喜爱的文本编辑器或者IDE（集成开发工具）
 - [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或更高版本
 - [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
 - 您也可以直接导入代码到IDE中：
    - [Spring Tool Suite (STS)][3]
    - [IntelliJ IDEA][4]

### 如何完成这份指南
和其他[Spring使用指南][5]一样，你可以从零开始完成每一步，或者略过你熟悉的一些基本步骤。不管哪种方式你最终都会得到一份工作代码。

如果从零开始，请移步Build with Gradle。

如果打算跳过基本步骤，按照如下操作：

 - [下载][6]并解压这份指南的源码包，或者使用[Git][7]克隆：
    `git clone https://github.com/spring-guides/gs-scheduling-tasks.git`
 - 进入目录 `gs-scheduling-tasks/initial`
 - 跳到 `创建定时任务章节`

当你完成以上操作，你可以在 `gs-scheduling-tasks/complete` 根据代码检查结果。


### 创建定时任务
既然已经搭建好了项目，下面就可以开始创建定时任务了。

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

`@Scheduled`注解定义一个方法的执行周期。**注意**：这个例子使用了`fixedRate`，
它指定了每次方法调用开始执行的间隔。有[其他的选项][8]，比如`fixedDelay`，它指定了从每次方法调用结束开始计算时间的调用间隔。你也可以[使用`@Scheduled(cron=". . .")`表达式实现更复杂的定时任务][9]

### 开启定时功能
虽然计划任务可以嵌入在Web应用程序和WAR包中，通过创建一个独立的应用程序，下面演示了一种更简单的方法。 您将所有内容都打包在一个可执行的JAR中，通过Java的`main()`方法就可以执行。

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
`@SpringBootApplication`是一个方便的注解，组合了以下注解：

 - `@Configuration` 将类标记为应用上下文获取bean定义的地方
 - `@EnableAutoConfiguration` 告诉Spring Boot从类路径、其他bean和属性配置中添加bean
 - 正常情况下如果是基于Spring MVC构建的应用需要添加注解`@EnableWebMvc`，但是Spring Boot当发现类路径下有**spring-webmvc**依赖时会自动添加这个注解。这个注解的作用标记一个应用为Web应用并且会自动激活一些功能，比如创建`DispatcherServlet`
 - `@ComponentScan` 告诉Spring在`hello`包下扫描其他的组件、配置和服务，从而让Spring找到Controller。

`main()`方法使用Spring Boot的`SpringApplication.run()`启用一个应用。不知道你是否注意到没有一行**XML**配置，也没有**web.xml**文件。这个web应用100%使用Java实现，你不需要任何添加额外的配置。

`@EnableScheduling`注解会在后台创建一个任务执行器，没有它定时任务无法正常工作。

**构建一个可执行JAR包**

你可以使用Maven或者Gradle在命令行中启动这个应用。也可以构建一个包含所有依赖、类文件和资源文件的可执行JAR包，然后运行它。这使得在整个开发生命周期中，在不同的环境部署一个服务变得简单了。

如果你使用的是Gradle，可以通过运行脚本`./gradlew bootRun`来启动应用。或者在Maven使用`./mvnw clean package`打包成JAR文件，然后运行：

`java -jar target/gs-scheduling-tasks-0.1.0.jar`

> 上面的程序会生成一个可执行的JAR，你也可以选择构建传统的WAR包。

从日志输出可以看到这是在后台的线程在输出，你应该可以看到你的定时任务是5秒输出一次的。

```
[...]
2016-08-25 13:10:00.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:00
2016-08-25 13:10:05.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:05
2016-08-25 13:10:10.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:10
2016-08-25 13:10:15.143  INFO 31565 --- [pool-1-thread-1] hello.ScheduledTasks : The time is now 13:10:15
```

### 总结
恭喜！ 您创建了一个有定时任务功能的应用程序。可以发现，实际代码比构建文件更少！ 这份代码适用于任何类型的应用。
 
> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可][10]协议进行许可。


  [1]: https://github.com/happyxiaofan
  [2]: https://github.com/codedrinker
  [3]: https://spring.io/guides/gs/sts
  [4]: https://spring.io/guides/gs/intellij-idea/
  [5]: https://spring.io/guides
  [6]: https://github.com/spring-guides/gs-scheduling-tasks/archive/master.zip
  [7]: https://spring.io/understanding/Git
  [8]: https://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html#scheduling-annotation-support-scheduled
  [9]: https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html
  [10]: http://creativecommons.org/licenses/by-nc-sa/4.0/