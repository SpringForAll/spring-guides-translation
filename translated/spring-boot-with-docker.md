# Spring Boot应用Docker化

> 原文：[Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)
> 
> 译者：[马勇斌](https://https://github.com/stormmaybin)
>
> 校对：[張偉文](https://github.com/carlzhangweiwen)


本教程将引导你通过建立一个Docker镜像来运行Spring Boot应用。

## 你将构建什么
Docker是一个具有社區性的Linux容器管理工具集，它允許用户发布镜像或者使用其他开发者发布的镜像。Docker镜像本質上是一个進程的运行環境。在這篇guide中，我們将构建一个運行Spring Boot应用程序的鏡像。

## 构建之前你要准备的东西

- 大约十五分钟的时间
- 一个你最喜欢的文本编辑器或者IDE
- [JDK1.8+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Gradle2.3+](http://www.gradle.org/downloads)或者[Maven3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入你的IDE
	- [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  	- [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

如果你当前使用的系统不是Linux，那么你需要一个虚拟机。通过安装VirtualBox或者像mac的boot2docker等其他工具都可以满足你的使用。
访问[VirtualBox下载地址](https://www.virtualbox.org/wiki/Downloads)选择你的系统版本来下载VirtualBox，并且安装运行起来。

当然你也需要安装[Docker](https://docker.com/)（只能在64位机器上运行），有关详细信息请参阅[https://docs.docker.com/installation/#installation](https://docs.docker.com/installation/#installation)来安装Docker到你的系统上。开始下一步之前，你要确认在你的shell上是否可以运行`docker`命令，如果你使用的是boot2docker，在运行`docker`命令之前你需要先启动boot2docker。


## 如何完成这篇教程
像大多数的Spring系列教程 [Getting Started guides](https://spring.io/guides)，你可以从头开始，完成每一步，也可以跳过已经熟悉的基本構建步骤。 无论哪种方式，你都可以成功。

**从头开始**, 参阅[使用Gradle构建](https://spring.io/guides/gs/spring-boot-docker/#scratch).

**跳过基础部分**, 执行以下操作:

- [下载](https://github.com/spring-guides/gs-spring-boot-docker/archive/master.zip)并解压本仓库代码，或者使用git命令克隆代码库：`git clone https://github.com/spring-guides/gs-spring-boot-docker.git`
- cd到如下目录：`gs-spring-boot-docker/initial`
- 前往[创建Spring Boot应用程序](https://spring.io/guides/gs/spring-boot-docker/#initial)

**完成以上操作之后**，你可以和如下代码进行比对：`gs-spring-boot-docker/complete`

## 使用Gradle构建
首先你得安装构建脚本. 你可以使用你喜欢的构建系统去构建Spring应用, 你需要的工具在下面都可以找到: [Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/) . 如果你对[Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/)都不熟悉，请参阅[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle)或者[使用Maven构建Java项目](https://spring.io/guides/gs/maven).

### 创建目录结构
在你当前项目工作目录中，创建如下子目录:

```text
└── src
    └── main
        └── java
            └── hello
```

如在Linux系统下使用`mkdir -p src/main/java/hello`来创建。

### 创建Gradle项目文件(build.gradle)
如下是一个[gradle项目文件](https://github.com/spring-guides/gs-spring-boot-docker/blob/master/initial/build.gradle)

`build.gradle`

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.4.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-spring-boot-docker'
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
```

 [Spring Boot gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的功能:

- 它集中了`classpath`下的所有jar，并构建一个可运行的jar，这样可以更方便地执行和发布服务。
- 它找到`public static void main()`方法并标记该类为可执行。
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改成你希望的版本，但它默认为Spring Boot选择的版本。

## 使用maven构建
首先你得安装构建脚本. 你可以使用你喜欢的构建系统去构建Spring应用，你可以前往[Maven官网](https://maven.apache.org/)来指导你安装。 如果你对[Maven](https://maven.apache.org/)不熟悉，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven).

### 创建目录结构
在你当前项目工作目录中，创建如下子目录:

```text
└── src
    └── main
        └── java
            └── hello
```

如在Linux系统下使用`mkdir -p src/main/java/hello`来创建。

`pom.xml`


```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-spring-boot-docker</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.4.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

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

[Spring Boot maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多便捷的功能:

- 它集中了`classpath`下的所有jar，并构建一个可运行的“uber-jar”，这样可以更方便地执行和发布服务。
- 它查找`public static void main()`方法并标记该类为可执行类。
- 它提供了一个内置的依赖解析器，将应用与Spring Boot依赖的版本号进行匹配。你可以修改（override）成你希望的版本，但它默认为Spring Boot选择的版本。

## 使用你的IDE构建
- 参阅[Spring Tool Suite](https://spring.io/guides/gs/sts/)如何直接导入到你的IDE。
- 参阅[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)如何来构建。

## 创建一个Spring Boot应用程序
现在你可以创建一个简单的Spring Boot应用程序。

`src/main/java/hello/Application.java`

```java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello Docker World";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

这个类使用`@SpringBootApplication`注解和`@RestController`注解标记，表示這個類已經准备好使用Spring MVC来处理web请求。`@RequestMapping`映射`/`在`home()`方法上，這個方法返回`Hello Docker World`的响应。`main()`方法调`SpringApplication.run()`方法来启动应用程序。

现在你可以在没有Docker的情况下运行该应用程序。

如果你使用gradle构建的Spring Boot应用程序，执行:

`./gradlew build && java -jar build/libs/gs-spring-boot-docker-0.1.0.jar`

如果你使用maven构建的，执行:
`./mvnw package && java -jar target/gs-spring-boot-docker-0.1.0.jar
`

然后访问[localhost:8080](http://localhost:8080/)就可以看见`Hello Docker World`字样。


## 使你的应用程序容器化
Docker有一个[Dockerfile](https://docs.docker.com/reference/builder/)文件格式的文件，并且使用指定的镜像作为基础镜像。接下来我们去创建一个Dockerfile在Spring Boot应用程序中。

`Dockerfile`


```dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD target/gs-spring-boot-docker-0.1.0.jar app.jar
ENV JAVA_OPTS=""
ENTRYPOINT exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar
```

这个Dockerfile很简单，但是已经包含了你运行这个Spring Boot应用程序的所有东西并且没有多余的东西：仅仅只有Java和一个Jar文件。应用程序的jar文件(文件名为app.jar)被添加到容器中，执行 `ENTRYPOINT`

> 我们增加了一个`VOLUME`指向"/tmp"，因为那是Spring Boot应用程序为Tomcat创建的默认工作目录。作用是在你的主机"/var/lib/docker"目录下创建一个临时的文件，并且链接到容器中的"/tmp"目录。对于简单程序这一步是可选的，但是对于其他想要真实写入文件系统的Spring Boot应用程序又是必选的。


> 我们增加了一个指向"/dev/urandom"的Tomcat系统属性来缩小Tomcat的启动时间。

> 如果你使用的是boot2docker，在你使用Docker命令或者使用构建工具构建(它运行在守护进程，在虚拟机上为你工作)之前，你必须先启动它。

构建Docker镜像，你可以使用一些诸如Gradle或Maven的工具或者借助社区(强烈感谢[Transmode](https://github.com/Transmode/gradle-docker)和[Spotify](https://github.com/spotify/dockerfile-maven)提供这些工具)。

### 使用Maven构建Docker镜像
在Maven的`pom.xml`中，你应该增加一个插件，如下。(参阅[the plugin documentation](https://github.com/spotify/dockerfile-maven)获取更多信息)。

`pom.xml`


```xml
<properties>
   <docker.image.prefix>springio</docker.image.prefix>
</properties>
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.3.4</version>
            <configuration>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
            </configuration>
        </plugin>
    </plugins>
</build>
```
配置指定了两个东西:

- 镜像名字，最终会在`springio/gs-spring-boot-docker`
- 可选，镜像的标签，未指定默认为`latest`，如果有需要的话，也可以指定`artifact id`。

> 继续下面的步骤之前(使用Docker CLI工具)，输入`docker ps`确保正常运行，如果有错误信息，大概因为没有设置其他什么信息。如果使用的Mac，在`.bash_profile`(或者是类似env-setting配置文件)最后增加`$(boot2docker shellinit 2> /dev/null)`并刷新你的shell，确保更改生效。

你可以使用下面的命令来构建Docker的镜像:


```bash
$ ./mvnw install dockerfile:build
```

你可以使用`./mvnw dockerfile:push`命令发布镜像到`Docker`镜像中心去。

> 你不用着急发布你刚刚制作的镜像，如果你不是"springio"组织的成员`push`命令会失败。改变构建的配置和命令行中的用户名为自己的而不是"springio"就可以正常工作。

> 你可以让`dockerfile:push`自动运行在安装或部署生命周期阶段，通过将其添加到插件配置中。

`pom.xml`


```xml
<executions>
	<execution>
		<id>default</id>
		<phase>install</phase>
		<goals>
			<goal>build</goal>
			<goal>push</goal>
		</goals>
	</execution>
</executions>

```

### 使用Gradle构建Docker镜像
如果你使用Gradle构建，添加如下插件:

`build.gradle`

```groovy
buildscript {
    ...
    dependencies {
        ...
        classpath('se.transmode.gradle:gradle-docker:1.2')
    }
}

group = 'springio'

...
apply plugin: 'docker'

task buildDocker(type: Docker, dependsOn: build) {
  applicationName = jar.baseName
  dockerfile = file('Dockerfile')
  doFirst {
    copy {
      from jar
      into "${stageDir}/target"
    }
  }
}
```

该配置指定了三个东西:
- 从jar文件属性设置镜像名字(或标签)，最终会出现在`springio/gs-spring-boot-docker`
- Dockerfile的位置
- 复制Maven构建目录中的jar文件到相同的目录，这样我们就可以用Maven或者Gradle构建镜像而且使用同一个`Dockerfile`。

你可以使用以下命令构建镜像并且发布到远程仓库:

```bash
$ ./gradlew build buildDocker

```

### 发布之后
对你来说"Docker镜像发布"会失败(除非你在Dockerhub中是"springio"组织的一员)，但是如果你修改配置符合你自己的docker ID之后同样会成功，你会有一个新的标签，并且部署镜像。

> 如果你没有注册docker，或者没有发布任何docker镜像。但是你有一个本地的镜像，你可以像这样让它运行起来:
> ```
> $ docker run -p 8080:8080 -t springio/gs-spring-boot-docker
....
2015-03-31 13:25:48.035  INFO 1 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2015-03-31 13:25:48.037  INFO 1 --- [           main] hello.Application                        : Started Application in 5.613 seconds (JVM running for 7.293)
> ```

然后应用程序可以在[http://localhost:8080/](http://localhost:8080/)访问(访问可以看到"Hello Docker World")。确保它是真的在工作，把"springio"前缀改成其他(比如`${env.USER}`并且重新构建运行)。

> 如果你使用mac的boot2docker，你通常在启动时可以看到这样的事情:
> 
```bash
Docker client to the Docker daemon, please set:
    export DOCKER_CERT_PATH=/Users/gturnquist/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1
    export DOCKER_HOST=tcp://192.168.59.103:2376
```
>查看应用程序，你必须访问Docker主机而不是本地主机的ip(localhost)，在这种情况下，[http://192.168.59.103:8080/](http://192.168.59.103:8080/)，对于VM来说就是开发的ip。

当它正在运行，你可以在容器列表中看到:

```bash
$ docker ps
CONTAINER ID        IMAGE                             COMMAND                CREATED             STATUS              PORTS                    NAMES
81c723d22865        springio/gs-spring-boot-docker:latest   "java -jar /app.jar"   34 seconds ago      Up 33 seconds       0.0.0.0:8080->8080/tcp   goofy_brown
```

你可以使用`docker stop`加上容器的id来关闭它。

```bash
$ docker stop 81c723d22865
81c723d22865
```

当然你喜欢你也可以删除容器(它们都保存在`/var/lib/docker`文件目录下)。


```bash
$ docker rm 81c723d22865
```

### 使用Spring Profiles
运行你新制作的镜像使用Spring Profiles，和运行docker命令传入一个环境变量一样简单。

```bash
$ docker run -e "SPRING_PROFILES_ACTIVE=prod" -p 8080:8080 -t springio/gs-spring-boot-docker
```
或


```bash
$ docker run -e "SPRING_PROFILES_ACTIVE=dev" -p 8080:8080 -t springio/gs-spring-boot-docker
```

### 在Docker容器中调试应用程序
使用[JPDA Transport](http://docs.oracle.com/javase/8/docs/technotes/guides/jpda/conninv.html#Invocation)可以调试应用程序，所以我们可以视容器为一个服务器，启用此功能通过java设置JAVA_OPTS变量和代理的端口映射到本地主机在一个容器中运行。对于[Docker for Mac](https://www.docker.com/products/docker#/mac)由于有限制,我们不能通过IP访问容器因为没有[black magic usage.](https://github.com/docker/for-mac/issues/171)。


```bash
$ docker run -e "JAVA_OPTS=-agentlib:jdwp=transport=dt_socket,address=5005,server=y,suspend=n" -p 8080:8080 -p 5005:5005 -t springio/gs-spring-boot-docker
```

## 总结
恭喜你!刚刚为Spring Boot应用程序创建了Docker容器!Spring Boot应用程序在容器中运行默认端口8080映射到相同的宿主机端口使用"-p"参数。

## 另请参阅
以下教程也可能是有用的:

- [Serving Web Content with Spring MVC
](https://spring.io/guides/gs/serving-web-content/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)

想写一篇新的教程？或者为旧教程出力？查看我们的[贡献参考](https://github.com/spring-guides/getting-started-guides/wiki)

