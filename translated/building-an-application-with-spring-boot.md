# 用Spring Boot创建应用
======================================================================
> 原文：[Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
>
> 译者：[nycgym](https://github.com/nycgym)
>
> 校对：

本指南提供了一个例子，旨在说明Spring Boot是怎样加速你开发应用程序的。如果你有兴趣
可以阅读更多关于Spring系列的入门指南，在这里你将会找到更多有用的例子。这个例子将会
让你体验Spring Boot的便捷快速。如果你想创建自己的Spring Boot项目，可以参照这个[ Spring Initializr](https://start.spring.io/)，
在这个页面你需要填写自己项目的详细信息，选择项目配置，接下来你可以选择生成Maven构建文件或者直接生成zip包来获取到自己的项目。

## 你会创建什么
你将会使用Spring Boot构建一个简单的web程序，而且可以在程序中加入一些实用的功能

## 你需要的准备

* 花费15分钟
* 你喜欢的编辑器
* [JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)以上
* [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
* 你也可以直接将现成的代码导入编辑器:
    > + [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
       + [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本项引导
就像大多数的Spring系列入门指南，你可以从零开始，按照指南中的每一步的指示来做，或者你也可以
跳过你已经熟悉的基本教程。不论那种方式，最后我们想要的结果都是可以运行的代码。

*从零开始*，移步至[ Build with Gradle](https://spring.io/guides/gs/spring-boot/#scratch)

*跳过基本教程*，需要做下列操作：
* [下载](https://github.com/spring-guides/gs-spring-boot/archive/master.zip)本指南的源码zip包并解压，或者直接使用Git获取源码
> git clone https://github.com/spring-guides/gs-spring-boot.git

* 打开目录 ``` gs-spring-boot/initial ```

* 初始化项目

*当你做完以上的几步*，你可以在如下目录检查你的代码
>gs-spring-boot/complete

## 使用Gradle创建项目

首先设置一个基本的构建脚本。 在使用Spring构建应用程序时，你可以使用任何你喜欢的项目构建系统，但是你用Maven或Gradle构建的代码必须
要在这个系统下。如果你还不熟悉，请参阅[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle)或[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

## 创建目录结构
在您选择的项目目录中，创建以下子目录结构;例如，在 unix系统使用
``` mkdir -p src / main / java / hello ```

    └── src
        └── main
            └── java
                └── hello

### 创建一个Gradle构建文件

下面是[Gradle的初始化构建文件](https://github.com/spring-guides/gs-spring-boot/blob/master/initial/build.gradle)

``` build.gradle ```

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

Spring Boot的[Gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多方便的功能

* 它会把类路径下的所有jar包都打包最后生成一个可运行的"über-jar"，可以方便你的运行和服务调用。
* 它会把 ``` public static void main() ```方法作为入口方法
* 它提供了一个内置的依赖解析器，可以自动设置适应Spring Boot依赖的版本。你也可以自定义版本号，但这个版本号必须是Spring Boot默认存在的版本号。

## 使用Maven创建项目

首先设置一个基本的构建脚本。 在使用Spring构建应用程序时，你可以使用任何你喜欢的项目构建系统，但是你用Maven构建的代码必须
要在这个系统下。如果你还不熟悉，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构
在您选择的项目目录中，创建以下子目录结构;例如，在 unix系统使用
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


        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>

    </project>

Spring Boot的[Maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多方便的功能

* 它会把类路径下的所有jar包都打包最后生成一个可运行的"über-jar"，可以方便你的运行和服务调用。
* 它会把 ``` public static void main() ```方法作为入口方法
* 它提供了一个内置的依赖解析器，可以自动设置适应Spring Boot依赖的版本。你也可以自定义版本号，但这个版本号必须是Spring Boot默认存在的版本号。

##使用你的编辑器创建项目

* 了解下如何把本项引导的代码导入[Spring Tool Suite](https://spring.io/guides/gs/sts/)。
* 了解下在[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)中如何完成本项引导。

## 了解使用Spring Boot你可以做哪些事
Spring Boot提供了快速构建应用程序的途径。 它查看你的类路径下的配置文件和你配置的bean，自动补充你缺少的配置。 使用Spring Boot，你可以更多地关注业务功能，减少基础架构的过程。

#### 例如

* 想要使用Spring MVC？有几个特定的bean你几乎总是需要，而Spring Boot会自动添加它们。Spring MVC应用程序还需要一个servlet容器，所以Spring Boot内置了Tomcat容器。
* 想要使用Jetty？你可能想使用Jetty而不是Tomca。 Spring Boot也内置了Jetty，为你解决后顾之忧。
* 想要使用Thymeleaf？有几个特定的bean你几乎总是引用，而Spring Boot已经自动添加它们。

这些只是Spring Boot自动配置的几个例子。Spring Boot的强大远不止如此。 例如，如果在你的类路径上配置Thymeleaf的配置文件，Spring Boot会自动向你的应用程序添加一个SpringTemplateEngine。 但是如果你使用自己定义的SpringTemplateEngine，那么Spring Boot将不会自动配置。 如此一来，你就不能随心所欲地配置项目，但是却可以减少很多文件的配置。

> Spring Boot不会生成代码或对文件进行编辑。相反，当您启动应用程序时，Spring Boot会动态地把你定义的bean和项目配置链接起来，并将其应用于应用程序。

###创建一个简单的Web程序

现在你可以在Web程序中创建一个controller类

``` src/main/java/hello/HelloController.java ```

        package hello;

        import org.springframework.web.bind.annotation.RestController;
        import org.springframework.web.bind.annotation.RequestMapping;

        @RestController
        public class HelloController {

            @RequestMapping("/")
            public String index() {
                return "Greetings from Spring Boot!";
            }

        }
该类被标记为```@RestController```，这意味着它可以使用Spring MVC来处理Web请求。 ```@RequestMapping```映射/到index（）方法。 当从浏览器调用或在命令行使用curl时，该方法返回纯文本。 这是因为```@RestController```结合了```@Controller```和```@ResponseBody```，两个注释导致Web请求返回数据而不是视图。

## 创建一个Application类

现在，你将要创建一个包含其他组件的 ``` Application ```类

``` src/main/java/hello/Application.java ```

    package hello;

    import java.util.Arrays;

    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.annotation.Bean;

    @SpringBootApplication
    public class Application {

        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }

        @Bean
        public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
            return args -> {

                System.out.println("Let's inspect the beans provided by Spring Boot:");

                String[] beanNames = ctx.getBeanDefinitionNames();
                Arrays.sort(beanNames);
                for (String beanName : beanNames) {
                    System.out.println(beanName);
                }

            };
        }

    }

```@SpringBootApplication```是一个方便开发的注解，它包含了下列几个注解：

* ```@Configuration```将该类标记为应用程序中bean定义的源。
* ```@EnableAutoConfiguration```告诉Spring Boot根据类路径中的项目配置，其他定义的bean，和其他的配置文件来加载bena。
* 通常，您将为Spring MVC应用程序添加```@EnableWebMvc```注解，但是当Spring Boot在类路径中看到*spring-webmvc*时，Spring Boot会自动添加这个注解。这样应用程序会被标记为Web程序，并激活一系列关键指令比如设置```DispatcherServlet```。
* ```@ComponentScan``` 告诉Spring在```hello```包中扫描components, configurations, 和services,也允许其扫描controllers.

```main（）```方法使用Spring Boot的```SpringApplication.run（）```方法启动应用程序。 你注意到没有？没有一行XML 没有```web.xml```文件。 此Web应用程序是纯Java代码，你无需担心任何其他的配置。

还有一个```CommandLineRunner```方法标记了```@Bean```注解，它在程序启动时运行。它检索由您的应用程序创建或者通过Spring Boot自动添加的所有bean，并会把他们排序打印出来。

## 运行应用程序

执行以下命令运行程序：

```./gradlew build && java -jar build/libs/gs-spring-boot-0.1.0.jar```

如果使用Maven，执行：

```mvn package && java -jar target/gs-spring-boot-0.1.0.jar```

你应该可以看到如下的输出：

    Let's inspect the beans provided by Spring Boot:
    application
    beanNameHandlerMapping
    defaultServletHandlerMapping
    dispatcherServlet
    embeddedServletContainerCustomizerBeanPostProcessor
    handlerExceptionResolver
    helloController
    httpRequestHandlerAdapter
    messageSource
    mvcContentNegotiationManager
    mvcConversionService
    mvcValidator
    org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration
    org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration
    org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration
    org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$DispatcherServletConfiguration
    org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$EmbeddedTomcat
    org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration
    org.springframework.boot.context.embedded.properties.ServerProperties
    org.springframework.context.annotation.ConfigurationClassPostProcessor.enhancedConfigurationProcessor
    org.springframework.context.annotation.ConfigurationClassPostProcessor.importAwareProcessor
    org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    org.springframework.context.annotation.internalCommonAnnotationProcessor
    org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    org.springframework.context.annotation.internalRequiredAnnotationProcessor
    org.springframework.web.servlet.config.annotation.DelegatingWebMvcConfiguration
    propertySourcesBinder
    propertySourcesPlaceholderConfigurer
    requestMappingHandlerAdapter
    requestMappingHandlerMapping
    resourceHandlerMapping
    simpleControllerHandlerAdapter
    tomcatEmbeddedServletContainerFactory
    viewControllerHandlerMapping

您可以清楚地看到*org.springframework.boot.autoconfigure* bean。 还有一个```tomcatEmbeddedServletContainerFactory```。

检查服务启动情况

    $ curl localhost:8080
    Greetings from Spring Boot!

## 增加单元测试

您将需要为创建的程序添加一个测试，Spring Test已经提供了一系列功能，并且与你所创建的项目完美兼容。

把这个加入你的构建文件中，导入相关依赖：

     testCompile("org.springframework.boot:spring-boot-starter-test")

如果你是用的是Maven，把这个加入依赖：

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

现在写一个简单的单元测试，通过你的程序模拟servlet请求和响应：

```src/test/java/hello/HelloControllerTest.java```

    package hello;

    import static org.hamcrest.Matchers.equalTo;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.http.MediaType;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

    @RunWith(SpringRunner.class)
    @SpringBootTest
    @AutoConfigureMockMvc
    public class HelloControllerTest {

        @Autowired
        private MockMvc mvc;

        @Test
        public void getHello() throws Exception {
            mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())
                    .andExpect(content().string(equalTo("Greetings from Spring Boot!")));
        }
    }

```MockMvc```在Spring Test中，可以让你通过一组方便的相关构造器向```DispatcherServlet```发送HTTP请求,并对结果进行断言。注意，```@AutoConfigureMockMvc```和```@SpringBootTest```要一起使用注入```MockMvc```实例。使用```@SpringBootTest```要求整个应用程序创建成功。还有一种办法，使用```@WebMvcTest```只访问Spring Boot创建的web层。无论哪一种方式，Spring Boot都会自动尝试去定位你应用程序中的主类。当然，你可以重写这个配置，或者缩小它的寻找范围。

除了模拟HTTP请求周期，我们还可以使用Spring Boot来编写一个非常简单的全栈集成测试。例如，我们可以这样做，而不是（或者说包含）上述的模拟测试

```src/test/java/hello/HelloControllerIT.java```

    package hello;

    import static org.hamcrest.Matchers.equalTo;
    import static org.junit.Assert.assertThat;

    import java.net.URL;

    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.context.embedded.LocalServerPort;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.boot.test.web.client.TestRestTemplate;
    import org.springframework.http.ResponseEntity;
    import org.springframework.test.context.junit4.SpringRunner;

    @RunWith(SpringRunner.class)
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    public class HelloControllerIT {

        @LocalServerPort
        private int port;

        private URL base;

        @Autowired
        private TestRestTemplate template;

        @Before
        public void setUp() throws Exception {
            this.base = new URL("http://localhost:" + port + "/");
        }

        @Test
        public void getHello() throws Exception {
            ResponseEntity<String> response = template.getForEntity(base.toString(),
                    String.class);
            assertThat(response.getBody(), equalTo("Greetings from Spring Boot!"));
        }
    }

内嵌的服务器将通过```webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT```在随机端口上启动，实际端口在运行时使用```@LocalServerPort```监控。

## 添加生产环境中实用的服务

如果您正在为您的业务构建一个网站，则可能需要添加一些管理服务。 Spring Boot提供几个开箱即用的[监控模块](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle/#production-ready)，如 health, audits, beans等。

把这个加入你的构建文件中，导入相关依赖：

      compile("org.springframework.boot:spring-boot-starter-actuator")

如果你是用的是Maven，把这个加入依赖：

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

然后重启程序：

    ./gradlew build && java -jar build/libs/gs-spring-boot-0.1.0.jar

如果你使用Maven，执行：

    mvn package && java -jar target/gs-spring-boot-0.1.0.jar

您将看到一组新的RESTful端点添加到应用程序中。这就是Spring Boot提供的管理服务。

    2014-06-03 13:23:28.119  ... : Mapped "{[/error],methods=[],params=[],headers=[],consumes...
    2014-06-03 13:23:28.119  ... : Mapped "{[/error],methods=[],params=[],headers=[],consumes...
    2014-06-03 13:23:28.136  ... : Mapped URL path [/**] onto handler of type [class org.spri...
    2014-06-03 13:23:28.136  ... : Mapped URL path [/webjars/**] onto handler of type [class ...
    2014-06-03 13:23:28.440  ... : Mapped "{[/info],methods=[GET],params=[],headers=[],consum...
    2014-06-03 13:23:28.441  ... : Mapped "{[/autoconfig],methods=[GET],params=[],headers=[],...
    2014-06-03 13:23:28.441  ... : Mapped "{[/mappings],methods=[GET],params=[],headers=[],co...
    2014-06-03 13:23:28.442  ... : Mapped "{[/trace],methods=[GET],params=[],headers=[],consu...
    2014-06-03 13:23:28.442  ... : Mapped "{[/env/{name:.*}],methods=[GET],params=[],headers=...
    2014-06-03 13:23:28.442  ... : Mapped "{[/env],methods=[GET],params=[],headers=[],consume...
    2014-06-03 13:23:28.443  ... : Mapped "{[/configprops],methods=[GET],params=[],headers=[]...
    2014-06-03 13:23:28.443  ... : Mapped "{[/metrics/{name:.*}],methods=[GET],params=[],head...
    2014-06-03 13:23:28.443  ... : Mapped "{[/metrics],methods=[GET],params=[],headers=[],con...
    2014-06-03 13:23:28.444  ... : Mapped "{[/health],methods=[GET],params=[],headers=[],cons...
    2014-06-03 13:23:28.444  ... : Mapped "{[/dump],methods=[GET],params=[],headers=[],consum...
    2014-06-03 13:23:28.445  ... : Mapped "{[/beans],methods=[GET],params=[],headers=[],consu...

它们包括:errors, environment, health, beans, info, metrics, trace, configprops, 和dump.,

> 还有一个```/shutdown```，但是默认情况下只能通过JMX显示。 [要将其作为HTTP端点启用](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle/#production-ready-customizing-endpoints)，请将```endpoints.shutdown.enabled = true```添加到```application.properties```文件中。

要检查应用的健康状况很简单：

    $ curl localhost:8080/health
    {"status":"UP","diskSpace":{"status":"UP","total":397635555328,"free":328389529600,"threshold":10485760}}}

你也可以通过curl调用关闭命令

    $ curl -X POST localhost:8080/shutdown
    {"timestamp":1401820343710,"error":"Method Not Allowed","status":405,"message":"Request method 'POST' not supported"}

因为我们没有启用它，所以请求被拒绝

有关每个REST点的更多详细信息以及如何使用```application.properties```文件调整其设置，您可以阅读有关端点的[详细文档](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle/#production-ready-endpoints)。

## 查看Spring Boot的各种staters

你已经看到过一些[starter](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle/#using-boot-starter)。你也可以直接参照[源码](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters)了解。

## JAR支持和Groovy支持

最后一个例子显示了Spring Boot简单的使应用中的bean链接，致使你可能意识不到自己到底需要。同时，也展示了Spring Boot是如何方便快捷地实现管理服务的。

不止如此，Spring Boot还有更多的功能。 它不仅支持传统的WAR文件部署，而且还可以轻松地将可执行JAR放在一起，这得益于Spring Boot的加载器模块。各种教程通过```spring-boot-gradle-plugin```和```spring-boot-maven-plugin```来实现这种双重支持。

除此之外，Spring Boot还支持Groovy，允许您使用仅一个文件构建Spring MVC Web应用程序。

创建一个名为*app.groovy*文件，在文件中写入下列代码：

    @RestController
    class ThisWillActuallyRun {

        @RequestMapping("/")
        String home() {
            return "Hello World!"
        }

    }

文件放在哪里无关紧要。你甚至可以在一个小的推特里嵌入一个程序。

接下来，安装[Spring Boot’s CLI](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle/#getting-started-installing-the-cli)

运行如下命令：

    $ spring run app.groovy

> 以上命令前提是你关闭了以前的应用程序，以避免端口冲突。

令起一个窗口：

    $ curl localhost:8080
    Hello World!

Spring Boot在你的代码中动态添加关键注释，并使用Groovy Grape来下拉使应用程序运行所需的库。

##总结

恭喜！ 您使用Spring Boot构建了一个简单的Web应用程序，并了解了如何提高开发速度。 你也添加了一些方便的生产服务。 这只是Spring Boot可以做的一些小例子。 如果要深入挖掘Spring Boot,请参阅[在线文档](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle)。

##你也可以看

下列的文章对你可能也有帮助：

*[保护wen应用](https://spring.io/guides/gs/securing-web/)
*[使用Spring MVC提供Web服务](https://spring.io/guides/gs/serving-web-content/)

------------------------------------------------------------------

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
