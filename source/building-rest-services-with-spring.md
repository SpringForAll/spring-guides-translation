# 使用Spring构建REST服务

> 原文：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) （这里为示例，译者需根据具体文章修改）
>
> 译者：戒子猪
>
> 校对：

在此处编写正文，基本翻译规范：

* 正文标题按层次结构 从 \#\# 开始
* 代码片段\`\`\`之后需要写明语言类型
* 如有图片更静态资源保存在static目录下，每篇文章建立自己的目录存储
* 尊重原作、不修改、不删减内容
* 每篇文章翻译完成之后提交PR，并在翻译交流群中找校对人员完成review，最后由管理员完成Merge
* 若译者与校对有不同建议，可以将争议部分发到交流群中一起讨论确定结果

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

## 术语表

* REST/RESTful：不翻译
* Repository：数据访问层组件名，不翻译，对应@Repository
* ## 正文

因为REST易于构建和使用，它很快成为网络中构建Web服务应用的标准。

关于REST如何在微服务的世界使用会有更广的讨论，但对于本教程，我们来看看构建REST服务。

为什么选择REST？借鉴[Martin Fowler](http://martinfowler.com/)的说法，[REST实践](https://www.amazon.com/gp/product/0596805829?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0596805829)中指出：“网络是大规模可扩展分布式系统的一个存在证据，它的工作原理很好，我们借鉴它更容易构建集成系统。”我认为这是一个很好的理由：REST包含了自身的网络规则、架构、优势以及所有。

有什么好处？主要因为HTTP自身是一个免费的平台，应用程序安全性（加密和验证）解决方案的数量是已知的，缓存直接内置在协议中，通过DNS的服务路由有良好的弹性，并且大部分系统都支持。

因此，REST无处不在，它并不是一种标准，而是一种方法，是一种基于HTTP约束的架构风格。它的实施方式可能会有所不同。作为API消费者，这可能是一个令人沮丧的经历，因为REST服务的质量差异会很大。

Leonard Richardson博士将REST总结汇集了一个成熟度模型，解释了系统遵循RESTful原则的程度，并对其进行了评分。它描述了从0级开始的四个级别，Martin Fowler在成熟度模型上写出了明确的解释：

* **0级（Level 0）**：POX沼泽——在这个级别，我们仅使用HTTP作为传输。您可以将SOAP称为0级技术，它使用HTTP协议，但主要是作为传输。值得一提的是，您也可以在没有HTTP的情况下使用SOAP，如[JMS](https://www.w3.org/TR/soapjms/)。因此，SOAP不是RESTful的，它只是HTTP感知的。
* **1级（Level 1）**：资源——在此级别，服务可能会使用HTTP URI来区分系统中的名词、实体。例如，您可以将请求路由到`/customers，/users`等。XML-RPC是1级技术的代表：它使用HTTP，并且可以使用URI来区分端点（Endpoint）。最终，尽管有这些定义，XML-RPC实际也不是Restful的：它使用HTTP做了其他东西（如RPC远程过程调用）的传输。
* **2级（Level 2）**：HTTP动词——这就是您想要的级别。如果您用Spring MVC时所有事都做错了，您将仍然在这个地方。在这个级别，服务利用了HTTP的原生术语如标题、状态代码、不同的URI等。这也将是我们旅程的起点。
* **3级（Level 3）**：超媒体控件——最后一级是我们将努力的地方。超媒体，作为[HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)（Hypermedia as the Engine of Application State）的实践，它是真正受欢迎的设计模式。超媒体通过服务的消费者与该服务的“表面积”和拓扑结构的亲密知识的解藕来促进服务寿命。它描述了REST服务，该服务可以回答调用什么，什么时候调用等问题，稍后我们再深入一下。                             

图1：Leonard Richardson成熟度模型

![](/static/s-rest-001.png)

## 开始

在本教程中，我们将用[Spring Boot](https://spring.io/projects/spring-boot)。Spring Boot删除了许多典型的应用程序开发样板。您可以前往[Spring Initializr](https://start.spring.io/)开始，选择和应用程序支持的工作相对应的复选框（来生成Spring Boot脚手架程序）。这样，我们将构建一个Web应用程序，使用JPA在H2数据库中建模，所以选择下边内容：

* Web
* JPA
* H2

然后点击“生成项目”，一个`.zip` 文件将会被下载。解压该文件您会发现一个简单的Maven或Gradle项目目录，配有Maven的`pom.xml`或Gradle中的`build.gradle`。本教程中的示例将基于Maven，尽管如此，对于Gradle部分，您也可以看看。这是很不错的。

Spring Boot可以与任何IDE一起使用，您可使用Eclipse、IntelliJ IDEA、Netbeans等。[Spring Tool Suite](https://spring.io/tools/)（STS）是一个开源的基于Eclipse的IDE发行版，它提供了Eclipse在开发JavaEE过程中的超集工具，而且它包含了使用Spring应用程序更方便的工具。这绝对不是必须的，但是若你想要您的击键变得额外有魅力（省掉重复性工作），您可以考虑它。这是一个演示如何开始使用STS和Spring Boot的视频，仅是一般的介绍，让您熟悉这些工具。

[https://www.youtube.com/watch?v=p8AdyMlpmPk&feature=youtu.be](https://www.youtube.com/watch?v=p8AdyMlpmPk&feature=youtu.be)

## 故事到此……

我们所有的例子都将基于Spring Boot，将会每个示例重温初始化代码。我们的例子是建立一个简单的书签服务，如Instapaper或其他云端书签服务，我们的书签服务只是收集一个URI，以及它的描述。所有的书签将隶属于一个用户账户。该关系模型会使用[模块中](https://github.com/joshlong/bookmarks/tree/tutorial/model/)的JPA和Spring Data JPA存储并进行建模。

我们不会太多地解析代码细节，将使用两个JPA实体对记录进行建模，之后它们将映射到数据库。我们使用标准的SQL关系数据库来存储记录，以便可立即受众于尽可能大的领域。

首先为我们用户账户建模创建第一个类，类名为`Account` 的实体。

_\*：令人惊讶，下列课程是最繁琐的课程——主要是因为Java语言本身的冗长。_

**model/src/main/java/bookmarks/Account.java**

```java
package bookmarks;

import com.fasterxml.jackson.annotation.JsonIgnore;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import java.util.HashSet;
import java.util.Set;


@Entity
public class Account {

    @OneToMany(mappedBy = "account")
    private Set<Bookmark> bookmarks = new HashSet<>();

    @Id
    @GeneratedValue
    private Long id;

    public Set<Bookmark> getBookmarks() {
        return bookmarks;
    }

    public Long getId() {
        return id;
    }

    public String getPassword() {
        return password;
    }

    public String getUsername() {
        return username;
    }

    @JsonIgnore
    public String password;
    public String username;

    public Account(String name, String password) {
        this.username = name;
        this.password = password;
    }

    Account() { // jpa only
    }
}
```

每个`Account`可能没有、或者一个、或者多个对应的`Bookmark`实体，这是一个`1:N`的关系（一对多），`Bookmark`实体的代码如下所示：

**model/src/main/java/bookmarks/Bookmark.java**

```java
package bookmarks;

import com.fasterxml.jackson.annotation.JsonIgnore;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
public class Bookmark {

    @JsonIgnore
    @ManyToOne
    private Account account;

    @Id
    @GeneratedValue
    private Long id;

    Bookmark() { // jpa only
    }

    public Bookmark(Account account, String uri, String description) {
        this.uri = uri;
        this.description = description;
        this.account = account;
    }

    public String uri;
    public String description;

    public Account getAccount() {
        return account;
    }

    public Long getId() {
        return id;
    }

    public String getUri() {
        return uri;
    }

    public String getDescription() {
        return description;
    }
}
```

我们将使用两个[Spring Data JPA repositories](https://spring.io/guides/gs/accessing-data-jpa/)来处理繁琐的数据库交互。Spring Data repositories通常是支持对后端数据进行读取、更新、删除和创建记录的方法接口。一些repositories通常还支持数据分页，并在适当的情况下进行排序，Spring Data根据接口中方法命名约定来实现。除了JPA之外，还有多个存储repositories的实现，您可以使用Spring Data MongoDB、Spring Data GemFire、Spring Data Cassandra等。

一个Repository将管理我们的`Account`实体，称为`AccountRepository`，如下所示。包含了一个自定义查找方法`findByUsername`，针对该方法创建一个基于JPA的查询：`select a from Account a where a.username = :username`。运行它（将方法参数`username`作为命名参数传递），最终返回我们需要的结果。方便！

`model/src/main/java/bookmarks/AccountRepository.java`

```java
package bookmarks;


import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUsername(String username);
}
```

下边是和`Bookmark`实体工作的Repository：

**model/src/main/java/bookmarks/BookmarkRepository.java**

```java
package bookmarks;


import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Collection;

public interface BookmarkRepository extends JpaRepository<Bookmark, Long> {
    Collection<Bookmark> findByAccountUsername(String username);
}
```

`BookmarkRepository`有一个类似finder的方法，但这个方法依赖`Bookmark`实体对应`Account`关系的`username`属性，最终它需要一种连接，它会生成对应的查询：

```sql
SELECT b from Bookmark b WHERE b.account.username = :username
```

我们的应用程序将使用Spring Boot，Spring Boot的应用最少会有一个`public static void main`作为主程序入口，并且该入口被`@SpringBootApplication`注解。不论如何，它都会告诉Spring Boot这些信息，我们的`Application`类也是一个很好的地方作为其他零碎模块的支点，和`@Bean`注解定义一样，下边是我们的示例：

`Application.java`的类就像这样：

```java
package bookmarks;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.Arrays;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner init(AccountRepository accountRepository,
            BookmarkRepository bookmarkRepository) {
        return (evt) -> Arrays.asList(
                "jhoeller,dsyer,pwebb,ogierke,rwinch,mfisher,mpollack,jlong".split(","))
                .forEach(a -> {
                        Account account = accountRepository.save(
                            new Account(a,"password"));
                        bookmarkRepository.save(
                            new Bookmark(account,"http://bookmark.com/1/" + a, "A description"));
                        bookmarkRepository.save(
                            new Bookmark(account,"http://bookmark.com/2/" + a, "A description"));
                }
            );
    }

}
```

一旦启动，Spring Boot将调用所有`CommandLineRunner`类型的Bean，给它们一个回调。这种情况下，`CommandLineRunner`是只有一个抽象方法的接口，这意味着在Java 8的世界中，我们可以用lambda表达式来替换它。本教程中所有的示例将使用Java 8，然后，若您不使用Java 8，而是6或7，那么只要将简洁的lambda语法替换成实现该接口的稍微复杂详细的匿名内部类就可以了。

## HTTP是平台

HTTP URI是描述层次结构或关系的自然方式，例如，我们可能会在账户一级启动我们的REST API，所有的URI都以账户的用户名开头。因此，对于名为bob的账户，我们可能会将该账户解析成`/users/bob`或者只是`/bob`。想要访问该账户的书签集合时，我们可以降一级（像文件系统一样），降到`/bob/bookmarks`资源。

REST不规定表述和编码。REST是Representational State Transfer（表述状态迁移）的缩写，适用于HTTP的内容协商机制，允许客户端和服务端在可能的情况下同意来自服务的数据相互理解的表述方式。处理内容协商有很多种方法，最简单的方式就是客户端发送一个`Accept`请求头，该`Accept`标头指定了可接受的MIME类型的逗号分隔列表（如：`application/json, application/xml`）。如果该服务可以生成和这些MIME想匹配的类型，它将以第一个解析的MIME类型的表述形式进行响应。

_（译者语：对于Accept的“第一个”的概念主要和参数q相关，q的值描述了这个MIME类型的优先级，并不是书写顺序，如：_`Accept: application/json;q=0.5,application/xml;q=1.0`_这种会优先尝试解析成XML格式。）_

我们可以使用HTTP动词来操作这些URI表示的数据。

* HTTP GET动词告诉服务获取、检索URI指定的资源，当然如何做到和具体的实现相关，后端代码可能会与数据库、文件系统、另一个Web服务通信，然而客户端不需要感知这一点。对客户端来说，所有的资源都是HTTP资源，在HTTP世界里，只有一种方式请求数据：`GET`。GET调用在请求中没有正文（Body）信息，但通常会返回一个内容体（Body），对`/bob/bookmarks/6`的HTTP GET请求的响应可能如下所示：

```json
    {
        id: 6,
        uri: "http://bookmark.com/2/bob",
        description: "A description"
    }
```

* HTTP DELETE动词告诉服务删除URI指定的资源，同样它的执行流程也和具体实现相关，DELETE调用也没有正文（Body）信息。

* HTTP PUT动词告诉服务使用附带的请求的主体更新URI指定的资源。因此，要更新`/bob/bookmarks`的资源，我可能会发送从GET调用返回的相同的JSON来表述，并更新字段，用新值替换旧值。

* HTTP POST动词告诉服务队请求的封闭体（Enclosed Body）执行某些操作，这里没有固定和快速的规则，但是通常对`/bob/bookmarks`的HTTP POST调用会将封闭体添加或追加到由`/bob/bookmarks`URI指定的集合（数据库、文件系统）中。这可能有点混乱，另一方面，对`/bob/bookmarks/1`的HTTP POST可能与HTTP PUT调用的方式相同，服务可以使用封闭体中的数据，来替换URI指定的资源。

当然有时候事情不一定会按照计划进行。也许浏览器超时、或者服务超时、或者服务遇到错误。尝试访问不存在或不能正确路由映射的资源时，我们会收到讨厌的`404`（找不到页面）错误。`404`在这里是[状态代码](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)，它描述了操作状态的相关信息。针对不同的目的，有很多状态代码沿着范围进行分割。当您在浏览器中向网页发出请求时，它是一个HTTP GET调用，如果页面项目处，那它将返回`200`（OK）的状态代码。它表示OK，你可能读不懂返回的内容，可它确实存在。

* **100x范围**（100 - 199）：这些状态代码是消息型的，描述了请求的处理（方式或过程）。
* **200x范围**（200 - 299）：这些状态代码表示客户端请求的操作已经成功接收、解析、接受和处理。
* **300x范围**（300 - 399）：这些状态代码表示客户端必须采取其他操作来完成该请求，比如重定向。
* **400x范围**（400 - 499）：这些状态代码适用于客户端有错的情况，并且必须在继续之前更正请求，前边的404就是这个例子。
* **500x范围**（500 - 599）：这些状态代码适用于服务端无法完成该请求或者有错的情况。

## 构建REST服务

书签REST服务的第一个功能至少支持从账户的书签读取和添加账户的书签，以及读取单个书签。一下是我们REST的第一个功能：

前文已经看到了`@SpringBootApplication`执行器的代码，我们的REST服务的基石是`BookmarkRestController`。

**rest/src/main/java/bookmarks/BookmarkRestController.java**

```java
@RestController
@RequestMapping("/{userId}/bookmarks")
class BookmarkRestController {

    private final BookmarkRepository bookmarkRepository;

    private final AccountRepository accountRepository;

    @Autowired
    BookmarkRestController(BookmarkRepository bookmarkRepository,
                           AccountRepository accountRepository) {
        this.bookmarkRepository = bookmarkRepository;
        this.accountRepository = accountRepository;
    }

    @RequestMapping(method = RequestMethod.GET)
    Collection<Bookmark> readBookmarks(@PathVariable String userId) {
        this.validateUser(userId);
        return this.bookmarkRepository.findByAccountUsername(userId);
    }

    @RequestMapping(method = RequestMethod.POST)
    ResponseEntity<?> add(@PathVariable String userId, @RequestBody Bookmark input) {
        this.validateUser(userId);

        return this.accountRepository
                .findByUsername(userId)
                .map(account -> {
                    Bookmark result = bookmarkRepository.save(new Bookmark(account,
                            input.uri, input.description));

                    URI location = ServletUriComponentsBuilder
                        .fromCurrentRequest().path("/{id}")
                        .buildAndExpand(result.getId()).toUri();

                    return ResponseEntity.created(location).build();
                })
                .orElse(ResponseEntity.noContent().build());

    }

    @RequestMapping(method = RequestMethod.GET, value = "/{bookmarkId}")
    Bookmark readBookmark(@PathVariable String userId, @PathVariable Long bookmarkId) {
        this.validateUser(userId);
        return this.bookmarkRepository.findOne(bookmarkId);
    }

    private void validateUser(String userId) {
        this.accountRepository.findByUsername(userId).orElseThrow(
                () -> new UserNotFoundException(userId));
    }
}
```

`BookmarkRestController`是一个简单的使用`@RestController`注解的Spring MVC控制器组件。`@RestController`使用`@RequestMapping`注解在每个被注解的Bean的方法上提供相关元数据将该方法作为HTTP端（Endpoint），如果传入的HTTP请求符合方法的`@RequestMapping`注解规定的限制条件，那么该方法在请求匹配时会被调用。

`@RestController`，当它（`@RequestMapping`）处于类级别（type-level）时，它会默认为该类级别的所有方法提供默认值，每个单独的方法都可以覆盖大多数类级的注解，有些事情是上下文环境的。例如，`BookmarkRestController`处理了以某一用户名（如bob）开头，后边跟了`/bookmarks`的所有请求。进一步限定URI的类型的任何方法（`readBookmark`）都将添加到根请求映射中。因此，`readBookmark`实际上被映射到了`/{userId}/bookmarks/{bookmarkId}`中。不指定路径的方法只是继承在类级别映射的路径。add方法响应类级别指定的URI，但它只响应带有动词（POST）的HTTP请求。

路径中的`{userId}`和`{bookmarkId}`标识路径变量，它们是一小部分（固定值）或者通配符，Spring MVC将提取URI的这部分，并将其作为参数传递给控制器中的方法，参数名和`@PathVariable`注解的参数名相同。对于HTTP GET请求：`/bob/bookmarks/4234`，（匹配方式如）

* `@PathVariable String userId`，参数将会是`"bob"`；
* `@PathVariable Long bookmarkId`，参数将会是长整型的`4234`。

这些控制器方法会返回简单的POJO（Plain Ordinary Java Object）——`Collection<Bookmark>`，以及`Bookmark`等。当一个HTTP请求指定一个`Accept`标头时，Spring MVC循环遍历已经配置的`HttpMessageConverter`，知道找到一个可以从POJO域模型类型转换为`Accept`头中指定的内容类型（如果配置的话）。Spring Boot将自动连接一个可以将Generic Object转换成[JSON](http://www.json.org/)的`HttpMessageConvertor`，不需要任何更具体的转换器。`HttpMessageConverter`会做双向转换：传入请求体（Body）转换成Java对象，将Java对象转换为HTTP响应体（\*：反序列化、序列化操作）。

使用命令`curl`或（浏览器）您可以从`http://localhost:8080/jhoeller/bookmarks`中查看JSON响应。

add方法指定一个类型为`Bookmark`的参数，一个POJO。Spring MVC将使用合适的`HttpMessageConverter`将传入的HTTP请求（包含可能有效的JSON）转换成POJO。

add方法接收传入的HTTP请求，保存它们，然后转换成`ResponseEntity<T>`。`ResponseEntity`是响应的包装器，它包含了可选的HTTP标头和状态代码。add方法响应了一个状态代码201（CREATED）和一个头（Location），它表示客户端可以知道它新创建的资源的状态以及引用地址，这有点像在数据库中保存记录之后提取刚刚生成的主键。

如果没有路径匹配，默认的Spring Boot设置了常用的`HttpMessageConverter`实现的集合，也很容易添加其他紧凑的网络高效格式（[如Spring 4.1中的Google Protocol Buffer实现](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/protobuf/ProtobufHttpMessageConverter.html)）的支持，这个使用通常的Spring MVC配置。

Spring MVC原生支持文件上传，在控制器中使用参数`MultipartFile multipartFile`。

在示例中如果用户不存在，则可抛出自定义异常：

**rest/src/main/java/bookmarks/UserNotFoundException.java**

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(String userId) {
        super("could not find user '" + userId + "'.");
    }
}
```

想要运行？

```bash
$ cd rest
$ mvn clean spring-boot:run
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::

2016-11-04 11:54:21.135  INFO 73989 --- [           main] bookmarks.Application                    : Starting Application on retina with PID 73989 (/Users/gturnquist/src/tutorials/tut-bookmarks/rest/target/classes started by gturnquist in /Users/gturnquist/src/tutorials/tut-bookmarks/security)
...
```

_\*：如果您从来没有做过这个教程，您需要先运行_`mvn install`_，它将会把项目编译好的结果放到POM的仓库中，_`~/.m2`_目录下，您可运行每一个章节的内容（每个章节是一个Maven的模块）。_

Spring MVC可以很容易地编写形状由Http Servlet API保存的面向服务的代码，该代码可以轻松进行单元测试，并通过Spring AOP扩展。我们将在下一节中介绍如何单元测试这些Spring MVC组件。

### 使用HTTP来表示错误

REST服务中的所有方法都调用了`validateUser`，这些用来验证有问题的用户是否存在，如果没有，则抛出`UserNotFoundException`异常，如前文所显示的。它使用Spring MVC的`@ResponseStatus(HttpStatus.NOT_FOUND)`进行注解，该异常触发时它会通知Spring MVC发回状态代码（404）。这是一个很好的安排：您可以在代码中考虑您的业务领域，并使用状态代码，您可以更好地了解如何将HTTP映射到客户端的HTTP信号错误中，Spring MVC提供了很多不同的地方，可以在通用的、应用程序全局和局部控制的错误逻辑中进行清晰的分层处理（[参考链接](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)）。

## 测试一个REST服务

Spring MVC提供了很好的基于HTTP端的[单元测试](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/testing.html#spring-mvc-test-framework)功能。它在单元测试和集成测试之间提供了很好的中间件，因为它可以让您站在基于Spring MVC的`DispatcherServlet`的位置——包括验证器、`HttpMessageConverter`等，然后对它们运行测试，而无需实际启动一个真正的HTTP服务：世界最好的两点！这是一个集成测试，因为您关心的业务逻辑实际上正在被执行，但它又是一个单元测试，实际上您并不需要等待Web服务器初始化并开始服务请求。

一下是`BookmarkRestController`的单元测试示例，任何已经编写过Junit单元测试的人应该看起来都很熟悉。在顶部，我们使用`@SpringApplicationConfiguration(class=Application.class)`注解来告诉`SpringJUnit4ClassRunner`应该在哪里获取相关需要测试的Spring应用程序信息。`@WebAppConfiguration`注释告诉JUnit，这是Spring MVC Web组件的单元测试，因此应该运行在WebApplicationContext种类下，而不是标准的ApplicationContext实现。

**rest/src/test/java/bookmarks/BookmarkRestControllerTest.java**

```java
package bookmarks;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;
import org.springframework.mock.http.MockHttpOutputMessage;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.web.context.WebApplicationContext;

import java.io.IOException;
import java.nio.charset.Charset;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import static org.hamcrest.Matchers.*;
import static org.junit.Assert.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*;

/**
 * @author Josh Long
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
@WebAppConfiguration
public class BookmarkRestControllerTest {


    private MediaType contentType = new MediaType(MediaType.APPLICATION_JSON.getType(),
            MediaType.APPLICATION_JSON.getSubtype(),
            Charset.forName("utf8"));

    private MockMvc mockMvc;

    private String userName = "bdussault";

    private HttpMessageConverter mappingJackson2HttpMessageConverter;

    private Account account;

    private List<Bookmark> bookmarkList = new ArrayList<>();

    @Autowired
    private BookmarkRepository bookmarkRepository;

    @Autowired
    private WebApplicationContext webApplicationContext;

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    void setConverters(HttpMessageConverter<?>[] converters) {

        this.mappingJackson2HttpMessageConverter = Arrays.asList(converters).stream()
            .filter(hmc -> hmc instanceof MappingJackson2HttpMessageConverter)
            .findAny()
            .orElse(null);

        assertNotNull("the JSON message converter must not be null",
                this.mappingJackson2HttpMessageConverter);
    }

    @Before
    public void setup() throws Exception {
        this.mockMvc = webAppContextSetup(webApplicationContext).build();

        this.bookmarkRepository.deleteAllInBatch();
        this.accountRepository.deleteAllInBatch();

        this.account = accountRepository.save(new Account(userName, "password"));
        this.bookmarkList.add(bookmarkRepository.save(new Bookmark(account, "http://bookmark.com/1/" + userName, "A description")));
        this.bookmarkList.add(bookmarkRepository.save(new Bookmark(account, "http://bookmark.com/2/" + userName, "A description")));
    }

    @Test
    public void userNotFound() throws Exception {
        mockMvc.perform(post("/george/bookmarks/")
                .content(this.json(new Bookmark()))
                .contentType(contentType))
                .andExpect(status().isNotFound());
    }

    @Test
    public void readSingleBookmark() throws Exception {
        mockMvc.perform(get("/" + userName + "/bookmarks/"
                + this.bookmarkList.get(0).getId()))
                .andExpect(status().isOk())
                .andExpect(content().contentType(contentType))
                .andExpect(jsonPath("$.id", is(this.bookmarkList.get(0).getId().intValue())))
                .andExpect(jsonPath("$.uri", is("http://bookmark.com/1/" + userName)))
                .andExpect(jsonPath("$.description", is("A description")));
    }

    @Test
    public void readBookmarks() throws Exception {
        mockMvc.perform(get("/" + userName + "/bookmarks"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(contentType))
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].id", is(this.bookmarkList.get(0).getId().intValue())))
                .andExpect(jsonPath("$[0].uri", is("http://bookmark.com/1/" + userName)))
                .andExpect(jsonPath("$[0].description", is("A description")))
                .andExpect(jsonPath("$[1].id", is(this.bookmarkList.get(1).getId().intValue())))
                .andExpect(jsonPath("$[1].uri", is("http://bookmark.com/2/" + userName)))
                .andExpect(jsonPath("$[1].description", is("A description")));
    }

    @Test
    public void createBookmark() throws Exception {
        String bookmarkJson = json(new Bookmark(
                this.account, "http://spring.io", "a bookmark to the best resource for Spring news and information"));

        this.mockMvc.perform(post("/" + userName + "/bookmarks")
                .contentType(contentType)
                .content(bookmarkJson))
                .andExpect(status().isCreated());
    }

    protected String json(Object o) throws IOException {
        MockHttpOutputMessage mockHttpOutputMessage = new MockHttpOutputMessage();
        this.mappingJackson2HttpMessageConverter.write(
                o, MediaType.APPLICATION_JSON, mockHttpOutputMessage);
        return mockHttpOutputMessage.getBodyAsString();
    }
}
```

首先要看`@Before`注解的`setup`方法，该方法的第一件事就是实例化一个需要引用应用程序`WebApplicationContext`的`MockMvc`。`MockMvc`是中心部分：所有测试会通过`MockMvc`类型来模拟服务的HTTP请求，就是这样！之后看看所有测试中的任何一个。 我们可以使用`org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*`的静态导入将HTTP请求链接在一起并验证响应。

所有测试都指定了一个`application/json`内容类型，并且期望得到该内容类型的响应。 测试使用`MockMvcResultMatchers＃jsonPath`方法来验证JSON响应的结构和内容。 反过来又使用`Jayway JSON Path API`在JSON结构上运行`X-Path`风格的遍历，就像我们在单元测试中的各个位置一样。

## 构建一个HATEOAS的REST服务

API的第一个切入点工作得很好，如果这个服务有详细的文档，那么对于许多不同语言的REST客户端来说，它将是可行的，它是一个干净的API，因为它以一种很好理解的方式使用了HTTP提供的原语。 API的一个定义原则是遵循[统一接口原理](https://en.wikipedia.org/wiki/Representational_state_transfer#Uniform_interface)，像我们迄今为止的HTTP REST API设计堆叠得都不错，每条消息都包含足够的信息来描述如何处理消息。例如，客户端可以根据请求消息中的`Content-Type`头来决定要调用哪个解析器，将系统中的状态映射到唯一标识资源URI，状态是可寻址的，状态突变是通过已知的HTTP动词（`POST，GET，DELETE，PUT`等）完成的。因此，当客户端持有资源的表示（包括附加的任何元数据）时，它具有足够的信息来修改或删除资源。

但我们可以做得更好。他们的服务是足够的任务，但缺乏……**持续力量**。如维基百科说：客户必须先知道该API。 API中的更改会破坏客户端，并且会破坏有关该服务的文档。超媒体作为应用程序状态的引擎（a.k.a.[HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)）是解决和消除这种耦合的另一个约束。客户端仅通过在超媒体中由服务器动态识别的动作（例如，通过超文本中的超链接）来进行状态转换。除了对应用程序的简单固定入口点之外，客户端不会假定任何特定的操作对于以前从服务器接收的表示中描述的任何特定资源的可用性。

我们来看看这个API的修改版（如下所示），这次用[Spring HATEOAS](https://spring.io/projects/spring-hateoas)来实现HATEOAS。简单的说，Spring HATEOAS可以很容易地提供链接——包含了返回到客户端的元数据——但这描述了我们将如何处理它。从根本上说，我们所做的就是使用Spring HATEOAS的`ResourceSupport`类型来包装我们的响应有效资源。 `ResourceSupport`包含了`Link`对象集，后者又描述了有用的相关资源。例如，描述电子商务解决方案中的帐户的资源，具体到该帐户订单的资源链接，到该帐户的当前购物车的链接，可用于再次检索该资源状态的链接。

**hateoas/src/main/java/bookmarks/Application.java**

```java
package bookmarks;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.Arrays;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner init(AccountRepository accountRepository, BookmarkRepository bookmarkRepository) {
        return (args) ->
            Arrays.asList("jhoeller,dsyer,pwebb,ogierke,rwinch,mfisher,mpollack,jlong".split(","))
                .forEach(a -> {
                    Account account = accountRepository.save(new Account(a, "password"));
                    bookmarkRepository.save(new Bookmark(account, "http://bookmark.com/1/" + a, "A description"));
                    bookmarkRepository.save(new Bookmark(account, "http://bookmark.com/2/" + a, "A description"));
                });
    }
}
```

这个类包含了`@SpringBootApplication`注解，它激活了自动配置，组件扫描，允许定义不同的Bean。

接下了，我们需要针对`bookmarks`定义REST资源的细节：

**hateoas/src/main/java/bookmarks/BookmarkResource.java**

```java
class BookmarkResource extends ResourceSupport {

    private final Bookmark bookmark;

    public BookmarkResource(Bookmark bookmark) {
        String username = bookmark.getAccount().getUsername();
        this.bookmark = bookmark;
        this.add(new Link(bookmark.getUri(), "bookmark-uri"));
        this.add(linkTo(BookmarkRestController.class, username).withRel("bookmarks"));
        this.add(linkTo(methodOn(BookmarkRestController.class, username)
                .readBookmark(username, bookmark.getId())).withSelfRel());
    }

    public Bookmark getBookmark() {
        return bookmark;
    }
}
```

Spring HATEOAS的扩展`ResourceSupport`允许我们嵌入一个链接到整个书签集合中，等价于`self`链接。

接下来看看REST端的详细信息：

**hateoas/src/main/java/bookmarks/BookmarkRestController.java**

```java
@RestController
@RequestMapping("/{userId}/bookmarks")
class BookmarkRestController {

    private final BookmarkRepository bookmarkRepository;

    private final AccountRepository accountRepository;

    BookmarkRestController(BookmarkRepository bookmarkRepository,
                           AccountRepository accountRepository) {
        this.bookmarkRepository = bookmarkRepository;
        this.accountRepository = accountRepository;
    }

    @RequestMapping(method = RequestMethod.GET)
    Resources<BookmarkResource> readBookmarks(@PathVariable String userId) {

        this.validateUser(userId);

        List<BookmarkResource> bookmarkResourceList = bookmarkRepository
                .findByAccountUsername(userId).stream().map(BookmarkResource::new)
                .collect(Collectors.toList());

        return new Resources<>(bookmarkResourceList);
    }

    @RequestMapping(method = RequestMethod.POST)
    ResponseEntity<?> add(@PathVariable String userId, @RequestBody Bookmark input) {

        this.validateUser(userId);

        return accountRepository.findByUsername(userId)
            .map(account -> {
                Bookmark bookmark = bookmarkRepository
                        .save(new Bookmark(account, input.uri, input.description));

                Link forOneBookmark = new BookmarkResource(bookmark).getLink("self");

                return ResponseEntity.created(URI.create(forOneBookmark.getHref())).build();
            })
            .orElse(ResponseEntity.noContent().build());
    }

    @RequestMapping(method = RequestMethod.GET, value = "/{bookmarkId}")
    BookmarkResource readBookmark(@PathVariable String userId,
                                  @PathVariable Long bookmarkId) {
        this.validateUser(userId);
        return new BookmarkResource(this.bookmarkRepository.findOne(bookmarkId));
    }

    private void validateUser(String userId) {
        this.accountRepository
            .findByUsername(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
    }
}
```

在本教程前面，我们看到这个控制器使用`Bookmark`对象，现在该版本将它们包装在`BookmarkResource`中，带来了HATEOAS的更多优点。

它还使用了Spring HATEOAS中的`Link` API，它可以替换成：

```java
URI location = ServletUriComponentsBuilder
    .fromCurrentRequest().path("/{id}")
    .buildAndExpand(result.getId()).toUri();

return ResponseEntity.created(location).build();
```

或者：

```java
Link forOneBookmark = new BookmarkResource(bookmark).getLink("self");

return ResponseEntity.created(URI.create(forOneBookmark.getHref())).build();
```

_Spring HATEOAS中的_`linkTo()`_和_`methodOn()`_API嘉定在Web请求的上下文环境中被调用了，这就是他们访问您刚才使用的相同的Servlet的详细信息，当从上下文环境之外的Servlet调用它时，会报告错误消息提示您。_

要很好地处理异常，请注意下边的控制器中注册的Advice：

**hateoas/src/main/java/bookmarks/BookmarkControllerAdvice.java**

```java
@ControllerAdvice
class BookmarkControllerAdvice {

    @ResponseBody
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    VndErrors userNotFoundExceptionHandler(UserNotFoundException ex) {
        return new VndErrors("error", ex.getMessage());
    }
}
```

它将`UserNotFoundException`转换成了HTTP 404状态代码（Not Found）和一个特殊类型`application/vnd.error`。实际上这个异常是一个运行时异常：

**hateoas/src/main/java/bookmarks/UserNotFoundException.java**

```java
class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(String userId) {
        super("could not find user '" + userId + "'.");
    }
}
```

顺便说一句，这里的`Link`对象与HTML中的用于导入CSS样式表的`<link/>`元素是一样的，它们具有href属性和rel属性。href属性URI类似于CSS样式表的Web路径，rel告诉客户端（浏览器）为什么这个资源很重要（它是一个用于呈现页面的样式表），将连接元素转换成JSON也不难。

[_HAL_](http://stateless.co/hal_specification.html)_（超文本应用程序语言）是最流行的超媒体格式之一，它是轻量级的，添加的JSON编码比_`_links`_属性更多，以包含超链接及其rels的列表。 它还提供通过_`_embedded`_条目对集合进行编码的功能，这是Spring HATEOAS提供的默认媒体类型，但也支持其他格式。_

`BookmarkResource`类型包装一个`Bookmark`对象，并提供了一个很好的地方来集中管理链接建立的逻辑。至少，一个资源应该提供一个自己的链接（通常是一个链接，其`rel`值是`self`）。我们可以简单地写出`http://127.0.0.1:8080/{userId}/bookmarks/{id}`（将路径变量替换为适当的值），但是一旦我们转到不同的主机，端口，和上下文根。我们可以使用`ServletUriComponentsBuilder`来简化这些工作。为什么我们应该如此？毕竟，Spring MVC已经知道了这个URI。它在每个控制器方法的`@RequestMapping`信息中！ Spring HATEOAS提供了方便的静态`ControllerLinkBuilder.linkTo`和`ControllerLinkBuilder.methodOn`方法来从控制器元数据本身提取URI——一个显着的改进，并与`DRY`保持一致（不要重复自己）的原则。该示例显示如何直接构建一个Link对象，为`href`指定任意值（在本例中为书签本身的URI），如何根据Spring MVC控制器类上的`@RequestMapping`元数据构建链接，以及如何基于特定的Spring MVC控制器方法的`@RequestMapping`元数据构建链接。

看看这是如何变化的？

```bash
$ cd ../hateoas
$ mvn clean spring-boot:run
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::

2016-11-04 11:54:21.135  INFO 73989 --- [           main] bookmarks.Application                    : Starting Application on retina with PID 73989 (/Users/gturnquist/src/tutorials/tut-bookmarks/hateoas/target/classes started by gturnquist in /Users/gturnquist/src/tutorials/tut-bookmarks/security)
...
```

使用此功能，剩下的更改将`Bookmark`类型替换为`BookmarkResource`类型。

### 改进使用VndErrors的错误处理

HATEOAS为客户提供了有关服务本身的改进元数据，我们也可以自定义改善错误处理的情况。 HTTP状态代码告诉客户 — 广泛 — 出现问题。如，400-499的HTTP状态代码告诉客户客户端出错了，来自500-599的HTTP状态代码告诉客户端服务器发生错误，如果您知道您的状态代码，那么这可能是了解如何使用API​​的开始。但是，我们可以做得更好！毕竟，在REST客户端启动和运行之前，有人需要开发它，而有用的错误消息在理解API方面是非常宝贵的，可以使用一致的方式更好地处理错误！

`VndError`是一种流行的、标准的mime类型，它描述了对REST服务的客户端的错误编码，它适用于40x和50x错误范围内的错误，Spring HATEOAS提供了`VndError`类型，可用于将错误报告回您的客户端。

我们修改后的服务引入了一个新类，`BookmarkControllerAdvice`，它使用Spring MVC的`@ControllerAdvice`注释。 `@ControllerAdvice`是一个有用的方法，将常见问题的配置（如错误处理）接藕到单独的地方，远离任何Spring MVC的主控制器。例如，Spring MVC定义了`@ExceptionHandler`方法，将特定的处理程序方法与异常或HTTP状态代码的相关事件联系起来。在这里，我们告诉Spring MVC，抛出`UserNotFoundException`的代码，最终应该由`userNotFoundExceptionHandler`方法处理。该方法简单地将传播的`Exception`包装在`VndErrors`中，并将其返回给客户端。在这个例子中，我们已经从异常本身移除了`@ResponseStatus`注解，并将其集中在`@ControllerAdvice`类型中。`@ControllerAdvice`类型是集中各种逻辑的一种方便的方法，并且与注释异常类型不同 —— 即使对于不包含源代码的异常类型也可以使用。

## 保护REST服务

到目前为止，我们都是假设所有的客户都值得信赖，并且他们应该有权限对所有的数据进行访问，实际上这样的情况很少，一个开放的REST API是通常是一个不安全的。但是这种问题很好解决，[Spring Security](https://spring.io/projects/spring-security)提供了保护应用程序访问权限的原语，从根本上说，Spring Security定义了应用程序的用户和他们的特权，这些权限可以回答这个问题：应用程序用户可以看到什么或做什么？

Spring Security的核心是`UserDetailsS​​ervice`接口，它有一个工作：提供一个用户名生成`UserDetails`实现类，`UserDetails`实现类必须能够回答有关帐户有效性、密码、用户名及其权限的所有问题（以`org.springframework.security.core.GrantedAuthority`类为例）。

```java
package org.springframework.security.core.userdetails;

public interface UserDetailsService {

    org.springframework.security.core.userdetails.UserDetails loadUserByUsername(java.lang.String s)
        throws org.springframework.security.core.userdetails.UsernameNotFoundException;

}
```

Spring Security提供了许多适应现有身份提供商（如Active Directory，LDAP，pam，CAAS等）的合约实施，[Spring Social](https://spring.io/projects/spring-social)甚至提供了一个和三方社交网络如Facebook，Twitter等基于不同的OAuth认证服务的更好的集成。

我们的示例已经有一个`Account`的概念，所以我们可以通过提供自定义的`UserDetailsS​​ervice`来实现，如下面的`WebSecurityConfiguration` ，`@Configuration`-类所示。对于任何认证请求有关的疑问，Spring Security将会检查`UserDetailsS​​ervice`。

### 使用Spring Security OAuth实现开放Web上的客户端身份认证和授权

我们可以通过无数方式验证客户端请求，客户端可以在每个请求上发送如HTTP基本用户名和密码、他们可以在每个请求上传送x509证书、实际上还有更多方法可以在这里使用、只是需要针对实际情况进行权衡。

我们的API旨在开放Web中的资源让终端消费，包括各种各样的HTML5资源、以及我们打算构建的本地移动和桌面客户端来使用。我们将使用具有多种安全功能的多样化客户端，我们选择的任何解决方案都应该能适合这一点。我们还应该将用户的用户名和密码与应用程序的会话分离；毕竟，如果我重置了我的Twitter密码，我不想被迫重新验证每个登录过的客户端。另一方面，如果有人劫持我们的一个客户端（也许用户已经丢失了一个电话），我们不希望窃取设备的骗子能够将我们的用户锁定在帐户之外。

[OAuth](https://oauth.net/)提供了一个清晰的方式来处理这些问题，毫无疑问您已经使用了OAuth：一个常见的情况是安装Facebook插件或游戏。通常流程如下所示：

1. 用户在网络上找到了一个游戏或者某些功能，这些功能需要让用户授权访问它的Facebook数据。一个常见的例子是“使用Facebook登录”的场景。
2. 用户点击“安装”或“添加”，然后重定向到受信任的域（如：Facebook.com），其中用户被提示授予某些权限（例如“发送到墙”或“读取基本信息”）。
3. 用户确认这些权限，并随后重定向回调到应用程序，其中源应用程序现在具有访问令牌（Access Token），它将使用此访问令牌代替您向Facebook发出请求访问您的资源。

在此示例中，任何旧客户端都可以与Facebook通话，并且客户端在进程结束时具有访问令牌，该访问令牌会用于所有后续的REST请求伴随传输，类似于HTTP cookie，而不需要重传用户名和密码，客户端也可以在有限或无限期内缓存访问令牌。例如，客户端每次打开应用程序时都不需要重新进行身份验证。更好的实现是：访问令牌和特定的客户端绑定，它们用于表示当前这个客户端比其他客户拥有更多的权限。在上述流程中，请求会在Facebook.com上终止，如果用户尚未登录过Facebook，则会提示他们登录，然后向客户端分配权限。这有利于确保在不受信任的应用程序中，永远不会在任何可能尝试恶意捕获该用户的用户名和密码的程序里输入任何敏感信息（如用户名和密码）。我们的应用程序将不会提供给任何旧的客户端，但是可以肯定，我们部署的任何客户端都是友好的，因为它将是我们的客户端之一。 OAuth支持一个更简单的流程，用户可以从客户端进行身份验证（通常是通过发送用户名和密码），并且服务直接返回OAuth访问令牌，避免重定向到受信任的域——这是我们将采取的方法：结果将是我们的客户端将具有与用户的用户名密码分离的访问令牌，并且访问令牌可用于在不同客户端上赋予不同级别的安全性。

为我们的应用设置OAuth安全性很简单，`OAuth2Configuration`配置类描述了需要`ROLE_USER`和`write`范围的一个客户端（这里是一个假设的Android客户端）。 Spring Security OAuth将从`AuthenticationManager`中读取最终使用我们自定义`UserDetailsS​​ervice`的配置信息。

OAuth非常灵活，例如，您可以部署许多REST API共享的授权服务器，在这种情况下，我们的OAuth实现与我们的书签REST API相似，实际上他们是一样的——这就是为什么我们在同一配置类中使用`@EnableResourceServer`和`@EnableAuthorizationServer`的原因。

### 您说的开放网页？

我们期望服务的一些客户端是基于HTML5的，他们会想从不同的域访问到我们的REST API，在大多数浏览器中，这都与跨站点脚本安全措施相冲突。我们可以通过公开CORS（跨原始请求脚本）头来明确为已知受信客户端启用XSS功能。这些标头在服务响应中存在时，向浏览器发出信号，即允许标头中描述的源、形状和配置的请求、甚至跨域。我们的API使用简单的`javax.servlet.Filter`进行封装，每个请求都添加这些头。在这个例子中，如果提供的话，我们将委托给一个属性 `- tagit.origin -`，或默认的`http//localhost9000`。我们可能会有一个JavaScript客户端向服务器发出请求、我们可以轻松地从数据存储区读取这些信息、我们可以在不重新编译代码的情况下进行更改。我们在这里只指定了一个源头（同源），但是我们也可以轻松地指定许多其他客户端。

### 使用HTTPS（SSL/TLS）来防止中间人攻击

Spring Boot提供了一个嵌入式Web服务器（默认情况下是Apache Tomcat），可以通过编程方式进行配置，以执行独立Apache Tomcat Web服务器可以执行的任何操作。在过去，配置HTTPS（SSL/TLS）的步骤十分繁琐，现在，Spring Boot使用声明性方式让它的配置变得非常简单。

首先，创建以下内容：

**security/src/main/resources/application-https.properties**

```properties
# Configure the server to run with SSL/TLS and using HTTPS
server.port = 8443
server.ssl.key-store = classpath:tomcat.keystore
server.ssl.key-store-password = password
server.ssl.key-password = password
```

_\*：仅当使用配置文件https配置SPRING\_PROFILES\_ACTIVE运行应用程序时，才会激活此属性文件。_

HTTPS需要使用属性值提供的签名证书证书和证书密码。为此，我们可以使用JDK的keytool，如下所示：

```auto
$ keytool -genkey -alias bookmarks -keyalg RSA -keystore src/main/resources/tomcat.keystore
Enter keystore password: password
Re-enter new password: password
What is your first and last name?
  Unknown:  Josh Long
What is the name of your organizational unit?
  Unknown:  Spring Team
What is the name of your organization?
  Unknown:  Pivotal
What is the name of your City or Locality?
  Unknown:  IoT
What is the name of your State or Province?
  Unknown:  Earth
What is the two-letter country code for this unit?
  Unknown:  US
Is CN=Josh Long, OU=Spring Team, O=Pivotal, L=IoT, ST=Earth, C=US correct?
  no:  yes

Enter key password for <learningspringboot>
    (RETURN if same as keystore password): <RETURN>
```

_\*：密钥库文件必须保存在文件系统上让嵌入式的Tomcat服务器读取它们，它们不能嵌入到JAR文件中。（坦白说，这种类型的安全项不应该属于您的可交付的结果的一部分。）_

这将在`src/main/resources`下创建一个密钥库，现在如果您运行应用程序，它将使用密钥库来运行启用了`SSL/TLS`的嵌入式Tomcat servlet容器。

如前文所述，此模式仅适用于名为`https`的Spring配置文件，这意味着您可以在不使用配置文件的情况下运行和测试应用程序，也可以轻松地将其切换。

## 放到一起

我们需要一个顶级类来运行这个应用：

**security/src/main/java/bookmarks/Application.java**

```java
//
// curl -X POST -vu android-bookmarks:123456 http://localhost:8080/oauth/token -H "Accept: application/json" -d "password=password&username=jlong&grant_type=password&scope=write&client_secret=123456&client_id=android-bookmarks"
// curl -v POST http://127.0.0.1:8080/bookmarks -H "Authorization: Bearer <oauth_token>""

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    // CORS
    @Bean
    FilterRegistrationBean corsFilter(
            @Value("${tagit.origin:http://localhost:9000}") String origin) {
        return new FilterRegistrationBean(new Filter() {
            public void doFilter(ServletRequest req, ServletResponse res,
                    FilterChain chain) throws IOException, ServletException {
                HttpServletRequest request = (HttpServletRequest) req;
                HttpServletResponse response = (HttpServletResponse) res;
                String method = request.getMethod();
                // this origin value could just as easily have come from a database
                response.setHeader("Access-Control-Allow-Origin", origin);
                response.setHeader("Access-Control-Allow-Methods",
                        "POST,GET,OPTIONS,DELETE");
                response.setHeader("Access-Control-Max-Age", Long.toString(60 * 60));
                response.setHeader("Access-Control-Allow-Credentials", "true");
                response.setHeader(
                        "Access-Control-Allow-Headers",
                        "Origin,Accept,X-Requested-With,Content-Type,Access-Control-Request-Method,Access-Control-Request-Headers,Authorization");
                if ("OPTIONS".equals(method)) {
                    response.setStatus(HttpStatus.OK.value());
                }
                else {
                    chain.doFilter(req, res);
                }
            }

            public void init(FilterConfig filterConfig) {
            }

            public void destroy() {
            }
        });
    }

    @Bean
    CommandLineRunner init(AccountRepository accountRepository,
            BookmarkRepository bookmarkRepository) {
        return (evt) -> Arrays.asList(
                "jhoeller,dsyer,pwebb,ogierke,rwinch,mfisher,mpollack,jlong".split(","))
                .forEach(
                        a -> {
                            Account account = accountRepository.save(new Account(a,
                                    "password"));
                            bookmarkRepository.save(new Bookmark(account,
                                    "http://bookmark.com/1/" + a, "A description"));
                            bookmarkRepository.save(new Bookmark(account,
                                    "http://bookmark.com/2/" + a, "A description"));
                        });
    }

}
```

除了您之前写的配置以外，此类已经扩展了包括CORS过滤器的设置。

接下来，我们需要配置从数据存储加载安全细节的方法：

**security/src/main/java/bookmarks/WebSecurityConfiguration.java**

```java
@Configuration
class WebSecurityConfiguration extends GlobalAuthenticationConfigurerAdapter {

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void init(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }

    @Bean
    UserDetailsService userDetailsService() {
        return (username) -> accountRepository
                .findByUsername(username)
                .map(a -> new User(a.username, a.password, true, true, true, true,
                        AuthorityUtils.createAuthorityList("USER", "write")))
                .orElseThrow(
                        () -> new UsernameNotFoundException("could not find the user '"
                                + username + "'"));
    }
}
```

此类使用`GlobalAuthenticationConfigurerAdapter`构建`UserDetailsService`并连接到我们的Spring数据存储库中。

接下来，我们需要制定我们的OAuth2安全策略：

**security/src/main/java/bookmarks/OAuth2Configuration.java**

```java
@Configuration
@EnableResourceServer
@EnableAuthorizationServer
class OAuth2Configuration extends AuthorizationServerConfigurerAdapter {

    String applicationName = "bookmarks";

    // This is required for password grants, which we specify below as one of the
    // {@literal authorizedGrantTypes()}.
    @Autowired
    AuthenticationManagerBuilder authenticationManager;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints)
            throws Exception {
        // Workaround for https://github.com/spring-projects/spring-boot/issues/1801
        endpoints.authenticationManager(new AuthenticationManager() {
            @Override
            public Authentication authenticate(Authentication authentication)
                    throws AuthenticationException {
                return authenticationManager.getOrBuild().authenticate(authentication);
            }
        });
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {

        clients.inMemory()
            .withClient("android-" + applicationName)
            .authorizedGrantTypes("password", "authorization_code", "refresh_token")
            .authorities("ROLE_USER")
            .scopes("write")
            .resourceIds(applicationName)
            .secret("123456");
    }
}
```

有了这些，我们可以创建前面提到的策略，`BookmarkResource`类则与以前相同。

引入安全性后，对`BookmarkRestController`做了一些修改：

**security/src/main/java/bookmarks/BookmarkRestController.java**

```java
@RestController
@RequestMapping("/bookmarks")
class BookmarkRestController {

    private final BookmarkRepository bookmarkRepository;

    private final AccountRepository accountRepository;

    @Autowired
    BookmarkRestController(BookmarkRepository bookmarkRepository,
                           AccountRepository accountRepository) {
        this.bookmarkRepository = bookmarkRepository;
        this.accountRepository = accountRepository;
    }

    @RequestMapping(method = RequestMethod.GET)
    Resources<BookmarkResource> readBookmarks(Principal principal) {
        this.validateUser(principal);

        List<BookmarkResource> bookmarkResourceList = bookmarkRepository
            .findByAccountUsername(principal.getName()).stream()
            .map(BookmarkResource::new)
            .collect(Collectors.toList());

        return new Resources<>(bookmarkResourceList);
    }

    @RequestMapping(method = RequestMethod.POST)
    ResponseEntity<?> add(Principal principal, @RequestBody Bookmark input) {
        this.validateUser(principal);

        return accountRepository
                .findByUsername(principal.getName())
                .map(account -> {
                    Bookmark bookmark = bookmarkRepository.save(
                        new Bookmark(account, input.uri, input.description));

                    Link forOneBookmark = new BookmarkResource(bookmark).getLink(Link.REL_SELF);

                    return ResponseEntity.created(URI
                        .create(forOneBookmark.getHref()))
                        .build();
                })
                .orElse(ResponseEntity.noContent().build());
    }

    @RequestMapping(method = RequestMethod.GET, value = "/{bookmarkId}")
    BookmarkResource readBookmark(Principal principal, @PathVariable Long bookmarkId) {
        this.validateUser(principal);
        return new BookmarkResource(
            this.bookmarkRepository.findOne(bookmarkId));
    }

    private void validateUser(Principal principal) {
        String userId = principal.getName();
        this.accountRepository
            .findByUsername(userId)
            .orElseThrow(
                () -> new UserNotFoundException(userId));
    }
}
```

唯一的改变是从URI中删除`{userId}`，让Spring Security把`Principal`注入到各种方法。

`UserNotFoundException`和`BookmarkControllerAdvice`和原来一样。

想要以安全的方式运行？

```bash
$ cd security
$ mvn clean spring-boot:run
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::

2016-11-04 11:54:21.135  INFO 73989 --- [           main] bookmarks.Application                    : Starting Application on retina with PID 73989 (/Users/gturnquist/src/tutorials/tut-bookmarks/security/target/classes started by gturnquist in /Users/gturnquist/src/tutorials/tut-bookmarks/security)
2016-11-04 11:54:21.137  INFO 73989 --- [           main] bookmarks.Application                    : No active profile set, falling back to default profiles: default
2016-11-04 11:54:21.181  INFO 73989 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@3db919e6: startup date [Fri Nov 04 11:54:21 CDT 2016]; root of context hierarchy
2016-11-04 11:54:22.116  WARN 73989 --- [           main] o.s.c.a.ConfigurationClassPostProcessor  : Cannot enhance @Configuration bean definition 'org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerEndpointsConfiguration$TokenKeyEndpointRegistrar' since its singleton instance has been created too early. The typical cause is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor return type: Consider declaring such methods as 'static'.
2016-11-04 11:54:22.392  INFO 73989 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [class org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$240a3410] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
...
```

使用Spring Security OAuth，我们可以对API进行一些修改，让用户标识贯穿在整个API的URI没有任何意义。毕竟，用户上下文已经暗示了它在我们端点的安全访问中，安全Spring MVC处理程序方法不需要在路径中使用`userId`，而是希望Spring Security为我们注入`javax.security.Principal`。这个类提供了一个可以替代userId的`javax.security.Principal#getName`方法。

## 结论

和无处不在、不同的客户端进行通信，REST是最自然的方式，它是基于HTTP协议工作的。一旦您了解REST如何在应用程序中实现，就可以设想一个具有众多单独关注统一接口的API架构，这些API通过REST的方式暴露出来，并且像任何其他HTTP服务一样实现负载均衡。因此，REST（通常但不一定）是微服务的关键并不奇怪，我们将在未来的教程中做更深入的研究。

