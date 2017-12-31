# 创建一个多模块项目

> 原文：[Creating a Multi Module Project](https://spring.io/guides/gs/multi-module/)
>
> 译者：[qiushile](https://github.com/qiushile)
>
> 校对：

本指南将向您介绍如何使用Spring Boot创建多模块项目，其中有一个库jar和一个使用该库的主应用程序。 您也可以使用它来查看如何自行构建库（即不是应用程序的jar文件）。。

## 你会构建什么
您将设置一个简单的库jar，为简单的Hello World消息公开服务，然后将该服务包含在使用该库作为依赖项的Web应用程序中。 你可以使用这个指南学习如何用Maven和Gradle做同样的事情，但是对于一个真正的项目你可能会选择其中的一个。

## 你需要准备什么

* 大约15分钟时间

* 一个喜欢的文本编辑器或者IDE

* [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

* [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

* 你也可以直接导入代码到IDE:

  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)



## 怎样完成指南？

像大多数 Spring [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#scratch)。


**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/draft-gs-multi-module.git)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

    `git clone https://github.com/spring-guides/draft-gs-multi-module.git`
* 进入 `draft-gs-multi-module/initial`目录
* 跳过前面的部分[创建一个库项目](https://spring.io/guides/gs/multi-module/#initial) 

**当你完成之后**，你可以在`draft-gs-multi-module/complete`根据代码检查下结果。

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建， 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

## 创建一个根项目
####创建目录结构

在您选择的项目目录中，创建以下子目录结构; 例如， 在*nix系统种使用`mkdir library applicationon`：
```
└── library
└── application
```
在项目的根目录下，您需要建立一个构建系统，本指南将向您展示如何使用Maven或Gradle来构建。

##<div id="grandle_configuration"></div>多模块项目的Grandle配置

我们推荐使用 [Grandle wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)。所以可以通过从现有项目中复制或通过`wrapper`任务执行Gradle（按照Gradle文档中的说明）来安装wapper。应当让顶级目录看起来像这样：
```
└── library
└── application
└── gradlew
└── gradlew.bat
└── gradle
    └── wrapper
        └── gradle-wrapper.properties
        └── gradle-wrapper.jar
```
Windows（非Cygwin）用户执行`gradlew.bat`脚本来构建项目; 其他用户都使用`gradlew`脚本。然后添加一个`settings.gradle`到根目录来驱动顶层的构建：

`settings.gradle`

```
rootProject.name = 'gs-multi-module'

include 'library'
include 'application'
```

##<div id="maven_configuration"></div>多模块项目的Maven配置

我们推荐使用[Maven wrapper](https://github.com/takari/maven-wrapper)。所以可以通过从现有项目中复制或通过执行带有`io.takari:maven:wrapper`目标的Maven （按照wapper文档中的说明）来安装wrapper。应当让顶级目录看起来像这样：
```
└── library
└── application
└── mvnw
└── mvnw.cmd
└── .mvn
    └── wrapper
        └── maven-wrapper.properties
        └── maven-wrapper.jar
```

Windows（非Cygwin）用户执行`mvnw.cmd`脚本来构建项目; 其他用户都使用`mvnw`脚本。然后添加一个`pom.xml`到根目录来驱动顶层的构建：

`pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-multi-module</artifactId>
    <version>0.1.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>library</module>
        <module>application</module>
    </modules>

</project>
```

##创建一个库项目
####创建目录结构

在"library"目录中，创建以下子目录结构; 例如，在*nix系统中使用 `mkdir -p src/main/java/hello/serviceon`：
```
└── src
    └── main
        └── java
            └── hello
                └── service
```
现在我们需要配置一个构建工具（Maven或Gradle）。在这两种情况下需要注意，Spring Boot插件根本**不在**库项目中使用。插件的主要功能是为了库创建一个我们不需要（也不想要）的可执行的“über-jar”。为了让你快速入门，下面是完整的配置：

##[库的Grandle配置](#grandle_configuration)

##[库的Maven配置](#maven_configuration)

##创建一个服务组件
该库将提供一个可供依赖于该库的应用使用的`Service`类：

`library/src/main/java/hello/service/Service.java`

```
package hello.service;

import org.springframework.stereotype.Component;

@Component
public class Service {

    private final String message;

    public Service(String message) {
        this.message = message;
    }

    public String message() {
        return this.message;
    }
}
```

为了使它在标准的Spring Boot风格中可配置（通过`application.properties`），你还可以添加一个`@ConfigurationProperties`类

`library/src/main/java/hello/service/ServiceProperties.java`

```
package hello.service;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("service")
public class ServiceProperties {

    /**
     * A message for the service.
     */
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

并且提供一种将服务及其配置都包含在应用中的方式，还可以通过@Configuration：

`library/src/main/java/hello/service/ServiceConfiguration.java`

```
package hello.service;

import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(ServiceProperties.class)
public class ServiceConfiguration {
    @Bean
    public Service service(ServiceProperties properties) {
        return new Service(properties.getMessage());
    }
}
```

不必这样做：库可能只是提供纯Java API，并没有Spring特性。这种情况下，使用库的应用将需要自己提供配置。

##测试服务组件

你会想为你的库组件编写单元测试。如果你提供可重用的Spring配置作为库的一部分，你可能还想编写一个集成测试以确保配置能够正常工作。要做到这一点，可以使用JUnit和`@SpringBootTest`注解。例如：

`library/src/test/java/hello/service/ServiceTest.java`

```
package hello.service;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest("service.message=Hello")
public class ServiceTest {

    @Autowired
    private Service service;

    @Test
    public void contextLoads() {
        assertThat(service.message()).isNotNull();
    }

    @SpringBootApplication
    @Import(ServiceConfiguration.class)
    static class TestConfiguration {
    }

}
```


> 在上面的示例中，我们使用`@SpringBootTest`注释的默认属性配置了`service.message`来测试。**不**建议将`application.properties`放入库中，因为在使用它的应用程序的运行时可能会发生冲突（`application.properties`在路径中只会被加载一次）。你**可以**将`application.properties`放在测试类路径，但不要将其放入jar内，例如可以放在`src/test/resources`。

##创建应用项目

####创建目录结构
在“应用”目录中，创建以下子目录结构; 例如，在*nix系统使用`mkdir -p src/main/java/hello/app`：
```
└── src
    └── main
        └── java
            └── hello
                └── app
```

除非在应用程序中通过`@ComponentScan`默认包含库中的所有Spring组件，否则不要使用与库（或库包的父级）相同的包。现在来构建配置：

##[应用的Grandle配置](#grandle_configuration)

##[应用的Maven配置](#maven_configuration)

##编写应用

应用程序中的主类可以是一个使用库中的`Service`来呈现信息的`@RestController`。要做到这一点，你需要从库中`@Import`类`ServiceConfiguration`。例如：

`application/src/main/java/hello/app/DemoApplication.java`

```
package hello.app;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Import;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import hello.service.Service;
import hello.service.ServiceConfiguration;

@SpringBootApplication
@Import(ServiceConfiguration.class)
@RestController
public class DemoApplication {

    private final Service service;

    @Autowired
    public DemoApplication(Service service) {
        this.service = service;
    }

    @GetMapping("/")
    public String home() {
        return service.message();
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

`@SpringBootApplication` 是一个方便的注释，增加了以下内容：

* `@Configuration` 将该类标记为应用程序上下文的bean定义的来源。
  
* `@EnableAutoConfiguration` 告诉Spring Boot开始添加基于类路径设置，其他bean和各种属性设置的bean。
  
* 通常情况下你会为Spring MVC应用添加`@EnableWebMvc`，但是Spring Boot会自动在类路径中看到**spring-webmvc**时添加它。这将该应用标记为Web应用，并激活像设置`DispatcherServlet`这样的关键行为。
  
* `@ComponentScan`告诉Spring在包hello中查找其他组件、配置和服务，以便找到控制器。
  
`main()`方法使用Spring Boot的`SpringApplication.run()`方法来启动一个应用。你有没有注意到一行XML都没有？也没有**web.xml**文件。这个Web应用程序是100％纯Java，您不必处理配置任何管道或基础设施。

##创建`application.properties`文件

我们需要通过`application.properties`为库中的服务提供信息。在sources文件夹中，你只需要创建一个文件`src/main/resources/application.properties`：

`service.message=Hello World`

##测试应用

通过启动应用来测试端到端的结果。你可以很容易地在集成编译环境中启动应用，或者按照以下说明使用命令行。在浏览器中访问客户端应用`http://localhost:8080/`。在那应当能看到响应内显示的字符串`Hello World`。

####Gradle命令行来运行应用程序
####Maven命令行来运行应用程序

##总结

恭喜！您刚刚使用Spring Boot创建了一个可重用的库，然后用它来构建一个应用。




> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。





