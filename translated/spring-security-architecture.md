# Spring Security 架构

> 原文：[Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture/)
>
> 译者：[徐靖峰](https://github.com/lexburner)
>
> 校对：马超君

**专题指南**

本文是 Spring Security 的入门指南，并对 Spring Security 的框架设计和基础组件进行深度解析。我们仅涉及应用程序安全性的基础知识，但这已足够消除开发人员在使用 Spring Security 时遇到的一些困惑。要做到这一点，我们需要了解如何使用过滤器和方法注解来保障Web应用程序的安全性。如果你需要了解高级别安全应用程序的工作方式，以及如何定制安全应用程序，或只需要学习如何思考应用程序的安全性，请使用本指南。

本指南不是用于解决基本问题的手册（有其他来源的手册用于解决基本问题），但对初学者和专家来说可能是有用的。 Spring Boot 在本文中也经常被提及，因为它为安全应用程序提供了一些默认的配置，了解它如何与整个体系结构相适应是非常有用的。 所有这些原则同样适用于不使用 Spring Boot 的应用程序。

## 身份认证和访问控制

应用程序安全性可以归结为差不多两个独立的问题：身份验证（你是谁？）和授权（你可以做什么？）。有时候，人们会说“访问控制”而不是“授权”，“授权”会让人感到困惑，可以这样想：“授权”在其他地方已经被使用，为了避免歧义而用“访问控制”来描述。 Spring Security 有一个旨在将认证与授权分开的体系结构，并兼备多种策略和扩展点。

### 认证

认证策略的核心接口是 `AuthenticationManager` ，它只有唯一的方法:

```java
public interface AuthenticationManager {

  Authentication authenticate(Authentication authentication)
    throws AuthenticationException;

}
```

一个 `AuthenticationManager` 认证管理者可能会在`authenticate()`方法中做下面三件事中的任意一个:

1. 如果认证成功，返回 `Authentication` (通常它的authenticated属性为true `authenticated=true`) .
2. 如果认证失败，抛出 `AuthenticationException`  异常.
3. 如果无法判断，返回 `null` .

`AuthenticationException` 是一个运行时异常。通常由应用程序以通用方式处理，具体取决于应用程序的风格或目的。 换句话说，用户代码通常不会捕获和处理。 例如，Web UI会呈现一个页面，表示认证失败，并且后端HTTP服务将发送401响应，可能包含 `WWW-Authenticate` 标头，具体取决于上下文。

`AuthenticationManager` 最常用的实现是 `ProviderManager`，它委托给一个`AuthenticationProvider` 实例链。 `AuthenticationProvider`有点像`AuthenticationManager`，但它有一个额外的方法来允许调用者询问它是否支持给定的认证类型

```java
public interface AuthenticationProvider {

	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;

	boolean supports(Class<?> authentication);

}
```

`supports()` 方法中的`Class <?>`参数实际上是 `Class<? extends Authentication>`（它只会被问到是否支持将被传递到 `authenticate()` 方法的东西）。 一个 `ProviderManager` 可以通过委托给一个 `AuthenticationProviders` 链来支持同一个应用程序中的多个不同认证机制。 如果一个 ProviderManager 不能识别一个特定的 `Authentication` 类型，它将被跳过。

`ProviderManager` 可以有一个父类认证器，如果所有的提供者返回null，则将再交给父类去认证。 如果父类不可用，则会导致 `AuthenticationException`。

有时应用程序具有受保护资源的逻辑组（例如所有与路径模式/ api / **相匹配的Web资源），并且每个组可以具有其自己的专用 `AuthenticationManager`。 通常，每个人都是一个 `ProviderManager`，他们共享一个父类。 父母是一种“全局”资源，充当所有提供者的失败回调。

![ProviderManagers with a common parent](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/authentication.png)

图 1.  `AuthenticationManager` 使用 `ProviderManager`

### 自定义身份验证管理器

Spring Security 提供了一些配置帮助类来快速获得应用程序中设置的通用身份验证管理器功能。 最常用的帮助类是 `AuthenticationManagerBuilder`，它非常适用于设置内存，JDBC或LDAP 中的用户详细信息，或添加自定义的`UserDetailsService`。 以下是配置全局（父类）`AuthenticationManager`的应用程序示例：

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

   ... // web stuff here

  @Autowired
  public initialize(AuthenticationManagerBuilder builder, DataSource dataSource) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

这个例子涉及到一个 Web 应用程序，但是 `AuthenticationManagerBuilder` 的使用更广泛地适用（更多细节请参见下面关于Web应用程序安全性的实现）。 请注意，`AuthenticationManagerBuilder` 是 `@Autowired` 到 `@Bean` 中的一个方法 - 使用它构建全局（父类）`AuthenticationManager`。 相反，如果我们这样做：

```java
@Configuration
public class ApplicationSecurity extends WebSecurityConfigurerAdapter {

  @Autowired
  DataSource dataSource;

   ... // web stuff here

  @Override
  public configure(AuthenticationManagerBuilder builder) {
    builder.jdbcAuthentication().dataSource(dataSource).withUser("dave")
      .password("secret").roles("USER");
  }

}
```

（在配置器中使用`@Override` 覆盖一个方法），那么 `AuthenticationManagerBuilder` 只用来构建一个“本地” `AuthenticationManager`，它是全局认证器的一个子实现。 在 Spring Boot 应用程序中，您可以 `@Autowired` 将全局认证器变成另一个bean，除非你自己明确暴露，否则不能使用本地变量。

Spring Boot 提供了一个默认的全局 `AuthenticationManager`（只有一个用户），除非你提供自定义`AuthenticationManager`类型的bean。 默认是足够安全的，不必担心太多，除非你主动需要一个自定义的全局`AuthenticationManager`。 如果你做任何构建 `AuthenticationManager`的配置，你可以本地化配置你保护的资源，而不用担心影响全局缺省。

### Authorization or Access Control

一旦认证成功，我们可以继续进行授权，这里的核心策略是 `AccessDecisionManager`。 框架提供了三个实现，并将所有三个委托连接到一个 `AccessDecisionVoter` 链，有点类似于 `ProviderManagerdelegates` 到`AuthenticationProviders`。

一个 `AccessDecisionVoter` 考虑一个认证（代表一个委托人）和一个安全的对象，它被`ConfigAttributes`装饰：

```java
boolean supports(ConfigAttribute attribute);

boolean supports(Class<?> clazz);

int vote(Authentication authentication, S object,
        Collection<ConfigAttribute> attributes);
```

对象在 `AccessDecisionManager` 和 `AccessDecisionVoter` 的签名中是完全通用的 - 它表示用户可能想要访问的任何内容（Web资源或Java类中的方法是最常见的两种情况）。 `ConfigAttributes` 也是相当通用的，用一些元数据表示安全对象的装饰，这些元数据决定了访问它所需的权限级别。 `ConfigAttribute` 是一个接口，但它只有一个非常通用的方法，并返回一个 `String`，所以这些字符串以某种方式编码资源所有者，表达允许访问规则。典型的 `ConfigAttribute` 是一个用户角色的名字（比如 `ROLE_ADMIN` 或者 `ROLE_AUDIT` ），它们通常有特殊的格式（比如ROLE_前缀）或者表示需要计算的表达式。

大多数人只是使用默认的 `AccessDecisionManager`，即`AffirmativeBased`（如果没有选举者拒绝，则授予访问）。任何自定义的配置都倾向于发生在选举者身上，要么增加新的自定义配置，要么改变现有的工作方式。

使用Spring表达式语言（SpEL）表达式的 `ConfigAttributes` 是很常见的，例如 `isFullyAuthenticated()&& hasRole('FOO')`。这由 `AccessDecisionVoter` 支持，可以处理表达式并为它们创建一个上下文。要扩展可以处理的表达式的范围，需要自定义实现 `SecurityExpressionRoot`，有时还需要 `SecurityExpressionHandler`。

## Web 安全

Web层中的Spring Security（用于UI和HTTP后端）基于Servlet过滤器，所以首先查看过滤器的作用是很有帮助的。 下图显示了单个HTTP请求的处理程序的典型分层结构。

![Filter chain delegating to a Servlet](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/filters.png)客户端向应用程序发送一个请求，容器根据请求URI的路径决定哪些过滤器和哪个servlet适用于它。最多一个servlet可以处理单个请求，但是过滤器形成一个链，所以它们是有序的，事实上，如果一个过滤器想要单独处理请求，过滤器可以否决链的其余部分。过滤器还可以修改在下游过滤器和servlet中使用的请求和/或响应。过滤器链的顺序是非常重要的，Spring Boot通过两种机制来管理它：一种是Filter类型的 `@Beans` 可以有 `@Order` 或实现`Ordered`，另一种是它们可以是 `FilterRegistrationBean` 本身的Order属性作为其API的一部分。一些现成的过滤器定义了自己的常量，以帮助表示它们喜欢相互之间的顺序（例如，来自 Spring Session 的`SessionRepositoryFilter` 具有默认顺序： `Integer.MIN_VALUE + 50`，这告诉我们它一般位于链的前端，但不排除在它之前存在其他过滤器）。

Spring Security 作为一个单独的过滤器安装在链中，其配置类型为 `FilterChainProxy`，原因很快很快就会被揭示。在Spring Boot应用程序中，安全过滤器是ApplicationContext中的`@Bean`，并具有默认配置，以便将其应用于每个请求。它被安装在由 `SecurityProperties.DEFAULT_FILTER_ORDER` 定义的位置，而该位置又由`FilterRegistrationBean.REQUEST_WRAPPER_FILTER_MAX_ORDER`（Spring Boot应用程序在包装请求时修改其行为的期望过滤器的最大顺序）决定。除此之外还有更多的内容：从容器的角度来看，Spring Security是一个单一的过滤器，但里面还有额外的过滤器，每个过滤器都扮演着特殊的角色。这是一张图片：

![Spring Security Filter](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters.png)

图 2. Spring Security 是一个单独的 `Filter` 但代理执行了一个内部的过滤器链

事实上，在安全过滤器中甚至还有一层间接寻址：它通常作为`DelegatingFilterProxy`安装在容器中，而不必是Spring 的 Bean。代理委托给一个 `FilterChainProxy`，通常使用固定的名称：`springSecurityFilterChain`。 `FilterChainProxy` 包含所有安全逻辑，内部安排为过滤器的一个或多个链。所有的过滤器都有相同的API（他们都实现了Servlet规范中的Filter接口），他们都有机会否决链的其余部分。

在同一个顶级`FilterChainProxy`中，可以有多个由 Spring Security 管理的过滤器链，并且容器都是未知的。 Spring Security筛选器包含一个筛选器链列表，并向与之匹配的第一个链派发一个请求。下图显示了匹配请求路径（`/foo/**` 在 `/**` 之前匹配）的转发情况。这是非常普遍的，但不是匹配请求的唯一方法。这个调度过程最重要的特点是只有一个链处理请求。

![Security Filter Dispatch](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/security-filters-dispatch.png)

图 3.   `Spring Security FilterChainProxy` 向第一个匹配的过滤器链转发请求.

没有自定义安全配置的Spring Boot应用程序有 n 个过滤器链，通常n = 6。 第一个链只是为了忽略静态资源，如`/css/**`和`/images/**`，错误视图/错误（路径可以通过`SecurityProperties` 中的 `security.ignored` 属性由用户来控制）。 最后一个链匹配所有路径`/**`，并且更加强大，包含认证，授权，异常处理，会话处理，头文件写等逻辑。默认情况下，链中总共有11个过滤器，但通常情况下 用户不必关心使用哪个过滤器以及何时使用过滤器。

> Note
>
> Spring Security内部的所有过滤器对于容器是未知的，这一点非常重要，尤其是在Spring Boot应用程序中，默认情况下，Filter类型的所有@Beans都会自动注册到容器中。 因此，如果你想要将自定义过滤器添加到安全链，则需要将其设置为@Bean，或者将其包装在明确禁用容器注册的FilterRegistrationBean中。

### 创建和自定义过滤器链

Spring Boot 应用程序（具有`/ **`请求匹配程序的应用程序）中的默认失败回调过滤器链具有预定义的`SecurityProperties.BASIC_AUTH_ORDER` 顺序。 你可以通过设置`security.basic.enabled = false`将其完全关闭，或者可以将其用作备用，只需定义其他较低顺序的规则即可。 要做到这一点，只需添加一个类型为`WebSecurityConfigurerAdapter`（或`WebSecurityConfigurer`）的`@Bean`，然后用`@Order`装饰类。 例：

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```

这个bean将使Spring Security添加一个新的过滤器链，并在回调之前对其进行排序。

对于一组资源，许多应用程序具有完全不同的访问规则。 例如，托管UI和支持API的应用程序可能支持基于cookie的身份验证，重定向到UI的登录页面，以及基于令牌的身份验证，对未经身份验证的API部件请求进行401响应。 每一组资源都有自己的`WebSecurityConfigurerAdapter`，它具有唯一的顺序和自己的请求匹配器。 如果匹配规则重叠，则最早排序的过滤器链将获胜。

### 请求匹配分发和授权

安全过滤器链（或等同于`WebSecurityConfigurerAdapter`）具有请求匹配器，用于决定是否将其应用于HTTP请求。 一旦决定采用特定的过滤器链，则不会应用其他过滤器。 但是在一个过滤链中，通过在HttpSecurity配置器中设置额外的匹配器，可以对授权进行更细粒度的控制。 例：

A security filter chain (or equivalently a `WebSecurityConfigurerAdapter`) has a request matcher that is used for deciding whether to apply it to an HTTP request. Once the decision is made to apply a particular filter chain, no others are applied. But within a filter chain you can have more fine grained control of authorization by setting additional matchers in the `HttpSecurity` configurer. Example:

```java
@Configuration
@Order(SecurityProperties.BASIC_AUTH_ORDER - 10)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
      .authorizeRequests()
        .antMatchers("/foo/bar").hasRole("BAR")
        .antMatchers("/foo/spam").hasRole("SPAM")
        .anyRequest().isAuthenticated();
  }
}
```

配置 Spring Security 最容易犯的一个错误是忘记这些匹配器适用于不同的进程，一个是整个过滤器链的请求匹配器，另一个只是选择应用的访问规则。

### 将应用安全规则与Actuator 相结合

如果你使用Spring Boot Actuator作为管理端点，你可能希望它们是安全的，默认情况下它们是。 事实上，只要将执行器添加到安全的应用程序中，您就会得到一个仅适用于执行器端点的附加过滤器链。 它是由一个请求匹配器定义的，它只匹配执行器端点，它的顺序是`ManagementServerProperties.BASIC_AUTH_ORDER`，比默认的`SecurityProperties`过滤器小5，所以在回调之前会被查询。

如果您希望您的应用程序安全规则适用于执行器端点，则可以添加一个比执行器更早的过滤器链，以及包含所有执行器端点的请求匹配器。 如果您更喜欢执行器端点的默认安全设置，那么最简单的方法是在执行器之后添加自己的过滤器，但早于回调（例如`ManagementServerProperties.BASIC_AUTH_ORDER + 1`）。 例：

```java
@Configuration
@Order(ManagementServerProperties.BASIC_AUTH_ORDER + 1)
public class ApplicationConfigurerAdapter extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.antMatcher("/foo/**")
     ...;
  }
}
```

> Note
>
> Web层中的Spring Security目前与Servlet API绑定在一起，因此只有在servlet容器中运行应用程序（嵌入式或其他方式）时才是真正适用的。 但是，它并不是绑定到Spring MVC或Spring Web堆栈的其余部分，所以它可以用在任何servlet应用程序中，例如使用JAX-RS的应用程序。

## 方法安全

除了支持保护Web应用程序，Spring Security还支持将访问规则应用于Java方法。 对于Spring Security来说，这只是一种不同类型的“受保护的资源”。 对于用户来说，这意味着使用相同格式的`ConfigAttribute`字符串（例如角色或表达式）来声明访问规则，但是在代码中具有不同的配置。 第一步是启用方法安全配置，例如在我们的应用程序的顶级配置中：

```java
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SampleSecureApplication {
}
```

然后，我们可以直接修饰方法资源，例如:

```java
@Service
public class MyService {

  @Secured("ROLE_USER")
  public String secure() {
    return "Hello Security";
  }

}
```

这个示例是一个安全方法的服务。 如果 Spring 创建了这种类型的 `@Bean`，那么它将被代理，调用者必须在方法被实际执行之前通过一个安全拦截器。 如果访问被拒绝，调用者将得到一个 `AccessDeniedException` 而不是实际的方法结果。

还有其他的注解可以用于强制执行安全约束的方法，特别是@PreAuthorize和@PostAuthorize，它们允许你编写包含对方法参数和返回值分别引用的表达式。

> Tip
>
> 将Web安全性和方法安全性结合起来并不罕见。 过滤器链提供用户体验功能，如身份验证和重定向到登录页面等，方法安全性提供更细粒度的保护。

## 与线程协同工作

Spring Security基本上是线程绑定的，因为它需要使当前的身份验证委托人可用于各种下游消费者。 基本构建块是SecurityContext，其中可能包含一个身份验证（并且当用户登录时它将是一个明确验证的身份验证）。 你总是可以通过SecurityContextHolder中的静态便利方法访问和操作SecurityContext，而后者只需操作一个TheadLocal，

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
assert(authentication.isAuthenticated);
```

用户应用程序代码执行此操作并**不**常见，但如果您需要编写自定义身份验证筛选器（尽管Spring Security中有基类可用于避免需要的地方 使用 `SecurityContextHolder`）。

如果你需要访问Web端点中当前已通过身份验证的用户，则可以在 `@RequestMapping` 中使用方法参数。 例如。

```java
@RequestMapping("/foo")
public String foo(@AuthenticationPrincipal User user) {
  ... // do stuff with user
}
```

这个注解将当前Authentication从SecurityContext中抽出，并调用其上的 `getPrincipal()` 方法来产生方法参数。 认证中的委托人类型取决于用于验证认证的认证管理器，所以这对于获得对用户数据的类型安全引用是一个有用的小技巧。

如果使用Spring Security，则HttpServletRequest中的`Principal`将是`Authentication`类型，因此也可以直接使用它：

```java
@RequestMapping("/foo")
public String foo(Principal principal) {
  Authentication authentication = (Authentication) principal;
  User = (User) authentication.getPrincipal();
  ... // do stuff with user
}
```

如果你需要编写在没有使用Spring Security的情况下工作的代码，那么这有时候会很有用（你需要在加载`Authentication`类时更加谨慎）。

### 异步安全配置

由于 `SecurityContext` 是线程绑定的，因此如果要执行任何调用安全方法的后台处理，例如与`@Async`，你需要确保上下文传播。 这归结为将 `SecurityContext` 包装在后台执行的任务（Runnable，Callable，etc）中。 Spring Security 提供了一些帮助器，使之变得简单，比如Runnable和Callable的包装器。 要将 `SecurityContext` 传播到`@Async`方法，你需要提供一个 `AsyncConfigurer` 并确保 `Executor` 的类型正确：

```java
@Configuration
public class ApplicationConfiguration extends AsyncConfigurerSupport {

  @Override
  public Executor getAsyncExecutor() {
    return new DelegatingSecurityContextExecutorService(Executors.newFixedThreadPool(5));
  }

}
```

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



