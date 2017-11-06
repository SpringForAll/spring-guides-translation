# YARN 的批处理应用程序

> 原文：[Batch YARN Application](https://spring.io/guides/gs/yarn-batch-processing/)
>
> 译者：[UniKrau](https://github.com/UniKrau)
>
> 校对：[dyc87112](https://github.com/dyc87112)


本指南主要探讨Spring批处理job在Hadoop YARN上执行的过程

## 你将学会构建什么

通过Spring Hadoop, Spring Batch and Spring Boot 构建一个简单Hadoop YARN 应用程序

## 你需要准备什么

* 大概15分钟
* 文本编辑器或IDE
* 需要 [JDK 1.6](http://www.oracle.com/technetwork/java/javase/downloads/index.html)以上的版本
* 编译工具版本[Gradle 2.3+](http://www.gradle.org/downloads) [ Maven 3.0+](https://maven.apache.org/download.cgi)
* 也可以将代码直接导入到IDE
  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
     *  使用本地单实例模式，需要Hadoop 2.2.0以上的版本

## 怎样完成指南

像大多数[Spring 入门文章](https://spring.io/guides)一样，即新手按部就班完成或者有基础的可以跳过这些基本步骤，不过最后，程序是可以运行的.

**如果从基础开始**，参考[配置工程](#set_up)

**如果已经熟悉跳过一些基本步骤**你可以这样

* [下载](https://github.com/spring-guides/gs-yarn-batch-processing/archive/master.zip)源码然后使unzip 命令解压，或者使用[Git](https://spring.io/understanding/Git)拷贝一份源代码，克隆命令：`git clone` [https://github.com/spring-guides/gs-yarn-batch-processing.git](https://github.com/spring-guides/gs-yarn-batch-processing.git)

* 使用以下命令跳转到目录 
```bash 
$ cd gs-yarn-batch-processing/initial 
```
 
* 跳转到创建[远程批处理步骤](#remote_batch)

**以上步骤结束之后**， 可以在 `gs-yarn-batch-processing/complete` 目录下检查代码.

<h2 id="set_up"> 配置工程</h2>

首先要配置编译脚本。Spring构建apps的时候可以使用的编译工具有很多，但是在这里需要用[Gradle](http://gradle.org/)编译代码。如果不熟悉，请参考[Gradle编译java工程](https://spring.io/guides/gs/gradle).

### 创建工程目录结构

在工程文件夹下，创建子文件夹

```groovy
├── gs-yarn-batch-processing-appmaster
│ └── src
│ └── main
│ ├── resources
│ └── java
│ └── hello
│ └── appmaster
├── gs-yarn-batch-processing-container
│ └── src
│ └── main
│ ├── resources
│ └── java
│ └── hello
│ └── container
├── gs-yarn-batch-processing-client
│ └── src
│ └── main
│ ├── resources
│ └── java
│ └── hello
│ └── client
└── gs-yarn-batch-processing-dist
└── src
└── test
└── java
└── hello
```

举个例子，使用unix或者Linux系统的同学使用 `mkdir -p` 命令分别创建以下文件夹

```bash
mkdir -p gs-yarn-batch-processing-appmaster/src/main/resources
mkdir -p gs-yarn-batch-processing-appmaster/src/main/java/hello/appmaster
mkdir -p gs-yarn-batch-processing-container/src/main/resources
mkdir -p gs-yarn-batch-processing-container/src/main/java/hello/container
mkdir -p gs-yarn-batch-processing-client/src/main/resources
mkdir -p gs-yarn-batch-processing-client/src/main/java/hello/client
mkdir -p gs-yarn-batch-processing-dist/src/test/java/hello
```

### 创建Gradle编译文件

[初始化Gradle编译文件 ](https://github.com/spring-guides/gs-yarn-batch-processing/blob/master/initial/build.gradle)，也可以用[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)工具直接导入源码

`build.gradle`

````groovy
buildscript {
    repositories {
        maven { url "http://repo.spring.io/libs-release" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.3.3.RELEASE")
    }
}

allprojects {
    apply plugin: 'base'
}

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
        into "$rootDir/gs-yarn-batch-processing-dist/target/gs-yarn-batch-processing-dist/"
        include "**/*.jar"
    }
    configurations {
        compile.exclude group: "org.slf4j", module: "slf4j-log4j12"
        runtime.exclude group: "org.slf4j", module: "slf4j-log4j12"
    }
    assemble.doLast {copyJars.execute()}
}

project('gs-yarn-batch-processing-client') {
    apply plugin: 'spring-boot'
}

project('gs-yarn-batch-processing-appmaster') {
    apply plugin: 'spring-boot'
    dependencies {
        compile("org.springframework.data:spring-yarn-batch:2.4.0.RELEASE")
        runtime("org.springframework.boot:spring-boot-starter-batch:1.3.3.RELEASE")
    }
}

project('gs-yarn-batch-processing-container') {
    apply plugin: 'spring-boot'
    dependencies {
        compile("org.springframework.data:spring-yarn-batch:2.4.0.RELEASE")
        runtime("org.springframework.boot:spring-boot-starter-batch:1.3.3.RELEASE")
    }
}

project('gs-yarn-batch-processing-dist') {
    dependencies {
        compile project(":gs-yarn-batch-processing-client")
        compile project(":gs-yarn-batch-processing-appmaster")
        compile project(":gs-yarn-batch-processing-container")
        testCompile("org.springframework.data:spring-yarn-boot-test:2.4.0.RELEASE")
        testCompile("org.hamcrest:hamcrest-core:1.2.1")
        testCompile("org.hamcrest:hamcrest-library:1.2.1")
    }
    test.dependsOn(':gs-yarn-batch-processing-client:assemble')
    test.dependsOn(':gs-yarn-batch-processing-appmaster:assemble')
    test.dependsOn(':gs-yarn-batch-processing-container:assemble')
    clean.doLast {ant.delete(dir: "target")}
}

task wrapper(type: Wrapper) {
    gradleVersion = '1.11'
}
````

依照gradle编译文件，简单的构建三个不同的jar包，Spring Boot’s Gradle插件会把这三个包编译成可执行jar包

`gesettings.gradle`文件定义其子项目

`settings.gradle` 内容

````groovy
include 'gs-yarn-batch-processing-client','gs-yarn-batch-processing-appmaster','gs-yarn-batch-processing-container','gs-yarn-batch-processing-dist'
````

### Spring 批处理介绍


可以用单线程或进程的job解决批处理问题，因此处理许多复杂的问题的时候优可能会先考虑是否能用批处理模式解决。Spring Batch提供了很多并行处理方法解决这些问题。有两种高级层次多并行处理模型：单进程，多线程；多进程

在Hadoop YARN 计算框架的集群上 Spring Hadoop 支持Spring Batch 类型的job。为了更好的并行处理， Spring Batch 做了分区，而且在YARN上执行的时候采用remote steps的方式。

要在YARN上运行Spring Batch Job 的关键地方在于Application Master能否判断这个job是不是简单类型的或者不要进行分区。job没有分区的时候，整个job都运行在 Application Master内并且不会启动额外的容器。好神奇的样子，竟然不要容器就能在YARN跑job，但是别忘了Application Master其实也是Hadoop 集群分配的资源，当然也是一个能运行在YARN上容器。

 要在Hadoop 集群上运行Spring Batch jobs，还是有些限制的
 
 * Job Context - Job 上下文，Application Master是运行job的主要实体
 * Job Repository - Job 仓库， Application Master需要访问驻留在内存或者关系型数据库的仓库。显然Spring Batch是支持这两种。
 * 远程工作集- 由于 Spring Batch有分区能力的基因，所以远程工作集需要访问工作仓库
 
 快速浏览Spring Batch 分区是怎样的机制，其理念是：一个分区好的job需要三样东西，第一 远程工作集，第二 分区处理器，第三 分区者。站在用户的角度来看，任何一个 remote step 都有点过于简化了。Spring Batch 本身不包含任何专门网格计算或者特殊远程过程调用实现。然而Spring Batch确实提供PartitionHandler的实现，PartitionHandler会根据Spring的TaskExecutor的策略，然后使用各自独立的线程运行工作集。Spring Hadoop针对Hadoop集群提供了远程工作集的实现。
 
 
> 了解更多关于Spring Batch Partitioning的信息，参照Spring Batch文档
 
 
 <h2 id="remote_batch"> 远程批处理步骤 </h2>
 
 这里创建一个PrintTasklet类
 
```groovy
 gs-yarn-batch-processing-container/src/main/java/hello/container/PrintTasklet.java
``` 

```java
package hello.container;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

public class PrintTasklet implements Tasklet {

	private static final Log log = LogFactory.getLog(PrintTasklet.class);

	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext)
			throws Exception {
		log.info("PrintTasklet says Hello");
		return RepeatStatus.FINISHED;
	}

}
```

一个job的step当中，Tasklet接口是Spring Batch最简单通俗易懂的概念之一。这个tasklet目的只是简单示范一下真正Partitioned Step是如何执行的。当然就不用介绍如何处理复杂job处理啦

PrintTasklet类负责输出简单的日志

然后创建ContainerApplication类

```groovy
gs-yarn-batch-processing-container/src/main/java/hello/container/ContainerApplication.java
```

````java
package hello.container;

import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.yarn.batch.config.EnableYarnRemoteBatchProcessing;

@Configuration
@EnableAutoConfiguration(exclude = { BatchAutoConfiguration.class })
@EnableYarnRemoteBatchProcessing
public class ContainerApplication {

	@Autowired
	private StepBuilderFactory stepBuilder;

	@Bean
	protected Tasklet tasklet() {
		return new PrintTasklet();
	}

	@Bean
	protected Step remoteStep() throws Exception {
		return stepBuilder
			.get("remoteStep")
			.tasklet(tasklet())
			.build();
	}

	public static void main(String[] args) {
		SpringApplication.run(ContainerApplication.class, args);
	}

}
````

 `@EnableYarnRemoteBatchProcessing`注解 Spring为YARN 容器提供Batch处理的功能 `@EnableBatchProcessing`自动使得JavaConfig的所有建造者都可用。
 
 * @AutoWired  step建造者把steps转换成beans
 * 把PrintTasklet转换成Bean
 * 一个step转换成bean以后就知道怎么执行一个tasklet了
 
 
 接下来为容器写一个application.yml文件
 
`gs-yarn-batch-processing-container/src/main/resources/application.yml`
 
 格式如下
 
 ````yaml
 spring:
     batch:
         job:
             enabled: false
     hadoop:
         fsUri: hdfs://localhost:8020
     yarn:
         batch:
             enabled: true

 ````

* 在Spring Boot core里禁用batch功能，因此就可以使用YARN特性
* 配置HDFS文件，在真实的集群里是可以自定义的
* 通过 `spring.yarn.batch.enabled property` 可以在YARN上使用批处理

### 创建一个批处理job

创建AppmasterApplication类

```groovy
gs-yarn-batch-processing-appmaster/src/main/java/hello/appmaster/AppmasterApplication.java
```

````java
package hello.appmaster;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParametersIncrementer;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.launch.support.RunIdIncrementer;
import org.springframework.batch.core.partition.PartitionHandler;
import org.springframework.batch.core.partition.support.Partitioner;
import org.springframework.batch.core.partition.support.SimplePartitioner;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.yarn.batch.config.EnableYarnBatchProcessing;
import org.springframework.yarn.batch.partition.StaticPartitionHandler;

@Configuration
@EnableAutoConfiguration
@EnableYarnBatchProcessing
public class AppmasterApplication {

	@Autowired
	private JobBuilderFactory jobFactory;

	@Autowired
	private StepBuilderFactory stepFactory;

	@Bean
	public Job job() throws Exception {
		return jobFactory.get("job")
			.incrementer(jobParametersIncrementer())
			.start(master())
			.build();
	}

	@Bean
	public JobParametersIncrementer jobParametersIncrementer() {
		return new RunIdIncrementer();
	}

	@Bean
	protected Step master() throws Exception {
		return stepFactory
			.get("master")
			.partitioner("remoteStep", partitioner())
			.partitionHandler(partitionHandler())
			.build();
	}

	@Bean
	protected Partitioner partitioner() {
		return new SimplePartitioner();
	}

	@Bean
	protected PartitionHandler partitionHandler() {
		StaticPartitionHandler handler = new StaticPartitionHandler();
		handler.setStepName("remoteStep");
		handler.setGridSize(2);
		return handler;
	}

	public static void main(String[] args) {
		SpringApplication.run(AppmasterApplication.class, args);
	}

}
````

* `@EnableYarnBatchProcessing` 为appmaster启动Batch处理功能
* 为steps和jobs提供建造者


为appmaster创建`application.yml`文件

`gs-yarn-batch-processing-appmaster/src/main/resources/application.yml`

````yaml
spring:
    batch:
        job:
            enabled: false
    hadoop:
        fsUri: hdfs://localhost:8020
        resourceManagerHost: localhost
    yarn:
        appName: gs-yarn-batch-processing
        applicationDir: /app/gs-yarn-batch-processing/
        batch:
            enabled: true
            name: job
            jobs:
              - name: job
                enabled: true
                next: true
        appmaster:
            keepContextAlive: false
            launchcontext:
                archiveFile: gs-yarn-batch-processing-container-0.1.0.jar
````
相关的解析说明
* 在Spring Boot core里禁用batch功能，因此就可以使用YARN特性
* 配置HDFS文件，在真实的集群里是可以自定义的
* 通过spring.yarn.batch.enabled property可以在YARN上使用批处理
* enabled 是否自动运行
* next 是否允许下一步操作

###  创建Yarn客户端

创建 ClientApplication 类

`gs-yarn-batch-processing-client/src/main/java/hello/client/ClientApplication.java`

````java
package hello.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.yarn.client.YarnClient;

@EnableAutoConfiguration
public class ClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ClientApplication.class, args)
			.getBean(YarnClient.class)
			.submitApplication();
	}

}
````


ClientApplication与其他指南的例子程序的类差不多，不过这里的目的只是提交一个YARN application

创建一个client `application.yml` 文件

`gs-yarn-batch-processing-client/src/main/resources/application.yml`

内容如下

````yaml
spring:
    hadoop:
        fsUri: hdfs://localhost:8020
        resourceManagerHost: localhost
    yarn:
        appName: gs-yarn-batch-processing
        applicationDir: /app/gs-yarn-batch-processing/
        client:
            files:
              - "file:target/gs-yarn-batch-processing-dist/gs-yarn-batch-processing-container-0.1.0.jar"
              - "file:target/gs-yarn-batch-processing-dist/gs-yarn-batch-processing-appmaster-0.1.0.jar"
            launchcontext:
                archiveFile: gs-yarn-batch-processing-appmaster-0.1.0.jar
````

* 为application需要提交的，定义好了所有的文件

### 编译程序

使用Gradle命令：clean清空工作目录，build编译

```bash
./gradlew clean build
```

跳过所有的单元测试

```bash
./gradlew clean build -x test
```

使用maven命令：clean清空工作目录，package打包

```bash
mvn clean package
```

Gradle 编译成功之后 在target里有以下三个jar包

````groovy
gs-yarn-batch-processing-dist/target/gs-yarn-batch-processing-dist/gs-yarn-batch-processing-client-0.1.0.jar
gs-yarn-batch-processing-dist/target/gs-yarn-batch-processing-dist/gs-yarn-batch-processing-appmaster-0.1.0.jar
gs-yarn-batch-processing-dist/target/gs-yarn-batch-processing-dist/gs-yarn-batch-processing-container-0.1.0.jar
````

运行Application

目前为止已经成功的编译打包了application了，该上Hadoop YARN上试试了

运行这个client jar包

```bash
$ cd gs-yarn-batch-processing-dist
$ java -jar target/gs-yarn-batch-processing-dist/gs-yarn-batch-processing-client-0.1.0.jar
```


如果这个程序没有出错，在YARN可以看到两段执行过程

###  创建一个单元测试类

以下这个执行应用程序的类可以作为单元测试类，并不需要在Hadoop集群上跑

`gs-yarn-batch-processing-dist/src/test/java/hello/AppIT.java`

````java
package hello;

import static org.hamcrest.CoreMatchers.containsString;
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.CoreMatchers.notNullValue;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.greaterThan;
import hello.client.ClientApplication;

import java.io.File;
import java.util.List;
import java.util.Scanner;

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
	public void testAppSubmission() throws Exception {

		ApplicationInfo info = submitApplicationAndWait(ClientApplication.class, new String[0]);
		assertThat(info.getYarnApplicationState(), is(YarnApplicationState.FINISHED));

		List<Resource> resources = ContainerLogUtils.queryContainerLogs(getYarnCluster(), info.getApplicationId());
		assertThat(resources, notNullValue());
		assertThat(resources.size(), is(6));

		for (Resource res : resources) {
			File file = res.getFile();
			if (file.getName().endsWith("stdout")) {
				// there has to be some content in stdout file
				assertThat(file.length(), greaterThan(0l));
				if (file.getName().equals("Container.stdout")) {
					Scanner scanner = new Scanner(file);
					String content = scanner.useDelimiter("\\A").next();
					scanner.close();
					// this is what container will log in stdout
					assertThat(content, containsString("PrintTasklet says Hello"));
				}
			} else if (file.getName().endsWith("stderr")) {
				String content = "";
				if (file.length() > 0) {
					Scanner scanner = new Scanner(file);
					content = scanner.useDelimiter("\\A").next();
					scanner.close();
				}
				// can't have anything in stderr files
				assertThat("stderr file is not empty: " + content, file.length(), is(0l));
			}
		}
	}
}
````

## 总结一下

恭喜了，可以开发Spring Batch job的Spring YARN程序了


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



