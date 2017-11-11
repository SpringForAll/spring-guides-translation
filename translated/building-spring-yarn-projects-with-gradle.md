# Gradle构建Spring YARN项目

> 原文：[Building Spring YARN Projects with Gradle](https://spring.io/guides/gs/gradle-yarn/) 
>
> 译者：[UniKrau](https://github.com/UniKrau)
>
> 校对：[dyc87112](https://github.com/dyc87112)


本指南主要探讨Gradle构建一个简单的Spring YARN工程

>
> 本指南集中在构建项目和模型上，不需要一个完全可用的YARN程序
>

## 你会学到什么

会创建和用Gradle编译一个YARN应用程序

## 准备工作

* 大约15分钟
* 文本编辑器或者是IDE
* 需要[JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)1.6以上版本 

## 如何完成指南

像大多数[Spring 入门文章](https://spring.io/guides)一样，你可以逐渐的完成每一步，也可以跳过一些你熟悉的步骤。不管怎样，最后你都将得到一份可执行的代码。

**如果从基础开始**参考[配置工程](#gradle_id)

**如果已经熟悉跳过一些基本步骤**你可以这样

* 需要[下载](https://github.com/spring-guides/gs-gradle-yarn/archive/master.zip)和解压源代码文件，或者用[Git](https://spring.io/understanding/Git)克隆: `git clone` [https://github.com/spring-guides/gs-gradle-yarn.git](https://github.com/spring-guides/gs-gradle-yarn.git)
* 使用以下命令跳转到目录
  ```bash 
  $ cd gs-gradle-yarn/initial 
  ``` 
* 或者戳[理解Gradle使用规则](#gradle_inital)

**完成上述步骤**，你可以根据代码检查结果`gs-gradle-yarn/complete`

如果不了解Gradle或者尚未安装Gradle，请参考[用Gradle构建Java工程](https://spring.io/guides/gs/gradle)

<h2 id="gradle_id">配置工程</h2>

建立一个Java工程，然后用Gradle编译，这个工程尽可能的简单，其目的是学会怎么使用Gradle，

### 创建工程目录结构

在你选择的项目目录中，创建以下子目录结构

```
├── gs-gradle-yarn-appmaster
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── appmaster
├── gs-gradle-yarn-container
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── container
├── gs-gradle-yarn-client
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── client
└── gs-gradle-yarn-dist
```


举个例子，在unix或者Linux系统下使用`mkdir -p`命令分别创建以下文件夹

```bash
mkdir -p gs-gradle-yarn-appmaster/src/main/resources
mkdir -p gs-gradle-yarn-appmaster/src/main/java/hello/appmaster
mkdir -p gs-gradle-yarn-container/src/main/resources
mkdir -p gs-gradle-yarn-container/src/main/java/hello/container
mkdir -p gs-gradle-yarn-client/src/main/resources
mkdir -p gs-gradle-yarn-client/src/main/java/hello/client
mkdir    gs-gradle-yarn-dist
```


第一步创建ContainerApplication类

文件路径

`gs-gradle-yarn-container/src/main/java/hello/container/ContainerApplication.java`

```java
package hello.container;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class ContainerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ContainerApplication.class, args);
	}

}
```

第二步创建AppmasterApplication类

文件路径

`
gs-gradle-yarn-appmaster/src/main/java/hello/appmaster/AppmasterApplication.java
`

```java
package hello.appmaster;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class AppmasterApplication {

	public static void main(String[] args) {
		SpringApplication.run(AppmasterApplication.class, args);
	}

}
```

第三步创建ClientApplication类

文件路径

`
gs-gradle-yarn-client/src/main/java/hello/client/ClientApplication.java
`

```java
package hello.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class ClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ClientApplication.class, args);
	}

}
```

在相应的目录下创建一个`application.yml`配置文件

```yaml
gs-gradle-yarn-container/src/main/resources/application.yml
gs-gradle-yarn-appmaster/src/main/resources/application.yml 
gs-gradle-yarn-client/src/main/resources/application.yml
```

```ymal
spring:
    yarn:
        appName: gs-yarn-gradle
```

<h3 id="gradle_inital">理解Gradle使用规则</h3>

创建`build.gradle`文件，在这个文件里定义好编译所需的内容

使用`Spring Boot Gradle plugin`，因此在`build script`里定义仓库和依赖

```groovy
buildscript {
    repositories {
        maven { url "http://repo.spring.io/libs-release" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.3.RELEASE")
    }
}
```

gradle为整个工程提供`base`插件，技术上讲`base`插件只为整个工程编译过程进行排序，比如`clean`任务。但是`base`插件不是必须的

```groovy
allprojects {
    apply plugin: 'base'
}
```

以下部分内容是，项目版本号为 `0.1.0`，而`apply plugins`定义了以下内容`java`，`eclipse`，`idea`。并且`repositories`选项通过`Maven repositories` 解决所有的第三方jar的依赖问题。接下来为所有的子项目添加`spring-boot plugin`插件和`spring-yarn-boot maven`依赖属性，最后一步，创建一个`copyJars` 任务将生成的jar包从子项目的目录拷贝到发布工程目录`gs-yarn-testing-dist`，这样就更加方便简单地测试和运行程序。

```groovy
subprojects { subproject ->
    apply plugin: 'java'
    apply plugin: 'eclipse'
    apply plugin: 'idea'
    version =  '0.1.0'
    repositories {
        mavenCentral()
        maven { url "http://repo.spring.io/libs-release" }
    }
    dependencies {
        compile("org.springframework.data:spring-yarn-boot:2.4.0.RELEASE")
    }
    task copyJars(type: Copy) {
        from "$buildDir/libs"
        into "$rootDir/gs-gradle-yarn-dist/target/gs-gradle-yarn-dist/"
        include "**/*.jar"
    }
    assemble.doLast {copyJars.execute()}
}
```

创建一个工程的时候，每一个子项目所需要依赖模块都可以在`configure section`进行配置，而且`Gradle plugin`为`Spring Boot`自动创建一个能从新打包`main jar file`的任务。

```groovy
project('gs-gradle-yarn-client') {
    apply plugin: 'spring-boot'
}

project('gs-gradle-yarn-appmaster') {
    apply plugin: 'spring-boot'
}

project('gs-gradle-yarn-container') {
    apply plugin: 'spring-boot'
}
```

和前面的步骤一样，`gs-gradle-yarn-dist`为其子工程提供解决编译依赖的问题。后面的指南里，在创建单元测试的时候需要这个编译步骤。

```groovy
project('gs-gradle-yarn-dist') {
    dependencies {
        compile project(":gs-gradle-yarn-client")
        compile project(":gs-gradle-yarn-appmaster")
        compile project(":gs-gradle-yarn-container")
        testCompile("org.springframework.data:spring-yarn-boot-test:2.4.0.RELEASE")
        testCompile("org.hamcrest:hamcrest-core:1.2.1")
        testCompile("org.hamcrest:hamcrest-library:1.2.1")
    }
    test.dependsOn(':gs-gradle-yarn-client:assemble')
    test.dependsOn(':gs-gradle-yarn-appmaster:assemble')
    test.dependsOn(':gs-gradle-yarn-container:assemble')
    clean.doLast {ant.delete(dir: "target")}
    jar.enabled = false
}
```

针对以上`gs-gradle-yarn-dist`的部分内容，有必要再检查一遍

* 最后将其他的artifacts一起纳入这个工程当中，然后构建一些测试，这就为什么将依赖的模块添加到其他工程当中

* 运行测试之前`Spring Boot`的插件会重新打包。其前提要为其他工程的`assembly`任务添加`test.dependsOn`选项

* `clean task`会包装`target`目录，虽说有点像`maven centric`的风格，但是就这个样例而言，maven和gradle都可以使用相同的工程结构，由于Hadoop测试类的限制，在`target`目录里要创建一些文件比较难。

* 启用`disabled jar`，简化这个工程，如果只存在测试类，禁用这个`jar`选项将为空，当然也没必要使用`disabled jar`


最后再添加一个简单`gradle wrapper`任务，其实也不是必须的，但是有一点好处就是使用它可以不必安装`gradle`二进制包。

```groovy
task wrapper(type: Wrapper) {
    gradleVersion = '1.11'
}
```

为根目录创建`settings.gradle`文件，将子工程名写入其中

```groovy
include 'gs-gradle-yarn-client'
include 'gs-gradle-yarn-appmaster'
include 'gs-gradle-yarn-container'
include 'gs-gradle-yarn-dist'
```


### 解决Hadoop依赖

通过`Maven POM`项目形式Spring可以很好解决Apache Hadoop的依赖问题。默认使用的是Apache Hadoop 2.6.x但是也可以使用其他发行商的Hadoop比如 Pivotal HD, Hortonworks Data Platform or Cloudera CDH。指定好版本，Maven会自动找到正确的Hadoop发行版本的依赖关系。例如以下案例

` Apache Hadoop 2.6.x `

```
dependencies {
    compile("org.springframework.data:spring-yarn-boot:2.1.0.RELEASE")
}
```

` Pivotal HD 2.1 `
```
dependencies {
    compile("org.springframework.data:spring-yarn-boot:2.1.0.RELEASE-phd21")
}
```

` Hortonworks Data Platform 2.2 `
```
dependencies {
    compile("org.springframework.data:spring-yarn-boot:2.1.0.RELEASE-hdp22")
}
```

` Cloudera CDH 5.x `

```
dependencies {
    compile("org.springframework.data:spring-yarn-boot:2.1.0.RELEASE-cdh5")
}
```

` Apache Hadoop 2.4.x `

```
dependencies {
    compile("org.springframework.data:spring-yarn-boot:2.1.0.RELEASE-hadoop24")
}
```

` Apache Hadoop 2.5.x `

```
dependencies {
    compile("org.springframework.data:spring-yarn-boot:2.1.0.RELEASE-hadoop25")
}
```

### 编译打包应用程序

运行编译命令

```bash
gradle clean build
```

将会编译成三个jar文件

```
gs-gradle-yarn-dist/target/gs-gradle-yarn-dist/gs-gradle-yarn-client-0.1.0.jar
gs-gradle-yarn-dist/target/gs-gradle-yarn-dist/gs-gradle-yarn-container-0.1.0.jar
gs-gradle-yarn-dist/target/gs-gradle-yarn-dist/gs-gradle-yarn-appmaster-0.1.0.jar
```

运行这个工程 

```bash
java -jar gs-gradle-yarn-dist/target/dist/gs-gradle-yarn-client-0.1.0.jar
```

当然还添加提交YARN框架上的代码，但是可以浏览输出的日子信息，可以通过其他的`Spring YARN`指南了解怎么创建可以提交到YARN上运行的完整应用程序

## 总结

恭喜了，现在可以通过高效的Maven构建一个简单Spring YARN工程了


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



