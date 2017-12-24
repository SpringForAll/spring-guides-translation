# 使用gemfire缓存数据

> 原文：[Caching Data with GemFire](https://spring.io/guides/gs/caching-gemfire/)
>
> 译者：译者：[UniKrau](https://github.com/UniKrau)
>
> 校对：

本指南阐述了使用Pivotal GemFire的数据网缓存代码需要的调用

## 你将学会构建什么

你将从CloudFoundry持有的服务中构建一个获取请求配额的服务并且能缓存在Pivotal GemFire。

然后，你会明白由于Spring’s Cache Abstraction和Pivotal GemFire备份，再次获取相同配额的时候会节省调用Quote服务的成本。
相同的请求会被放在缓存中。

Quote服务的地址如下

``` 
http://gturnquist-quoters.cfapps.io
```

Quote服务有以下API

```
GET /api         - get all quotes
GET /api/random  - get random quote
GET /api/{id}    - get specific quote
```

## 你需要准备什么

* 大概15分钟
* 文本编辑器或IDE
* [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)以上
* [Gradle 2.3](http://www.gradle.org/downloads)以上 或者 [Maven 3.0](https://maven.apache.org/download.cgi)以上
* 你也可以直接将代码导入到本地开发工具中:
  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)


## 怎样完成本指南
   
像大多数[Spring 入门文章](https://spring.io/guides)一样，你可以逐渐的完成每一步，也可以跳过一些你熟悉的步骤。不管怎样，最后你都将得到一份可执行的代码。

**如果从基础开始**参考[使用Gradle构建](#scratch).

**如果已经熟悉跳过一些基本步骤** 你可以这样：

* 可以通过[下载](https://github.com/spring-guides/gs-caching-gemfire/archive/master.zip) 并解压, 
或者使用[Git](https://spring.io/understanding/Git)克隆: `git clone` [https://github.com/spring-guides/gs-caching-gemfire.git](https://github.com/spring-guides/gs-caching-gemfire.git)

* 跳转到目录`gs-caching-gemfire/initial`

* 跳转到[为获取数据创建一个可以绑定的对象](#initial).

**完成上述步骤**，你可以根据代码检查结果`gs-caching-gemfire/complete`

<h2 id="scratch">使用Gradle构建</h2>
<h2>使用Maven构建</h2>
<h2>特定IDE构建</h2>
<h3 id="initial">为获取数据创建一个可以绑定的对象</h3>

现在，可以配置工程和构建系统，关注定义这些领域对象，它们能从Quote服务中拉取数据中获得字节码

```
src/main/java/hello/Quote.java
```

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

import org.springframework.util.ObjectUtils;

import lombok.Data;

@Data
@JsonIgnoreProperties(ignoreUnknown = true)
@SuppressWarnings("unused")
public class Quote {

	private Long id;

	private String quote;

	@Override
	public boolean equals(Object obj) {

		if (this == obj) {
			return true;
		}

		if (!(obj instanceof Quote)) {
			return false;
		}

		Quote that = (Quote) obj;

		return ObjectUtils.nullSafeEquals(this.getId(), that.getId());
	}

	@Override
	public int hashCode() {
		int hashValue = 17;
		hashValue = 37 * hashValue + ObjectUtils.nullSafeHashCode(getId());
		return hashValue;
	}

	@Override
	public String toString() {
		return getQuote();
	}
}
```

Quote领域类拥有自己id和配额属性。在本指南中将会详细涉猎到这两个主要的属性。[Project Lombok](https://projectlombok.org/)简单实现了Quote类。
Quote还添加了QuoteResponse，QuoteResponse抓取了一个请求中Quote服务中发送的完整的response负载信息。
它包括跟随配额请求的状态（a.k.a 类型）。[Project Lombok](https://projectlombok.org/)含有其简化版的实现。

```
src/main/java/hello/QuoteResponse.java   
```

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonProperty;

import lombok.Data;

@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class QuoteResponse {

	@JsonProperty("value")
	private Quote quote;

	@JsonProperty("type")
	private String status;

	@Override
	public String toString() {
		return String.format("{ @type = %1$s, quote = '%2$s', status = %3$s }",
			getClass().getName(), getQuote(), getStatus());
	}
}
```

以下是Quote服务中的一个典型的响应

```
{
  "type":"success",
  "value": {
    "id":1,
    "quote":"Working with Spring Boot is like pair-programming with the Spring developers."
  }
}
```

两个类都使用了`@JsonIgnoreProperties(ignoreUnknown=true)`标记. 
这意味着即使通过其它JSON属性进行检索，一样会被忽略

### 在Quote服务中查询数据

下一步是创建一个查询配额的服务类

```
src/main/java/hello/QuoteService.java   
```

```java
package hello;

import java.util.Collections;
import java.util.Map;
import java.util.Optional;

import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.web.client.RestTemplate;

@SuppressWarnings("unused")
public class QuoteService {

	protected static final String ID_BASED_QUOTE_SERVICE_URL = "http://gturnquist-quoters.cfapps.io/api/{id}";
	protected static final String RANDOM_QUOTE_SERVICE_URL = "http://gturnquist-quoters.cfapps.io/api/random";

	private volatile boolean cacheMiss = false;

	private final RestTemplate quoteServiceTemplate = new RestTemplate();

	/**
	 * Determines whether the previous service method invocation resulted in a cache miss.
	 *
	 * @return a boolean value indicating whether the previous service method invocation resulted in a cache miss.
	 */
	public boolean isCacheMiss() {
		boolean cacheMiss = this.cacheMiss;
		this.cacheMiss = false;
		return cacheMiss;
	}

	protected void setCacheMiss() {
		this.cacheMiss = true;
	}

	/**
	 * Requests a quote with the given identifier.
	 *
	 * @param id the identifier of the {@link Quote} to request.
	 * @return a {@link Quote} with the given ID.
	 */
	@Cacheable("Quotes")
	public Quote requestQuote(Long id) {
		setCacheMiss();
		return requestQuote(ID_BASED_QUOTE_SERVICE_URL, Collections.singletonMap("id", id));
	}

	/**
	 * Requests a random quote.
	 *
	 * @return a random {@link Quote}.
	 */
	@CachePut(cacheNames = "Quotes", key = "#result.id")
	public Quote requestRandomQuote() {
		setCacheMiss();
		return requestQuote(RANDOM_QUOTE_SERVICE_URL);
	}

	protected Quote requestQuote(String URL) {
		return requestQuote(URL, Collections.emptyMap());
	}

	protected Quote requestQuote(String URL, Map<String, Object> urlVariables) {

		return Optional.ofNullable(this.quoteServiceTemplate.getForObject(URL, QuoteResponse.class, urlVariables))
			.map(QuoteResponse::getQuote)
			.orElse(null);
	}
}
```

这个`QuoteService`使用Spring’s`RestTemplate`查询Quote服务的[API](https://gturnquist-quoters.cfapps.io/api)Quote
服务返回一个JSON对象。但是Spring使用Jackson绑定数据到`QuoteResponse`，最终作为`Quote`对象。
 
这个服务类的精妙之处是`requestQuote`一直能够使用`@Cacheable("Quotes")`注解。[Spring’s Caching Abstraction](https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#cache)
拦截调用然后专递给`requestQuote`去检查服务方法是否已经被调用了。如果是这样，Spring’s Caching Abstraction这是返回了缓存拷贝。否则
Spring会处理这个方法调用，并且存储这个响应在缓存中，然后返回结果给调用者。


当然，在`requestRandomQuote`服务方法中还使用了`@CachePut`注解。因为从服务的调用方法中返回的配额是随机的，不知道具体收到的配额。
所以，对于一个调用请求来讲，无法其干涉缓存（ `Quotes`）的优先级，但是可以缓存调用的结果，假设配额是随机选择的和已经缓存了。然后在随后的`requestQuote(id)`的调用上添加一个
positive效果，`@CachePut`使用了SpEL表达式（“#result.id”）访问服务方法调用的结构和检索`Quote`的ID并且作为缓存键。
更多Spring’s Cache Abstraction SpEL的内容[请点击](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#cache-spel-context).


> 缓存必须要有名字。
使用"Quotes"只是作为演示的目的，但是在生产环境中，推荐使用一个恰当可描述的名字。
换句话说，不同的方法对应不同的缓存名。特别是在不同的缓存配置文件中。比如不同的过期策略，等待
 

运行代码的时候，每一个调用都会显示消耗的时间，能够了解缓存在服务响应时间的作用。
这只是演示缓存特定调用的意义。
如果应用程序持续查询相同的数据，缓存的结果能非常明显的够提高性能

### 使应用程序可执行
    
虽然Pivotal GemFire缓存能嵌入在web apps和 WAR文件中。以下是简单的演示了创建一个标准的范例
将所有的都打包在单个，可执行的JAR文件中，入口在`main()`方法。

```
src/main/java/hello/Application.java
```

```java
package hello;

import java.util.Optional;

import org.apache.geode.cache.GemFireCache;
import org.apache.geode.cache.client.ClientRegionShortcut;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.gemfire.cache.config.EnableGemfireCaching;
import org.springframework.data.gemfire.client.ClientRegionFactoryBean;
import org.springframework.data.gemfire.config.annotation.ClientCacheApplication;

@ClientCacheApplication(name = "CachingGemFireApplication", logLevel = "error")
@EnableGemfireCaching
@SuppressWarnings("unused")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean("Quotes")
    public ClientRegionFactoryBean<Integer, Integer> quotesRegion(GemFireCache gemfireCache) {

        ClientRegionFactoryBean<Integer, Integer> quotesRegion = new ClientRegionFactoryBean<>();

        quotesRegion.setCache(gemfireCache);
        quotesRegion.setClose(false);
        quotesRegion.setShortcut(ClientRegionShortcut.LOCAL);

        return quotesRegion;
    }

    @Bean
    QuoteService quoteService() {
        return new QuoteService();
    }

    @Bean
    ApplicationRunner runner() {

        return args -> {
            Quote quote = requestQuote(12L);
            requestQuote(quote.getId());
            requestQuote(10L);
        };
    }

    private Quote requestQuote(Long id) {

        QuoteService quoteService = quoteService();

        long startTime = System.currentTimeMillis();

        Quote quote = Optional.ofNullable(id)
            .map(quoteService::requestQuote)
            .orElseGet(quoteService::requestRandomQuote);

        long elapsedTime = System.currentTimeMillis();

        System.out.printf("\"%1$s\"%nCache Miss [%2$s] - Elapsed Time [%3$s ms]%n", quote,
            quoteService.isCacheMiss(), (elapsedTime - startTime));

        return quote;
    }
}
```

`@SpringBootApplication` 是非常方便的注解，添加以下所有的内容：

* 应用程序的上下文中`@Configuration`标记这个类作为bean定义的出始处

* `@EnableAutoConfiguration`告知Spring Boot根据类路径，其它beans和属性配置进行添加beans

* Spring MVC app，一般情况下需要添加`@EnableWebMvc`，但是类路径中含有spring-webmvc的时候，Spring Boot能够自动添加这个app，标记这个app是web app和会激活关键的行为，比如设置`DispatcherServlet`

* `@ComponentScan` 告知Spring在`hello` package寻找其它的组件，配置文件，服务,允许app找到controller。


这个`main()`使用Spring Boot’s `SpringApplication.run()`方法启动app. 注意到XML文件内容不是单行配置，同时也没有web.xml文件
这个web app是纯java并且无需配置必要配置或者基础配置

在配置顶端是单个，重要的注解：`@EnableGemfireCaching`。这个启用了缓存（比如，元注解使用Spring’s `@EnableCaching` ）并且声明了额外重要的beans在后台。
使用Pivotal GemFire作为缓存提供器支持缓存

第一bean是`QuoteService`的实例，用来访问Quotes REST-ful Web服务

其它两个beans用作缓存Quote和执行app的动作

* quotesRegion 定义了一个GemFire`LOCAL`区在缓存中存储quotes.在`QuoteService`方法中它使用"Quotes"匹配`@Cacheable("Quotes")`.

* `runner` 是Spring Boot的实例，使用`ApplicationRunner`接口运行app.

The first time a quote is requested (using ), 
第一次的时候，发送一个（`requestQuote(id)`）quote请求，缓存不命中，转而调用service方法。伴随一个明显的延迟。
这种情况下，缓存会被service的`requestQuote`方法输入的参数（`id`）链接，换句话说，`id`的方法参数是这个缓存键。
接下来由ID识别对于相同的请求，，结果是缓存命中，因此避免了高成本的服务调用。

出于演示的目的，调用的`QuoteService`包装在一个单独方法里（`requestQuote`方法内置在`Application`类中）以便计算服务调用时间。


### 编译一个可执行的JAR
     
或者编译成单个的可执行JAR文件，当然这个JAR文件包含了所有的依赖、类、资源文件。这使得jar文件作为app比较容易装载，发布版本，部署服务通过开发周期，在不同的环境下，等等


如果使用Gradle，使用`./gradlew bootRun`运行app，或者`./gradlew build`编译JAR文件。然后使用以下命令运行JAR文件

```bash
java -jar build/libs/gs-caching-gemfire-0.1.0.jar
```

如果使用Maven，则可以使用`./mvnw spring-boot:run`命令
或者使用`./mvnw clean package`编译JAR文件。然后运行JAR文件

```bash
java -jar target/gs-caching-gemfire-0.1.0.jar
```


> 以上过程将会产生一个可以运行的JAR。当然也可以选择这种方式[build a classic WAR file](https://spring.io/guides/gs/convert-jar-to-war/) 

以下是输出的日志，服务很快会启动和运行。

```
"@springboot with @springframework is pure productivity! Who said in #java one has to write double the code than in other langs? #newFavLib"
Cache Miss [true] - Elapsed Time [776 ms]
"@springboot with @springframework is pure productivity! Who said in #java one has to write double the code than in other langs? #newFavLib"
Cache Miss [false] - Elapsed Time [0 ms]
"Really loving Spring Boot, makes stand alone Spring apps easy."
Cache Miss [true] - Elapsed Time [96 ms]
```


首次调用Quote服务花费了776ms并且发生了缓存命中失败。然而，第二次毫不费时地直接缓存命中。很明显第二次调用被缓存了而且没有调用Quote服务。
不管怎样，最终服务调用一个特殊，没有缓存的quote请求的，同样花费了96ms，结果显示缓存命中失败，因为这是一个新的quote。

## Summary

恭喜，你已经学会构建一个执行和标记高成本的操作，并且能缓存结果的服务

## 其它

以下指南也许有用:

* [Caching Data with Spring](https://spring.io/guides/gs/caching/)
* [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



