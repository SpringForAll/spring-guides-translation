# 创建SOAP Web Service服务

> 原文：[Producing a SOAP web service](https://spring.io/guides/gs/producing-web-service/) 
>
> 译者：feilangrenM
>
> 校对：

本指南将帮助你了解使用Spring构建SOAP Web Service 服务端程序的基本流程。

## 你将创建什么

你将创建一个基于WSDL的SOAP web service服务器，用它发布不同欧洲国家的数据信息。

> 为了简化示例，接下来将使用英国、西班牙和波兰三个国家的硬编码数据。

## 你需要准备什么

- 大约15分钟
- 一个富文本编辑器或者IDE


- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 及以上版本


- [Gradle 2.3+](https://gradle.org/install/) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)


- 你也可以直接将代码导入到本地的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 怎样使用本指南

与大多数[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。不管采用哪种方式，最终都可以写出能够运行的代码。

如果是**从零开始**，则可以从[使用Gradle构建项目](#使用gradle构建项目)小节开始。

如果想**跳过基本的设置步骤**，可以按照以下步骤执行：

- [下载](https://github.com/spring-guides/gs-soap-service/archive/master.zip) 并解压本指南相关的源文件，或者直接通过git命令克隆到本地：

  ```
  git clone https://github.com/spring-guides/gs-soap-service.git
  ```


- 进入 `gs-soap-service/initial` 目录
- 直接跳到[添加Spring-WS依赖](#添加-spring-ws-依赖)小节。

**当完成所有的编码以后**，可以将你的代码与`gs-soap-service/complete`目录下的示例代码进行对比，以检查结果是否正确。

## 使用Gradle构建项目
* xx

首先需要设置一个基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，这里准备了在使用[Gradle](https://gradle.org/)和[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Gradle和Maven都不熟悉，可以参照[使用Gradle构建Java项目](https://spring.io/guides/gs/gradle/)或[使用Maven构建Java项目](https://spring.io/guides/gs/gradle/)。

### 创建目录结构

选择需要创建项目的目录，参照照以下目录结构创建子目录；如果使用的是UNIX系统，可以使用`mkdir -psrc/main/java/hello`命令创建：

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle 的构建文件

下面是一个[原始的Gradle build文件](https://github.com/spring-guides/gs-producing-web-service/blob/master/initial/build.gradle)。

`build.gradle`

```groovy
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
    baseName = 'gs-producing-web-service'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
}
```

 [Spring Boot gradle 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，把它作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

*xx## 使用Maven构建项目

首先，设置基本的构建脚本。在使用Spring构建应用程序时，你可以使用任何自己喜欢的构建系统，在这里为你提供了使用[Maven](https://maven.apache.org/)构建项目时需要的代码。如果你对Maven不熟悉，可以参照[使用maven构建JAVA项目工程](https://spring.io/guides/gs/maven) 。

### 创建目录结构

选择需要创建项目的目录，参照以下目录结构创建子目录；如果使用的是UNIX系统，可以使用`mkdir -psrc/main/java/hello` 命令创建：

`mkdir -psrc/main/java/hello`

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
    <artifactId>gs-producting-web-service</artifactId>
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
```

 [Spring Boot maven 插件 ](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了很多便捷的特性：

- 该插件可以把类路径下所有的jar打包成一个可以运行的“über-jar”，给程序的执行和传输带来很大的方便。
- 该插件会自动搜索程序中的`public static void main()` 方法，作为程序运行的入口。
- 它还提供了一个内置的依赖解析器，可以自动调整版本号与 [Spring Boot 的依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)相一致。你可以覆盖其中的任何一个版本，但是默认情况下它会使用Spring Boot自身版本集中的版本。

## *xx使用IDE构建项目

- 在Spring Tool Suite中构建项目，请参照 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。 
- 在IntelliJ IDEA中构建项目，请参照[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea)。 

## 添加 Spring-WS 依赖

在创建项目时需要在构建文件中添加`spring-ws-core` 和 `wsdl4j` 的相关依赖。

对于Maven项目：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web-services</artifactId>
</dependency>
<dependency>
	<groupId>wsdl4j</groupId>
	<artifactId>wsdl4j</artifactId>
</dependency>
```

对于Gradle项目：

```
sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web-services")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    compile("wsdl4j:wsdl4j:1.6.1")
    jaxb("org.glassfish.jaxb:jaxb-xjc:2.2.11")
    compile(files(genJaxb.classesDir).builtBy(genJaxb))
}
```

## 创建XML schema 来定义domain

Web service 的domain定义在一个XML schema 文件(XSD)中，Spring-WS 会基于此文件自动生成一个WSDL文件。

下面来创建XSD文件，在文件中定义国家的**名字(name)、人口(population)、首都（capital）**和**货币（currency）**： 

`src/main/resources/countries.xsd`

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:tns="http://spring.io/guides/gs-producing-web-service"
           targetNamespace="http://spring.io/guides/gs-producing-web-service" elementFormDefault="qualified">

    <xs:element name="getCountryRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="name" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="getCountryResponse">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="country" type="tns:country"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:complexType name="country">
        <xs:sequence>
            <xs:element name="name" type="xs:string"/>
            <xs:element name="population" type="xs:int"/>
            <xs:element name="capital" type="xs:string"/>
            <xs:element name="currency" type="tns:currency"/>
        </xs:sequence>
    </xs:complexType>

    <xs:simpleType name="currency">
        <xs:restriction base="xs:string">
            <xs:enumeration value="GBP"/>
            <xs:enumeration value="EUR"/>
            <xs:enumeration value="PLN"/>
        </xs:restriction>
    </xs:simpleType>
</xs:schema>
```

## 基于XML schema 生成java类文件

下一步是基于上述定义的XSD文件来生成JAVA类文件。正确的做法是使用maven或者gradle插件，在构建项目的时候同时自动生成JAVA类文件。

Maven的插件配置：

```xml
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>jaxb2-maven-plugin</artifactId>
	<version>1.6</version>
	<executions>
		<execution>
			<id>xjc</id>
			<goals>
				<goal>xjc</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<schemaDirectory>${project.basedir}/src/main/resources/</schemaDirectory>
		<outputDirectory>${project.basedir}/src/main/java</outputDirectory>
		<clearOutputDir>false</clearOutputDir>
	</configuration>
</plugin>
```

生成的类文件放置在`target/generated-sources/jaxb/` 路径下

如果想通过gradle实现相同的效果需要在build文件中配置JAXB:

```
configurations {
    jaxb
}

jar {
    baseName = 'gs-producing-web-service'
    version =  '0.1.0'
    from genJaxb.classesDir
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web-services")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    compile("wsdl4j:wsdl4j:1.6.1")
    jaxb("org.glassfish.jaxb:jaxb-xjc:2.2.11")
    compile(files(genJaxb.classesDir).builtBy(genJaxb))
}
```

> 上面的构建文件中如果有`tag`和 `end`注释，在你自己的构建文件中是不需要添加这些注释的。
> 这里引入这些注释只是为了解释得更详细。

下一步，添加Gradle生成java类文件时需要的task  `genJaxb`:

```
task genJaxb {
    ext.sourcesDir = "${buildDir}/generated-sources/jaxb"
    ext.classesDir = "${buildDir}/classes/jaxb"
    ext.schema = "src/main/resources/countries.xsd"

    outputs.dir classesDir

    doLast() {
        project.ant {
            taskdef name: "xjc", classname: "com.sun.tools.xjc.XJCTask",
                    classpath: configurations.jaxb.asPath
            mkdir(dir: sourcesDir)
            mkdir(dir: classesDir)

            xjc(destdir: sourcesDir, schema: schema) {
                arg(value: "-wsdl")
                produces(dir: sourcesDir, includes: "**/*.java")
            }

            javac(destdir: classesDir, source: 1.6, target: 1.6, debug: true,
                    debugLevel: "lines,vars,source",
                    classpath: configurations.jaxb.asPath) {
                src(path: sourcesDir)
                include(name: "**/*.java")
                include(name: "*.java")
            }

            copy(todir: classesDir) {
                fileset(dir: sourcesDir, erroronmissingdir: false) {
                    exclude(name: "**/*.java")
                }
            }
        }
    }
}
```

由于gradle还没有JAXB插件，所以这里包含了一个ant的task，这使得它比在maven中配置要复杂一些。

在上述两种情况下，通过JAXB生成java类文件的过程已经与构建工具的运行周期同步，因此不需要再进行其他的操作就可以自动生成相关的Java文件。

## 创建国家的信息库

在项目构建过程中，需要一个国家信息库向web service服务提供各个国家的数据。
在本指南中，将通过硬编码的方式创建一个虚拟的国家信息库。

```java
package hello;

import javax.annotation.PostConstruct;
import java.util.HashMap;
import java.util.Map;

import io.spring.guides.gs_producing_web_service.Country;
import io.spring.guides.gs_producing_web_service.Currency;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;

@Component
public class CountryRepository {
	private static final Map<String, Country> countries = new HashMap<>();

	@PostConstruct
	public void initData() {
		Country spain = new Country();
		spain.setName("Spain");
		spain.setCapital("Madrid");
		spain.setCurrency(Currency.EUR);
		spain.setPopulation(46704314);

		countries.put(spain.getName(), spain);

		Country poland = new Country();
		poland.setName("Poland");
		poland.setCapital("Warsaw");
		poland.setCurrency(Currency.PLN);
		poland.setPopulation(38186860);

		countries.put(poland.getName(), poland);

		Country uk = new Country();
		uk.setName("United Kingdom");
		uk.setCapital("London");
		uk.setCurrency(Currency.GBP);
		uk.setPopulation(63705000);

		countries.put(uk.getName(), uk);
	}

	public Country findCountry(String name) {
		Assert.notNull(name, "The country's name must not be null");
		return countries.get(name);
	}
}
```

## 创建访问国家信息的服务端点

要创建服务端点，只需要使用几个Spring WS注解创建一个[POJO](https://spring.io/understanding/POJO)来处理传入的SOAP请求。

```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;

import io.spring.guides.gs_producing_web_service.GetCountryRequest;
import io.spring.guides.gs_producing_web_service.GetCountryResponse;

@Endpoint
public class CountryEndpoint {
	private static final String NAMESPACE_URI = "http://spring.io/guides/gs-producing-web-service";

	private CountryRepository countryRepository;

	@Autowired
	public CountryEndpoint(CountryRepository countryRepository) {
		this.countryRepository = countryRepository;
	}

	@PayloadRoot(namespace = NAMESPACE_URI, localPart = "getCountryRequest")
	@ResponsePayload
	public GetCountryResponse getCountry(@RequestPayload GetCountryRequest request) {
		GetCountryResponse response = new GetCountryResponse();
		response.setCountry(countryRepository.findCountry(request.getName()));

		return response;
	}
}
```

[`@Endpoint`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/server/endpoint/annotation/Endpoint.html)  ： 通过 Spring WS 注册该类作为其中一个服务端点，用来处理传入的SOAP消息。

[`@PayloadRoot`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/server/endpoint/annotation/PayloadRoot.html)  ：Spring WS 根据消息的**namespace** 和**localPart**选择需要调用的方法。

[`@RequestPayload`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/server/endpoint/annotation/RequestPayload.html) ： 表示将传入的消息映射到方法的`request`参数上。

[`@ResponsePayload `](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/server/endpoint/annotation/ResponsePayload.html)  ：通知Spring WS 将返回的值映射到reponse的有效负载。

> 代码块中的`io.spring.guides`类可以报告IDE中的编译时错误，除非正在访问WSDL文件生成Java类。

## 配置 web service beans

使用Spring WS的Bean配置创建一个新的Java类：

```java
package hello;

import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.ws.config.annotation.EnableWs;
import org.springframework.ws.config.annotation.WsConfigurerAdapter;
import org.springframework.ws.transport.http.MessageDispatcherServlet;
import org.springframework.ws.wsdl.wsdl11.DefaultWsdl11Definition;
import org.springframework.xml.xsd.SimpleXsdSchema;
import org.springframework.xml.xsd.XsdSchema;

@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {
	@Bean
	public ServletRegistrationBean messageDispatcherServlet(ApplicationContext applicationContext) {
		MessageDispatcherServlet servlet = new MessageDispatcherServlet();
		servlet.setApplicationContext(applicationContext);
		servlet.setTransformWsdlLocations(true);
		return new ServletRegistrationBean(servlet, "/ws/*");
	}

	@Bean(name = "countries")
	public DefaultWsdl11Definition defaultWsdl11Definition(XsdSchema countriesSchema) {
		DefaultWsdl11Definition wsdl11Definition = new DefaultWsdl11Definition();
		wsdl11Definition.setPortTypeName("CountriesPort");
		wsdl11Definition.setLocationUri("/ws");
		wsdl11Definition.setTargetNamespace("http://spring.io/guides/gs-producing-web-service");
		wsdl11Definition.setSchema(countriesSchema);
		return wsdl11Definition;
	}

	@Bean
	public XsdSchema countriesSchema() {
		return new SimpleXsdSchema(new ClassPathResource("countries.xsd"));
	}
}
```

- Spring WS通过特定类型的servlet来处理SOAP消息：[`MessageDispatcherServlet`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/transport/http/MessageDispatcherServlet.html)。在[`MessageDispatcherServlet`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/transport/http/MessageDispatcherServlet.html)中必须设置应用程序的上下文[ApplicationContext](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/context/ApplicationContext.html)，否则Spring WS扫描不到Spring beans。


- 将bean命名为 `messageDispatcherServlet`，这样可以与[Spring Boot的默认DispatcherServlet bean](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/reference/htmlsingle/#howto-switch-off-the-spring-mvc-dispatcherservlet)相区分。


- [DefaultMethodEndpointAdapter](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/server/endpoint/adapter/DefaultMethodEndpointAdapter.html)配置注释驱动的SpringWS编程模型。 这使得前面提到的各种注解都可以使用，如[`@Endpoint`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/server/endpoint/annotation/Endpoint.html)。


- [`DefaultWsdl11Definition`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/wsdl/wsdl11/DefaultWsdl11Definition.html)使[`XsdSchema`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/xml/xsd/XsdSchema.html)发布了一个标准的 WSDL1.1

需要强调的是，[`MessageDispatcherServlet`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/transport/http/MessageDispatcherServlet.html)和[`DefaultWsdl11Definition`](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/wsdl/wsdl11/DefaultWsdl11Definition.html)必须指定bean名称。 这两个bean的名称决定了访问WSDL文件的URL。比如，上述代码生成的WSDL文件位于

`http：//<host>：<port> /ws/countries.wsdl`

此外，上述配置中还开启了WSDL地址转换：`servlet.setTransformWsdlLocations(true)`。 如果访问

`http：// localhost：8080 / ws /countries.wsdl`，`则WSDL文件中的soap:address标签显示的是正确的地址：<soap:address location="http://localhost:8080/ws"/>`。 如果你的计算机IP地址是面向公共的IP地址，则访问WSDL文件时，`soap:address` 显示的是你计算机的地址。

## 使应用程序可执行

虽然可以将此服务打包为传统WAR文件以部署到外部应用程序服务器，但下面将演示一种创建了独立应用程序的简单方法：将所有内容都打包在一个可执行的JAR文件中，通过传统的Java `main()`方法驱动。采用这种方式，可以使用Spring的支持将内嵌的Tomcat servlet容器作为HTTP的运行环境，而不是在Tomcat servlet容器中部署一个外部实例。

`src/main/java/hello/Application.java`

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

`@SpringBootApplication` 是一个方便的注释，它添加以下所有内容：

- @Configuration将该类标记为应用程序上下文的bean定义的源。


- @EnableAutoConfiguration指示Spring Boot根据类路径设置，其他bean和各种属性设置开始添加bean。


-   通常，您将为Spring MVC应用程序添加@EnableWebMvc，但是当SpringBoot在类路径中看到spring-webmvc时，Spring Boot会自动添加它。这将应用程序标记为Web应用程序，并激活诸如设置DispatcherServlet等关键行为。
- @ComponentScan告诉Spring在hello包中查找其他组件，配置和服务，允许它找到控制器。

`main()`方法使用Spring Boot的SpringApplication.run（）方法启动应用程序。 你注意到没有一行XML吗？ 没有**web.xml**文件。 此Web应用程序是100%纯Java，您无需处理配置任何管道或基础设施。

## 构建可执行的JAR

程序创建好以后，可以使用Gradle或Maven从命令行运行。或者，也可以将所有必需的依赖项，类和资源打包成一个可执行的JAR文件，并运行该文件。这种方式使得在整个开发生命周期中，应用程序可以轻松地发布，更新版本和部署服务。

如果你使用的是Gradle，则可以使用`./gradlew bootRun`运行应用程序。或者使用`./gradlew build`来构建JAR文件。然后运行：

```
java -jar build/libs/gs-soap-service-0.1.0.jar
```

如果你使用的是Maven，可以使用./mvnw spring-boot:run运行应用程序，或者使用./mvnw clean package来构建JAR文件。然后运行JAR文件：

```
java -jar target/gs-soap-service-0.1.0.jar
```

> 上面的过程创建了一个可运行的JAR。也可以选择[构建一个经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

上述操作完成后，将会看到有日志信息输出，服务程序将会在几秒内启动并运行。

## 测试应用程序

现在应用程序正在运行，可以进行测试。创建一个包含SOAP请求的文件`request.xml：`

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
				  xmlns:gs="http://spring.io/guides/gs-producing-web-service">
   <soapenv:Header/>
   <soapenv:Body>
      <gs:getCountryRequest>
         <gs:name>Spain</gs:name>
      </gs:getCountryRequest>
   </soapenv:Body>
</soapenv:Envelope>
```

在测试SOAP接口时，可以使用[SoapUI](https://www.soapui.org/)，或者如果是在Unix / Mac系统上，可以使用命令行工具，如下所示：

```
$ curl --header "content-type: text/xml" -d @request.xml http://localhost:8080/ws
```

响应结果如下所示：

```xml
<?xml version="1.0"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">
  <SOAP-ENV:Header/>
  <SOAP-ENV:Body>
    <ns2:getCountryResponse xmlns:ns2="http://spring.io/guides/gs-producing-web-service">
      <ns2:country>
        <ns2:name>Spain</ns2:name>
        <ns2:population>46704314</ns2:population>
        <ns2:capital>Madrid</ns2:capital>
        <ns2:currency>EUR</ns2:currency>
      </ns2:country>
    </ns2:getCountryResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

> 不足的是，输出的是一个紧凑的XML文档，格式不好看。
> 如果你的系统上安装了xmllib2，可以通过命令`curl -s <args above> | xmllint --format –` 生成好看的格式。

## 小结

恭喜！你已经使用Spring Web Services创建了一个基于SOAP的web service 服务。

想要写一个新的指南或者为现有的一个贡献力量吗？请查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

> 所有指南中的代码都将遵照ASLv2许可，指南的编写遵照 [Attribution, NoDerivatives creative commons license](https://creativecommons.org/licenses/by-nd/3.0/) 。
>
> 

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

