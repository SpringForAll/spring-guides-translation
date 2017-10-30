# 创建批量服务

> 原文：[Creating a Batch Service](https://spring.io/guides/gs/batch-processing/)
>
> 译者：[Mr.lzc](https://github.com/cleverlzc)
>
> 校对：

本指南将引导您完成创建基本的批量驱动解决方案的过程。

## 你将构建什么

您将构建一个从CSV电子表格导入数据的服务，并使用自定义代码进行转换，并将最终结果存储在数据库中。

## 你需要准备什么

- 大约15分钟
- 一个自己喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或以上版本
- [Gradle 2.3+](https://gradle.org/install/) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到本地的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本指南

像大多数[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。不管采用哪种方式，最终都可以得到能够运行的代码。

如果是**从零开始**，则可以从[使用Gradle构建项目](#使用gradle构建项目)小节开始。

如果想**跳过基本的设置步骤**，可以按照以下步骤执行：

- [下载](https://github.com/spring-guides/gs-batch-processing/archive/master.zip) 并解压本指南相关的源文件，或者直接通过[Git](https://spring.io/understanding/Git)命令克隆到本地：

```bash
git clone https://github.com/spring-guides/gs-batch-processing.git
```

- 进入到 `gs-batch-processing/initial` 目录
- 直接跳到[创建业务类](#创建业务类)小节。

**当完成所有的编码以后**，可以将你的代码与`gs-batch-processing/complete`目录下的示例代码进行对比，以检查结果是否正确。

## 使用Gradle构建项目

首先需要设置一个基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，这里准备了在使用[Gradle](https://gradle.org/)和[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Gradle和Maven都不熟悉，可以参照[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle/)或[使用Maven构建Java项目](https://spring.io/guides/gs/gradle/)。

### 创建目录结构

选择需要创建项目的目录，参照照以下目录结构创建子目录；如果使用的是UNIX/LINUX系统，可以使用`mkdir -p src/main/java/hello`命令创建：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle 的构建文件

下面是一个[原始的Gradle build文件](https://github.com/spring-guides/gs-batch-processing/blob/master/initial/build.gradle)。

`build.gradle`

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.8.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-batch-processing'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-batch")
    compile("org.hsqldb:hsqldb")
    testCompile("junit:junit")
}
```

 [Spring Boot gradle 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，把它作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用Maven构建项目

首先，设置基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，在这里为你提供了使用[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Maven不熟悉，可以参照[使用maven构建JAVA项目工程](https://spring.io/guides/gs/maven) 。

### 创建目录结构

选择需要创建项目的目录，参照以下目录结构创建子目录；如果使用的是UNIX/LINUX系统，可以使用`mkdir -p src/main/java/hello` 命令创建：

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
    <artifactId>gs-batch-processing</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-batch</artifactId>
        </dependency>
        <dependency>
            <groupId>org.hsqldb</groupId>
            <artifactId>hsqldb</artifactId>
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

[Spring Boot maven 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## 使用IDE构建项目

- 在Spring Tool Suite中构建项目，请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。 
- 在IntelliJ IDEA中构建项目，请参照[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)。 

## 业务数据

通常您的客户或业务分析师提供电子表格。在这种情况下，你可以做到这一点。

`src/main/resources/sample-data.csv`

```csv
Jill,Doe
Joe,Doe
Justin,Doe
Jane,Doe
John,Doe
```

此电子表格每行包含名字和姓氏，用逗号分隔。 这是一个相当常见的模式，正如您将看到的那样，Spring会处理开箱即用的情况。

接下来，您编写一个SQL脚本来创建一个表存储数据。

`src/main/resources/schema-all.sql`

```sql
DROP TABLE people IF EXISTS;

CREATE TABLE people  (
    person_id BIGINT IDENTITY NOT NULL PRIMARY KEY,
    first_name VARCHAR(20),
    last_name VARCHAR(20)
);
```

> Spring Boot在启动过程中自动运行`schema-@@platform@@.sql`。 `-all`是所有平台的默认值。

## 创建业务类

现在看到数据输入和输出的格式，您编写代码来表示一行数据。

`src/main/java/hello/Person.java`

```java
package hello;

public class Person {
    private String lastName;
    private String firstName;

    public Person() {

    }

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return "firstName: " + firstName + ", lastName: " + lastName;
    }

}
```

您可以通过构造函数或通过设置属性来实例化具有名字和姓氏的Person类。

## 创建中间处理器

批处理中的一个常见范例是摄取数据，转换数据，然后将其导出到其他位置。 在这里，您编写一个简单的变换器，将名字转换为大写。

`src/main/java/hello/PersonItemProcessor.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.batch.item.ItemProcessor;

public class PersonItemProcessor implements ItemProcessor<Person, Person> {

    private static final Logger log = LoggerFactory.getLogger(PersonItemProcessor.class);

    @Override
    public Person process(final Person person) throws Exception {
        final String firstName = person.getFirstName().toUpperCase();
        final String lastName = person.getLastName().toUpperCase();

        final Person transformedPerson = new Person(firstName, lastName);

        log.info("Converting (" + person + ") into (" + transformedPerson + ")");

        return transformedPerson;
    }

}
```

`PersonItemProcessor`实现Spring Batch的`ItemProcessor`接口。这样可以方便地将代码连接到本指南中进一步定义的批处理作业中。根据接口，您会收到一个传入的`Person`对象，然后将其转换为大写形式的`Person`。

> 不要求输入和输出类型相同。事实上，在读取一个数据源之后，有时应用程序的数据流需要不同的数据类型。

## 将批处理工作放在一起

现在你把实际的批处理工作放在一起。Spring Batch提供了许多实用程序类，可以减少编写自定义代码的需要。相反，您可以专注于业务逻辑。

`src/main/java/hello/BatchConfiguration.java`

```java
package hello;

import javax.sql.DataSource;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecutionListener;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.launch.support.RunIdIncrementer;
import org.springframework.batch.item.database.BeanPropertyItemSqlParameterSourceProvider;
import org.springframework.batch.item.database.JdbcBatchItemWriter;
import org.springframework.batch.item.file.FlatFileItemReader;
import org.springframework.batch.item.file.mapping.BeanWrapperFieldSetMapper;
import org.springframework.batch.item.file.mapping.DefaultLineMapper;
import org.springframework.batch.item.file.transform.DelimitedLineTokenizer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
@EnableBatchProcessing
public class BatchConfiguration {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Autowired
    public DataSource dataSource;

    // tag::readerwriterprocessor[]
    @Bean
    public FlatFileItemReader<Person> reader() {
        FlatFileItemReader<Person> reader = new FlatFileItemReader<Person>();
        reader.setResource(new ClassPathResource("sample-data.csv"));
        reader.setLineMapper(new DefaultLineMapper<Person>() {{
            setLineTokenizer(new DelimitedLineTokenizer() {{
                setNames(new String[] { "firstName", "lastName" });
            }});
            setFieldSetMapper(new BeanWrapperFieldSetMapper<Person>() {{
                setTargetType(Person.class);
            }});
        }});
        return reader;
    }

    @Bean
    public PersonItemProcessor processor() {
        return new PersonItemProcessor();
    }

    @Bean
    public JdbcBatchItemWriter<Person> writer() {
        JdbcBatchItemWriter<Person> writer = new JdbcBatchItemWriter<Person>();
        writer.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<Person>());
        writer.setSql("INSERT INTO people (first_name, last_name) VALUES (:firstName, :lastName)");
        writer.setDataSource(dataSource);
        return writer;
    }
    // end::readerwriterprocessor[]

    // tag::jobstep[]
    @Bean
    public Job importUserJob(JobCompletionNotificationListener listener) {
        return jobBuilderFactory.get("importUserJob")
                .incrementer(new RunIdIncrementer())
                .listener(listener)
                .flow(step1())
                .end()
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Person, Person> chunk(10)
                .reader(reader())
                .processor(processor())
                .writer(writer())
                .build();
    }
    // end::jobstep[]
}
```

对于初学者，`@EnableBatchProcessing`注释添加了许多支持作业的关键beans，并节省了大量的工作。 此示例使用基于内存的数据库（由`@EnableBatchProcessing`提供），这意味着完成后，数据就会消失。

分解如下：

`src/main/java/hello/BatchConfiguration.java`

```java
 @Bean
    public FlatFileItemReader<Person> reader() {
        FlatFileItemReader<Person> reader = new FlatFileItemReader<Person>();
        reader.setResource(new ClassPathResource("sample-data.csv"));
        reader.setLineMapper(new DefaultLineMapper<Person>() {{
            setLineTokenizer(new DelimitedLineTokenizer() {{
                setNames(new String[] { "firstName", "lastName" });
            }});
            setFieldSetMapper(new BeanWrapperFieldSetMapper<Person>() {{
                setTargetType(Person.class);
            }});
        }});
        return reader;
    }

    @Bean
    public PersonItemProcessor processor() {
        return new PersonItemProcessor();
    }

    @Bean
    public JdbcBatchItemWriter<Person> writer() {
        JdbcBatchItemWriter<Person> writer = new JdbcBatchItemWriter<Person>();
        writer.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<Person>());
        writer.setSql("INSERT INTO people (first_name, last_name) VALUES (:firstName, :lastName)");
        writer.setDataSource(dataSource);
        return writer;
    }
 ```
 
第一个代码块定义了输入，处理器和输出。- `reader()`创建一个`ItemReader`类。它查找一个名为`sample-data.csv`的文件，并解析具有足够信息的每个行项目，将其转换为一个`Person`。- `processor()`创建一个我们先前定义的`PersonItemProcessor`的一个实例，用于转换大写数据。- `write（DataSource）`创建一个ItemWriter类。这个目标是针对JDBC，并自动获取由`@EnableBatchProcessing`创建的dataSource的副本。它包括插入由Java bean属性驱动的单个`Person`所需的SQL语句。

下一个重点是实际的工作配置。

`src/main/java/hello/BatchConfiguration.java`

```java
 @Bean
    public Job importUserJob(JobCompletionNotificationListener listener) {
        return jobBuilderFactory.get("importUserJob")
                .incrementer(new RunIdIncrementer())
                .listener(listener)
                .flow(step1())
                .end()
                .build();
    }

    @Bean
    public Step step1() {
        return stepBuilderFactory.get("step1")
                .<Person, Person> chunk(10)
                .reader(reader())
                .processor(processor())
                .writer(writer())
                .build();
    }
```
    
第一个方法定义了作业，第二个方法定义了一个步骤。作业是从步骤构建的，每个步骤都可以涉及读取器，处理器和写入器。

在此作业定义中，您需要一个增量器，因为作业使用数据库来维护执行状态。然后您列出每个步骤，其中该作业只有一步。作业结束后，Java API生成完美配置的作业。

在步骤定义中，您可以定义一次写入的数据量。在这种情况下，它最多可以写入十条记录。接下来，您使用前面的注入位配置读取器，处理器和写入器。

chunk()由前缀`<Person，Person>`修饰，因为它是一个泛型方法。 这表示处理的每个“块”的输入和输出类型，并与`ItemReader <Person>`和`ItemWriter <Person>`排列。

`src/main/java/hello/JobCompletionNotificationListener.java`

```java
package hello;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.batch.core.BatchStatus;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.listener.JobExecutionListenerSupport;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Component;

@Component
public class JobCompletionNotificationListener extends JobExecutionListenerSupport {

	private static final Logger log = LoggerFactory.getLogger(JobCompletionNotificationListener.class);

	private final JdbcTemplate jdbcTemplate;

	@Autowired
	public JobCompletionNotificationListener(JdbcTemplate jdbcTemplate) {
		this.jdbcTemplate = jdbcTemplate;
	}

	@Override
	public void afterJob(JobExecution jobExecution) {
		if(jobExecution.getStatus() == BatchStatus.COMPLETED) {
			log.info("!!! JOB FINISHED! Time to verify the results");

			List<Person> results = jdbcTemplate.query("SELECT first_name, last_name FROM people", new RowMapper<Person>() {
				@Override
				public Person mapRow(ResultSet rs, int row) throws SQLException {
					return new Person(rs.getString(1), rs.getString(2));
				}
			});

			for (Person person : results) {
				log.info("Found <" + person + "> in the database.");
			}

		}
	}
}
```

此代码侦听作业是否为`BatchStatus.COMPLETED`状态时，然后使用`JdbcTemplate`检查结果。

## 使应用程序可以执行

虽然批处理可以嵌入到Web应用程序和WAR文件中，但下面演示的更简单的方法创建了一个独立的应用程序。 您将所有内容都包装在一个可执行的JAR文件中，由一个好的旧的Java `main()`方法驱动。

`src/main/java/hello/Application.java`

```java
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

`@SpringBootApplication` 是一个方便的注解，它会自动添加以下所有内容：

- `@Configuration`将该类标记为应用程序上下文中bean定义的源。
- `@EnableAutoConfiguration`指示Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。
- 通常，您将为Spring MVC 应用程序添加`@EnableWebMvc`注解，但是当Spring Boot在类路径中发现**spring-webmvc**时会自动添加该注解。通过添加该注解将应用程序标记为Web应用程序，并进行一些关键操作，比如设置`DispatcherServlet`。
- `@ComponentScan`通知Spring在`hello`包中查找其他组件，配置和服务，允许Spring扫描到控制器。

`main()`方法使用Spring Boot的`SpringApplication.run()`方法启动应用程序。你注意到我们没有写过一行XML代码吗？而且也没有**web.xml**配置文件。此Web应用程序是100%纯Java编写的，无需再配置其他基础设施。

为了演示的目的，创建一个`JdbcTemplate`，查询数据库，并打印出批处理作业插入的人的名字的代码。

### 构建可执行的JAR

程序创建好以后，可以使用Gradle或Maven从命令行运行。或者，也可以将所有必需的依赖项，类和资源打包成一个可执行的JAR文件，并运行该文件。这种方式使得在整个开发生命周期中，应用程序可以轻松地发布，更新版本和部署服务。

如果你使用的是Gradle，则可以使用`./gradlew bootRun`运行应用程序。或者使用`./gradlew build`来构建JAR文件。然后运行这个JAR文件：

```
java -jar build/libs/gs-soap-service-0.1.0.jar
```

如果你使用的是Maven，可以使用`./mvnw spring-boot:run`运行应用程序，或者使用`./mvnw clean package`来构建JAR文件。然后运行这个JAR文件：

```
java -jar target/gs-soap-service-0.1.0.jar
```

> 上面的过程创建了一个可运行的JAR。也可以选择[构建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

上述操作完成后，将会看到有日志信息输出，服务程序将会在几秒内启动并运行。

该作业为每个被转换成大写的人的信息打印出一行。作业运行后，您还可以查看查询数据库的输出。

```
Converting (firstName: Jill, lastName: Doe) into (firstName: JILL, lastName: DOE)
Converting (firstName: Joe, lastName: Doe) into (firstName: JOE, lastName: DOE)
Converting (firstName: Justin, lastName: Doe) into (firstName: JUSTIN, lastName: DOE)
Converting (firstName: Jane, lastName: Doe) into (firstName: JANE, lastName: DOE)
Converting (firstName: John, lastName: Doe) into (firstName: JOHN, lastName: DOE)
Found <firstName: JILL, lastName: DOE> in the database.
Found <firstName: JOE, lastName: DOE> in the database.
Found <firstName: JUSTIN, lastName: DOE> in the database.
Found <firstName: JANE, lastName: DOE> in the database.
Found <firstName: JOHN, lastName: DOE> in the database.
```

## 总结

恭喜！您构建了一个批处理作业，从电子表格中获取数据，对其进行处理，并将其写入数据库。

## 了解更多

以下指南也可能有帮助：

- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
- [Accessing Data with GemFire](https://spring.io/guides/gs/accessing-data-gemfire/)
- [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)

想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

