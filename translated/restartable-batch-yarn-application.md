#  可重启的YARN批处理程序

> 原文：[Restartable Batch YARN Application](https://spring.io/guides/gs/yarn-batch-restart/)
>
> 译者：[UniKrau](https://github.com/UniKrau)
>
> 校对：[dyc87112](https://github.com/dyc87112)

## 导言

本指南不仅让你知道一个Spring Batch job的执行过程以及在Hadoop YARN上的partitioned steps，而且模拟了在分区步骤过程出错的过程。凸显重启job的完整性。

## 你将学会什么

使用Spring Hadoop and Spring Boot构建一个简单Hadoop YARN程序。它含有一个job，当这个job在YARN partitioned steps执行完毕的时候产生两master steps。同样也模拟了每个step出错的情况，所以能知道job的重启点是在step运行失败的地方开始执行

## 你要准备什么

* 大约15分钟
* 文本编辑器或者是IDE
* [JDK 1.6](http://www.oracle.com/technetwork/java/javase/downloads/index.html)以上
* [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
* 可以使用以下两种IDE直接将代码导入:
  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
    * 使用本地单实例模式，需要Hadoop 2.2.0以上的版本
    
> 测试样例程序没有必要在Hadoop实例上运行.

## 怎样完成这个指南

正如和其他的[Spring Getting Started guides](https://spring.io/guides)指南一样，可以按部就班的从零开始完成每个步骤，或者跳过熟悉的基本步骤。无论何种方式。你的代码都能完成本指南的任务。

**如果是零基础开始** , 可以直接从[配置工程](#set_up)开始学习.

**如果有些基础** 按照以下步骤:

* [下载](https://github.com/spring-guides/gs-yarn-batch-restart/archive/master.zip) 并解压本指南的代码库, 或者使用[Git](https://spring.io/understanding/Git)克隆: git clone [https://github.com/spring-guides/gs-yarn-batch-restart.git](https://github.com/spring-guides/gs-yarn-batch-restart.git)

* 使用以下命令跳转到目录 
```bash
 $ cd   gs-yarn-batch-restart/initial
```

* 直接跳转到[Remote Batch Step](#rbs).

**以上步骤完成后** 可以到 `gs-yarn-batch-restart/complete` 目录查看.


<h2 id="set_up">配置工程 </h2>


首先配置脚本， 可以使用[Gradle](http://gradle.org/)工具构建Spring app. 如果不熟悉请参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle).

### 创建工程目录结构

选定工程文件夹以后，建立子文件夹：

```
├── gs-yarn-batch-restart-appmaster
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── appmaster
├── gs-yarn-batch-restart-container
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── container
├── gs-yarn-batch-restart-client
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── client
└── gs-yarn-batch-restart-dist
    └── src
        └── test
            └── java
                └── hello
```


```bash
mkdir -p gs-yarn-batch-restart-appmaster/src/main/resources
mkdir -p gs-yarn-batch-restart-appmaster/src/main/java/hello/appmaster
mkdir -p gs-yarn-batch-restart-container/src/main/resources
mkdir -p gs-yarn-batch-restart-container/src/main/java/hello/container
mkdir -p gs-yarn-batch-restart-client/src/main/resources
mkdir -p gs-yarn-batch-restart-client/src/main/java/hello/client
mkdir -p gs-yarn-batch-restart-dist/src/test/java/hello
```


### 创建一个gradle编译文件

完整文件参考[the initial Gradle build file](https://github.com/spring-guides/gs-yarn-batch-restart/blob/master/initial/build.gradle). 可以使用[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)工具直接导入源码.

`build.gradle`

```groovy
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
        into "$rootDir/gs-yarn-batch-restart-dist/target/gs-yarn-batch-restart-dist/"
        include "**/*.jar"
    }
    configurations {
        compile.exclude group: "org.slf4j", module: "slf4j-log4j12"
        runtime.exclude group: "org.slf4j", module: "slf4j-log4j12"
    }
    assemble.doLast {copyJars.execute()}
}

project('gs-yarn-batch-restart-client') {
    apply plugin: 'spring-boot'
}

project('gs-yarn-batch-restart-appmaster') {
    apply plugin: 'spring-boot'
    dependencies {
        compile("org.springframework.data:spring-yarn-batch:2.4.0.RELEASE")
        runtime("org.springframework.boot:spring-boot-starter-batch:1.3.3.RELEASE")
        runtime("org.hsqldb:hsqldb:2.3.1")
        runtime("commons-dbcp:commons-dbcp:1.2.2")
    }
}

project('gs-yarn-batch-restart-container') {
    apply plugin: 'spring-boot'
    dependencies {
        compile("org.springframework.data:spring-yarn-batch:2.4.0.RELEASE")
        runtime("org.springframework.boot:spring-boot-starter-batch:1.3.3.RELEASE")
    }
}

project('gs-yarn-batch-restart-dist') {
    dependencies {
        compile project(":gs-yarn-batch-restart-client")
        compile project(":gs-yarn-batch-restart-appmaster")
        compile project(":gs-yarn-batch-restart-container")
        testCompile("org.hsqldb:hsqldb:2.3.1")
        testCompile("org.springframework.data:spring-yarn-boot-test:2.4.0.RELEASE")
        testCompile("org.hamcrest:hamcrest-core:1.2.1")
        testCompile("org.hamcrest:hamcrest-library:1.2.1")
    }
    test.dependsOn(':gs-yarn-batch-restart-client:assemble')
    test.dependsOn(':gs-yarn-batch-restart-appmaster:assemble')
    test.dependsOn(':gs-yarn-batch-restart-container:assemble')
    clean.doLast {ant.delete(dir: "target")}
}

task wrapper(type: Wrapper) {
    gradleVersion = '1.11'
}
```

以上的gradle文件只是简单的创建类三个jar包— client, appmaster and container, 每一个类都有相应的功能角色. Spring Boot's gradle 插件会从新把这些jars打包成可执行的jar。

还需要 `settings.gradle` 文件定义其子工程.

`settings.gradle`

```groovy
include 'gs-yarn-batch-restart-client','gs-yarn-batch-restart-appmaster','gs-yarn-batch-restart-container','gs-yarn-batch-restart-dist'

```

<h2 id="rbs">Create a Remote Batch Step </h2> 
     
第一、创建一个`HdfsTasklet` 类.

`gs-yarn-batch-restart-container/src/main/java/hello/container/HdfsTasklet.java `

```java
package hello.container;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.apache.hadoop.conf.Configuration;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.hadoop.fs.FsShell;

public class HdfsTasklet implements Tasklet {

	private static final Log log = LogFactory.getLog(HdfsTasklet.class);

	@Autowired
	private Configuration configuration;

	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
		String fileName = chunkContext.getStepContext().getStepName().replaceAll("\\W", "");
		FsShell shell = new FsShell(configuration);
		boolean exists = shell.test("/tmp/" + fileName);
		shell.close();
		if (exists) {
			log.info("File /tmp/" + fileName + " exist");
			log.info("Hello from HdfsTasklet ok");
			return RepeatStatus.FINISHED;
		} else {
			log.info("Hello from HdfsTasklet fail");
			throw new RuntimeException("File /tmp/" + fileName + " does't exist");
		}
	}

}
```

* 要使用 `FsShell` 首先🉐️要使用注解 ` @AutoWired` Hadoop’s `Configuration` 。

* 文件不存在HDFS的时候 `RuntimeException` 类抛异常。`Tasklet` 类实现了这个功能。如果文件不存在则会从`Tasklet`返回 `FINISHED`值。

* 要了解文件名如何传递的，需要访问类似 `remoteStep1:partition0` 这样的 `stepname`. 然后过滤掉无关的字符。

第二、 创建 `ContainerApplication` 类.

`gs-yarn-batch-restart-container/src/main/java/hello/container/ContainerApplication.java`


```java
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
	private StepBuilderFactory steps;

	@Bean
	public Step remoteStep1() throws Exception {
		return steps
			.get("remoteStep1")
			.tasklet(tasklet())
			.build();
	}

	@Bean
	public Step remoteStep2() throws Exception {
		return steps
			.get("remoteStep2")
			.tasklet(tasklet())
			.build();
	}

	@Bean
	public Tasklet tasklet() {
		return new HdfsTasklet();
	}

	public static void main(String[] args) {
		SpringApplication.run(ContainerApplication.class, args);
	}

}
```

* 简单创建了两个 `step` 分别命名为 `remoteStep1` 和 `remoteStep2`。还附加了 `HdfsTasklet` 到这个两个 `step` 上

第三、 为container app 创建 `application.yml` 文件.

`gs-yarn-batch-restart-container/src/main/resources/application.yml`

```yaml
spring:
    batch:
        job:
            enabled: false
    hadoop:
        fsUri: hdfs://localhost:8020
        resourceManagerHost: localhost
    yarn:
        batch:
            enabled: true
```

* 在Spring Boot core可以通过禁用job的功能来使用YARN的相关特性

* 为HDFS添加配置文件，当然要访问集群可以自定义配置文件

* 在YARN上用过启用 ` spring.yarn.batch.enable` 属性来启用批处理功能.

### Create a Batch Job

创建一个`AppmasterApplication`类.

`gs-yarn-batch-restart-appmaster/src/main/java/hello/appmaster/AppmasterApplication.java`

```java
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
			.start(master1())
			.next(master2())
			.build();
	}

	@Bean
	public JobParametersIncrementer jobParametersIncrementer() {
		return new RunIdIncrementer();
	}

	@Bean
	protected Step master1() throws Exception {
		return stepFactory.get("master1")
			.partitioner("remoteStep1", partitioner())
			.partitionHandler(partitionHandler1())
			.build();
	}

	@Bean
	protected Step master2() throws Exception {
		return stepFactory.get("master2")
			.partitioner("remoteStep2", partitioner())
			.partitionHandler(partitionHandler2())
			.build();
	}

	@Bean
	protected Partitioner partitioner() {
		return new SimplePartitioner();
	}

	@Bean
	protected PartitionHandler partitionHandler1() {
		StaticPartitionHandler handler = new StaticPartitionHandler();
		handler.setStepName("remoteStep1");
		handler.setGridSize(2);
		return handler;
	}

	@Bean
	protected PartitionHandler partitionHandler2() {
		StaticPartitionHandler handler = new StaticPartitionHandler();
		handler.setStepName("remoteStep2");
		handler.setGridSize(2);
		return handler;
	}

	public static void main(String[] args) {
		SpringApplication.run(AppmasterApplication.class, args);
	}

}
```

* 简单起见，分别创建`master1` 和 `master2`两个master steps。然后配置这两个steps，以便在YARN上进行partitioned，并将grid的大小设置为2
 
为appmaster创建一个`application.yml`文件.


  
`gs-yarn-batch-restart-appmaster/src/main/resources/application.yml`


```yaml
spring:
    batch:
        job:
            enabled: false
    datasource:
        url: jdbc:hsqldb:hsql://localhost:9001/testdb
        driverClassName: org.hsqldb.jdbcDriver
    hadoop:
        fsUri: hdfs://localhost:8020
        resourceManagerHost: localhost
    yarn:
        appName: gs-yarn-batch-restart
        applicationDir: /app/gs-yarn-batch-restart/
        batch:
            enabled: true
            name: job
            jobs:
              - name: job
                enabled: true
                next: true
                failNext: false
                restart: true
                failRestart: false
        appmaster:
            keepContextAlive: false
            launchcontext:
                archiveFile: gs-yarn-batch-restart-container-0.1.0.jar
```   

* 同样地，在Spring Boot core可以通过禁用job的功能来使用YARN的相关特性
* 为HDFS添加配置文件，当然要访问集群可以自定义配置文件
* 在YARN上用过启用 ` spring.yarn.batch.enable` 属性来启用批处理功能
* 给job取个名称以便能够自动运行
* 启用job的命名功能，并且允许job下一次运行的时候，即使没有自定自增的参数也不会运行失败
* 启用job自动重启功能，使得如果job重启失败后job不会出现失败

### 创建一个Yarn Client

现在创建一个`ClientApplication`类.

`gs-yarn-batch-restart-client/src/main/java/hello/client/ClientApplication.java`

```java
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
```

* [@EnableAutoConfiguration](https://docs.spring.io/spring-boot/docs/1.2.1) 它能使Spring Boot添加各类属性配置     

* auto-configuration 能使得Spring YARN 组件像一般的Spring Boot app 一样运行
这个 ` main()` 方法使用了Spring Boot的`SpringApplication.run()` 方法启动application. From there, we simply request a bean of type `YarnClient` and execute its `submitApplication()` method. What happens next depends on application configuration, which we go through later in this guide. Did you notice that there wasn’t a single line of XML?


为client创建一个`application.yml`文件

`gs-yarn-batch-restart-client/src/main/resources/application.yml`

```yaml
spring:
    hadoop:
        fsUri: hdfs://localhost:8020
        resourceManagerHost: localhost
    yarn:
        appName: gs-yarn-batch-restart
        applicationDir: /app/gs-yarn-batch-restart/
        client:
            files:
              - "file:target/gs-yarn-batch-restart-dist/gs-yarn-batch-restart-container-0.1.0.jar"
              - "file:target/gs-yarn-batch-restart-dist/gs-yarn-batch-restart-appmaster-0.1.0.jar"
            launchcontext:
                archiveFile: gs-yarn-batch-restart-appmaster-0.1.0.jar
```

* 在这个文件里配置了应用程序提交时需要的元数据信息

### 编译程序


简单的执行gradle clean和build命令

```bash
./gradlew clean build
```

跳过测试命令:

```bash
./gradlew clean build -x test
```

执行maven的clean和package命令

```bash
mvn clean package
```

跳过单元测试使用以下命令:

```bash
mvn clean package -DskipTests=true
```

gradle编译成功后会有以下三个jar包

```groovy
gs-yarn-batch-restart-dist/target/gs-yarn-batch-restart-dist/gs-yarn-batch-restart-client-0.1.0.jar
gs-yarn-batch-restart-dist/target/gs-yarn-batch-restart-dist/gs-yarn-batch-restart-appmaster-0.1.0.jar
gs-yarn-batch-restart-dist/target/gs-yarn-batch-restart-dist/gs-yarn-batch-restart-container-0.1.0.jar
```


     
### 运行程序流程

现在准备将已经编译和打包程序在Hadoop YARN上执行。另外需要将Spring Batch job的状态保存在数据库里。`HSQL`实例可以很方便的使用内存模型运行程序。在终端运行以下命令：

```bash
$ cd db/
$ unzip hsqldb-2.3.1.zip
$ cd hsqldb-2.3.1/hsqldb/data/
$ java -cp ../lib/hsqldb.jar org.hsqldb.server.Server --database.0 mem:testdb --dbname.0 testdb --silent false --trace true
```


> 备注: 可以下载HSQL源码压缩文件[http://sourceforge.net/projects/hsqldb/files/hsqldb/hsqldb_2_3/](http://sourceforge.net/projects/hsqldb/files/hsqldb/hsqldb_2_3/)，如果想从源码编译HSQL jar包。


配置程序以便运行，首先要在HDFS上创建两个空文件`/tmp/remoteStep1partition0`和`/tmp/remoteStep1partition1` 

```bash
$ hdfs dfs -touchz /tmp/remoteStep1partition0
$ hdfs dfs -touchz /tmp/remoteStep1partition1
```

运行程序

```bash
$ cd gs-yarn-batch-restart-dist
$ java -jar target/gs-yarn-batch-restart-dist/gs-yarn-batch-restart-client-0.1.0.jar
```

可以在YARN资源管理器看到程序是处于失败状态，因为第二阶段的partitioned steps导致的，在HDFS上创建`/tmp/remoteStep2partition0`和`/tmp/remoteStep2partition1`文件：

```bash
$ hdfs dfs -touchz /tmp/remoteStep2partition0
$ hdfs dfs -touchz /tmp/remoteStep2partition1
```

重新运行

```bash
$ java -jar target/dist/gs-yarn-batch-restart-client-0.1.0.jar
```

现在程序已经运行成功，并且只执行之前失败的partitioned steps


### 测试程序

这个类可以作为执行这个程序的单元测试类，而且不需要在Hadoop集群上运行


`gs-yarn-batch-restart-dist/src/test/java/hello/AppIT.java`

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
import org.hsqldb.Server;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.springframework.core.io.Resource;
import org.springframework.data.hadoop.fs.FsShell;
import org.springframework.yarn.boot.test.junit.AbstractBootYarnClusterTests;
import org.springframework.yarn.test.context.MiniYarnClusterTest;
import org.springframework.yarn.test.junit.ApplicationInfo;
import org.springframework.yarn.test.support.ContainerLogUtils;

@MiniYarnClusterTest
public class AppIT extends AbstractBootYarnClusterTests {

	@Test
	public void testApp() throws Exception {
		FsShell shell = new FsShell(getConfiguration());
		shell.touchz("/tmp/remoteStep1partition0");
		shell.touchz("/tmp/remoteStep1partition1");

		ApplicationInfo info1 = submitApplicationAndWait(ClientApplication.class, new String[0], 4, TimeUnit.MINUTES);
		assertThat(info1.getYarnApplicationState(), is(YarnApplicationState.FINISHED));
		assertLogs(ContainerLogUtils.queryContainerLogs(getYarnCluster(), info1.getApplicationId()), 10, 2, 2);

		shell.touchz("/tmp/remoteStep2partition0");
		shell.touchz("/tmp/remoteStep2partition1");
		shell.close();

		ApplicationInfo info2 = submitApplicationAndWait(ClientApplication.class, new String[0], 4, TimeUnit.MINUTES);
		assertThat(info2.getYarnApplicationState(), is(YarnApplicationState.FINISHED));
		assertLogs(ContainerLogUtils.queryContainerLogs(getYarnCluster(), info2.getApplicationId()), 6, 2, 0);
	}

	private void assertLogs(List<Resource> resources, int numResources, int numOk, int numFail) throws Exception {
		int ok = 0;
		int fail = 0;
		assertThat(resources, notNullValue());
		assertThat(resources.size(), is(numResources));
		for (Resource res : resources) {
			File file = res.getFile();
			String content = ContainerLogUtils.getFileContent(file);
			if (file.getName().endsWith("stdout")) {
				assertThat(file.length(), greaterThan(0l));
				if (file.getName().equals("Container.stdout")) {
					assertThat(content, containsString("Hello from HdfsTasklet"));
					if (content.contains("Hello from HdfsTasklet ok")) {
						ok++;
					}
					if (content.contains("Hello from HdfsTasklet fail")) {
						fail++;
					}
				}
			} else if (file.getName().endsWith("stderr")) {
				assertThat("stderr file is not empty: " + content, file.length(), is(0l));
			}
		}
		assertThat("Failed to find ok's from logs", ok, is(numOk));
		assertThat("Failed to find fail's from logs", fail, is(numFail));
	}

	private Server server = null;

	@Before
	public void startDb() {
		if (server == null) {
			server = new Server();
			server.setSilent(false);
			server.setTrace(true);
			server.setDatabaseName(0, "testdb");
			server.setDatabasePath(0, "mem:testdb");
			server.start();
		}
	}

	@After
	public void stopDb() {
		if (server != null) {
			server.stop();
			server = null;
		}
	}

}
```

     
##  总结
    
恭喜！ 你已经开发一个 Spring YARN 程序
   

### 其他类似内容
   
   以下有用的相关链接:

* [Simple YARN Application](https://spring.io/guides/gs/yarn-basic/)
* [Simple Single Project YARN Application](https://spring.io/guides/gs/yarn-basic-single/)
* [Batch YARN Application](https://spring.io/guides/gs/yarn-batch-processing/)
* [Building Spring YARN Projects with Maven](https://spring.io/guides/gs/maven-yarn/)
* [Building Spring YARN Projects with Gradle](https://spring.io/guides/gs/gradle-yarn/)


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



