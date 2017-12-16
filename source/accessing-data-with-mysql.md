# 使用MySQL数据库访问数据

> 原文：[Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
>
> 译者：nemoAndroid
>
> 校对：

本指南将引导你创建一个连接MySQL数据库的Spring应用的过程，相对于其它使用内置于内存或嵌入式数据库的向导和许多示例应用。本向导使用Spring Data JPA的方式来访问数据库，但这只是许多可能选择中的一种（例如：你可以使用的Spring JDBC方式）。

## 你将创建什么
你将创建一个MySQL数据库，并构建一个连接此新创建数据库的Spring应用。

* MySQL使用GPL许可证，因此你分发时，任何使用它的二进制程序都也必须使用GPL许可证。可参考GNU公共许可证。

## 开始之前你需要准备

MySQL5.6或更高版本，若你已安装了docker，docker将作为运行容器对运行数据库是有帮助的

大约15分钟

一个自己喜欢的文本编辑器或集成开发环境IDE

[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

[Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

你也可以直接将代码导入到你的本地IDE中：

[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成指南？
像大多数 Spring [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#scratch)。

**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/gs-accessing-data-mysql/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

`git clone https://github.com/spring-guides/gs-accessing-data-mysql.git`
* 进入 `gs-accessing-data-mysql/initial`目录
* 直接跳转到创建数据库部分.

**当你完成之后**，你可以将你的代码与gs-accessing-data-mysql/complete目录下的示例代码进行对比，以核对你的结果是否正确。
 `gs-accessing-data-mysql/complete`
 
 ## 使用Gradle构建项目
 首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建， 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

选择一个需要创建的项目，参照以下目录结构创建子目录结构，例如，在Unix或Linux系统使用的创建命令是`mkdir -p src/main/java/hello`：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一个[初始的Gradle构建文件](https://github.com/spring-guides/gs-accessing-data-mysql/blob/master/initial/build.gradle)

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
    baseName = 'gs-accessing-data-rest'
    version = '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-rest")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 此插件可以把类路径下的所有jar打包成一个单独运行的“über-jar”，使得运行和发布你的服务变得更加便捷。
* 此插件通过搜索 `public static void main()` 方法并且将它标记为可执行类。
* 此插件提供了一个内置的依赖解析器，可自动调整版本号来匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。你可以覆盖其中的任何一个版本，但默认情况下它会使用Spring Boot版本集中的版本。

## 使用Maven构建

首先，你需要创建一个基本的构建脚本。当使用 Spring 构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

选择需要创建项目的目录，参照以下目录结构创建如下的子目录结构；例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

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
    <artifactId>gs-mysql-data</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
  
    <!-- JPA Data (我们将使用Repositories, Entities, Hibernate, etc...) -->
  
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
  
     <!-- 使用 MySQL Connector-J -->
  
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
  
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它可以把类路径下的所有jar打包成一个单独运行的“über-jar”，这样可以更方便地执行和发布你的服务。
- 它通过搜索public static void main()方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器,可自动调整版本号来匹配[Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。你可以覆盖其中的任何一个版本，但默认情况下它会使用Spring Boot版本集中的版本。

## 使用你的IDE构建

- 在Spring Tool Suite中构建项目，请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 在IntelliJ IDEA中构建项目，请参照 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)。

## 创建数据库
进入终端（微软Windows下就是使用cmd进入命令提示符）。使用一个有创建新用户权限的用户打开MySQL客户端。

例如：在Linux平台，就使用如下命令打开：
```
$dudo mysql --password
```

* 使用root用户连接MySQL，对于生产环境的服务器，这种方式是不推荐使用的。

创建一个新数据库的命令如下：
```
mysql> create database db_example;--Create the new database
mysql> create user 'springuser'@'localhost' identified by 'ThePassword';
mysql> grant all on db_example.* to 'springuser'@'localhost';--Gives all privileges to the new user on the newly created database
```
## 创建application.properties配置文件

Spring Boot给你提供了关于所有事情的默认值，默认的数据库是`H2`数据库，因此当你想改变默认值，而用其它的数据库时，你必须在`application.properties`配置文件中定义这个数据库的连接属性值。

在资源source文件夹值，你需创建一个资源文件，路径如下：`src/main/resources/application.properties`

```
spring.jpa.hibernate.ddl-auto=create
spring.datasource.url=jdbc:mysql://localhost:3306/db_example
spring.datasource.username=springuser
spring.datasource.password=ThePassword
```

这里的`spring.jpa.hibernate.ddl-auto`属性取值可以是`none`，`update`，`create`，`create-drop`,可以从Hibernate文档中参考更详细的信息。

* `none`   这是`MySQL`的默认值，不对数据库结构做任何改变；
* `update` 根据已知的Entitiy实体结构，Hibernate改变数据库；
* `create` 每次都创建数据库，但在关闭时不销毁；
* `create-drop` 每次都创建数据库，但在`SessionFactory`关闭时，创建的数据库会被销毁；

这里我们一开始使用`create`属性值，因为我们一开始还没有数据库的结构。在第一次运行之后，我们就可以根据程序的需要改变这个值为`update`或`none`。当你想对数据库结构进行一些修改时，就直接使用`update`属性值。

对于`H2`数据库和其它嵌入式数据库来说，spring.jpa.hibernate.ddl-auto属性取值是`create-drop`，但对于其它数据库，像`MySQL`，属性值就是`none`。

当你的数据库在生产环境状态下，这是一个好的安全实践，属性值是none，你就可以取消所有MySQL用户连接Spring应用的所有权限，给他们的权限只有SELECT，UPDATE，INSERT，DELETE。

接下来的内容就是本篇指南的结尾了。

## 创建@Entitiy模型

`src/main/java/hello/User.java`

```
package hello;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity // 此标签会告诉Hibernate根据此类创建对应的一张数据表
public class User {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;

    private String name;

    private String email;

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

}
```
这是一个实体类，Hibernate会自动将它翻译为数据库中的一张数据表。


`src/main/java/hello/UserRepository.java`

```
package hello;

import org.springframework.data.repository.CrudRepository;

import hello.User;

//本接口将会被Spring自动实现为一个名字为userRepository的Bean
//  CRUD就是：创建Create, 查询Read, 更新Update, 删除Delete

public interface UserRepository extends CrudRepository<User, Long> {

}
```

## 创建对应的仓库（repository）

这是一个repository接口，它会被Spring自动实现，并使用和接口相同的名字，只是首字母改为小写，即实现接口的类名是`userRepository`。


## 为你的Spring应用创建一个新的控制器类controller

`src/main/java/hello/MainController.java`

```
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import hello.User;
import hello.UserRepository;

@Controller    // 此标签标记此类是一个控制器类Controller
@RequestMapping(path="/demo") // 此标签表示以/demo开头的URL请求都会被此控制器类拦截
public class MainController {
	@Autowired // 此标签表示Spring框架会自动产生一个userRepository类，
	           // 我们直接使用此类处理数据就可以了
	private UserRepository userRepository;

	@GetMapping(path="/add") // 只拦截GET请求Requests
	public @ResponseBody String addNewUser (@RequestParam String name
			, @RequestParam String email) {
		// @ResponseBody标签表示响应是一个返回的字符串, 不是一个视图名
		// @RequestParam标签表示是从GET 或 POST请求传过来的参数

		User n = new User();
		n.setName(name);
		n.setEmail(email);
		userRepository.save(n);
		return "Saved";
	}

	@GetMapping(path="/all")
	public @ResponseBody Iterable<User> getAllUsers() {
		// 此返回值是所有user信息，信息是JSON格式或XML格式的
		return userRepository.findAll();
	}
}
```

* 上面例子没有明确指定是`GET`请求、`PUT`请求或`POST`请求等等，因为`@GetMapping`标签是`@RequestMapping(method=GET)`标签的缩写。`@RequestMapping`标签默认会拦截所有的HTTP请求。使用`@RequestMapping(method=GET)`或[其它缩写形式的标签](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/package-summary.html)都缩小了拦截的范围。


## 将应用变得可直接运行

尽管可以将这个程序的服务打包为一个传统的[WAR](https://spring.io/understanding/WAR)文件，并将其部署到一个外部的应用服务器上。可下面展示了一个更简单的方法，就是创建一个单独的应用。你可以将所有的文件打包为一个单独的、可运行的JAR文件，并使用Java的`main()`方法作为程序运行的入口。同时你使用Spring支持的内嵌的Tomcat Servlet容器作为HTTP的运行时环境，而不是部署到一个外部的[Tomcat](https://spring.io/understanding/Tomcat)实例上。

`src/main/java/hello/Application.java`

```
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

构建一个可执行的JAR文件

你可以在从命令行中，使用Gradle或Maven运行这个应用。或者你也可以构建一个可直接执行的JAR文件，此文件包含了所有必要的依赖文件、类文件和资源文件，并可以直接执行它。这使它在整个开发周期都很容易运输、版本控制和部署为一个应用，且可跨不同的环境等等。

若你正在使用的是Gradle，你可以使用`./gradlew bootRun`命令来运行应用，或你可以使用`./gradlew build`命令构建这个JAR文件，并使用如下命令来运行这个JAR文件：`java -jar build/libs/gs-accessing-data-mysql-0.1.0.jar`

若你正在使用的是Maven，你可以使用`./mvnw spring-boot:run`命令来运行应用，或使用`./mvnw clean package`命令来构建这个JAR文件。使用如下命令来运行此JAR文件：`java -jar target/gs-accessing-data-mysql-0.1.0.jar`

* 上面的命令将会创建一个可直接运行的JAR文件，你也可以选择[构建一个WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

日志输出是可以显示的。这个程序就可以几秒运行起来的。


## 测试此应用

现在，此应用正在运行中，你可以测试它了。

例如可使用`curl`命令来测试。现在你有2个可测试的REST Web服务。通过`localhost:8080/demo/all`访问，可以得到所有的用户数据，通过`localhost:8080/demo/add`访问，可新增一条用户数据。

```
$ curl 'localhost:8080/demo/add?name=First&email=someemail@someemailprovider.com'
```

得到的响应应该是 
```
Saved
```

```
$ curl 'localhost:8080/demo/all'
```

得到的响应应该是：
```
[{"id":1,"name":"First","email":"someemail@someemailprovider.com"}]
```


## 做一些安全方面的优化

现在当你处在生产环境时，你可能会遭到SQL注入的攻击。黑客可能会注入`DROP TABLE`或任何其它有害的SQL命令。因此作为应该安全实践，在用户开始使用你的应用之前，对你的数据库执行下面的命令来做这些改变。

```
mysql> revoke all on db_example.* from 'springuser'@'localhost';
```

上面这条命令会收回所有连接此Spring应用的用户的权限。现在此Spring应用不能对数据库做任何事情。但我们不希望这样，因此可以执行下面的命令：
```
mysql> grant select, insert, delete, update on db_example.* to 'springuser'@'localhost';
```
这条命令只赋给你的Spring应用必要的修改数据的权限，没有修改数据库结构的权限。

现在可对你的`src/main/resources/application.properties`做些更改：
```
spring.jpa.hibernate.ddl-auto=none
```

上面命令改变了此属性的初始值`create`，此初始值是在Hibernate从你的实体创建数据库数据表时设置的。

当你想对数据库做些改变，可以通过改变`spring.jpa.hibernate.ddl-auto`属性的值为`update`来重新获得授权。此时，重新运行你的应用，或更好的方法是使用专用的迁移工具，例如Flyway或Liquibase。`


## 总结

祝贺你！你已经开发了一个连接MySQL数据库的Spring应用。


## 了解更多

下面的指南可能对你有用：
* [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
* [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
* [Accessing Data with Gemfire](https://spring.io/guides/gs/accessing-data-gemfire/)

想写一个新的指南或贡献一个现有的？可查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。



> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



