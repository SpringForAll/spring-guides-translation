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

