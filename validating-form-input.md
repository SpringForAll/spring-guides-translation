# 表单输入的校验

> 原文：[Validating Form Input](https://spring.io/guides/gs/validating-form-input) 
>
> 译者：[JohnHello](https://github.com/JohnHello)


本篇指南会引导你配置生成一个支持表单校验的web项目。

## 你将收获什么
你将学会搭建一个基本的Spring MVC程序，该程序会使用标准的校验注解去检查用户的表单输入。你也可以学习如何展示错误信息去引导用户输入正确的表单。

## 开始之前你需要准备
- 大约15分钟时间
- 一个顺手的文本编辑器或IDE
- JDK 1.8 或 更高版本
- Gradle 2.3+ 或 Maven 3.0+
- 你也可以直接导入代码到IDE:
    - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts/)
    - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    
## 如何完成指南
像大多数 Spring 入门指南一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

如果从基础开始，你可以往下查看怎样使用 Gradle 构建项目。

如果已经熟悉跳过一些基本步骤，你可以：

- 下载并解压源码库，或者通过 Git克隆：
> git clone https://github.com/spring-guides/gs-validating-form-input.git

- 进入 gs-validating-form-input/initial 目录

- 跳过前面的部分，直接从创建一个PersonForm对象开始往下进行

当你完成之后，你可以在gs-validating-form-input/complete根据检查下代码运行结果。

## 使用Gradle构建

首先您需要编写基础构建脚本。在构建 Spring 应用的时候，您可以使用任何您喜欢的工具来构建。这里提供一份对您可能有帮助的事例 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 代码。 如果您对这两者都不是很熟悉, 您可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

在您的项目根目录，创建如下的子目录结构; 例如，如果您使用的是\*nix系统，您可以使用`mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 使用Gradle构建文件

下面是一份初始化Gradle构建文件

`build.gradle`

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
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-validating-form-input'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    compile("org.hibernate:hibernate-validator")
    compile("org.apache.tomcat.embed:tomcat-embed-el")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布您的服务变得更加便捷。
* 搜索 `public static void main()` 方法并且将它标记为可执行类。
* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。您可以根据您的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建

首先，您需要设置一个基本的构建脚本。当使用 Spring 构建应用程序时，您可以使用任何您喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在您选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

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
    <artifactId>gs-validating-form-input</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

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

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-el</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>spring-releases</id>
            <name>Spring Releases</name>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <name>Spring Releases</name>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>

</project>
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布您的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。您可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。



## 使用您的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。


## 创建一个PersonForm对象
程序需要验证用户的姓名和年龄，所以首先需要创建一个类来支持表单来创建一个Person对象。

> src/main/java/hello/PersonForm.java


```
package hello;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class PersonForm {

    @NotNull
    @Size(min=2, max=30)
    private String name;

    @NotNull
    @Min(18)
    private Integer age;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String toString() {
        return "Person(Name: " + this.name + ", Age: " + this.age + ")";
    }
}
```
这个PersonForm类有两个属性：名字和年龄。它用了几个规范验证注解：
- @Size(min=2, max=30) 只允许name的字符长度在2-30之间

- @NotNull 不允许为null

- @Min(18) 不允许age小于18

此外，你还可以看到name和age的getter和setter方法以及一个方便的tostring()方法。

## 创建一个web controller
现在你已经定义了一个表单对象，现在需要创建一个简单的web controller。
> src/main/java/hello/WebController.java


```
package hello;

import javax.validation.Valid;

import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;


@Controller
public class WebController extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/results").setViewName("results");
    }

    @GetMapping("/")
    public String showForm(PersonForm personForm) {
        return "form";
    }

    @PostMapping("/")
    public String checkPersonInfo(@Valid PersonForm personForm, BindingResult bindingResult) {

        if (bindingResult.hasErrors()) {
            return "form";
        }

        return "redirect:/results";
    }
}
```
这个controller有一个GET方法和一个POST方法都映射到 `/`。

该`showForm`方法返回`form`模板。它在方法签名包括一个`PersonForm`以使模板可以将表格属性与`PersonForm`对应上。

这个`checkPersonFormInfo`方法包含两个参数：

- 一个有注解@Valid的`PersonForm`对象，这个注解是为了标识你要校验的表格对象。

-  一个`bindingResult`对象，因此您可以测试和检索验证错误。

你可以检索所有属性关联到到`PersonForm`对象中。在代码中，您可以测试错误信息，如果是的话，将用户界面跳转到`form`模板。在这种情况下，所有的错误属性都会显示出来。

如果所有person的属性校验通过，然后重定向到最终的`results`模板。

## 构建一个HTML前端

现在构建 "main" 页面

`src/main/resources/templates/form.html`


```
<html>
    <body>
        <form action="#" th:action="@{/}" th:object="${personForm}" method="post">
            <table>
                <tr>
                    <td>Name:</td>
                    <td><input type="text" th:field="*{name}" /></td>
                    <td th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Name Error</td>
                </tr>
                <tr>
                    <td>Age:</td>
                    <td><input type="text" th:field="*{age}" /></td>
                    <td th:if="${#fields.hasErrors('age')}" th:errors="*{age}">Age Error</td>
                </tr>
                <tr>
                    <td><button type="submit">Submit</button></td>
                </tr>
            </table>
        </form>
    </body>
</html>
```
页面包含一个简单的表单，每个字段有一个单独的输入框。这个表格是`/`的post请求。在web controller的get方法中，它会映射到`person`对象。这被称为bean支持的表单。在`PersonForm`的bean中有两个属性，你可以看到这些属性标记为`th:field="{name}"`和`th:field="{age}"`。每个字段旁边的`th:errors`是用于显示验证错误的辅助元素。

最后，您可以按下按钮提交表单。一般来说，如果用户输入了一个名称或年龄，该名称或年龄违反了有效的约束，它将返回此页，并显示错误信息。如果输入了有效的名称和年龄，用户将被路由到下一个Web页面。

`src/main/resources/templates/results.html`

```
<html>
	<body>
		Congratulations! You are old enough to sign up for this site.
	</body>
</html>
```

> 在这个简单的例子中，这些网页没有任何复杂的CSS或JavaScript。但是对于任何生产环境下的网站，学习如何设计网页是很有价值的。

## 构建一个Application启动类
在这个程序中，使用了一个模板语言[Thymeleaf](http://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)。这个应用程序使用的不是原生HTML。

`src/main/java/hello/Application.java`


```
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```
使用Spring MVC，需要在`Application`上添加`@EnableWebMvc`注解。但是Spring Boot的`@SpringBootApplication`在你的classpath下检测spring-webmvc，并自动添加该注解。

Thymeleaf也会被`@SpringBootApplication`自动配置：默认的模板位置在classpath下的`templates/`，以'.html'为文件名后缀的文件才会被解析成views。Thymeleaf配置是可以改变的，你可以实现多种方式重写配置，但这方面知识不是本指南的重点。

## 创建一个可执行的JAR包

你可以通过Gradle或者Maven的命令行来启动这个应用程序。或者你可以构建一个包含所有需要的依赖，类文件和资源文件的独立可执行的JAR文件并且可以运行它。这样就可以很方便的传输，进行版本控制并且部署这个服务作为一个应用的时候无论在什么开发生命周期，不同的环境等等。

如果你使用Gradle，你可以运行你的应用通过使用`./gradlew bootRun`命令。或者你也可以构建一个JAR文件通过使用`./gralew build`。之后你就可以运行这个JAR文件了。

```
java -jar build/libs/gs-validating-form-input-0.1.0.jar
```
如果你使用Maven，你可以运行你的应用通过使用`./mvnw spring-boot:run`命令。或者你也可以构建一个JAR文件通过使用`./mvnw clean package`。之后你就可以运行这个JAR文件了。
```
java -jar target/gs-validating-form-input-0.1.0.jar
```
> 上述过程将会创建一个可运行的JAR文件，你也可以选择生成一个WAR包来代替。


该程序将在几秒后运行起来。
访问`http://localhost:8080/`，你将会看到如下信息：
![image](https://spring.io/guides/gs/validating-form-input/images/valid-01.png)
在name输入框内填写A，在age输入框中填写15，然后按下Submit按钮，会发生什么呢？
![image](https://spring.io/guides/gs/validating-form-input/images/valid-02.png)
![image](https://spring.io/guides/gs/validating-form-input/images/valid-03.png)
在这里你可以看到，因为它违反了`PersonForm`类的约束，你会返回到“main”的页面。如果您在表单中不输入任何信息点击提交，则会得到一个不同的错误。
![image](https://spring.io/guides/gs/validating-form-input/images/valid-04.png)
如果输入的name和age通过校验，你将会最终跳到结果页上！
![image](https://spring.io/guides/gs/validating-form-input/images/valid-05.png)

## 总结
恭喜！您已经编写了一个简单的Web应用程序，并包含了一个对象信息验证功能。通过这种方式，您可以确保数据符合某些标准，用户可以正确地输入数据。

## 其他
下面的指南可能也会有帮助:
- [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
- [Handling Form Submission](https://spring.io/guides/gs/handling-form-submission/)
- [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)


想写一个新的指南或为现有的指南提意见？ 请查看我们的[贡献准则](https://github.com/spring-guides/getting-started-guides/wiki)。
> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。
