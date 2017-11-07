# 使用 Restdocs 创建API文档

> 原文：[Creating API Documentation with Restdocs](https://spring.io/guides/gs/testing-restdocs/)
>
> 译者：HoldDie
>
> 校对：Jitianyu

本指南将引导你了解在 Spring 应用程序中为 HTTP 端点（HTTP endpoints）生成文档的过程。

### 你会建立什么

你将构建一个简单的 Spring 应用程序，其中包含一些暴露 API 的 HTTP 端点（HTTP endpoints）。你将使用  Spring MockMVC 以及 JUnit 来进行 Web 层测试，然后你将使用相同的测试，来为使用 [Spring REST Docs](https://projects.spring.io/spring-restdocs) 的 API 生成[文档](https://projects.spring.io/spring-restdocs)。

### 你需要什么

+ 约 15 分钟
+ 最喜欢的文本编辑器或IDE
+ [JDK 1.8 ](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或更高版本
+ [Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)
+ 你还可以将代码直接导入到IDE中：
  + [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  + [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

### 如何完成本指南

与大多数“ [入门指南](https://spring.io/guides) ”一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。无论哪种方式，你都会得到可以成功运行的代码。

要**从头开始**，请跳转到使用 [Gradle构建](https://spring.io/guides/gs/testing-restdocs/#scratch)。

要**跳过基本操作**，请执行以下操作：

+ [下载](https://github.com/spring-guides/gs-testing-restdocs/archive/master.zip) 并解压缩本指南的源代码库，或使用 [Git](https://spring.io/understanding/Git) 克隆它：`git clone https://github.com/spring-guides/gs-testing-restdocs.git`
+ cd进入 `gs-testing-restdocs/initial`
+ 跳转到 [创建一个简单的应用程序](https://spring.io/guides/gs/testing-restdocs/#initial)。

**完成后**，你可以根据代码检查结果 `gs-testing-restdocs/complete`。

### 使用 Gradle 构建

第一步，建立基本的构建脚本（build script）。 当使用 Spring 构建 apps 的时候，几乎可以使用任何你喜欢的构建工具， 但是此指南只介绍了如何使用 Gradle 和 Maven 来构建目标 app。如果这两个工具你都不熟悉，请参考 [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) 或 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构。例如，在 *nix 系统中使用命令 `mkdir -p src/main/java/hello`  来创建该目录结构。

```
└── src 
    └── main
        └── java 
            └── hello
```

### 创建一个Gradle构建文件

以下是 [初始化的Gradle构建文件](https://github.com/spring-guides/gs-testing-restdocs/blob/master/initial/build.gradle)。

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

在 [Spring Boot Gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多方便的功能：

+ 将 classpath 中的所有 jar 文件集中起来，构建成单独的可运行的 "über-jar", 这使得服务的运行和转移更加便捷。
+ 搜索 `public static void main()` ，并对该可执行类进行标记。
+ 提供了一个内置的依赖解析功能，该功能将依赖的版本与 [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 相匹配。用户可以按照需求覆盖依赖（dependency）的任何版本号，但是默认版本号是 Spring Boot中已经选择好的版本号的集合。

### 使用 Maven 构建应用程序

第一步，建立基本构建脚本(build script)。 当使用Spring构建apps的时候，几乎可以使用任何你喜欢的构建工具， 但是此部分只介绍了如何使用 [Maven](https://maven.apache.org/) 来构建目标app。如果你不熟悉这个工具，请参考 [Building Java Projects with Maven](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选定的工程目录中，创建如下的子目录结构。 例如，在 *nix 中使用命令  `mkdir -p src/main/java/hello` 来创建该目录结构。

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

+ 在 [Spring Boot Maven plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了许多方便的功能：
  + 将 classpath 中的所有 jar 文件集中起来，构建成单独的可运行的 "über-jar", 这使得服务的运行和转移更加便捷。
  + 搜索 `public static void main()` ，并对该可执行类进行标记。
  + 提供了一个内置的依赖解析功能，该功能将依赖的版本与 [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 相匹配。用户可以按照需求覆盖依赖（dependency）的任何版本号，但是默认版本号是 Spring Boot中已经选择好的版本号的集合。

### 使用IDE构建

+ 阅读如何将本指南直接导入到 [Spring Tool Suite](https://spring.io/guides/gs/sts/) 中。
+ 阅读如何在 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 中使用的指南。

### 创建一个简单的应用

为你的 Spring 应用程序创建一个 Controller：

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

创建一个“main”类，你可以使用它来启动应用程序：

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

`@SpringBootApplication` 是一个方便的注解，它添加以下所有内容：

+ `@Configuration` 将类标记为应用程序上下文的 bean 定义的来源。
+ `@EnableAutoConfiguration` 告诉 Spring Boot 根据类路径设置，其他bean和各种属性设置开始添加 bean。
+ 通常，你将添加 `@EnableWebMvc`一个 Spring MVC 应用程序，但 Spring Boot 在类路径中看到 **spring-webmvc** 时会自动添加它。这将应用程序标记为 Web 应用程序，并激活诸如设置 a 的关键行为 `DispatcherServlet`。
+ `@ComponentScan `告诉 Spring 寻找包中的其他组件，配置和服务 `hello`，让它找到 `HelloController`。

该 `main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法启动应用程序。你注意到没有一行 XML 吗？没有 **web.xml** 文件。此 Web 应用程序是 100％ 纯 Java，你无需处理配置任何管道或基础信息。

### 构建可执行的 JAR 文件

可以从 Gradle 或者 Maven 的命令行运行此程序，也可以构建一个单独的可执行的JAR文件，此文件包含了应用程序所有必需的依赖、类以及资源。由于应用程序存在不同的开发周期，也会部署于不同的环境，这种方法使应用程序的转移、版本管理、以及发布都变得更加简单。

如果使用 Gradle，可以使用 `./gradlew bootRun` 运行程序。或者使用 `./gradlew build` 构建 JAR 文件，然后运行此 JAR 文件：

```shell
java -jar build / libs / gs-testing-restdocs-0.1.0.jar
```

如果使用 Maven，可以使用 `./mvnw spring-boot:run` 运行程序。或者使用 `./mvnw clean package` 构建 JAR 文件，然后运行此 JAR 文件：

```shell
java -jar target / gs-testing-restdocs-0.1.0.jar
```

> 上述的过程将创建一个可运行的JAR。你也可以参考 [如何构建一个 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

日志会输出，上述服务应该在几秒钟内准备就绪，开始运行。

### 测试应用程序

 既然应用程序已经在运行了，就可以测试一下了。如果应用正在运行，那么可以访问 [http://localhost:8080](http://localhost:8080)  来加载主页。但是为了在进行修改的时候，让自己对此应用能正常运行有信心，需要进行自动化测试。想要发布 HTTP endpoint 的文档，作为使用 Spring REST Docs 进行测试的一部分，可以用来生成 HTTP endpoint 文档的动态部分。

首先要做的是进行简单的可用性测试，如果应用程序上下文无法启动，该测试就会失败。先把 Spring Test 和 Spring REST Docs 作为 test scope 的依赖加入到工程中，如果使用 Maven 的话：

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

如果你使用 Gradle：

`build.gradle`

```shell
    testCompile("org.springframework.boot:spring-boot-starter-test")
    testCompile("org.springframework.restdocs:spring-restdocs-mockmvc")
```

> 目前此应用程序已经包含了 "mockmvc" 风格的 Rest Docs, 此文档使用 Spring MockMvc 来捕获 HTTP content。如果你的应用不使用 Spring MVC, 也有 "restassured" 风格的Rest Docs，适用于全栈的集成测试

然后使用 `@RunWith` 和 `@SpringBootTest` 注解创建一个测试用例和一个空的测试方法：

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

你可以在IDE或命令行（`mvn test` 或 `gradle test`）中成功运行此测试。

虽然已经有了一个可用性的测试，但是你也应该写一些测试用例来确保程序正常工作。一个有用的方法是只对 MVC 层 进行测试，就是 Spring 接收传入的 HTTP 请求，并将其移交给控制器处理。要做到上述处理，可以使用 Spring `MockMvc`，在测试用例上使用 `@WebMvcTest`  注解进行依赖注入：

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

### 生成代码段文档

上述测试模拟了HTTP请求并验证相应的HTTP相应，所创建的HTTP API 含有动态内容，因此其能够探测测试、收集HTTP请求信息并用在文档中。Spring REST 文档允许你通过生成“片段”来实现。你可以轻松使其正常工作，只需要将“注解”加入到测试用例和额外的"断言“中，以下是一个完整的测试：

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

新的注解是 `@AutoConfigureRestDocs`（来自 Spring Boot），该注解的参数作为生成代码片段的位置。而新的断言是 `MockMvcRestDocumentation.document` ，其参数作为代码段的字符串标识符。

>   Gradle用户对于输出目录可能更喜欢使用 `build ` 而不是 `target` ，但实际上并不重要。这取决于你的选择。 

运行此测试，然后查看 `target/snippets`。你可以找到一个名为 `home`（标识符）的目录，其中包含 [Asciidoctor](http://asciidoctor.org/) 代码片段：

```sh
└── target
    └── snippets
        └── home
            └── httpie-request.adoc
            └── curl-request.adoc
            └── http-request.adoc
            └── http-response.adoc
```

默认代码片段为 Asciidoctor 格式，用于 HTTP 请求和响应，以及命令行示例 `curl` 和 `httpie`（在 HTTP客户端（HTTP clients）两个常见且常用的命令）。

你可以在测试中向 `document()` 添加断言来创建其他代码段。例如，你可以使用 `PayloadDocumentation.responseFields()` 代码段记录JSON响应中的每个字段：

`src/test/java/hello/WebLayerTest.java`

```java
this.mockMvc.perform(get("/"))
    ...
    .andDo(document("home", responseFields(
        fieldWithPath("message").description("The welcome message for the user.")
    ));
```

如果你执行上述测试用例，你会找到一个名为“response-fields.adoc”的文件，其中包含字段anames和描述表。如果你省略了一个字段或将其名称写错，则测试失败 - 这就是 REST 文档的强大功能。

>   你可以创建自定义片段，还可以更改片段的格式并自定义一些参数如：主机名。有关更多详细信息，请查看 [Spring REST文档的文档](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/)。 

### 使用代码片段

要使用生成的代码片段，你希望在项目中有一些 Asciidoctor 的内容，然后在构建时添加代码片段。为了让生成的代码段运行，创建一个文件，`src/main/asciidoc/index.adoc` 并根据需要添加代码片段。

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

其主要特点是它包含 2 个片段，使用 Asciidoctor `include` 指令（冒号和尾部括号告诉解析器在这些行上执行特殊操作）。请注意，包含的片段的路径 `{snippets}` 表示为占位符 - 是Asciidoctor 中的一个“属性”。在这种简单的情况下，唯一的标记是在“.”在片段之前（“请求”和“响应”）顶部的“=”，它是一级标题。

然后在构建配置中，你需要将此源文件处理为你选择的文档格式。例如，使用 Maven 生成 HTML（`target/generated-docs` 在执行时生成 `mvnw package`）：

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

如果你使用Gradle（`build/asciidoc` 当你执行时生成 `gradlew asciidoctor`）：

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

>   Asciidoctor Gradle 插件不在 Maven Central 中，所以你还必须添加 `jcenter()` 到 Gradle 中的 buildscipt 依赖项。 

> Gradle 中 asciidoctor 源的默认位置是 `src/doc/asciidoc`。我们只需要设置，`sourceDir` 因为我们更改了位置以匹配 Maven 的默认值。

### 总结

恭喜！你刚刚开发了一个 Spring 应用程序，并使用 Spring Restdocs 生成文档。你可以将你创建的 HTML 文档发布到静态网站，或将其打包为应用程序本身所用。你的文档应始终是最新的，否则，测试将失败。

### 也可以学习其他

以下指南也可能有帮助：

+ [构建RESTful Web服务](https://spring.io/guides/gs/rest-service/)
+ [使用RESTful Web服务](https://spring.io/guides/gs/consuming-rest/)
+ [用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。



> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

