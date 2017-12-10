# 消费SOAP web服务

> 原文：[Consuming a SOAP web service](https://spring.io/guides/gs/consuming-web-service/)
>
> 译者：[dongyang1993](https://github.com/dongyang1993)
>
> 校对：[feilangrenM](https://github.com/feilangrenM)

本指南将引导你创建一个消费基于Spring的[SOAP web service](https://spring.io/understanding/SOAP)服务的应用程序。

## 你会构建什么

您将构建一个基于WSDL的web服务并使用SOAP从远程获取天气数据的客户端。你可以在这里找到关于这个被引用服务的更多信息[ http://www.webservicex.com/stockquote.asmx?op=GetQuote.]( http://www.webservicex.com/stockquote.asmx?op=GetQuote.)

该服务提供过往的行情引用。 您可以使用自己的ticker符号。

## 你需要什么

* 大约15分钟
* 一个喜欢的文本编辑器或者IDE
* [JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或者更新的版本
* [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
* 你也可以直接将这些代码导入到你的IDE里面：
    * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
    * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 怎么完成这个指南

与大多数Spring[入门指南](https://spring.io/guides)一样，您可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。 无论哪种方式，你最后都将完成代码。

从头开始，进入到[用Gradle构建](https://spring.io/guides/gs/consuming-web-service/#scratch)。

跳过基本过程，请按照以下步骤执行：

* [下载](https://github.com/spring-guides/gs-consuming-web-service/archive/master.zip)并解压这个指南的资源库，或者用[Git](https://spring.io/understanding/Git)克隆一份:
```
   git cline [https://github.com/spring-guides/gs-consuming-web-service.git](https://github.com/spring-guides/gs-consuming-web-service.git)
```

* cd into `gs-consuming-web-service/initial`

* [基于WSDL生成域对象](https://spring.io/guides/gs/consuming-web-service/#initial)。

完成后，您可以根据`gs-consumption-web-service/complete`中的代码检查结果。

### 使用Gradle构建

首先你设置一个基本的构建脚本。 在使用Spring构建应用程序时，您可以使用任何您喜欢的构建系统，不过你用[Gradle](http://gradle.org/)或[Maven](https://maven.apache.org/)构建的代码包括在这里。 如果您还不熟悉，请参阅[使用Gradle构建Java项目](http://spring.io/guides/gs/gradle)或[使用Maven构建Java项目](http://spring.io/guides/gs/maven)。

#### 创建目录结构

在您选择的项目目录中，创建以下子目录结构; 例如，在 * nix系统中使用`mkdir -p src/main/java/hello`：
```
└── src
    └── main
        └── java
            └── hello
```

#### 创建一个Gradle构建文件

以下是[Gradle初始化构建文件](https://github.com/spring-guides/gs-consuming-web-service/blob/master/initial/build.gradle)。

`build.gradle`
```
configurations {
    jaxb
}

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

repositories {
    mavenCentral()
}

// tag::wsdl[]
task genJaxb {
    ext.sourcesDir = "${buildDir}/generated-sources/jaxb"
    ext.classesDir = "${buildDir}/classes/jaxb"
    ext.schema = "http://www.webservicex.com/stockquote.asmx?WSDL"

    outputs.dir classesDir

    doLast() {
        project.ant {
            taskdef name: "xjc", classname: "com.sun.tools.xjc.XJCTask",
                    classpath: configurations.jaxb.asPath
            mkdir(dir: sourcesDir)
            mkdir(dir: classesDir)

            xjc(destdir: sourcesDir, schema: schema,
                    package: "hello.wsdl") {
                arg(value: "-wsdl")
                produces(dir: sourcesDir, includes: "**/*.java")
            }

            javac(destdir: classesDir, source: 1.8, target: 1.8, debug: true,
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
// end::wsdl[]

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter")
    compile("org.springframework.ws:spring-ws-core")
    compile(files(genJaxb.classesDir).builtBy(genJaxb))

    jaxb "com.sun.xml.bind:jaxb-xjc:2.1.7"
}

jar {
    baseName = 'gs-consuming-web-service'
    version =  '0.1.0'

    from genJaxb.classesDir
}


task afterEclipseImport {
    dependsOn genJaxb
}
```

[Spring Boot gradle插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了许多方方便的功能：
+ 收集classpath下所有的jar包，并且构建成一个可执行的更便于执行和传输服务的"über-jar"。
+ 搜索`public static void main()`方法并标记为可运行类。
+ 提供内置依赖解析器，设置版本号匹配[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)。它默认的版本是Boot's选用的版本，不过你可以重写成你想要的任何版本。

### 使用Maven构建

首先你设置一个基本的构建脚本。在使用Spring构建应用程序时，您可以使用任何构建系统，不过你用[Maven](https://maven.apache.org/)构建的代码包括在了这里。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](http://spring.io/guides/gs/maven)。

#### 创建目录结构

在你选择的项目目录中，创建以下子目录，例如，在 * nix系统中使用`mkdir -p src/main/java/hello`:
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
    <artifactId>gs-consuming-web-service</artifactId>
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
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
           <dependency>
               <groupId>org.springframework.ws</groupId>
               <artifactId>spring-ws-core</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- tag::wsdl[] -->
            <plugin>
                <groupId>org.jvnet.jaxb2.maven2</groupId>
                <artifactId>maven-jaxb2-plugin</artifactId>
                <version>0.12.3</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <schemaLanguage>WSDL</schemaLanguage>
                    <generatePackage>hello.wsdl</generatePackage>
                    <schemas>
                        <schema>
                            <url>http://www.webservicex.com/stockquote.asmx?WSDL</url>
                        </schema>
                    </schemas>
                </configuration>
            </plugin>
            <!-- end::wsdl[] -->
        </plugins>
    </build>

</project>
```

[Spring Boot Maven插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin)提供了许多方便的功能：
+ 收集classpath下所有的jar包，并且构建成一个可执行的更便于执行和传输服务的"über-jar"。
+ 搜索`public static void main()`方法并标记为可运行类。
+ 提供内置依赖解析器，设置版本号匹配[Spring Boot依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)。它默认的版本是Boot's选用的版本，不过你可以重写成你想要的任何版本。

### 使用你的IDE构建

+ 阅读如何将本指南直接导入到[Spring Tool Suite](http://spring.io/guides/gs/sts/)中。
+ 阅读如何z在IntelliJ IDEA中使用本指南。
>如果你阅读了[生产SOAP web service](http://spring.io/guides/gs/producing-web-service)，你可能会很想知道为什么本指南不使用 **spring-boot-starter-ws**?因为Spring Boot starter只适用于服务端的web service。这个启动器带有嵌入式的Tomcat，而不需要再进行网络通话。

### 基于WSDL生成域对象

SOAP web service服务的接口可以在[WSDL](https://en.wikipedia.org/wiki/Web_Services_Description_Language)捕获到。 JAXB提供了一种从WSDL生成Java类的简单方法（或者是：XSD包含在WSDL文件中的`<Types />`标签）。 所引用服务的 WSDL文件可以通过http://www.webservicex.com/stockquote.asmx?WSDL访问。

用maven通过WSDL生成Java类，需要安装以下插件：

```
<plugin>
    <groupId>org.jvnet.jaxb2.maven2</groupId>
    <artifactId>maven-jaxb2-plugin</artifactId>
    <version>0.13.1</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <schemaLanguage>WSDL</schemaLanguage>
        <generatePackage>hello.wsdl</generatePackage>
        <schemas>
            <schema>
                <url>http://www.webservicex.com/stockquote.asmx?WSDL</url>
            </schema>
        </schemas>
    </configuration>
</plugin>
```

上述设置会根据特定URL下的WSDL文件生成Java类，并将这些类文件放在`hello.wsdl`包中。

如果要使用gradle做同样的事情，您的构建文件中将需要以下内容：
```
task genJaxb {
    ext.sourcesDir = "${buildDir}/generated-sources/jaxb"
    ext.classesDir = "${buildDir}/classes/jaxb"
    ext.schema = "http://www.webservicex.com/stockquote.asmx?WSDL"

    outputs.dir classesDir

    doLast() {
        project.ant {
            taskdef name: "xjc", classname: "com.sun.tools.xjc.XJCTask",
                    classpath: configurations.jaxb.asPath
            mkdir(dir: sourcesDir)
            mkdir(dir: classesDir)

            xjc(destdir: sourcesDir, schema: schema,
                    package: "hello.wsdl") {
                arg(value: "-wsdl")
                produces(dir: sourcesDir, includes: "**/*.java")
            }

            javac(destdir: classesDir, source: 1.8, target: 1.8, debug: true,
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

由于gradle没有JAXB插件，它涉及到一个ant任务，这使得它比在maven中复杂一些。

在这两种情况下，JAXB域对象生成过程已经连接到构建工具的生命周期中，因此没有额外的步骤可以运行。

### 创建一个天气服务客户端

要创建一个Web服务客户端，你只需要继承[WebServiceGatewaySupport](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/client/core/support/WebServiceGatewaySupport.html)类并实现相关操作代码：

`src/main/java/hello/QuoteClient.java`

```
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.ws.client.core.support.WebServiceGatewaySupport;
import org.springframework.ws.soap.client.core.SoapActionCallback;

import hello.wsdl.GetQuote;
import hello.wsdl.GetQuoteResponse;

public class QuoteClient extends WebServiceGatewaySupport {

	private static final Logger log = LoggerFactory.getLogger(QuoteClient.class);

	public GetQuoteResponse getQuote(String ticker) {

		GetQuote request = new GetQuote();
		request.setSymbol(ticker);

		log.info("Requesting quote for " + ticker);

		GetQuoteResponse response = (GetQuoteResponse) getWebServiceTemplate()
				.marshalSendAndReceive("http://www.webservicex.com/stockquote.asmx",
						request,
						new SoapActionCallback("http://www.webserviceX.NET/GetQuote"));

		return response;
	}

}
```

客户端包含一个`getQuote`方法，该方法通过SOAP协议进行数据交换。

在上述方法中，`GetQuote`类和`GetQuoteResponse`类都是由前述的JABX通过WSDL文件生成出来的。
它创建`GetQuote`请求对象 建议改成： 它创建GetQuoterequest对象
打印出代码之后 建议改成 打印出tricker以后
SOAP交换 建议改成 SOAP数据交换
它将GetQuoterequest对象和一个SoapActionCallback对象放入SOAPAction请求头头，因为在WSDL文件soap:operation/的节点描述中需要这中类型的请求头。最后，将response转换成一个GetQuoteResponse对象，然后返回该对象。

在上述方法中，`GetQuote`类和`GetQuoteResponse`类都是由前述的JABX通过WSDL文件生成出来的。。 它创建`GetQuote`request对象，并使用`ticker`参数进行设置。 打印出tricker之后，它使用`WebServiceGatewaySupport`基类提供的[WebServiceTemplate](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/client/core/WebServiceTemplate.html)进行实际的SOAP数据交换。 它将`GetQuote`request对象和一个`SoapActionCallback`对象放入[SOAPAction](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383528)请求头，因为WSDL文件`<soap：operation />`的节点描述中需要这中类型的请求头。最后，将response转换成一个`GetQuoteResponse`对象，然后返回该对象。

### 配置Web服务组件

Spring WS使用Spring Framework的OXM模块的`Jaxb2Marshaller`来对XML请求进行序列化和反序列化。

`src/main/java/hello/QuoteConfiguration.java`

```
package hello;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;

@Configuration
public class QuoteConfiguration {

	@Bean
	public Jaxb2Marshaller marshaller() {
		Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
		// this package must match the package in the <generatePackage> specified in
		// pom.xml
		marshaller.setContextPath("hello.wsdl");
		return marshaller;
	}

	@Bean
	public QuoteClient quoteClient(Jaxb2Marshaller marshaller) {
		QuoteClient client = new QuoteClient();
		client.setDefaultUri("http://www.webservicex.com/stockquote.asmx");
		client.setMarshaller(marshaller);
		client.setUnmarshaller(marshaller);
		return client;
	}

}
```

`marshaller`指向生成的域对象的集合，并将使用它们在XML和[POJOs](https://spring.io/understanding/POJO)之间进行序列化和反序列化。

`quoteClient`被创建并配置了上面显示的天气服务的URI。 它也被配置为使用JAXB编组器。

### 使应用程序可执行

该应用程序被打包，从控制台运行，并为给定的邮政编码检索单个天气预报。

`src/main/java/hello/Application.java`

```
package hello;

import hello.wsdl.GetQuoteResponse;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class);
	}

	@Bean
	CommandLineRunner lookup(QuoteClient quoteClient) {
		return args -> {
			String ticker = "MSFT";

			if (args.length > 0) {
				ticker = args[0];
			}
			GetQuoteResponse response = quoteClient.getQuote(ticker);
			System.err.println(response.getGetQuoteResult());
		};
	}

}
```

`main()`方法参照[SpringApplication](https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/api/org/springframework/boot/SpringApplication.html)帮助类的做法，提供`QuoteConfiguration.class`作为其`run()`方法的参数。 这告诉Spring从`QuoteConfiguration`读取注释元数据，并将其作为[Spring应用程序上下文](https://spring.io/understanding/application-context)中的组件进行管理。

>此应用程序是硬编码查找邮政编码94304，帕洛阿尔托，CA. 在本指南的最后，您将看到如何插入不同的邮政编码，而无需编辑代码。

### 构建可执行的JAR

您可以从命令行运行Gradle或Maven应用程序。 或者，您可以构建一个包含所有必需依赖项，类和资源的单个可执行JAR文件，并运行该文件。 这使得在整个开发生命周期中，跨不同的环境传输，版本控制并且部署服务变得简单。

如果您使用的是Gradle，则可以使用`./gradlew bootRun`运行应用程序。 或者您可以使用`./gradlew build`来构建JAR文件。 然后可以运行JAR文件：

`java -jar build/libs/gs-consuming-web-service-0.1.0.jar`

如果使用Maven，可以使用`./mvnw spring-boot:run`运行应用程序。 或者您可以使用`./mvnw clean package`来构建JAR文件。 然后可以运行JAR文件：

`java -jar target/gs-consuming-web-service-0.1.0.jar`

>上面的过程将创建一个可运行的JAR。 您也可以选择构建一个[经典的WAR文件](https://spring.io/guides/gs/convert-jar-to-war/)。

显示记录输出。 该服务应该在几秒钟内启动并运行。

```
Requesting quote for MSFT

<StockQuotes><Stock><Symbol>MSFT</Symbol><Last>62.70</Last>...</StockQuotes>
```

您可以通过键入插入不同的代码

`java -jar build/libs/gs-consuming-web-service-0.1.0.jar ORCL`

```
Requesting quote for ORCL

<StockQuotes><Stock><Symbol>ORCL</Symbol><Last>39.26</Last>...</StockQuotes>
```

### 总结

恭喜！ 您刚刚使用Spring开发了一个客户端，用来消费基于SOAP的web service服务。

### 请参阅

* [生成SOAP Web服务](https://spring.io/guides/gs/producing-web-service/)

* [用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)

>本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可 协议进行许可](http://creativecommons.org/licenses/by-nc-sa/4.0/%29%E5%8D%8F%E8%AE%AE%E8%BF%9B%E8%A1%8C%E8%AE%B8%E5%8F%AF%E3%80%82)。
