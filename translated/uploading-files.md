# 上传文件

> 原文：[Uploading Files](https://spring.io/guides/gs/uploading-files/)
>
> 译者：[xiudongxu](https://github.com/xiudongxu/spring-guides-translation)
>
> 校对：

本指南将帮助您创建一个可以实现接收HTTP多部分文件上传的服务器应用程序。

## 你将构建什么

你将创建一个可以接收文件上传的Spring Boot web应用程序。你也将创建一个简单的HTML接口来上传一个测试文件。

## 你需要准备什么

* 大约15分钟
* 您最喜欢的文本编辑器或者IDE
* [JDK 6](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或者以上的版本
* [Gradle 2.3+](http://www.gradle.org/downloads)或者[Maven 3.0+](https://maven.apache.org/download.cgi)
* 你也可以直接导入代码到你的IDE中：
  * [Spring Tool Suite(STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本指南

像大多数[Spring系列入门指南](https://spring.io/guides)一样，你可以从零开始，按照指南中的每一步的指示来做，或者你也可以跳过你已经熟悉的基本教程。不论那种方式，最后我们想要的结果都是可以运行的代码。

**从零开始**，移步至[ Build with Gradle](https://spring.io/guides/gs/spring-boot/#scratch)

**跳过基本教程**，需要做下列操作：
* [下载](https://github.com/spring-guides/gs-spring-boot/archive/master.zip)本指南的源码zip包并解压，或者直接使用[Git](https://spring.io/understanding/Git)获取源码
> git clone <https://github.com/spring-guides/gs-uploading-files.git>

* 进入到如下目录```gs-uploading-files/initial```
* 跳转到[创建一个应用程序类](https://spring.io/guides/gs/uploading-files/#initial)

**当你做完以上的几步**，你可以在如下目录检查你的代码

>gs-uploading-files/complete

## 使用Gradle创建项目

首先设置一个基本的构建脚本。 在使用Spring构建应用程序时，你可以使用任何你喜欢的系统构建项目，但是你用Maven或Gradle构建的代码必须要在这个系统下。如果你还不熟悉，请参阅[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle)或[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构
在你选择的项目目录中，创建以下子目录结构；例如，在 unix系统使用
``` mkdir -p src / main / java / hello ```

    └── src
        └── main
            └── java
                └── hello

### 创建一个Gradle构建文件

下面是[Gradle的初始化构建文件](https://github.com/spring-guides/gs-spring-boot/blob/master/initial/build.gradle)

``` build.gradle ```

```gradle
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-uploading-files'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```



Spring Boot的[Gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多方便的功能

* 它会把类路径下的所有jar包都打包最后生成一个可运行的"über-jar"，可以方便你的运行和服务调用。
* 它会把 ``` public static void main() ```方法作为入口方法
* 它提供了一个内置的依赖解析器，可以自动设置适应[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)的版本。你也可以自定义版本号，但这个版本号必须是Spring Boot默认存在的版本号。

## 使用Maven创建项目

首先设置一个基本的构建脚本。 在使用Spring构建应用程序时，你可以使用任何你喜欢的系统构建项目，但是你用Maven构建的代码必须要在这个系统下。如果你还不熟悉，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构
在你选择的项目目录中，创建以下子目录结构：例如，在 unix系统使用
``` mkdir -p src / main / java / hello ```

    └── src
        └── main
            └── java
                └── hello

``` pom.xml ```


    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>org.springframework</groupId>
        <artifactId>gs-spring-boot</artifactId>
        <version>0.1.0</version>
    
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>1.5.7.RELEASE</version>
        </parent>
    
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
    
        <properties>
            <java.version>1.8</java.version>
        </properties>


```maven
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-uploading-files</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

Spring Boot的[Maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多方便的功能

* 它会把类路径下的所有jar包都打包最后生成一个可运行的"über-jar"，可以方便你的运行和服务调用。
* 它会把 ``` public static void main() ```方法作为入口方法
* 它提供了一个内置的依赖解析器，可以自动设置适应[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)的版本。你也可以自定义版本号，但这个版本号必须是Spring Boot默认存在的版本号。

## 使用你的编辑器创建项目

* 了解下如何把本项引导的代码导入[Spring Tool Suite](https://spring.io/guides/gs/sts/)。
* 了解下在[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)中如何完成本项引导。

## 创建一个Application类

为了开始一个Spring Boot MVC应用，我们首先需要一个启动装置；这里```spring-boot-starter-thymeleaf```和```spring-boot-starter-web```已经被添加进依赖里面了。用servlet容器上传文件，你需要等级一个```MultipartConfigElement```类，感谢Spring Boot，一切已经为您自动的配置好了。

启动此应用程序所需的所有操作就是以下应用程序类。```src/main/java/hello/Application.java```

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

作为自动配置Spring MVC的一部分，Spring Boot将会创建一个```MultipartConfigElement```bean并且已经为上传文件做好了准备。

## 创建一个上传文件controller

初始应用程序已经包含几个类，用于处理在磁盘上存储和加载上传的文件；它们都位于```hello.storage```包中，我们将用这些在我们的新```FileUploadController```类中。

> src/main/java/hello/FileUploadController.java

```java
package hello;

import java.io.IOException;
import java.util.stream.Collectors;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.method.annotation.MvcUriComponentsBuilder;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import hello.storage.StorageFileNotFoundException;
import hello.storage.StorageService;

@Controller
public class FileUploadController {

    private final StorageService storageService;

    @Autowired
    public FileUploadController(StorageService storageService) {
        this.storageService = storageService;
    }

    @GetMapping("/")
    public String listUploadedFiles(Model model) throws IOException {

        model.addAttribute("files", storageService.loadAll().map(
                path -> MvcUriComponentsBuilder.fromMethodName(FileUploadController.class,
                        "serveFile", path.getFileName().toString()).build().toString())
                .collect(Collectors.toList()));

        return "uploadForm";
    }

    @GetMapping("/files/{filename:.+}")
    @ResponseBody
    public ResponseEntity<Resource> serveFile(@PathVariable String filename) {

        Resource file = storageService.loadAsResource(filename);
        return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION,
                "attachment; filename=\"" + file.getFilename() + "\"").body(file);
    }

    @PostMapping("/")
    public String handleFileUpload(@RequestParam("file") MultipartFile file,
            RedirectAttributes redirectAttributes) {

        storageService.store(file);
        redirectAttributes.addFlashAttribute("message",
                "You successfully uploaded " + file.getOriginalFilename() + "!");

        return "redirect:/";
    }

    @ExceptionHandler(StorageFileNotFoundException.class)
    public ResponseEntity<?> handleStorageFileNotFound(StorageFileNotFoundException exc) {
        return ResponseEntity.notFound().build();
    }

}
```

这个类被```@Controller```注解标注了，所以Spring MVC可以发现它并且路由到它。每一个方法都被```@GetMapping```or```@PostMapping```标注了为了将路径和HTTP请求绑定到特定的Controller中。

在这种情况下：

* ```GET/```从```StorageService```中寻找当前的上传文件列表并将它加载到Thymeleaf模板中，通过```MvcUriComponentsBuilder```类来绑定一个链接到实际的资源。
* ```GET /files/{filename}```加载那些存在的资源，并将它发送到浏览器去下载通过使用一个```"Content-Disposition"```response头
* ```POST / ```用于处理一个multi-part消息```file```并且将其传递到```StorageService```类中去保存

>在生产情况下，你可能更多的将文件存储到一个临时位置，数据库，或者一个NoSQL 存储中例如[Mongo`s GridFS](http://docs.mongodb.org/manual/core/gridfs).最好不要在你的应用程序中加载文件系统。

你将需要提供一个```StorageService```为controller去与存储层(例如文件系统)来互动。像这样的一个接口：

>src/main/java/hello/storage/StorageService.java

```java
package hello.storage;

import org.springframework.core.io.Resource;
import org.springframework.web.multipart.MultipartFile;

import java.nio.file.Path;
import java.util.stream.Stream;

public interface StorageService {

    void init();

    void store(MultipartFile file);

    Stream<Path> loadAll();

    Path load(String filename);

    Resource loadAsResource(String filename);

    void deleteAll();

}
```

在示例程序中有一个例子实现了接口，如果你想节省时间的话就可以复制粘贴过去。

## 创建一个简单的HTML模板

为了建造有趣的东西，下面这个Thymeleaf模板是一个极好的上传文件的例子以及显示上传的内容。

```src/main/resources/templates/uploadForm.html```

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>

	<div th:if="${message}">
		<h2 th:text="${message}"/>
	</div>

	<div>
		<form method="POST" enctype="multipart/form-data" action="/">
			<table>
				<tr><td>File to upload:</td><td><input type="file" name="file" /></td></tr>
				<tr><td></td><td><input type="submit" value="Upload" /></td></tr>
			</table>
		</form>
	</div>

	<div>
		<ul>
			<li th:each="file : ${files}">
				<a th:href="${file}" th:text="${file}" />
			</li>
		</ul>
	</div>

</body>
</html>
```

这个模板有三部分：

* 在顶部有一个可选的消息由Spring MVC写的一个[flash-scoped message](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-flash-attributes)
* 一个允许用户上传文件的表单
* 一个从后端提供的文件列表

## 调整文件上传限制

通常是非常有用的去配置文件上传的大小限制。想象一下处理一个5GB大小的文件上传。使用Spring Boot我们可以调整它的自动配置```MultipartConfigElement```通过一些属性配置。

添加如下配置在你存在的配置文件中：```src/main/resources/application.properties```

```properties
spring.http.multipart.max-file-size=128KB
spring.http.multipart.max-request-size=128KB
```

多数设置约定如下：

* ```spring.http.multipart.max-file-size```就是设置成128KB，意思是总文件大小不能超过128KB。
* ```spring.http.multipart.max-request-size```就是设置成128KB，意思是一个```multipart/form-data```的请求大小不能超过128KB。

## 使应用程序可以执行

虽然可以将此服务打包为传统的WAR文件，以便部署到外部应用程序服务器,下面演示了一个更简单的方法去创建一个*独立应用*。你打包所有的文件到一个独立的可运行的JAR文件中，通过一个好用的古老的java```main()```方法。你使用Spring提供的嵌入式[Tomcat](https://spring.io/understanding/Tomcat) servlet 容器作为HTTP运行容器，代替部署到额外的实例上。

你还需要一个目标文件夹作为文件上传目的地，让我们一起来增强这个基本的```Application```类并且添加一个Boot```CommandLineRunner```在启动时可以删除和重新创建该文件夹:```src/main/java/hello/Application.java```

```java
package hello;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;

import hello.storage.StorageProperties;
import hello.storage.StorageService;

@SpringBootApplication
@EnableConfigurationProperties(StorageProperties.class)
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner init(StorageService storageService) {
        return (args) -> {
            storageService.deleteAll();
            storageService.init();
        };
    }
}
```

```@SpringBootApplication```是一个十分方便的注解因为他包含了以下内容：

* ```@Configuration```标注了这个类是一个bean定义在应用上下文中。
* ```@EnableAutoConfiguration```这个注解告诉Spring Boot开始添加beans通过classpath设置，其他beans和各种属性的设置。
* 通常你都会添加```@EnableWebMvc```这个注解作为一个Spring MVC应用，但是Spring Boot将自动添加它如果在classpath中看到了**spring-webmvc**这样的配置。这个标志中的应用作为一个web应用程序，并且激活一个关键行为例如设置```DispatcherServlet```。
* ```@ComponentScan```这个注解就会告诉Spring去寻找其他的components，configurations，和services在```hello```这个包中，允许它去找到controllers。

```main()```函数使用Spring Boot`s的```SpringApplication.run()```方法来启动一个应用程序。你是否注意到没有一行XML配置，而且也没有web.xml文件。这是一个100%纯Java程序而且你也不需要做任何的配置。

##创建一个可执行的JAR包

你可以通过Gradle或者Maven的命令行来启动这个应用程序。或者你可以构建一个包含所有需要的依赖，类文件和资源文件的独立可执行的JAR文件并且可以运行它。这样就可以很方便的传输，进行版本控制并且部署这个服务作为一个应用的时候无论在什么开发生命周期，不同的环境等等。

如果你使用Gradle，你可以运行你的应用通过使用```./gradlew bootRun```命令。或者你也可以构建一个JAR文件通过使用```./gralew build```。之后你就可以运行这个JAR文件了。

```java
java -jar build/libs/gs-uploading-files-0.1.0.jar
```

如果你使用Maven，你可以运行你的应用通过使用```./mvnw spring-boot:run```命令。或者你也可以构建一个JAR文件通过使用```./mvnw clean package```。之后你就可以运行这个JAR文件了。

```java
java -jar target/gs-uploading-files-0.1.0.jar
```

>上述过程将会创建一个可运行的JAR文件，你也可以选择[创建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)来代替。

这样就运行了可接收文件上传的服务端程序。日志输入显示。服务将在几秒后运行起来。

服务在运行状态时，你需要打开浏览器输入```http://localhost:8080/```来查看上传表单。挑选一个小文件然后按"Upload"然后你就能看见从controller返回的成功页面。选择一个过大的文件那样你就能看到一个丑陋的错误页面了。

你可以在你的浏览器窗口中看到如下信息：

```html
You successfully uploaded <name of your file>!
```

## 测试你的应用程序

在我们的应用中有很多种方式来测试这个特性。这有一个利用```MockMvc```的例子，所以它不需要启动Servlet容器。

```src/test/java/hello/FileUploadTests.java```

```java
package hello;

import java.nio.file.Paths;
import java.util.stream.Stream;

import org.hamcrest.Matchers;
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.mock.web.MockMultipartFile;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.BDDMockito.given;
import static org.mockito.BDDMockito.then;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.fileUpload;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.header;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.model;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import hello.storage.StorageFileNotFoundException;
import hello.storage.StorageService;

@RunWith(SpringRunner.class)
@AutoConfigureMockMvc
@SpringBootTest
public class FileUploadTests {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private StorageService storageService;

    @Test
    public void shouldListAllFiles() throws Exception {
        given(this.storageService.loadAll())
                .willReturn(Stream.of(Paths.get("first.txt"), Paths.get("second.txt")));

        this.mvc.perform(get("/")).andExpect(status().isOk())
                .andExpect(model().attribute("files",
                        Matchers.contains("http://localhost/files/first.txt",
                                "http://localhost/files/second.txt")));
    }

    @Test
    public void shouldSaveUploadedFile() throws Exception {
        MockMultipartFile multipartFile = new MockMultipartFile("file", "test.txt",
                "text/plain", "Spring Framework".getBytes());
        this.mvc.perform(fileUpload("/").file(multipartFile))
                .andExpect(status().isFound())
                .andExpect(header().string("Location", "/"));

        then(this.storageService).should().store(multipartFile);
    }

    @SuppressWarnings("unchecked")
    @Test
    public void should404WhenMissingFile() throws Exception {
        given(this.storageService.loadAsResource("test.txt"))
                .willThrow(StorageFileNotFoundException.class);

        this.mvc.perform(get("/files/test.txt")).andExpect(status().isNotFound());
    }

}
```

在这些测试当中，我们使用了各种各样的mock来为Controller和```StorageSercive```建立关联，而且也通过是用```MockMultipartFile```来与Servlet容器本身建立关联。

作为一个整合测试的例子，请检出```FileUploadIntergrationTests```类。

## 总结

恭喜你！你已经写了一个web应用使用Spring来上传文件了。

## 也可以看

下面的这些指南可能是有帮助的：

* [通过Spring MVC运行web组件](https://spring.io/guides/gs/serving-web-content/)
* [处理表单提交](https://spring.io/guides/gs/handling-form-submission/)
* [保护web应用的安全](https://spring.io/guides/gs/securing-web/)
* [用Spring Boot创建应用](https://spring.io/guides/gs/spring-boot/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



