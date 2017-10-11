# 中文标题[dongyang1993翻译中]

> 原文：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) （这里为示例，译者需根据具体文章修改）
>
> 译者：dongyang1993
>
> 校对：
## 使用SOAP服务

本指南将引导你完成基于Spring的[SOAP服务](https://spring.io/understanding/SOAP)操作步骤。

## 你会构建什么

您将构建一个基于WSDL的web服务并使用SOAP从远程获取天气数据的客户端。你可以在这里找到更多信息关于这个被引用的服务[ http://www.webservicex.com/stockquote.asmx?op=GetQuote.]( http://www.webservicex.com/stockquote.asmx?op=GetQuote.)

该服务提供股票市场报价。 您将能够使用自己的股票代码。

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

从头开始，进入到[用Gradle构建]()。

跳过基础，完成下面的步骤：

    [下载](https://github.com/spring-guides/gs-consuming-web-service/archive/master.zip)并解压这个指南的资源库，或者用Git克隆一份:
```
   git cline [https://github.com/spring-guides/gs-consuming-web-service.git](https://github.com/spring-guides/gs-consuming-web-service.git)
```

    cd into `gs-consuming-web-service/initial`

    [基于WSDL生成领域对象](https://spring.io/guides/gs/consuming-web-service/#initial)。

完成后，您可以根据`gs-consumption-web-service/complete`中的代码检查结果。

### 使用Gradle构建

### 使用Maven构建

### 使用你的IDE构建

### 基于WSDL生成领域对象

SOAP Web服务的接口在[WSDL](https://en.wikipedia.org/wiki/Web_Services_Description_Language)中被捕获。 JAXB提供了一种从WSDL生成Java类的简单方法（或者是：WSDL中`<Types />`部分中包含的XSD）。 WSDL中的报价服务可以在http://www.webservicex.com/stockquote.asmx?WSDL找到。

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

此安装程序将为指定URL中找到的WSDL生成类，将这些类放在`hello.wsdl`包中。

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

要创建一个Web服务客户端，你只需要扩展[WebServiceGatewaySupport](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/client/core/support/WebServiceGatewaySupport.html)类并对你的操作进行编码：

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

客户端包含一个方法：`getQuote`，它执行实际的SOAP交换。

在该方法中，`GetQuote`和`GetQuoteResponse`类都是从WSDL派生的，并且是在上一步中描述的JAXB生成过程中生成的。 它创建`GetQuote`请求对象，并使用`ticker`参数进行设置。 打印出代码之后，它使用`WebServiceGatewaySupport`基类提供的[WebServiceTemplate](https://docs.spring.io/spring-ws/sites/2.0/apidocs/org/springframework/ws/client/core/WebServiceTemplate.html)进行实际的SOAP交换。 它传递`GetQuote`请求对象以及`SoapActionCallback`以传递带有请求头[SOAPAction](https://www.w3.org/TR/2000/NOTE-SOAP-20000508/#_Toc478383528)，因为WSDL描述了它需要`<soap：operation />`元素中的头。 它将响应转换为`GetQuoteResponse`对象，然后返回。

### 配置Web服务组件

Spring WS使用Spring Framework的OXM模块，它有`Jaxb2Marshaller`来对XML请求进行序列化和反序列化。

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

`marshaller`指向生成的领域对象的集合，并将使用它们在XML和[POJOs](https://spring.io/understanding/POJO)之间进行序列化和反序列化。

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

`main()`方法会推迟到SpringApplication帮助类，提供`QuoteConfiguration.class`作为其`run()`方法的参数。 这告诉Spring从`QuoteConfiguration`读取注释元数据，并将其作为[Spring应用程序上下文](https://spring.io/understanding/application-context)中的组件进行管理。

>此应用程序是硬编码查找邮政编码94304，帕洛阿尔托，CA. 在本指南的最后，您将看到如何插入不同的邮政编码，而无需编辑代码。

构建可执行的JAR

您可以从命令行运行应用程序与Gradle或Maven。 或者，您可以构建一个包含所有必需依赖项，类和资源的单个可执行JAR文件，并运行该文件。 这使得在整个开发生命周期中，跨不同的环境等运输，版本和部署服务成为一个应用程序。

如果您使用的是Gradle，则可以使用./gradlew bootRun运行应用程序。 或者您可以使用./gradlew构建来构建JAR文件。 然后可以运行JAR文件：

`java -jar build/libs/gs-consuming-web-service-0.1.0.jar`

如果使用Maven，可以使用./mvnw spring-boot运行应用程序：run。 或者您可以使用./mvnw clean包来构建JAR文件。 然后可以运行JAR文件：

`java -jar target/gs-consuming-web-service-0.1.0.jar`

>上面的过程将创建一个可运行的JAR。 您也可以选择构建一个经典的WAR文件。

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

恭喜！ 您刚刚开发了一个客户端，使用Spring来使用基于SOAP的Web服务。

### 请参阅

* [生成SOAP Web服务](https://spring.io/guides/gs/producing-web-service/)

* [用Spring Boot构建应用程序](https://spring.io/guides/gs/spring-boot/)

想写一个新的指南或贡献一个现有的？ 查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

>所有指南都将发布ASLv2许可证的代码，以及[署名，没有衍生性](https://creativecommons.org/licenses/by-nd/3.0/)的写作许可证。

>本文由spring4all.com翻译小分队创作，采用知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可 协议进行许可。



