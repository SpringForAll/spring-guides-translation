# 单元测试之YARN应用程序

> 原文：[Testing YARN Application](https://spring.io/guides/gs/yarn-testing/)
>
> 译者：[UniKrau](https://github.com/UniKrau)
>
> 校对：

## 写一个YARN应用程序的单元测试

本指南告诉你测试一个简单的YARN应用程序

## 你将学会什么

使用已经开发好的application样例，写单元测试进行验证。

## 你要准备什么

* 大约15分钟
* 文本编辑器或者是IDE
* [JDK 1.6](https://spring.io/guides/gs/yarn-testing/)以上
* [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
* 可以使用以下两种IDE直接将代码导入:
  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    * 使用本地单实例模式，需要Hadoop 2.2.0以上的版本
    
> 测试样例程序没有必要在Hadoop实例上运行.


### 怎样完成这个指南

像大多数[Spring入门](https://spring.io/guides)文章一样,即新手按部就班学习或者如果有基础可以跳过这些基本步骤，不过最后，程序是可以跑的.

**如果是零基础开始**， 可以直接从[配置工程](#set_up)开始学习.

**如果有些基础** 按照以下步骤:

* [下载](https://github.com/spring-guides/gs-yarn-testing/archive/master.zip) 并解压本指南的代码库, 或者使用[Git](https://spring.io/understanding/Git)克隆: `git clone` [ https://github.com/spring-guides/gs-yarn-testing.git](https://github.com/spring-guides/gs-yarn-testing.git)

* 使用以下命令跳转到目录 

```bash
$ cd gs-yarn-testing/initial 
```

* 直接跳转到 [创建一个Junit单元测试类](#junit_class).

**以上结束后** 可以到 `gs-yarn-testing/complete` 目录查看.

如果不了解Gradle， 请参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle).


<h3 id="set_up"> 配置工程 </h3>

本指南重点在用 `JUnit` 测试样例程序。这个gradle工程以及所有的程序文件都是基于这个样例[YARN Basic Sample](https://spring.io/guides/gs/yarn-basic).剩下的事情只需创建一个 `JUnit` 测试类 

首先配置脚本， 可以使用[Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/)工具构建Spring app. 如果不熟悉请参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle) or [Building Java Projects with Maven](https://spring.io/guides/gs/maven
).


另外还有关于构建Spring YARN工程的指南说明。如果不熟悉，请阅读[Building Spring YARN Projects with Gradle](https://spring.io/guides/gs/gradle-yarn) or [Building Spring YARN Projects with Maven](https://spring.io/guides/gs/maven-yarn).

#### 创建工程目录结构

选定工程文件夹以后，建立子文件夹：

```
├── gs-yarn-testing-appmaster
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── appmaster
├── gs-yarn-testing-container
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── container
├── gs-yarn-testing-client
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── client
└── gs-yarn-testing-dist
    └── src
        └── test
            └── java
                └── hello
```

例如在Unix或者Linux系统下
                  
```bash
mkdir -p gs-yarn-testing-appmaster/src/main/resources
mkdir -p gs-yarn-testing-appmaster/src/main/java/hello/appmaster
mkdir -p gs-yarn-testing-container/src/main/resources
mkdir -p gs-yarn-testing-container/src/main/java/hello/container
mkdir -p gs-yarn-testing-client/src/main/resources
mkdir -p gs-yarn-testing-client/src/main/java/hello/client
mkdir -p gs-yarn-testing-dist/src/test/java/hello
```
##### 创建一个gradle编译文件

可以参考这个[initial Gradle build file](https://github.com/spring-guides/gs-yarn-testing/blob/master/initial/build.gradle).


<h4 id="junit_class"> 创建一个JUnit单元测试类 </h4>

``gs-yarn-testing-dist/src/test/java/hello/AppIT.java
``

```java
package hello;

import static org.hamcrest.CoreMatchers.containsString;
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.CoreMatchers.notNullValue;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.greaterThan;
import hello.client.ClientApplication;

import java.io.File;
import java.util.List;
import java.util.concurrent.TimeUnit;

import org.apache.hadoop.yarn.api.records.YarnApplicationState;
import org.junit.Test;
import org.springframework.core.io.Resource;
import org.springframework.yarn.boot.test.junit.AbstractBootYarnClusterTests;
import org.springframework.yarn.test.context.MiniYarnClusterTest;
import org.springframework.yarn.test.junit.ApplicationInfo;
import org.springframework.yarn.test.support.ContainerLogUtils;

@MiniYarnClusterTest
public class AppIT extends AbstractBootYarnClusterTests {

	@Test
	public void testApp() throws Exception {
		String[] args = new String[] {
			"--spring.yarn.client.files[0]=file:target/gs-yarn-testing-dist/gs-yarn-testing-container-0.1.0.jar",
			"--spring.yarn.client.files[1]=file:target/gs-yarn-testing-dist/gs-yarn-testing-appmaster-0.1.0.jar" };
		ApplicationInfo info = submitApplicationAndWait(ClientApplication.class, args, 120, TimeUnit.SECONDS);
		assertThat(info.getYarnApplicationState(), is(YarnApplicationState.FINISHED));

		List<Resource> resources = ContainerLogUtils.queryContainerLogs(
				getYarnCluster(), info.getApplicationId());
		assertThat(resources, notNullValue());
		assertThat(resources.size(), is(4));

		for (Resource res : resources) {
			File file = res.getFile();
			String content = ContainerLogUtils.getFileContent(file);
			if (file.getName().endsWith("stdout")) {
				assertThat(file.length(), greaterThan(0l));
				if (file.getName().equals("Container.stdout")) {
					assertThat(content, containsString("Hello from HelloPojo"));
				}
			} else if (file.getName().endsWith("stderr")) {
				assertThat("stderr with content: " + content, file.length(), is(0l));
			}
		}
	}

}
```


接下来一步一步分解 `JUnit` 类。不过之前提到过不需要Hadoop实例，是因为Spring YARN的测试框架很方便地开启一个独立地mini cluster测试环境


* 命名一个 `AppIT` 类，运行编译tests的时候，它能兼容gradle和maven。编译的时候。如果Tests添加了artifacts标识，那么只能用jar的形式运行并且通过 `Spring Boot’s` repackage 插件完成这件事情。

* 由于test运行的是一个 `gs-yarn-testing-dist` 工程，所以要用 `spring.yarn.client.files` 覆盖 `application.yml` 的配置

* `@MiniYarnClusterTest` 是一个composed注解，它能使Spring开启一个有`HDFS` 和 `YARN` 组件的Hadoop mini Cluster，Hadoop的配置文件能从mini cluster 自动的注入测试上下文

* `AbstractBootYarnClusterTests` 这个类包含类许多test需要的基本功能。

发布程序到miniCluster上

* `submitApplicationAndWait()` 方法调用了 `ClientApplication` 并且它具有发布程序的功能。默认等待60秒然后返回当前的状态

* 程序的各个运行状态无误

在miniCluster上，可以通过`ContainerLogUtils` 找到 container的日志文件

* assert 日志文件的行数

* 日志文件记录了相关内容

* `stderr` 是个空文件

#### 运行测试

使用gradle命令：

```bash
./gradlew clean build
```

使用maven命令:

```bash
mvn clean package
```

可以通过日志目录的YARN container的输出日志，检查测试是否执行成功：

```bash
$ find gs-yarn-testing-dist/target/|grep std
target/yarn--1502101888/yarn--1502101888-logDir-nm-0_0/application_1392395851515_0001/container_1392395851515_0001_01_000002/Container.stdout
target/yarn--1502101888/yarn--1502101888-logDir-nm-0_0/application_1392395851515_0001/container_1392395851515_0001_01_000002/Container.stderr
target/yarn--1502101888/yarn--1502101888-logDir-nm-0_0/application_1392395851515_0001/container_1392395851515_0001_01_000001/Appmaster.stdout
target/yarn--1502101888/yarn--1502101888-logDir-nm-0_0/application_1392395851515_0001/container_1392395851515_0001_01_000001/Appmaster.stderr
```

```bash
$ grep Hello gs-yarn-testing-dist/target/yarn--1502101888/yarn--1502101888-logDir-nm-0_0/application_1392395851515_0001/container_1392395851515_0001_01_000002/Container.stdout
[2014-02-14 16:37:54.278] boot - 18453  INFO [main] --- HelloPojo: Hello from HelloPojo
[2014-02-14 16:37:54.278] boot - 18453  INFO [main] --- HelloPojo: About to list from hdfs root content
[2014-02-14 16:37:55.157] boot - 18453  INFO [main] --- HelloPojo: FileStatus{path=hdfs://localhost:33626/; isDirectory=true; modification_time=1392395854968; access_time=0; owner=jvalkealahti; group=supergroup; permission=rwxr-xr-x; isSymlink=false}
[2014-02-14 16:37:55.157] boot - 18453  INFO [main] --- HelloPojo: FileStatus{path=hdfs://localhost:33626/app; isDirectory=true; modification_time=1392395854968; access_time=0; owner=jvalkealahti; group=supergroup; permission=rwxr-xr-x; isSymlink=false}
```


### 总结

恭喜！现在已经能写Spring YARN工程的 `JUnit` 单元测试。

## 其他类似内容
   
   以下有用的相关链接:
   
* [Simple YARN Application](https://spring.io/guides/gs/yarn-basic/)
* [Simple Single Project YARN Application](https://spring.io/guides/gs/yarn-basic-single/)
* [Batch YARN Application](https://spring.io/guides/gs/yarn-batch-processing/)
* [Restartable Batch YARN Application](https://spring.io/guides/gs/yarn-batch-restart/)
* [Building Spring YARN Projects with Maven](https://spring.io/guides/gs/maven-yarn/)
* [Building Spring YARN Projects with Gradle](https://spring.io/guides/gs/gradle-yarn/)




> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



