# 使用 LDAP 鉴权用户

> 原文：[Authenticating a User with LDAP(https://spring.io/guides/gs/authenticating-ldap/)
>
> 译者：[francischan](https://github.com/francischan714)
>
> 校对：


本指南引导您完成创建应用程序的过程，该应用程序通过[Spring Security](https://projects.spring.io/spring-security) 的 LDAP模块。


## 你将创建什么  

您将创建一个简单的 Web 应用，应用的安全由 Spring Security 中的 LDAP 服务完全，LDAP 服务将通过读取包含了一组用户的数据文件提供认证功能。


## 开始之前你需要准备

大约15分钟时间

一个喜欢的文本编辑器或者IDE

[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

[Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

你也可以直接导入代码到IDE:

[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)


## 如何完成指南？  

像大多数 `Spring` [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#使用Gradle构建)。

**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/gs-accessing-mongodb-data-rest/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/gs-accessing-mongodb-data-rest.git`
* 进入 `gs-accessing-mongodb-data-rest/initial`目录
* 跳过前面的部分[安装并启动MongoDB](#安装并启动MongoDB)

**当你完成之后**，你可以在`gs-accessing-mongodb-data-rest/complete`根据代码检查下结果。

## 使用Gradle构建

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建， 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

在你的项目根目录，创建如下的子目录结构; 例如，如果你使用的是\*nix系统，你可以使用`mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份[初始化Gradle构建文件](https://github.com/spring-guides/gs-accessing-mongodb-data-rest/blob/master/initial/build.gradle)

`build.gradle`

```
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
    baseName = 'gs-authenticating-ldap'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

// tag::security[]
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
// end::security[]```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷。
* 搜索 `public static void main()` 方法并且将它标记为可执行类。
* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建

首先，你需要设置一个基本的构建脚本。当使用 Spring 构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

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
    <artifactId>gs-authenticating-ldap</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!-- tag::security[] -->
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
    <!-- end::security[] -->

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

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布你的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。  

## 使用你的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。

## 创建一个简单的 web controller
在Spring中，REST 通常使用 Spring MVC controllers，以下遵循 Spring MVC controller 处理 `GET /` 请求并返回简单的信息。

`src/main/java/hello/HomeController.java`

~~~java
package hello;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HomeController {

    @GetMapping("/")
    public String index() {
        return "Welcome to the home page!";
    }
}
~~~

这个实体类使用了`RestControler`的注解，所以 Spring MVC 可以自动探测到这个controller使用内置的扫描功能并自动的配置web路由。

这个方法使用了`@RequestMapping`的注解，用来标记是方法的路由路径和REST的动作，这这里，GET是默认的行为，他将返回指向主页的信息。

`@RestController` 也告诉了Spring MVC，这将直接返回文本形式的http响应，而不是视图。换句话说，当你访问主页时，你将得到简单的信息在浏览器上。

## 创建不安全的 web 应用

在保证 web 应用安全之前，为了校验它确实有用，你需要创建一些关键的bean，比如 `Application` 类。

`src/main/java/hello/Application.java`

~~~java
package hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
~~~

`@SpringBootApplication` 是非常便捷的注解
* `@Configuration` 帮助应用上下文声明bean。
* `@EnableAutoConfiguration` 基于classpath的方式装载bean，其他bean可以有不同属性的配置。
* 通常你需要添加`@EnableWebMvc`来使用Spring MVC应用，但是Spring Boot 会自动添加 Spring MVC 当发现Spring—webmvc在classpath中。这个注解可以开启web应用并激活核心的功能，比如配置 DispatcherServlet。
* `@ComponentScan` 告知Spring自动加载其他在`hello`包中的components,configurations和services，让其可以发现controller。

这个`main()`方法可以使用Spring Boot的`SpringApplication.run()`方法来启动应用，你注意到这里没有一行XML吗？而且也没有 web.xml 文件。这个 web 应用是 100% java写的，你不需要处理其他结构的配置。

## 构建一个可执行的应用程序

尽管可以将此服务作为传统 `WAR` 文件打包以部署到外部应用程序服务器，但下面演示的更简单的方法创建了独立的应用程序。 你把所有东西都封装在一个单独的，可执行的 `JAR` 文件中，由`Java main`方法执行。 当然 `Spring` 同样支持内置的 `Tomcat`容器，这样你也不需要部署到其他的 `Tomcat`。
```
src/main/java/hello/Application.java
```
```java
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

`@SpringBootApplication`作为一个方便使用的注解，提供了如下的功能：

- `@Configuration` 表明使用该注解的类是应用程序上下文(Applicaiton Context)中Bean定义的来源。
- `@EnableAutoConfiguration` 注解根据 `classpath` 的配置、其他bean的定义或者不同的属性设置(property settings)等条件，使 `Spring Boot` 自动加入所需的bean。
- 对于 `Spring MVC` 应用，通常需要加入`@EnableWebMvc`注解，但是当**spring-webmvc** 存在于 `classpath`中时，`Spring Boot` 自动加入该注解。该注解将当前应用标记为web应用，并激活 `web` 应用的关键行为，例如开启 `DispatcherServlet`。
- `@ComponentScan` 注解使 `Spring` 在 `hello` 包(package)中搜索其他的组件、配置(configurations)和服务(service),在本例中，spring会搜索到控制器(controllers)。

`main()` 方法使用 `Spring Boot` 的 `SpringApplication.run()` 方法来启动应用。你有没有注意到本例子中一行 `XML` 代码都没有吗？也没有 `web.xml` 文件。此web应用100%使纯java代码，因此不需花精力处理任何像基础设施或者下水管道一般的配置工作。

## 构建可执行的JAR

您可以在命令行使用 `Gradle` 或 `Maven` 运行应用程序。或者您可以构建一个可执行的 `JAR` 文件,其中包含所有必需的依赖关系,类,和资源,并运行。这使得在整个开发生命周期中非常容易，跨不同的环境，版本等传输和部署服务成为一个应用程序,等等。

如果您使用的是 `Gradle`，则可以使用该应用程序 `./gradlew bootRun`,或者您可以使用 `JAR` 文件构建 `./gradlew build`,然后可以运行 `JAR` 文件：

```
java -jar build/libs/gs-authenticating-ldap-0.1.0.jar
```

如果您使用 `Maven`，则可以使用该命令 `./mvnw spring-boot:run`,或者您可以使用 `JAR` 文件来构建 `./mvnw clean package`,然后可以运行 `JAR` 文件：

```
java -jar target/gs-authenticating-ldap-0.1.0.jar
```

> 上面的过程将创建一个可运行的 `JAR`。您也可以选择 [构建一个 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

输出日志，该服务应该在几秒钟内启动并运行。

## 设置 Spring Security
配置Spring Security 的配置，搭建应用时首先要添加一些额外的依赖。

对于基于Gradle的构建

`build.gradle`

~~~
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-security")
    compile("org.springframework.ldap:spring-ldap-core")
    compile("org.springframework.security:spring-security-ldap")
    compile("org.springframework:spring-tx")
    compile("com.unboundid:unboundid-ldapsdk")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    testCompile("org.springframework.security:spring-security-test")
}
~~~
> 由于这个问题是由gradle产生的，spring-tx 必须要引入，否则Gradle将自动获取不能适配的旧版本。

对于基于Maven构建的

`pom.xml`

~~~xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.ldap</groupId>
        <artifactId>spring-ldap-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-ldap</artifactId>
    </dependency>
    <dependency>
        <groupId>com.unboundid</groupId>
        <artifactId>unboundid-ldapsdk</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
~~~
这些依赖添加 Spring Security 和Unboundld 的开源 LDAP server，在这里，你可以使用纯java来配置你的安全策略。

`src/main/java/hello/WebSecurityConfig.java`

~~~java
package hello;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.encoding.LdapShaPasswordEncoder;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.anyRequest().fullyAuthenticated()
				.and()
			.formLogin();
	}

	@Override
	public void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth
			.ldapAuthentication()
				.userDnPatterns("uid={0},ou=people")
				.groupSearchBase("ou=groups")
				.contextSource()
					.url("ldap://localhost:8389/dc=springframework,dc=org")
					.and()
				.passwordCompare()
					.passwordEncoder(new LdapShaPasswordEncoder())
					.passwordAttribute("userPassword");
	}

}
~~~

`@EnableWebSecurity` 是打开许多需要用到Spring Security bean 的开关。

你也可以使用 LDAP server，Sping Boot提供了自动配置对于内置的纯java写的server，这将被本指导使用。`ldapAuthentication()`方法的配置，在登录表单插入了，如`uid={0},ou=people,dc=springframework,dc=org`在LDAP server，同样 `passwordCompare()` 方法配置了加密和密码的属性。

## 设置用户数据

LDAP servers 使用 LDIF（LDAP 数据交换格式）文件用于交换用户数据，`spring.ldap.embedded.ldif` 属性在 `application.properties`中，允许Spring Boot 拉去 LDIF 数据文件，使得预加载demon数据更简单。

`src/main/resources/test-server.ldif`

~~~
dn: dc=springframework,dc=org
objectclass: top
objectclass: domain
objectclass: extensibleObject
dc: springframework

dn: ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: groups

dn: ou=subgroups,ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: subgroups

dn: ou=people,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: people

dn: ou=space cadets,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: space cadets

dn: ou=\"quoted people\",dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: "quoted people"

dn: ou=otherpeople,dc=springframework,dc=org
objectclass: top
objectclass: organizationalUnit
ou: otherpeople

dn: uid=ben,ou=people,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Ben Alex
sn: Alex
uid: ben
userPassword: {SHA}nFCebWjxfaLbHHG1Qk5UU4trbvQ=

dn: uid=bob,ou=people,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Bob Hamilton
sn: Hamilton
uid: bob
userPassword: bobspassword

dn: uid=joe,ou=otherpeople,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Joe Smeth
sn: Smeth
uid: joe
userPassword: joespassword

dn: cn=mouse\, jerry,ou=people,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Mouse, Jerry
sn: Mouse
uid: jerry
userPassword: jerryspassword

dn: cn=slash/guy,ou=people,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: slash/guy
sn: Slash
uid: slashguy
userPassword: slashguyspassword

dn: cn=quote\"guy,ou=\"quoted people\",dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: quote\"guy
sn: Quote
uid: quoteguy
userPassword: quoteguyspassword

dn: uid=space cadet,ou=space cadets,dc=springframework,dc=org
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Space Cadet
sn: Cadet
uid: space cadet
userPassword: spacecadetspassword



dn: cn=developers,ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: groupOfUniqueNames
cn: developers
ou: developer
uniqueMember: uid=ben,ou=people,dc=springframework,dc=org
uniqueMember: uid=bob,ou=people,dc=springframework,dc=org

dn: cn=managers,ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: groupOfUniqueNames
cn: managers
ou: manager
uniqueMember: uid=ben,ou=people,dc=springframework,dc=org
uniqueMember: cn=mouse\, jerry,ou=people,dc=springframework,dc=org

dn: cn=submanagers,ou=subgroups,ou=groups,dc=springframework,dc=org
objectclass: top
objectclass: groupOfUniqueNames
cn: submanagers
ou: submanager
uniqueMember: uid=ben,ou=people,dc=springframework,dc=org
~~~

> 对于生产环境，使用 LDIF 文件不是标准的配置，然而，他对于测试和案例来说非常有效。

如果你访问网站 ` http://localhost:8080` ，你应该会重定向到Spring Security提供的登录页。

输入用户名 **ben** 和 密码 **benspassword**，你可以看到信息在你的浏览器上

~~~
Welcome to the home page!
~~~



## 总结

恭喜你!实现了一个通过基于[Spring Security](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/)的应用程序，在此例中，使用了 `基于LDAP的用户存储` 。

## 了解更多
下面的指南也非常有帮助：
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot)  


想写一个新的指南或贡献一个现有的？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
