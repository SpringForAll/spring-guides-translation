
> 原文：[Integrating Data](https://spring.io/guides/gs/integration/)
>
> 译者：[xuxiaoxie](https://github.com/xuxiaoxie)
>
> 校对：[null]()

本指南将引导您完成使用Spring Integration创建一个简单的应用程序，该应用程序从RSS源（Spring Blog）检索数据，操作数据，然后将其写入文件。
本指南使用传统的Spring Integration XML配置; 存在其他指南，其中显示了使用具有和不具有JDK 8 Lambda表达式的JavaConfig / DSL。

## 你会得到什么？

你将学会使用传统的xml配置方式创建一个spring Integration项目

## 你需要准备什么？

- 大概15分钟时间
- 一个自己喜欢的文本编辑器或者IDE
- [JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或者更高版本
- [Gradle 2.3+](http://www.gradle.org/downloads) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也能直接将代码导入到你的IDE里面:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts/)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
  
## 如何完成这份指南？

和大多数的[Spring入门指南](https://spring.io/guides)一样，你可以从头开始完成每一步，或者你可以绕过你已经熟悉的基本设置步骤。无论哪种方式，你最后都会得到一份可执行的代码。

没有相关基础知识的话, 请跳转到 [怎样使用 Gradle 构建项目](https://spring.io/guides/gs/integration/#scratch).

如果已经熟悉一些基本步骤，你可以:

- [下载并解压源码库](https://github.com/spring-guides/gs-integration/archive/master.zip)，或者通过 Git 工具克隆一份代码: `git clone https://github.com/spring-guides/gs-integration.git`
- cd into `gs-integration/initial`
- 往下看[定义一个集成流](https://spring.io/guides/gs/integration/#initial)。

**当你完成上述准备工作后**, 你可以在`gs-integration/complete`检查一下结果.

#  Gradle构建

首先你一个基本的构建脚本。 在使用Spring构建应用程序时，您可以使用任何您喜欢的构建系统，但是您需要使用与Gradle和Maven一起使用的代码。 如果您还不熟悉，请参阅使用Gradle构建Java项目或使用Maven构建Java项目。

首先你设置一个基本的构建脚本。 在使用Spring构建应用程序时，您可以使用任何您喜欢的构建系统, 但是你的代码需要跟 [Gradle](http://gradle.org/) 和 [Maven](https://maven.apache.org/)配套使用。 如果您还不熟悉，请参阅使用[Gradle构建Java项目](https://spring.io/guides/gs/gradle)或使用[Maven构建Java项目](https://spring.io/guides/gs/maven)。 

创建目录结构

在您选择的项目目录中，创建以下子目录结构;比如：在Linux/Unix系统中使用命令 `mkdir -p src/main/java/hello` 

```
└── src
    └── main
        └── java
            └── hello
```
创建Gradle构建文件

下面就是[Gradle初始化文件](https://github.com/spring-guides/gs-integration/blob/master/initial/build.gradle)

`build.gradle`

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
    baseName = 'gs-integration'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-integration")
    compile("org.springframework.integration:spring-integration-feed")
    compile("org.springframework.integration:spring-integration-file")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    testCompile("junit:junit")
}

tasks.withType(JavaExec) {
    standardInput = System.in
}


eclipse {
    project {
        natures += 'org.springframework.ide.eclipse.core.springnature'
    }
}
```
[Springboot gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多便捷的特性:

- 它收集类路径上的所有jar，并构建一个可运行的“über-jar”，这样可以更方便地执行和传输您的服务。
- 它搜索`public static void main（）`方法来标记为可运行类。
- 它提供了一个内置的依赖解析器，它将版本号设置为与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)关系相匹配。 您可以覆盖任何您想要的版本，但它将默认为Boot所选择的一组版本。

#  maven 构建

首先你设置一个基本的构建脚本。 但的代码需要与Maven一起配套使用。 如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

创建项目目录

在项目里面创建类似于如下的项目结构，例如，你也可以在*nix系统里通过命令`mkdir -p src/main/java/hello`来创建

```
└── src
    └── main
        └── java
            └── hello
```
`pom.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-integration</artifactId>
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
            <artifactId>spring-boot-starter-integration</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-feed</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-file</artifactId>
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
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>

</project>
```
SpringBoot maven插件提供了很多便捷的特性：

- 它收集类路径上的所有jar，并构建一个可运行的“über-jar”，这样可以更方便地执行和传输您的服务。
- 它搜索`public static void main（）`方法来标记为可运行类。
- 它提供了一个内置的依赖解析器，它将版本号设置为与[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)关系相匹配。 您可以覆盖任何您想要的版本，但它将默认为Boot所选择的一组版本。


# 使用你的IDE构建
- 阅读何将本指南直接导入到 [Spring Tool Suite](https://spring.io/guides/gs/sts/)中
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建.

## 定义集成流程

本指南的示例程序，将定义一个Spring Integration的集成流，从Spring IO的RSS提要中读取博客文章，将其转换为一个易于阅读的字符串，其中包含帖子标题和帖子的URL，并将其追加到文件`/tmp/si/SpringBlog`中

要定义集成流程，您只需创建Spring XML配置，其中包含Spring Integration的XML命名空间中的少量元素。 具体来说，对于所需的集成流程，您可以使用这些Spring Integration命名空间中的元素：core，feed和file。

以下的XML配置定义了集成流程：

`src/main/resources/hello/integration.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:int="http://www.springframework.org/schema/integration"
	xmlns:file="http://www.springframework.org/schema/integration/file"
	xmlns:feed="http://www.springframework.org/schema/integration/feed"
	xsi:schemaLocation="http://www.springframework.org/schema/integration/feed http://www.springframework.org/schema/integration/feed/spring-integration-feed.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/integration/file http://www.springframework.org/schema/integration/file/spring-integration-file.xsd
		http://www.springframework.org/schema/integration http://www.springframework.org/schema/integration/spring-integration.xsd">

    <feed:inbound-channel-adapter id="news" url="http://spring.io/blog.atom" auto-startup="${auto.startup:true}">
        <int:poller fixed-rate="5000"/>
    </feed:inbound-channel-adapter>

    <int:transformer
            input-channel="news"
            expression="payload.title + ' @ ' + payload.link + '#{systemProperties['line.separator']}'"
            output-channel="file"/>

    <file:outbound-channel-adapter id="file"
            mode="APPEND"
            charset="UTF-8"
            directory="/tmp/si"
            filename-generator-expression="'${feed.file.name:SpringBlog}'"/>

</beans>
```
正如你所看到的那样，这里有三个集成的元素:

`<feed:inbound-channel-adapter>`. 
一个入站适配器用来拉取推送的消息。每个轮询一个，正如这里所配置的，它每5秒轮询一次，帖子被放置到名为“news”的通道中（与适配器的ID相对应）

`<int:transformer>`. 在`news`通道中转换条目 (`com.rometools.rome.feed.synd.SyndEntry`)
，提取条目的标题（`payload.title`）和链接（`payload.link`）并将其连接成可读字符串（添加换行符）。 然后将字符串发送到名为“file”的输出通道。

`<file:outbound-channel-adapter`>.
出站通道适配器将通道（这里称为“file”）里的内容写入一个文件。具体来说，如在这里配置的，它将在“file”通道中的任何东西都写入`/tmp/si/SpringBlog`文件

这里有个简单的流程示意图:
![image](https://spring.io/guides/gs/integration/images/blogToFile.png)

现在这里忽略`auto-startup`属性，我们将会在讨论测试的时候会回顾一下;在默认情况下，只要注意，默认情况下将是true，这意味着应用程序启动时将会提取帖子。同时还要主意`filename-generator-expression`中的配置占位符，这意味着默认值会springblog，但可以用一个配置将其覆盖。

## 使程序可执行

尽管在较大的应用程序中甚至在Web应用程序中配置Spring Integration流程是很常见的，但是没有理由不能在简单的独立应用程序中定义它。这就是您接下来要做的，创建一个主类来启动集成流程，并声明一小部分bean来支持集成流。你也可以将应用程序构建到一个独立的可执行jar文件中。我们将用spring boot的
`SpringApplication`来创建应用的上下文。
由于这个指南给集成流程使用xml命名空间，我们使用`@ImportResource` 来加载应用的上下文

`src/main/java/hello/Application.java`

```
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ImportResource;

@SpringBootApplication
@ImportResource("/hello/integration.xml")
public class Application {
    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new SpringApplication(Application.class).run(args);
        System.out.println("Hit Enter to terminate");
        System.in.read();
        ctx.close();
    }

}
```
构建一个可执行的JAR

你可以通过命令行来运行Gradle或者Maven，或者构建一个单一的jar文件，这个jar文件包含所有的必须的依赖、类和配置。这使得在整个开发生命周期中，跨不同的环境等情况下，更容易对程序进行传输，版本管理，和服务部署。

如果你使用的是Gradle，你可以使用 `./gradlew bootRun`运行程序，或者你可以用`./gradlew build`构建JAR文件，然后使用下面的命令运行jar文件：

```
java -jar build/libs/gs-integration-0.1.0.jar
```

当你使用maven的话，你可以通过`./mvnw spring-boot:run`命令来运行项目，或者你可以用`./mvnw clean package`命令来构建一个jar文件的项目：

```
java -jar target/gs-integration-0.1.0.jar
```

> 上述的过程将创建一个可执行的jar文件，当然，您也可以选择构建一个经典的WAR文件。


## 运行项目

现在你可以将下面的jar应用运行起来：

```
java -jar build/libs/{project_id}-0.1.0.jar

... app starts up ...
```
一旦应用运气起来的话，他将会连接RSS 订阅，并且抓取博客的推送，程序将会通过你定义的集成流处理，最终将把获取的信息写到文件`/tmp/si/SpringBlog`里

当程序运行一段时间后，你可以打开文件`/tmp/si/SpringBlog`去看看是否已经处理了一批数据了。在Linux/NUIX的操作系统里面，你可以用tail命令去查看写入的数据。

```
tail -f /tmp/si/SpringBlog
```
你将会看到这样的东西（实际的新闻将会有所不同）

```
Spring Integration Java DSL 1.0 GA Released @ https://spring.io/blog/2014/11/24/spring-integration-java-dsl-1-0-ga-released
This Week in Spring - November 25th, 2014 @ https://spring.io/blog/2014/11/25/this-week-in-spring-november-25th-2014
Spring Integration Java DSL: Line by line tutorial @ https://spring.io/blog/2014/11/25/spring-integration-java-dsl-line-by-line-tutorial
Spring for Apache Hadoop 2.1.0.M2 Released @ https://spring.io/blog/2014/11/14/spring-for-apache-hadoop-2-1-0-m2-released
```
**测试**

仔细看`complete`项目，你将会看见一个测试用例


`src/test/java/hello/FlowTests.java`

```
package hello;

import static org.assertj.core.api.Assertions.assertThat;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;

import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.integration.endpoint.SourcePollingChannelAdapter;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.MessageChannel;
import org.springframework.test.context.junit4.SpringRunner;

import com.rometools.rome.feed.synd.SyndEntryImpl;

@RunWith(SpringRunner.class)
@SpringBootTest({ "auto.startup=false",      // we don't want to start the real feed
                  "feed.file.name=Test" })   // use a different file
public class FlowTests {

	@Autowired
	private SourcePollingChannelAdapter newsAdapter;

	@Autowired
	private MessageChannel news;

	@Test
	public void test() throws Exception {
		assertThat(this.newsAdapter.isRunning()).isFalse();
		SyndEntryImpl syndEntry = new SyndEntryImpl();
		syndEntry.setTitle("Test Title");
		syndEntry.setLink("http://foo/bar");
		File out = new File("/tmp/si/Test");
		out.delete();
		assertThat(out.exists()).isFalse();
		this.news.send(MessageBuilder.withPayload(syndEntry).build());
		assertThat(out.exists()).isTrue();
		BufferedReader br = new BufferedReader(new FileReader(out));
		String line = br.readLine();
		assertThat(line).isEqualTo("Test Title @ http://foo/bar");
		br.close();
		out.delete();
	}

}
```
这里使用了Spring Boot的测试功能，将`auto.startup`属性设置成`false`，一般来说，依赖于网络连接的测试并不是一个好主意。尤其是在CI环境中。相反，我们会阻止feed适配器启动并将`SyndEntry`注入到`news`通道，以便其他的流程进行处理。这个测试同时设置了`feed.file.name`,所以输出的内容将写入不同的文件

- 验证适配器是否已经停止
- 创建一个测试 `SyndEntry`
- 删除测试生成的文件（如果有的话）
- 发送消息
- 校验文件是否存在
- 读取文件并验证数据是否符合预期

**总结**

恭喜你！你已经开发了一个简单的 用来获取spring.io博客推送并写入文件的 Spring Integration 应用。

**参见**

下面的指南也许也同样很有帮助：

- [如何创建一个springboot应用](https://spring.io/guides/gs/spring-boot/)




> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](https://creativecommons.org/licenses/by-nc-sa/4.0/%29%E5%8D%8F%E8%AE%AE%E8%BF%9B%E8%A1%8C%E8%AE%B8%E5%8F%AF%E3%80%82)