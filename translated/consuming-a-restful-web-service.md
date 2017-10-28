# 使用RESTful Web服务

> 原文：[Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/)
>
> 译者：[hapihapidoge](https://github.com/hapihapidoge)
>
> 校对：[maskwang520](https://github.com/maskwang520)

本指南将引导您完成创建一个使用RESTful Web服务的应用程序的过程。

## 您将构建什么

您将构建一个使用Spring RestTemplate的应用程序，并通过访问 http://gturnquist-quoters.cfapps.io/api/random 来获取一段 Spring Boot 随机字符串。

## 您需要准备什么

- 大约15分钟
- 一个喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)  或1.8以上
- [Grandle2.3以上](http://www.gradle.org/downloads) 或 [Maven3.0以上](https://maven.apache.org/download.cgi)
- 您也可以直接将代码引入您的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成该指南

与大多数Spring“入门指南”指南一样，您可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。 无论哪种方式，<u>你都会得到工作代码</u>。

从头开始，请参考 [Build with Gradle](https://spring.io/guides/gs/consuming-rest/#scratch);

要跳过基本操作，请执行以下操作：

- 下载并解压本指南引用的源代码，或使用Git克隆:git clone https://github.com/spring-guides/gs-consuming-rest.git
- 使用cd命令进入gs-consuming-rest/initial 目录
- 获取REST资源

完成后，您可以根据gs-consumption-rest / complete目录中的代码检查结果。

## 通过Gradle构建

首先你设置一个基本的构建脚本。 在使用Spring构建应用程序时，您可以使用任何您喜欢的构建系统，但是您需要使用与Gradle和Maven一起使用的代码。 如果您还不熟悉，请参阅 [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) or [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构; 例如， 在 *nix系统上使用命令 mkdir -p src / main / java / hello 创建

```
└── src
    └── main
        └── java
            └── hello
```

### 创建一个Gradle构建文件

以下是原始的Gradle构建文件。

`build.gradle`

```groovy
buildscript {
    repositories {
        mavenCentral()
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
    baseName = 'gs-consuming-rest'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter")
    compile("org.springframework:spring-web")
    compile("com.fasterxml.jackson.core:jackson-databind")
    testCompile("junit:junit")
}
```

Spring Boot gradle插件提供了许多方便的功能：

- 它收集类路径上的所有jar包，并构建一个可运行的“über-jar”，这样可以更方便地执行和传输您的服务。
- 它搜索public static void main（）方法来标记为可运行类。
- 它提供了一个内置的依赖解析器，使版本号与Spring Boot 依赖的版本相匹配。 您可以覆盖任何您想要的版本，但它将默认为Boot所选择的一组版本。

## 通过Maven构建

首先你设置一个基本的构建脚本。 在使用Spring构建应用程序时，您可以使用任何构建系统，但您需要使用与Maven一起使用的代码。 如果您不熟悉Maven，请参阅[Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构; 例如，在 * nix系统上使用命令 mkdir -p src / main / java / hello创建 

```
└── src
    └── main
        └── java
            └── hello
```

`pom.xml`

```Xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-consuming-rest</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
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

Spring Boot Maven插件提供了许多方便的功能：

- 它收集类路径上的所有jar包，并构建一个可运行的“über-jar”，这样可以更方便地执行和传输您的服务。
- 它搜索public static void main（）方法来标记为可运行类。
- 它提供了一个内置的依赖解析器，使版本号与Spring Boot 依赖的版本相匹配。 您可以覆盖任何您想要的版本，但它将默认为Boot所选择的一组版本。

## 通过IDE构建

- 在 [Spring Tool Suite](https://spring.io/guides/gs/sts/)中构建。
- 在 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)构建。

## 获取REST资源

完成项目设置后，您可以创建一个简单的应用程序来使用RESTful服务。

一个RESTful服务已经在http://gturnquist-quoters.cfapps.io/api/random上启动起来了。 它随机地获取 Spring Boot 字符串，并将其作为JSON文档返回。

如果您通过Web浏览器或curl 请求该URL，您将收到一个如下所示的JSON文档：

```json
{
   type: "success",
   value: {
      id: 10,
      quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
   }
}
```

相当简单，但通过浏览器或通过curl获取还不是最有用的。

通过编程式的方式来使用REST web服务更加有用。 为了帮助您完成该任务，Spring提供了一个名为RestTemplate的便捷的模板类。 RestTemplate使我们能用单行代码就可以与大多数RESTful服务交互。 它甚至可以将该数据绑定到自定义域类型。

首先，创建一个域类以包含所需的数据。

`src/main/java/hello/Quote.java`

```Java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

    private String type;
    private Value value;

    public Quote() {
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public Value getValue() {
        return value;
    }

    public void setValue(Value value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Quote{" +
                "type='" + type + '\'' +
                ", value=" + value +
                '}';
    }
}
```

如您所见，这是一个简单的Java类，具有少量属性和匹配的getter方法。 它使用Jackson JSON处理库中的@JsonIgnoreProperties进行注释，以表明任何未绑定在此Java类中的属性都应被忽略。

为了直接将数据绑定到您的自定义类型，您需要指定与从API返回的JSON文档中的键完全相同的变量名称。 为防止变量名称和JSON文档中的键不匹配，您需要使用@JsonProperty注解来指定与之对应的JSON中的键。

需要一个额外的类来嵌入内部引用本身。

`src/main/java/hello/Value.java`

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

    private Long id;
    private String quote;

    public Value() {
    }

    public Long getId() {
        return this.id;
    }

    public String getQuote() {
        return this.quote;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setQuote(String quote) {
        this.quote = quote;
    }

    @Override
    public String toString() {
        return "Value{" +
                "id=" + id +
                ", quote='" + quote + '\'' +
                '}';
    }
}
```

这里虽然使用了相同的注解，但却是映射到其他数据字段。

## 使应用程序可执行

虽然可以将此服务打包为传统WAR文件以部署到外部应用服务器，但下面演示的方法能够更为简便地创建一个独立的应用程序。 您将所有内容都打包在一个可执行的JAR文件中，由main（）方法驱动。 同时，使用Spring提供的内嵌Tomcat servlet 容器作为HTTP运行时环境, 而不是部署到外部实例当中。

现在，您可以编写使用RestTemplate的Application类来从Spring Boot服务中获取数据。

`src/main/java/hello/Application.java`

```Java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.client.RestTemplate;

public class Application {

    private static final Logger log = LoggerFactory.getLogger(Application.class);

    public static void main(String args[]) {
        RestTemplate restTemplate = new RestTemplate();
        Quote quote = restTemplate.getForObject("http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
        log.info(quote.toString());
    }

}
```

由于Jackson JSON处理库位于类路径中，所以RestTemplate将使用它（通过消息转换器）将传入的JSON数据转换为Quote对象。 在那里，Quote对象的内容将被记录到控制台。

目前您只使用了RestTemplate进行HTTP GET请求。 但是RestTemplatealso还支持其他HTTP请求，如POST，PUT和DELETE。

## 使用Spring Boot管理应用程序生命周期

到目前为止，我们在应用程序中没有使用Spring Boot，然而使用Spring Boot有一些优势，并且这并不难。 其中一个优点是，我们可能希望让Spring Boot管理RestTemplate的消息转换器，以便通过声明的方式来进行定制。 为了做到这一点，我们在主类上使用@SpringBootApplication，并转换了main方法来启动它，就像任何Spring Boot应用程序一样。 最后我们将RestTemplate移动到一个CommandLineRunner回调，因此它在启动时由Spring Boot执行：

`src/main/java/hello/Application.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class Application {

	private static final Logger log = LoggerFactory.getLogger(Application.class);

	public static void main(String args[]) {
		SpringApplication.run(Application.class);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
```

RestTemplateBuilder由Spring注入，如果您使用它来创建一个RestTemplate，那么您将从Spring Boot的自动配置中受益。 我们还将RestTemplate解压缩成@Bean，使其更易于测试（这样可以更容易模拟）。

### 构建可执行的JAR

您可以通过Gradle或Maven从命令行运行应用程序。 或者，您可以构建一个包含所有依赖，类和资源的单个可执行JAR文件，并运行该文件。 这使得在整个开发生命周期中，在不同的环境中迁移，打版本和部署服务变得更加容易。

如果您使用的是Gradle，则可以使用./gradlew bootRun运行应用程序。 或者您可以使用./gradlew构建来构建JAR文件。 然后可以运行JAR文件：

```shell
java -jar build/libs/gs-consuming-rest-0.1.0.jar
```

如果您使用的是Maven，则可以使用./mvnw spring-boot :run运行应用程序。 或者您可以使用./mvnw clean package 来构建JAR文件。 然后可以运行JAR文件：

```shell
java -jar target/gs-consuming-rest-0.1.0.jar
```

> 上面的程序会生成一个可执行的JAR，你也可以选择构建传统的WAR包。

你会看到下面会输出一段随机字符串：

```java
2015-09-23 14:22:26.415  INFO 23613 --- [main] hello.Application  : Quote{type='success', value=Value{id=12, quote='@springboot with @springframework is pure productivity! Who said in #java one has to write double the code than in other langs? #newFavLib'}}
```

> 如果您看到错误无法提取响应：Could not extract response: no suitable HttpMessageConverter found for response type [class hello.Quote]，您可能处于无法连接到后端服务（如果能连上会返回JSON）。 也许你用的是代理， 那么尝试将标准系统属性http.proxyHost和http.proxyPort设置为适合您环境的值。 

# Summary 总结

恭喜！ 您刚刚使用Spring开发了一个简单的REST客户端程序。

## 您也可以看看

以下指南也可能有帮助：

- [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
- [Consuming a RESTful Web Service with AngularJS](https://spring.io/guides/gs/consuming-rest-angularjs/)
- [Consuming a RESTful Web Service with jQuery](https://spring.io/guides/gs/consuming-rest-jquery/)
- [Consuming a RESTful Web Service with rest.js](https://spring.io/guides/gs/consuming-rest-restjs/)
- [Accessing GemFire Data with REST](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
- [Accessing MongoDB Data with REST](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)
- [Accessing Neo4j Data with REST](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
- [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
- [Creating API Documentation with Restdocs](https://spring.io/guides/gs/testing-restdocs/)
- [Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)
- [Building a Hypermedia-Driven RESTful Web Service](https://spring.io/guides/gs/rest-hateoas/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
