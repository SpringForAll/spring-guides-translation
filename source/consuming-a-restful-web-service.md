【hapihapidoge翻译中】

> 原文：[Consuming a RESTful Web Service](https://spring.io/guides/gs/consuming-rest/)
>
> 译者：[hapihapidoge](https://github.com/hapihapidoge)
>
> 校对：

# Consuming a RESTful Web Service

This guide walks you through the process of creating an application that consumes a RESTful web service.

## What you’ll build

You’ll build an application that uses Spring’s `RestTemplate` to retrieve a random Spring Boot quotation at [http://gturnquist-quoters.cfapps.io/api/random](https://gturnquist-quoters.cfapps.io/api/random).

## What you’ll need

- About 15 minutes
- A favorite text editor or IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
- [Gradle 2.3+](http://www.gradle.org/downloads) or [Maven 3.0+](https://maven.apache.org/download.cgi)
- You can also import the code straight into your IDE:
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## How to complete this guide

Like most Spring [Getting Started guides](https://spring.io/guides), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Build with Gradle](https://spring.io/guides/gs/consuming-rest/#scratch).

To **skip the basics**, do the following:

- [Download](https://github.com/spring-guides/gs-consuming-rest/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone https://github.com/spring-guides/gs-consuming-rest.git`
- cd into `gs-consuming-rest/initial`
- Jump ahead to [Fetch a REST resource](https://spring.io/guides/gs/consuming-rest/#initial).

**When you’re finished**, you can check your results against the code in `gs-consuming-rest/complete`.

## Build with Gradle

## Build with Maven

## Build with your IDE

## Fetch a REST resource

With project setup complete, you can create a simple application that consumes a RESTful service.

A RESTful service has been stood up at [http://gturnquist-quoters.cfapps.io/api/random](https://gturnquist-quoters.cfapps.io/api/random). It randomly fetches quotes about Spring Boot and returns them as a JSON document.

If you request that URL through your web browser or curl, you’ll receive a JSON document that looks something like this:

```json
{
   type: "success",
   value: {
      id: 10,
      quote: "Really loving Spring Boot, makes stand alone Spring apps easy."
   }
}
```

Easy enough, but not terribly useful when fetched through a browser or through curl.

A more useful way to consume a REST web service is programmatically. To help you with that task, Spring provides a convenient template class called [`RestTemplate`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html). `RestTemplate`makes interacting with most RESTful services a one-line incantation. And it can even bind that data to custom domain types.

First, create a domain class to contain the data that you need.

`src/main/java/hello/Quote.java`

```Java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

    private String type;
    private Value value;

    public Quote() {
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public Value getValue() {
        return value;
    }

    public void setValue(Value value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Quote{" +
                "type='" + type + '\'' +
                ", value=" + value +
                '}';
    }
}
```

As you can see, this is a simple Java class with a handful of properties and matching getter methods. It’s annotated with `@JsonIgnoreProperties` from the Jackson JSON processing library to indicate that any properties not bound in this type should be ignored.

In order for you to directly bind your data to your custom types, you need to specify the variable name exact same as the key in the JSON Document returned from the API. In case your variable name and key in JSON doc are not matching, you need to use `@JsonProperty`annotation to specify the exact key of JSON document.

An additional class is needed to embed the inner quotation itself.

`src/main/java/hello/Value.java`

```java
package hello;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

    private Long id;
    private String quote;

    public Value() {
    }

    public Long getId() {
        return this.id;
    }

    public String getQuote() {
        return this.quote;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public void setQuote(String quote) {
        this.quote = quote;
    }

    @Override
    public String toString() {
        return "Value{" +
                "id=" + id +
                ", quote='" + quote + '\'' +
                '}';
    }
}
```

This uses the same annotations but simply maps onto other data fields.

## Make the application executable

Although it is possible to package this service as a traditional [WAR](https://spring.io/understanding/WAR) file for deployment to an external application server, the simpler approach demonstrated below creates a standalone application. You package everything in a single, executable JAR file, driven by a good old Java `main()` method. Along the way, you use Spring’s support for embedding the [Tomcat](https://spring.io/understanding/Tomcat) servlet container as the HTTP runtime, instead of deploying to an external instance.

Now you can write the `Application` class that uses `RestTemplate` to fetch the data from our Spring Boot quotation service.

`src/main/java/hello/Application.java`

```Java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.client.RestTemplate;

public class Application {

    private static final Logger log = LoggerFactory.getLogger(Application.class);

    public static void main(String args[]) {
        RestTemplate restTemplate = new RestTemplate();
        Quote quote = restTemplate.getForObject("http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
        log.info(quote.toString());
    }

}
```

Because the Jackson JSON processing library is in the classpath, `RestTemplate` will use it (via a [message converter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html)) to convert the incoming JSON data into a `Quote` object. From there, the contents of the `Quote` object will be logged to the console.

Here you’ve only used `RestTemplate` to make an HTTP `GET` request. But `RestTemplate`also supports other HTTP verbs such as `POST`, `PUT`, and `DELETE`.

## Managing the Application Lifecycle with Spring Boot

So far we haven’t used Spring Boot in our application, but there are some advantages in doing so, and it isn’t hard to do. One of the advantages is that we might want to let Spring Boot manage the message converters in the `RestTemplate`, so that customizations are easy to add declaratively. To do that we use `@SpringBootApplication` on the main class and convert the main method to start it up, like in any Spring Boot application. Finally we move the `RestTemplate` to a `CommandLineRunner` callback so it is executed by Spring Boot on startup:

`src/main/java/hello/Application.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class Application {

	private static final Logger log = LoggerFactory.getLogger(Application.class);

	public static void main(String args[]) {
		SpringApplication.run(Application.class);
	}

	@Bean
	public RestTemplate restTemplate(RestTemplateBuilder builder) {
		return builder.build();
	}

	@Bean
	public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
		return args -> {
			Quote quote = restTemplate.getForObject(
					"http://gturnquist-quoters.cfapps.io/api/random", Quote.class);
			log.info(quote.toString());
		};
	}
}
```

The `RestTemplateBuilder` is injected by Spring, and if you use it to create a `RestTemplate`then you will benefit from all the autoconfiguration that happens in Spring Boot with message converters and request factories. We also extract the `RestTemplate` into a `@Bean` to make it easier to test (it can be mocked more easily that way).

### Build an executable JAR

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using `./gradlew bootRun`. Or you can build the JAR file using `./gradlew build`. Then you can run the JAR file:

```shell
java -jar build/libs/gs-consuming-rest-0.1.0.jar
```

If you are using Maven, you can run the application using `./mvnw spring-boot:run`. Or you can build the JAR file with `./mvnw clean package`. Then you can run the JAR file:

```shell
java -jar target/gs-consuming-rest-0.1.0.jar
```

| **   | The procedure above will create a runnable JAR. You can also opt to [build a classic WAR file](https://spring.io/guides/gs/convert-jar-to-war/)instead. |      |      |      |      |
| ---- | ---------------------------------------- | ---- | ---- | ---- | ---- |
|      |                                          |      |      |      |      |

You should see output like the following, with a random quote:

```java
2015-09-23 14:22:26.415  INFO 23613 --- [main] hello.Application  : Quote{type='success', value=Value{id=12, quote='@springboot with @springframework is pure productivity! Who said in #java one has to write double the code than in other langs? #newFavLib'}}
```

| **   | If you see the error `Could not extract response: no suitable HttpMessageConverter found for response type [class hello.Quote]`it’s possible you are in an environment that cannot connect to the backend service (which sends JSON if you can reach it). Maybe you are behind a corporate proxy? Try setting the standard system properties `http.proxyHost` and `http.proxyPort` to values appropriate for your environment. |
| ---- | ---------------------------------------- |
|      |                                          |

## Summary

Congratulations! You have just developed a simple REST client using Spring.

## See Also

The following guides may also be helpful:

- [Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/)
- [Consuming a RESTful Web Service with AngularJS](https://spring.io/guides/gs/consuming-rest-angularjs/)
- [Consuming a RESTful Web Service with jQuery](https://spring.io/guides/gs/consuming-rest-jquery/)
- [Consuming a RESTful Web Service with rest.js](https://spring.io/guides/gs/consuming-rest-restjs/)
- [Accessing GemFire Data with REST](https://spring.io/guides/gs/accessing-gemfire-data-rest/)
- [Accessing MongoDB Data with REST](https://spring.io/guides/gs/accessing-mongodb-data-rest/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)
- [Accessing JPA Data with REST](https://spring.io/guides/gs/accessing-data-rest/)
- [Accessing Neo4j Data with REST](https://spring.io/guides/gs/accessing-neo4j-data-rest/)
- [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
- [Creating API Documentation with Restdocs](https://spring.io/guides/gs/testing-restdocs/)
- [Enabling Cross Origin Requests for a RESTful Web Service](https://spring.io/guides/gs/rest-service-cors/)
- [Building a Hypermedia-Driven RESTful Web Service](https://spring.io/guides/gs/rest-hateoas/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

