# 使用Restdocs创建API文档

> 原文：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) 
>
> 译者：HoldDie
>
> 校对：

本指南将引导您了解在Spring应用程序中为HTTP端点生成记录的过程。

### 你会建立什么

您将构建一个简单的 Spring 应用程序，其中包含一些暴露 API 的 HTTP 端点。您将使用 JUnit 和 Spring 来测试 Web 层 `MockMvc`，然后您将使用相同的测试来生成使用 [Spring REST文档](https://projects.spring.io/spring-restdocs) 的API [文档](https://projects.spring.io/spring-restdocs)。

### 你需要什么

+ 约 15 分钟
+ 最喜欢的文本编辑器或IDE
+ [JDK 1.8 ](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或更高版本
+ [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
+ 您还可以将代码直接导入到IDE中：
  + [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  + [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

### 如何完成本指南

与大多数“ [入门指南”指南一样](https://spring.io/guides)，您可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。无论哪种方式，你都会得到工作代码。

要**从头开始**，请继续 [使用Gradle构建](https://spring.io/guides/gs/testing-restdocs/#scratch)。

要**跳过基本操作**，请执行以下操作：

+ [下载](https://github.com/spring-guides/gs-testing-restdocs/archive/master.zip) 并解压缩本指南的源存储库，或使用 [Git](https://spring.io/understanding/Git) 克隆它：`git clone https://github.com/spring-guides/gs-testing-restdocs.git`
+ cd进入 `gs-testing-restdocs/initial`
+ 跳转到 [创建一个简单的应用程序](https://spring.io/guides/gs/testing-restdocs/#initial)。

**完成后**，您可以根据代码检查结果 `gs-testing-restdocs/complete`。

### 建立与Gradle

首先你设置一个基本的构建脚本。在使用Spring构建应用程序时，您可以使用任何您喜欢的构建系统，但是您需要使用与 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/) 一起使用的代码。如果您还不熟悉，请参阅[使用Gradle ](https://spring.io/guides/gs/gradle)[构建Java项目](https://spring.io/guides/gs/maven)或[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构; 例如，使用 `mkdir -p src/main/java/hello` 在 *nix 系统：

```
└── src 
    └── main
        └── java 
            └── hello
```

### 创建一个Gradle构建文件

以下是 [最初的Gradle构建文件](https://github.com/spring-guides/gs-testing-restdocs/blob/master/initial/build.gradle)。

`build.gradle`

```shell
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.4.0.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
    baseName = 'gs-testing-web'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}

task wrapper(type: Wrapper) {
    gradleVersion = '2.3'
}
```

在 [Spring Boot Gradle 启动插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多便捷的功能：

+ 它收集类路径上的所有罐子，并构建一个可运行的“über-jar”，这样可以更方便地执行和传输您的服务。
+ 它搜索 `public static void main()` 标记为可运行类的方法。
+ 它提供了一个内置的依赖解析器，它将版本号设置为与 [Spring Boot依赖关系](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 相匹配。您可以覆盖任何您想要的版本，但它将默认为Boot所选择的一组版本。

### 搭建Maven

首先你设置一个基本的构建脚本。在使用Spring构建应用程序时，您可以使用任何构建系统，但您需要使用与 [Maven](https://maven.apache.org/) 一起使用的代码。如果您不熟悉 Maven，请参阅使用 Maven [构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构; 例如，使用 `mkdir -p src/main/java/hello` 在 *nix 系统：

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
    <artifactId>gs-testing-restdocs</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

在 [Spring Boot Maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了许多便捷的功能：

+ 它收集类路径上的所有罐子，并构建一个可运行的“über-jar”，这样可以更方便地执行和传输您的服务。
+ 它搜索 `public static void main()` 标记为可运行类的方法。
+ 它提供了一个内置的依赖解析器，它将版本号设置为与 [Spring Boot依赖关系](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 相匹配。您可以覆盖任何您想要的版本，但它将默认为Boot所选择的一组版本。

### 使用IDE构建

+ 阅读如何将本指南直接导入到 [Spring Tool Suite](https://spring.io/guides/gs/sts/) 中。
+ 阅读如何在 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 中使用的指南。

### 创建一个简单的应用

为您的Spring应用程序创建一个新的控制器：

`src/main/java/hello/HomeController.java`

```java
package hello;

import java.util.Collections;
import java.util.Map;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HomeController {

    @GetMapping("/")
    public Map<String, Object> greeting() {
        return Collections.singletonMap("message", "Hello World");
    }

}
```

### 使应用程序可执行

创建一个“main”类，您可以使用它来启动应用程序：

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

`@SpringBootApplication` 是一个方便的注释，它添加以下所有内容：

+ `@Configuration` 将类标记为应用程序上下文的 bean 定义的来源。
+ `@EnableAutoConfiguration` 告诉 Spring Boot 根据类路径设置，其他bean和各种属性设置开始添加 bean。
+ 通常，您将添加 `@EnableWebMvc`一个 Spring MVC 应用程序，但 Spring Boot 在类路径中看到 **spring-webmvc** 时会自动添加它。这将应用程序标记为 Web 应用程序，并激活诸如设置 a 的关键行为 `DispatcherServlet`。
+ `@ComponentScan `告诉 Spring 寻找包中的其他组件，配置和服务 `hello`，让它找到 `HelloController`。

该 `main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法启动应用程序。你注意到没有一行 XML 吗？没有 **web.xml** 文件。此 Web 应用程序是 100％ 纯 Java，您无需处理配置任何管道或基础信息。

### 构建可执行的JAR

您可以从命令行运行应用程序与 Gradle 或 Maven。或者，您可以构建一个包含所有必需依赖项，类和资源的单个可执行 JAR 文件，并运行该文件。这使得在整个开发生命周期中，跨不同的环境等运输，版本和部署服务成为一个应用程序。

如果您使用的是 Gradle，则可以使用该应用程序 `./gradlew bootRun`。或者您可以使用JAR文件构建 `./gradlew build`。然后可以运行 JAR 文件：

```shell
java -jar build / libs / gs-testing-restdocs-0.1.0.jar
```

如果您使用 Maven，则可以使用该应用程序 `./mvnw spring-boot:run`。或者您可以使用 JAR 文件来构建 `./mvnw clean package`。然后可以运行 JAR 文件：

```shell
java -jar target / gs-testing-restdocs-0.1.0.jar
```

> 上面的过程将创建一个可运行的JAR。您也可以选择 [构建一个WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

显示记录输出。该服务应该在几秒钟内启动并运行。

### 测试应用程序

现在应用程序正在运行，您可以测试它。如果它正在运行，您可以在 [http:// localhost:8080](http://localhost:8080/) 上加载主页。但是，为了让自己更有信心，当您进行更改时，应用程序正在运行，您希望自动化测试。您还要发布 HTTP 端点的文档，您可以使用 Spring REST 文档生成动态部分作为测试的一部分。

您可以做的第一件事是写一个简单的健康检查测试，如果应用程序上下文无法启动，它将失败。首先在测试范围内添加 Spring Test 和 Spring REST 文档作为项目的依赖关系。如果你使用 Maven：

`pom.xml`

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-mockmvc</artifactId>
            <scope>test</scope>
        </dependency>
```

或者如果您使用 Gradle：

`build.gradle`

```shell
    testCompile("org.springframework.boot:spring-boot-starter-test")
    testCompile("org.springframework.restdocs:spring-restdocs-mockmvc")
```

> 您已经包含REST文档的“mockmvc”风格，它使用Spring MockMvc来捕获HTTP内容。如果你自己的应用程序不使用Spring MVC，那么还有一个“restassured”的风格，可以用于堆栈集成测试。

然后使用 `@RunWith` 和 `@SpringBootTest` 注释创建一个测试用例和一个空的测试方法：

`src/test/java/hello/ApplicationTest.java`

```java
package hello;

import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTest {

    @Test
    public void contextLoads() throws Exception {
    }
}
```

您可以在IDE或命令行（`mvn test` 或 `gradle test`）中运行此测试，它应该通过。

很高兴有一个这样的健全检查，但你也应该写一些测试来证明我们的应用程序的行为。一个有用的方法是仅测试 MVC 层，其中Spring处理传入的 HTTP 请求，并将其移交给控制器。要做到这一点，可以使用 Spring `MockMvc`，并通过使用 `@WebMvcTest` 测试用例上的注释来要求为我们注入：

`src/test/java/hello/WebLayerTest.java`

```java
@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
public class WebLayerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void shouldReturnDefaultMessage() throws Exception {
        this.mockMvc.perform(get("/"))
            .andExpect(status().isOk())
            .andExpect(content().string(containsString("Hello World")));
    }
}
```

### 为文档生成代码段

上面的测试使（mock）HTTP 请求和断言响应。您创建的 HTTP API 具有动态内容（至少原则上），因此能够窥探测试并清除 HTTP 请求以便在文档中使用是非常好的。Spring REST 文档允许您通过生成“片段”来实现。你可以很容易地得到这个工作，只需要添加一个注释到你的测试和一个额外的“断言”。这是完整的测试：

`src/test/java/hello/WebLayerTest.java`

```java
package hello;

import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)
@AutoConfigureRestDocs(outputDir = "target/snippets")
public class WebLayerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void shouldReturnDefaultMessage() throws Exception {
        this.mockMvc.perform(get("/")).andDo(print()).andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello World")))
                .andDo(document("home"));
    }
}
```

新的注释是 `@AutoConfigureRestDocs`（来自 Spring Boot），它将生成的片段的目录位置作为参数。而新的断言是 `MockMvcRestDocumentation.document` 一个参数，作为代码段的字符串标识符。

>   Gradle用户可能更喜欢使用 `build `而不是 `target` 输出目录，但实际上并不重要。这取决于你的选择。 

运行此测试，然后查看 `target/snippets`。您应该找到一个名为 `home`（标识符）的目录，其中包含 [Asciidoctor](http://asciidoctor.org/) 片段：

```sh
└── target
    └── snippets
        └── home
            └── httpie-request.adoc
            └── curl-request.adoc
            └── http-request.adoc
            └── http-response.adoc
```

默认代码片段为 Asciidoctor 格式，用于 HTTP 请求和响应，以及命令行示例 `curl` 和 `httpie`（两个常见且常用的命令行HTTP客户端）。

您可以通过 `document()` 在测试中向断言添加参数来创建其他代码段。例如，您可以使用 `PayloadDocumentation.responseFields()` 代码段记录JSON响应中的每个字段：

`src/test/java/hello/WebLayerTest.java`

```java
this.mockMvc.perform(get("/"))
    ...
    .andDo(document("home", responseFields(
        fieldWithPath("message").description("The welcome message for the user.")
    ));
```

如果您尝试并执行测试，您应该找到一个名为“response-fields.adoc”的附加代码段文件，其中包含字段anames和描述表。如果您省略了一个字段或将其名称错误，则测试将失败 - 这是 REST 文档的强大功能。

>   您可以创建自定义片段，还可以更改片段的格式并自定义主机名。有关更多详细信息，请查看 [Spring REST文档的文档](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/)。 

### 使用该片段

要使用生成的片段，您希望在项目中具有一些Asciidoctor内容，然后在构建时添加片段。要看到这个工作，创建一个新文件，`src/main/asciidoc/index.adoc`并根据需要添加该片段。例如

`src/main/asciidoc/index.adoc`

```adoc
= Spring REST文档入门

这是在 http://localhost:8080 运行的服务的示例输出：

.request
include::{snippets}/home/http-request.adoc[]

.response
include::{snippets}/home/http-response.adoc[]

你可以看到格式很简单，其实你总是得到同样的信息。
```

其主要特点是它包含 2 个片段，使用 Asciidoctor `include` 指令（冒号和尾部括号告诉解析器在这些行上执行特殊操作）。请注意，包含的片段的路径 `{snippets}` 表示为占位符 - 称为 Asciidoctor 中的“属性”。在这种简单的情况下，唯一的其他标记是顶部的“=”，它是 1 级标题，而“.”在片段之前（“请求”和“响应”）。

然后在构建配置中，您需要将此源文件处理为您选择的文档格式。例如，使用 Maven 生成 HTML（`target/generated-docs` 在执行时生成 `mvnw package`）：

`pom.xml`

```xml
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process-asciidoc</goal>
            </goals>
            <configuration>
                <sourceDocumentName>index.adoc</sourceDocumentName>
                <backend>html</backend>
                <attributes>
                    <snippets>${project.build.directory}/snippets</snippets>
                </attributes>
            </configuration>
        </execution>
    </executions>
</plugin>
```

或者如果您使用Gradle（`build/asciidoc`当您执行时生成`gradlew asciidoctor`）：

`build.gradle`

```
buildscript {
    repositories {
        ...
        jcenter()
    }
    ...
    dependencies {
        ...
        classpath("org.asciidoctor:asciidoctor-gradle-plugin:1.5.3")
    }
}

...
apply plugin: 'org.asciidoctor.convert'

asciidoctor {
    sourceDir 'src/main/asciidoc'
    attributes \
      'snippets': file('target/snippets')
}
```

>   Asciidoctor Gradle 插件不在 Maven Central 中，所以您还必须添加 `jcenter()` 到 Gradle 中的 buildscipt 依赖项。 

> Gradle 中 asciidoctor 源的默认位置是 `src/doc/asciidoc`。我们只需要设置，`sourceDir` 因为我们更改了位置以匹配 Maven 的默认值。

### 总结

恭喜！您刚刚开发了一个 Spring 应用程序，并使用 Spring Restdocs 进行了记录。您可以将您创建的 HTML 文档发布到静态网站，或将其打包并从应用程序本身提供。您的文档将始终是最新的，如果不是，测试将失败。

### 也可以学习其他

以下指南也可能有帮助：

+ [构建RESTful Web服务](https://spring.io/guides/gs/rest-service/)
+ [使用RESTful Web服务](https://spring.io/guides/gs/consuming-rest/)
+ [用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。



> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

