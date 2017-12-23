# Spring Boot 与 OAuth2

> 原文：[Spring Boot and OAuth2](https://spring.io/guides/tutorials/spring-boot-oauth2/)
>
> 译者：[nycgym](https://github.com/nycgym)
>
> 校对：

本指南将向你展示如何使用[OAuth2](https://tools.ietf.org/html/rfc6749)和[Spring Boot](https://projects.spring.io/spring-boot/)构建的具有“社交登录”功能的应用程序去做完成各种事情。它从一个简单单点登录开始，运行一个自我托管的OAuth2授权服务器，此服务器带有一个身份验证提供者（[Facebook](https://developers.facebook.com/)或[Github](https://developer.github.com/)）。这些示例它们都在前端使用了普通的[jQuery](https://jquery.org/)，但是转换到不同的JavaScript框架或使用服务器端渲染的改动将非常小。

在每个添加新功能的例子中都有以下特点:

简单：一个非常基本的静态应用程序只有一个主页，并通过Spring Boot的`EnableOAuth2Sso`无条件登录（如果你访问主页，你将自动重定向到Facebook）。

点击：添加用户必须单击才能登录的显式链接。

登出：为通过身份验证的用户添加了登出链接。

手动配置：通过取消选中并手动配置来展示`@EnableOAuth2Sso`是如何工作的。

GitHub：在Github中添加了第二个登录提供方，以便用户可以在主页上选择使用哪一个。

认证服务：将应用程序变成一个完全成熟的OAuth2授权服务器，能够发出自己的令牌，但仍然使用外部OAuth2提供程序进行身份验证。

自定义错误：为未经身份验证的用户添加错误消息，并基于Github API添加自定义身份验证。
    
>从一个应用程序迁移到功能阶梯的下一个应用程序所需要的更改可以在源代码中跟踪(源代码在Github中)。存储库中的前6个更改正在转变一个应用程序，这样你就可以很容易地看到差异。
  在早期提交的应用程序和在指南中看到的最终程序，你看到的任何进一步的差异都是修饰性的。
    
它们中的每一个都可以被导入到一个IDE中，并且有一个可以在那里运行的主类`SocialApplication`来启动应用程序。 他们都在[http//localhost8080](http//localhost8080)上提供了一个主页（如果你想登录并查看内容，所有这些都需要你至少有一个Facebook帐户）。你也可以使用`mvn spring-boot:run`或通过构建jar文件并使用`mvn package`和`java -jar target / *.jar`（根据[Spring Boot](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-first-application-run)文档和[其他可用文档](https://spring.io/guides/gs/spring-boot/)）运行命令行中的所有应用程序。
如果你使用了[maven-wrapper](https://github.com/takari/maven-wrapper)，则不需要安装Maven。比如：

    $ cd simple
    $ ../mvnw package
    $ java -jar target/*.jar

>这些应用程序都在`localhost8080`上运行，因为他们使用在Facebook和Github上注册的OAuth2客户端来访问该地址。 要在不同的主机或端口上运行它们，你需要注册自己的应用程序，并将凭据放在配置文件中。 如果你使用默认值，则不会在本地主机之外泄漏你的Facebook或Github凭据，但要小心你在互联网上公开的内容，并且不要将你自己的应用程序注册置于公共源代码管理中。

## 用FaceBook做单点登录

在本节中，我们创建一个使用Facebook进行身份验证的应用程序。如果我们利用Spring Boot中的自动配置功能，这一过程将相当容易。

### 创建一个新的工程

首先，我们需要创建一个Spring Boot应用程序，可以通过多种方式来完成。 最简单的是去http://start.spring.io并生成一个空的项目（选择“Web”依赖项作为起点）。相当于于在命令行上执行此操作：

    $ mkdir ui && cd ui
    $ curl https://start.spring.io/starter.tgz -d style=web -d name=simple | tar -xzvf -
    
然后，你可以将该项目导入到你最喜欢的IDE中（默认情况下，这是一个普通的Maven Java项目），或者只是在命令行上配合“mvn”使用这些文件。

### 添加主页

    index.html
    <!doctype html>
    <html lang="en">
    <head>
        <meta charset="utf-8"/>
        <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
        <title>Demo</title>
        <meta name="description" content=""/>
        <meta name="viewport" content="width=device-width"/>
        <base href="/"/>
        <link rel="stylesheet" type="text/css" href="/webjars/bootstrap/css/bootstrap.min.css"/>
        <script type="text/javascript" src="/webjars/jquery/jquery.min.js"></script>
        <script type="text/javascript" src="/webjars/bootstrap/js/bootstrap.min.js"></script>
    </head>
    <body>
    	<h1>Demo</h1>
    	<div class="container"></div>
    </body>
    </html>

这些对于演示OAuth2登录功能来说都不是必须的，但是我们希望最终能有一个好看的用户界面，所以我们不妨从构造主页中一些基本的东西开始。

如果你启动应用程序并加载主页，则会注意到样式尚未加载。所以我们也需要添一些依赖：

    pom.xml
    <dependency>
    	<groupId>org.webjars</groupId>
    	<artifactId>jquery</artifactId>
    	<version>2.1.1</version>
    </dependency>
    <dependency>
    	<groupId>org.webjars</groupId>
    	<artifactId>bootstrap</artifactId>
    	<version>3.2.0</version>
    </dependency>
    <dependency>
    	<groupId>org.webjars</groupId>
    	<artifactId>webjars-locator</artifactId>
    </dependency>
    
我们添加了Twitter bootstrap和jQuery（这就是我们现在所需要的） 另一个依赖是webjars“定位器”，它由webjars站点作为提供库，Spring可以使用这个定位器在webjars中定位静态资源，而不需要知道确切的版本（因此versionless `/webjars/**`链接 在`index.html`中）。 只要不关闭MVC自动配置，webjar定位器在Spring Boot应用程序中默认激活。

在做了以上改变，我们应用程序的主页应该更加美观了。

### 保护应用程序

为了使应用程序安全，我们只需要添加Spring Security作为依赖。如果我们这样做，默认情况下是使用HTTP Basic来保护它，所以既然我们想做一个“社交”登录（委托给Facebook），我们也添加了Spring Security OAuth2依赖项：

    pom.xml
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
    	<groupId>org.springframework.security.oauth</groupId>
    	<artifactId>spring-security-oauth2</artifactId>
    </dependency>
    
为了能够链接到FaceBook我们需要在主类添加注解`@EnableOAuth2Sso`

    SocialApplication.java
    @SpringBootApplication
    @EnableOAuth2Sso
    public class SocialApplication {
    
      ...
    
    }
    
还有一些配置(这里把`application.properties`转换为YAML形式，这样可读性强)

    application.yml
    security:
      oauth2:
        client:
          clientId: 233668646673605
          clientSecret: 33b17e044ee6a4fa383f46ec6e28ea1d
          accessTokenUri: https://graph.facebook.com/oauth/access_token
          userAuthorizationUri: https://www.facebook.com/dialog/oauth
          tokenName: oauth_token
          authenticationScheme: query
          clientAuthenticationScheme: form
        resource:
          userInfoUri: https://graph.facebook.com/me
    ...

该配置是指向Facebook[开发者站点](https://developers.facebook.com/)注册的客户端应用程序，其中你必须为该应用程序提供注册的重定向（主页）。这个注册到“localhost:8080”，所以只有运行在该地址上的应用才能生效。

做了以上改变，你可以再次运行应用程序，并访问`http//localhost:8080`的主页。接下来你应该重定向到Facebook登录而不是主页。如果你登陆了并同意你要求的任何授权，你将被重定向回到本地应用程序并且可以看到主页。如果你保持登录到Facebook，即使你使用新的浏览器不使用Cookie和缓存数据打开它也不必使用本地应用重新进行身份验证。（这就是单点登录）

>如果你正在做示例应用程序的这一部分，请务必清除你的Cookie和HTTP Basic凭据的浏览器缓存。在Chrome中，最好在访问每个服务器主业的时候打开一个新的隐身窗口。

对这个示例进行访问是安全的，因为只有本地运行的应用程序可以使用令牌并且它要求的范围是有限的。当你登录到这样的应用程序时，要注意你所批准的内容:他们可能会要求你做更多的事情(比如他们可能会要求你更改你的个人资料，这可能不符合你的利益)。

### 刚刚发生了什么？

你刚刚用OAuth2的编写的应用程序是一个客户端应用程序，它使用[授权代码授权](https://tools.ietf.org/html/rfc6749#section-4)从Facebook（授权服务器）获取访问令牌。 然后，它使用访问令牌向Facebook询问一些个人信息（仅限于你允许的内容），包括你的登录ID和你的姓名。 在这个阶段，facebook充当了一个资源服务器，对你发送的令牌进行解码，并检查它给了应用程序访问用户详细信息的权限。如果该过程成功，则应用程序将用户详细信息插入到Spring Security上下文中，以便进行身份验证。

如果你用浏览器工具（Chrome上的F12），追踪查看所有路由的网络流量操作，你将看到Facebook的重定向，最后用新的`Set-Cookie`请求头返回主页。 这个cookie（默认情况下是`JSESSIONID`）是Spring（或任何基于servlet的）应用程序的认证细节的一个标记。

所以我们有一个安全的应用程序，用户查看任何内容必须通过外部供应服务器（Facebook）认证。 我们不希望将其用于网上银行网站，而是用于基本的身份识别，并将网站内的不同用户之间的内容隔离开来，这是一个很好的开端，这就解释了为什么这种认证现在非常流行。在下一节中，我们将为应用程序添加一些基本功能，并且使用户更清楚的看到最初重定向到Facebook时发生的事情。

### 添加一个欢迎页面

在本节中，我们将修改我们刚刚构建的应用程序，通过添加一个显式的链接登录Facebook。新的链接不会立即被重定向，而是可以在主页上看到，用户可以选择登录或不经过身份验证。只有当用户点击了链接，他才会显示内容。

#### 主页中受保护的内容

我们可以使用服务器端渲染页面（例如，使用Freemarker或Tymeleaf）通过用户是否通过验证来确定其是否可访问受保护的内容，或者我们可以使用一些JavaScript请求浏览器。 要做到这一点，我们要使用[AngularJS](https://angularjs.org/)，但是如果你更喜欢使用不同的框架，你可把客户端的代码转换成其他框架。

要开始使用动态内容，我们需要在部分内容中标记HTML，以显示它:

    index.html
    <div class="container unauthenticated">
        With Facebook: <a href="/login">click here</a>
    </div>
    <div class="container authenticated" style="display:none">
        Logged in as: <span id="user"></span>
    </div>
    
这个HTML满足了一些客户端代码的需求，这些客户端代码可以操作`authenticated`、`unauthenticated`和`user`元素。下面是这些功能的简单实现(将它们放在`<body>`的末尾):
    
    index.html
    <script type="text/javascript">
        $.get("/user"· function(data) {
            $("#user").html(data.userAuthentication.details.name);
            $(".unauthenticated").hide()
            $(".authenticated").show()
        });

### 服务器端的改动

为此，我们需要在服务器端进行一些更改。“home”控制器需要一个描述当前身份验证用户的“/user”端点。这很容易做到，例如在我们的主类:

    SocialApplication
    @SpringBootApplication
    @EnableOAuth2Sso
    @RestController
    public class SocialApplication {
    
      @RequestMapping("/user")
      public Principal user(Principal principal) {
        return principal;
      }
    
      public static void main(String[] args) {
        SpringApplication.run(SocialApplication.class, args);
      }
    
    }
    
注意`@restcontroller`和`@requestmapping`和`java.security`的使用。我们将其注入到了处理方法中。

>在/user端点中返回一个完整的用户信息主体不是一个好主意(它可能包含你不愿向浏览器客户机显示的信息)。我们这样做只是为了让应用尽快正常运行。在后面的指南中，我们将转换端点来隐藏浏览器不需要的信息。

这个应用程序现在可以像以前一样正常运行了，但是用户还不能点击我们刚刚新添加的链接。为了使链接可见，我们还需要通过添加`WebSecurityConfigurer`来关闭主页上一些不必要的安全保护:

    SocialApplication
    @SpringBootApplication
    @EnableOAuth2Sso
    @RestController
    public class SocialApplication extends WebSecurityConfigurerAdapter {
    
      ...
    
      @Override
      protected void configure(HttpSecurity http) throws Exception {
        http
          .antMatcher("/**")
          .authorizeRequests()
            .antMatchers("/", "/login**", "/webjars/**")
            .permitAll()
          .anyRequest()
            .authenticated();
      }
    
    }
    
完成以上步骤，我们的应用就完整了，如果你运行它并访问主页，该看到一个美观的的HTML链接“登录到Facebook”。链接不会直接传送到Facebook，而是定位到处理身份验证的本地路径（并将重定向发送到Facebook）。一旦你通过身份验证，你会被重定向回到本地的应用程序，本地应用将会显示你的名字（假设你已经在Facebook上设置了允许访问这些数据的权限）。

### 添加登出按钮

在本节中,我们修改了应用通过添加一个按钮,允许用户退出程序。这似乎是一个简单的功能,但实际上需要仔细考虑它的实现,所以它值得花一些时间讨论如何去做。大多数改动都是由于我们正在将应用程序从只读资源转换为读写操作(注销需要状态更改)，因此在任何实际应用程序中都需要相同的更改，而不仅仅是静态内容。

#### 客户端改动

在客户端，我们只需要提供一个注销按钮和一些JavaScript，以调用服务器请求取消身份验证。首先，在UI的“验证”部分中，我们添加按钮:

    index.html
    <div class="container authenticated">
      Logged in as: <span id="user"></span>
      <div>
        <button onClick="logout()" class="btn btn-primary">Logout</button>
      </div>
    </div>
    
接下来我们在JavaScript中提供`logout()`函数，以供调用：

    index.html
    var logout = function() {
        $.post("/logout", function() {
            $("#user").html('');
            $(".unauthenticated").show();
            $(".authenticated").hide();
        })
        return true;
    }

`logout()`函数执行一个POST `/logout`，然后屏蔽受保护内容。现在我们可以切换到服务器端来实现这个端点。

### 添加一个Logout端点

Spring Security已经构建了一个支持`/logout`的端点，它将为我们做正确的事情（清除会话并使Cookie无效）。 要配置端点，我们只需在`WebSecurityConfigurer`中扩展现有的`configure（)`方法：

    SocialApplication.java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http.antMatcher("/**")
        ... // existing code here
        .and().logout().logoutSuccessUrl("/").permitAll();
    }


340/5000
`/logout`端点需要我们用POST方法请求，并保护用户免受跨站点请求伪造（CSRF，发音为“sea surf”）攻击，它要求在请求中包含一个令牌。该令牌的值与当前提供保护的会话相关联，因此我们需要一种方法将这些数据放入到我们的JavaScript应用程序中。

许多JavaScript框架都支持CSRF（例如，在Angular中，他们称之为XSRF），但是它通常以与Spring Security的开箱即用方式稍有不同的方式实现。例如，在Angular中，前端希望服务器发送一个叫做“XSRF-TOKEN”的cookie，如果它看到的话，它会把这个值作为一个名为“X-XSRF-TOKEN”的请求头发回去。我们可以用简单的jQuery客户端来实现相同的行为，然后服务器端做很少的改动去与其他前端实现一起工作。为了让Spring Security适应这些，我们需要添加一个创建cookie的过滤器，同时我们还需要告诉现有的CRSF过滤器相应的请求头名。 在`WebSecurityConfigurer`中：

    SocialApplication.java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http.antMatcher("/**")
        ... // existing code here
        .and().csrf().csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }
    
### 在客户端添加CSRF令牌

由于我们在这个示例中没有使用封装更好的框架，所以我们需要显式地添加CSRF令牌，这是我们从后端提供的cookie。为了使代码更简单，我们还需要依赖一个额外的库:

    pom.xml
    <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>js-cookie</artifactId>
        <version>2.1.0</version>
    </dependency>
    
在HTML页面中引入：

    index.html
    <script type="text/javascript" src="/webjars/js-cookie/js.cookie.js"></script>
    
然后我们可以使用xhr中比较便利的`Cookies`方法:

    index.html
    $.ajaxSetup({
    beforeSend : function(xhr, settings) {
      if (settings.type == 'POST' || settings.type == 'PUT'
          || settings.type == 'DELETE') {
        if (!(/^http:.*/.test(settings.url) || /^https:.*/
            .test(settings.url))) {
          // Only send the token to relative URLs i.e. locally.
          xhr.setRequestHeader("X-XSRF-TOKEN",
              Cookies.get('XSRF-TOKEN'));
        }
      }
    }
    });

### 准备工作完成！

做了以上改动，我们可以准备运行应用程序，并尝试新的注销按钮。启动应用程序并在新的浏览器窗口中加载主页。点击“登录”链接将你带到Facebook（如果你已经登录，你可能不会注意到重定向）。点击“注销”按钮取消当前会话，并将应用程序返回到未认证状态。如果你足够细心，你应该能够在浏览器与本地服务器交换的请求中看到新的cookie和请求头。

请注意，现在logout端点与浏览器一起工作，那么所有其他HTTP请求(POST、PUT、DELETE等)也会正常工作。因此，对于一些具有更实际的特性的应用程序来说，这应该是一个很好的平台。

### 手动配置OAuth2客户端

在本节中，我们通过选择`@EnableOAuth2Sso`注释中的“magic”来修改我们已经构建的应用程序，手动配置其中的所有内容以使其显式化。

#### 客户端与认证

`@EnableOAuth2Sso`有两个特性:OAuth2客户端和身份验证。客户端是可重用的，因此你还可以使用它与你的授权服务器(在本例中是Facebook)提供的OAuth2资源进行交互(在本例中为[Graph API](https://developers.facebook.com/docs/graph-api))。认证件将你的应用与Spring安全的其他部分结合在一起，所以一旦你的应用程序与Facebook的同步，它就会和其他安全的Spring应用程序一样。

客户端是由Spring Security OAuth2提供的，并由一个不同的注释`@EnableOAuth2Client`开启。因此，这个改动的第一步是删除`@EnableOAuth2Sso`并将其替换为较低级别的注释:

    SocialApplication
    @SpringBootApplication
    @EnableOAuth2Client
    @RestController
    public class SocialApplication extends WebSecurityConfigurerAdapter {
      ...
    }
    
通过这样的更改，我们创建了一些有用的东西。首先，我们可以注入一个`OAuth2ClientContext`，并使用它来构建一个身份验证过滤器，以添加到我们的安全配置中:

    SocialApplication
    @SpringBootApplication
    @EnableOAuth2Client
    @RestController
    public class SocialApplication extends WebSecurityConfigurerAdapter {
    
      @Autowired
      OAuth2ClientContext oauth2ClientContext;
    
      @Override
      protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/**")
          ...
          .and().addFilterBefore(ssoFilter(), BasicAuthenticationFilter.class);
      }
    
      ...
    
    }
    
这个过滤器是在我们使用`OAuth2ClientContext`的新方法中创建的:

    SocialApplication
    private Filter ssoFilter() {
      OAuth2ClientAuthenticationProcessingFilter facebookFilter = new OAuth2ClientAuthenticationProcessingFilter("/login/facebook");
      OAuth2RestTemplate facebookTemplate = new OAuth2RestTemplate(facebook(), oauth2ClientContext);
      facebookFilter.setRestTemplate(facebookTemplate);
      UserInfoTokenServices tokenServices = new UserInfoTokenServices(facebookResource().getUserInfoUri(), facebook().getClientId());
      tokenServices.setRestTemplate(facebookTemplate);
      facebookFilter.setTokenServices(tokenServices);
      return facebookFilter;
    }
    
过滤器还需要知道客户端注册了Facebook:

    SocialApplication
      @Bean
      @ConfigurationProperties("facebook.client")
      public AuthorizationCodeResourceDetails facebook() {
        return new AuthorizationCodeResourceDetails();
      }
      
要完成身份验证，它需要知道用户信息端点在Facebook中的位置:

    SocialApplication
      @Bean
      @ConfigurationProperties("facebook.resource")
      public ResourceServerProperties facebookResource() {
        return new ResourceServerProperties();
      }
      
注意，在这些“静态”数据对象(`facebook()`和`facebookResource()`)中，我们使用了一个`@Bean`修饰的`@ConfigurationProperties`。这意味着我们可以创建`application.yml`的新格式，在这里，配置的前缀是`facebook`，而不是`security.oauth2`:

    application.yml
    facebook:
      client:
        clientId: 233668646673605
        clientSecret: 33b17e044ee6a4fa383f46ec6e28ea1d
        accessTokenUri: https://graph.facebook.com/oauth/access_token
        userAuthorizationUri: https://www.facebook.com/dialog/oauth
        tokenName: oauth_token
        authenticationScheme: query
        clientAuthenticationScheme: form
      resource:
        userInfoUri: https://graph.facebook.com/me
        
最后，我们需要将登录的路径改为在上面的过滤器声明中特定于facebook的，因此我们要在HTML中做出相同的更改:

    index.html
    <h1>Login</h1>
    <div class="container unauthenticated">
    	<div>
    	With Facebook: <a href="/login/facebook">click here</a>
    	</div>
    </div>
    
#### 处理重定向

我们需要做的最后一个改变是显式地支持从我们的应用程序到Facebook的重定向。这是在Spring OAuth2中使用servlet`Filter`处理的，并且过滤器已经在应用程序上下文中可用，因为我们使用了`@EnableOAuth2Client`。所需要的是将过滤器连接起来，以便在Spring Boot应用程序中以正确的顺序调用它。要做到这一点，我们需要一个`FilterRegistrationBean`:

    SocialApplication.java
    @Bean
    public FilterRegistrationBean oauth2ClientFilterRegistration(
        OAuth2ClientContextFilter filter) {
      FilterRegistrationBean registration = new FilterRegistrationBean();
      registration.setFilter(filter);
      registration.setOrder(-100);
      return registration;
    }
    
我们对已可用的过滤器进行自动连接，并在主Spring Security过滤器之前以较为靠后的顺序对其进行注册。通过这种方式，我们可以使用它来处理在身份验证请求中所表示的重定向。

做完以上改动，应用就可以很好的运行了，在运行时就相当于我们在上一节中构建的注销示例。通过这样将配置分解明确告诉我们Spring Boot所做的事情并没有什么神奇之处(它只是配置锅炉版)，而且它还提供了自动注入的功能让我们的应用程序继承，添加我们自己的代码和业务需求。

## 用GitHub登陆

在本节中，我们将修改已经构建好的应用程序，添加一个链接，这样除了原始的Facebook链接外，用户还可以选择使用Github进行身份验证。

### 增加Github链接

在客户端，更改很简单，我们只需添加另一个链接:

    index.html
    <div class="container unauthenticated">
      <div>
        With Facebook: <a href="/login/facebook">click here</a>
      </div>
      <div>
        With Github: <a href="/login/github">click here</a>
      </div>
    </div>
    
原则上，一旦我们开始添加身份验证提供者，我们可能需要对从“/user”端点返回的数据更加小心。事实证明，Github和Facebook在他们的用户信息中都有一个“name”字段，所以我们的端点没有任何变化。

### 增加Github认证过滤器

服务器上的主要变化是添加一个附加的安全过滤器来处理来自我们的新链接的“/login/github”请求。我们已经在我们的`ssoFilter()`方法中创建了一个用于Facebook的自定义验证过滤器，所以我们需要做的就是用一个可以处理多个身份验证路径的函数来替换它:

    SocialApplication.java
    private Filter ssoFilter() {
    
      CompositeFilter filter = new CompositeFilter();
      List<Filter> filters = new ArrayList<>();
    
      OAuth2ClientAuthenticationProcessingFilter facebookFilter = new OAuth2ClientAuthenticationProcessingFilter("/login/facebook");
      OAuth2RestTemplate facebookTemplate = new OAuth2RestTemplate(facebook(), oauth2ClientContext);
      facebookFilter.setRestTemplate(facebookTemplate);
      UserInfoTokenServices tokenServices = new UserInfoTokenServices(facebookResource().getUserInfoUri(), facebook().getClientId());
      tokenServices.setRestTemplate(facebookTemplate);
      facebookFilter.setTokenServices(tokenServices);
      filters.add(facebookFilter);
    
      OAuth2ClientAuthenticationProcessingFilter githubFilter = new OAuth2ClientAuthenticationProcessingFilter("/login/github");
      OAuth2RestTemplate githubTemplate = new OAuth2RestTemplate(github(), oauth2ClientContext);
      githubFilter.setRestTemplate(githubTemplate);
      tokenServices = new UserInfoTokenServices(githubResource().getUserInfoUri(), github().getClientId());
      tokenServices.setRestTemplate(githubTemplate);
      githubFilter.setTokenServices(tokenServices);
      filters.add(githubFilter);
    
      filter.setFilters(filters);
      return filter;
    
    }
    
我们的旧`ssoFilter()`的代码已经被复制，一次是用于Facebook，一次是Github，两个过滤器合并成一个复合过滤器。

请注意，相对于`github()`和`githubResource()`方法，我们已经添加了类似的方法`facebook()`和`facebookResource()`:

    SocialApplication.java
    @Bean
    @ConfigurationProperties("github.client")
    public AuthorizationCodeResourceDetails github() {
    	return new AuthorizationCodeResourceDetails();
    }
    
    @Bean
    @ConfigurationProperties("github.resource")
    public ResourceServerProperties githubResource() {
    	return new ResourceServerProperties();
    }
    
和相应的配置:

    application.yml
    github:
      client:
        clientId: bd1c0a783ccdd1c9b9e4
        clientSecret: 1a9030fbca47a5b2c28e92f19050bb77824b5ad1
        accessTokenUri: https://github.com/login/oauth/access_token
        userAuthorizationUri: https://github.com/login/oauth/authorize
        clientAuthenticationScheme: form
      resource:
        userInfoUri: https://api.github.com/user
        
这里的客户端详细信息是用[Github](https://github.com/settings/developers)注册的，地址也指向`localhost:8080`(与Facebook相同)。

现在，这个应用可以运行了，而且用户可以选择用Facebook登陆，或者Github登陆

### 如何添加本地用户数据库

即使身份验证被委托给外部提供者，许多应用程序也需要在本地保存其用户的数据。在这里我们没有展示代码，但是很容易通过两个步骤完成。

1.为数据库选择后端，并为自定义`User`对象设置一些存储库(例如，使用Spring Data)，该对象符合你的需求，并且可以通过外部验证服务器完成全部或部分身份验证。

2.通过检查`/User`端点中的数据库，为登录的每个唯一用户配置`User`对象。如果已存在具有当前主体`Principal`的用户，则可以更新该用户，否则将创建该用户。

提示:在`User`对象中添加一个字段以链接到外部提供程序中的唯一标识符(不是用户名，而是外部提供程序中帐户的唯一标志)。

## 托管授权服务器

在本节中，我们将修改我们构建的Github应用程序，使其成为一个成熟的oauth2授权服务器，仍然使用Facebook和Github进行身份验证，但能够创建自己的访问令牌。然后，可以使用这些令牌来保护后端资源，或者对我们碰巧需要以同样方式保护的其他应用程序执行SSO。

### 整理身份验证配置

在开始使用授权服务器功能之前，我们只需整理两个外部提供程序的配置代码。`ssoFilter()`方法中有一些重复的代码，因此我们将它们提取到共同方法中:

    SocialApplication.java
    private Filter ssoFilter() {
      CompositeFilter filter = new CompositeFilter();
      List<Filter> filters = new ArrayList<>();
      filters.add(ssoFilter(facebook(), "/login/facebook"));
      filters.add(ssoFilter(github(), "/login/github"));
      filter.setFilters(filters);
      return filter;
    }
    
新的方法具有旧方法的所有重复代码:

    SocialApplication.java
    private Filter ssoFilter(ClientResources client, String path) {
      OAuth2ClientAuthenticationProcessingFilter filter = new OAuth2ClientAuthenticationProcessingFilter(path);
      OAuth2RestTemplate template = new OAuth2RestTemplate(client.getClient(), oauth2ClientContext);
      filter.setRestTemplate(template);
      UserInfoTokenServices tokenServices = new UserInfoTokenServices(
          client.getResource().getUserInfoUri(), client.getClient().getClientId());
      tokenServices.setRestTemplate(template);
      filter.setTokenServices(tokenServices);
      return filter;
    }
    
并且它使用了一个新的包装对象`ClientResources`，它整合了`OAuth2ProtectedResourceDetails`和`ResourceServerProperties`，在应用程序的最后一个版本中，它们被声明为单独的`@Bean`:

    SocialApplication.java
    class ClientResources {
    
      @NestedConfigurationProperty
      private AuthorizationCodeResourceDetails client = new AuthorizationCodeResourceDetails();
    
      @NestedConfigurationProperty
      private ResourceServerProperties resource = new ResourceServerProperties();
    
      public AuthorizationCodeResourceDetails getClient() {
        return client;
      }
    
      public ResourceServerProperties getResource() {
        return resource;
      }
    }
    
>包装器使用`@NestedConfigurationProperty`通知注释处理器对该类型标记为元数据，它不表示单个值，而是完整的嵌套类型。

有了这个包装器，我们可以像以前一样使用相同的YAML配置，但一个方法只能给一个提供程序使用:

    SocialApplication.java
    
    @Bean
    @ConfigurationProperties("github")
    public ClientResources github() {
      return new ClientResources();
    }
    
    @Bean
    @ConfigurationProperties("facebook")
    public ClientResources facebook() {
      return new ClientResources();
    }
    
### 启用授权服务器

如果我们想把我们的应用程序变成oauth2授权服务器，不需要做得很麻烦，从一些基本功能(一个客户端和创建访问令牌的能力)开始。授权服务器不过是一堆端点，它们在Spring oaut 2中实现为SpringMVC handler。我们已经有了一个安全的应用程序，因此只需添加`@EnableuthorizationServer`注解即可:

    SocialApplication.java
    
    @SpringBootApplication
    @RestController
    @EnableOAuth2Client
    @EnableAuthorizationServer
    public class SocialApplication extends WebSecurityConfigurerAdapter {
    
       ...
    
    }
    
添加新注解后，Spring Boot将安装所有必要的端点并为其设置安全性，前提是我们提供了我们想要支持的OAuth2客户端的详细信息:

    application.yml
    security:
      oauth2:
        client:
          client-id: acme
          client-secret: acmesecret
          scope: read,write
          auto-approve-scopes: '.*'
          
此客户端相当于`facebook.client*`和`github.client*`所提供的外部身份验证。通过外部提供服务器，我们必须注册并获取客户端ID和在应用程序中使用的密码。在这种情况下，我们提供了相同的功能，因此我们需要(至少一个)客户端才能工作。

我们已经将`auto-approve-scopes`设置为匹配所有作用域的正则表达式。这并不一定要留在线上系统中，但它可以让我们快速工作，而无需重新放置Spring OAuth2在用户需要访问令牌时会为他们弹出的白色标签审批页面。要向令牌授予中添加显式批准步骤，我们需要提供一个UI来替换whitelabel(`/oauth/confirm_access`)。

要完成授权服务器，我们只需要为其UI提供安全配置。事实上，在这个应用程序中没有多少用户界面，但是我们仍然需要保护`/oauth/authorize`端点，并确保带有“登录”按钮的主页可见。这就是为什么我们需要这个方法:

    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http.antMatcher("/**")                                       (1)
        .authorizeRequests()
          .antMatchers("/", "/login**", "/webjars/**").permitAll() (2)
          .anyRequest().authenticated()                            (3)
        .and().exceptionHandling()
          .authenticationEntryPoint(new LoginUrlAuthenticationEntryPoint("/")) (4)
        ...
    }

*1* 默认情况下，所有请求都受到保护
*2* 明确排除主页和登录端点
*3* 所有其他端点都需要经过身份验证的用户
*4* 未经身份验证的用户将重新定向到主页

### 如何获取访问令牌

现在可以从我们的新授权服务器获得访问令牌。到目前为止，获取令牌的最简单方法是获取一个作为“acme”客户端的令牌。如果你运行应用程序并对其curl，你可以看到这一点:

    $ curl acme:acmesecret@localhost:8080/oauth/token -d grant_type=client_credentials
    {"access_token":"370592fd-b9f8-452d-816a-4fd5c6b4b8a6","token_type":"bearer","expires_in":43199,"scope":"read write"}
    
客户端凭据令牌在某些情况下很有用(如测试令牌端点是否正常工作)，但为了利用服务器的所有功能，我们希望能够为用户创建令牌。要代表应用程序的用户获取令牌，我们需要能够对用户进行身份验证。如果在应用程序启动时仔细查看日志，你可能会看到为默认Spring Boot用户记录了随机密码(根据[SpringBoot用户指南](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-security))。你可以使用此密码代表id为“user”的用户获取令牌:

    $ curl acme:acmesecret@localhost:8080/oauth/token -d grant_type=password -d username=user -d password=...
    {"access_token":"aa49e025-c4fe-4892-86af-15af2e6b72a2","token_type":"bearer","refresh_token":"97a9f978-7aad-4af7-9329-78ff2ce9962d","expires_in":43199,"scope":"read write"}
    
其中“…”应替换为实际密码。这称为“密码”授权，你可以在其中更改用户名和密码获取访问令牌。

密码授权对于测试也很有用，但当你有本地用户数据库来存储和验证凭据时，它可以适用于本机或移动应用程序。对于大多数应用程序或任何具有“社交”登录的应用程序(如我们的应用程序)，你需要“授权代码”授权，这意味着你需要浏览器(或行为类似浏览器的客户端)来处理重定向和cookie，并从外部提供程序呈现用户界面。

### 创建客户端

我们的授权服务器的客户端应用程序本身就是一个web应用程序，使用SpringBoot很容易创建。下面是一个示例:

    ClientApplication.java
    @EnableAutoConfiguration
    @Configuration
    @EnableOAuth2Sso
    @RestController
    public class ClientApplication {
    
      @RequestMapping("/")
      public String home(Principal user) {
        return "Hello " + user.getName();
      }
    
      public static void main(String[] args) {
        new SpringApplicationBuilder(ClientApplication.class)
            .properties("spring.config.name=client").run(args);
      }
    
    }
  
>`ClientApplication`类不能在`SocialApplication`类的同一包(或子包)中创建。否则，Spring将在启动`SocialApplication`server时加载一些`ClientApplication`自动配置，从而导致启动错误。

客户端的组成部分是主页(只打印用户名)和配置文件的显式名称(通过`spring.config.name=client`)。当我们运行此应用程序时，它将查找我们提供的配置文件，如下所示:

    client.yml
    
    server:
      port: 9999
      context-path: /client
    security:
      oauth2:
        client:
          client-id: acme
          client-secret: acmesecret
          access-token-uri: http://localhost:8080/oauth/token
          user-authorization-uri: http://localhost:8080/oauth/authorize
        resource:
          user-info-uri: http://localhost:8080/me
          
该配置看起来与我们在主应用程序中使用的配置非常相似，但使用的是“acme”客户端，而不是Facebook或Github客户端。应用程序将在端口9999上运行，以避免与主应用程序冲突。它指向的是用户信息端点“/me”，我们还没有实现它。

请注意，`server.context-path`是显式设置的，因此如果运行应用程序进行测试，请记住主页是[http://localhost:9999/client](http://localhost:9999/client)。单击该链接应该会将你带到auth服务器，并且在你通过所选的身份验证服务器进行身份验证后，你将被重定向回客户端应用程序

>如果同时在localhost上运行客户端和auth服务器，则上下文路径必须是显式的，否则cookie路径会冲突，并且两个应用程序无法就会话标识符达成一致。

### 保护用户信息端点

要使用我们的新授权服务器进行单点登录，就像我们使用Facebook和Github一样，它需要有一个受其创建的访问令牌保护的`/user`端点。到目前为止，我们有一个`/user`端点，它是通过用户身份验证时创建的cookie来保护的。要使用本地授予的访问令牌来保护它，我们可以重新使用现有端点并在新路径上为其设置别名:

    SocialApplication.java
    
    @RequestMapping({ "/user", "/me" })
    public Map<String, String> user(Principal principal) {
      Map<String, String> map = new LinkedHashMap<>();
      map.put("name", principal.getName());
      return map;
    }
    
>我们已将`Principal`转换为`Map`，以便隐藏不想向浏览器公开的部分，并取消两个外部身份验证提供程序之间端点的行为。原则上，我们可以在这里添加更详细的信息，例如提供程序特定的唯一标识符，或者电子邮件地址(如果有的话)。

现在可以通过声明我们的应用程序是资源服务器(以及授权服务器)来使用访问令牌保护“/me”路径。我们创建了一个新的配置类(作为主应用程序中的n个内部类，但也可以将其拆分为单独的独立类) :

    SocialApplication.java    
    @Configuration
    @EnableResourceServer
    protected static class ResourceServerConfiguration
        extends ResourceServerConfigurerAdapter {
      @Override
      public void configure(HttpSecurity http) throws Exception {
        http
          .antMatcher("/me")
          .authorizeRequests().anyRequest().authenticated();
      }
    }

此外，我们还需要为主应用程序指定`@Order`:

    SocialApplication.java    
    @SpringBootApplication
    ...
    @Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
    public class SocialApplication extends WebSecurityConfigurerAdapter {
      ...
    }
    
`@EnableResourceServer`注释使用`@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER-1)`创建安全过滤器，因此通过将主应用程序移动到`@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)`确保“/me”的规则优先。

### 测试OAuth2客户端

要测试新功能，你只需运行这两个应用程序，然后在浏览器中访问[http://localhost:9999/client](http://localhost:9999/client)。客户端应用程序将重定向到本地授权服务器，然后用户可以选择使用Facebook或Github进行身份验证。完成后返回到测试客户端，授予本地访问令牌并完成身份验证(你应该在浏览器中看到“Hello”消息)。如果你已经使用Github或Facebook进行了身份验证，你甚至可能不会注意到远程身份验证。

## 为未经身份验证的用户添加错误页

在本节中，我们将修改前面构建的注销应用程序，切换到Github身份验证，并向无法进行身份验证的用户提供一些反馈。同时，我们利用这个机会扩展身份验证逻辑，以包括一个规则，该规则只允许用户属于特定Github组织。“组织”是一个Github域特定的概念，但是也可以为其他提供商设计类似的规则，例如，使用Google，你可能只需要验证来自特定域的用户。

### 切换到Github

注销示例使用Facebook作为OAuth2提供程序。我们可以通过更改本地配置轻松切换到Github :

    application.yml    
    security:
      oauth2:
        client:
          clientId: bd1c0a783ccdd1c9b9e4
          clientSecret: 1a9030fbca47a5b2c28e92f19050bb77824b5ad1
          accessTokenUri: https://github.com/login/oauth/access_token
          userAuthorizationUri: https://github.com/login/oauth/authorize
          clientAuthenticationScheme: form
        resource:
          userInfoUri: https://api.github.com/user
          
### 检测客户端中的失败的认证

在客户端上，我们需要能够为无法进行身份验证的用户提供一些反馈。为此，我们添加了一个div，其中包含一条消息:

    index.html
    
    <div class="container text-danger error" style="display:none">
    There was an error (bad credentials).
    </div>
    
此文本将仅在显示“error”元素时显示，因此我们需要一些代码来执行此操作:

    index.html
    
    $.ajax({
      url : "/user",
      success : function(data) {
        $(".unauthenticated").hide();
        $("#user").html(data.userAuthentication.details.name);
        $(".authenticated").show();
      },
      error : function(data) {
        $("#user").html('');
        $(".unauthenticated").show();
        $(".authenticated").hide();
        if (location.href.indexOf("error=true")>=0) {
          $(".error").show();
        }
      }
    });
    
身份验证函数在加载时检查浏览器位置，如果发现其中包含“error=true”的URL，则将设置标记。

### 添加错误页面

为了支持客户端中的标志设置，我们需要能够捕获身份验证错误，并使用在查询参数中设置的标志重定向到主页。因此，在一个常规的`@Controller`中我们需要一个端点，如下所示:

    SocialApplication.java
    
    @RequestMapping("/unauthenticated")
    public String unauthenticated() {
      return "redirect:/?error=true";
    }

在示例应用程序中，我们将它放在主应用程序类中，该类现在是`@Controller` (而不是`@RestController`)，因此它可以处理重定向。我们最需要的是从未经验证的响应( HTTP 401,a.k.a. UNAUTHORIZED )到刚刚添加的“/unauthenticated”端点的映射:

    ServletCustomizer.java
    
    @Configuration
    public class ServletCustomizer {
      @Bean
      public EmbeddedServletContainerCustomizer customizer() {
        return container -> {
          container.addErrorPages(new ErrorPage(HttpStatus.UNAUTHORIZED, "/unauthenticated"));
        };
      }
    }
    
(在示例中，为了简明起见，这是作为主应用程序内部的嵌套类添加的)

### 服务端响应401

如果用户不能或不希望使用Github登录，则Spring Security会返回401，因此如果你未能进行身份验证(例如，拒绝令牌授予)，则说明应用程序已经在运行。

为了使事情更有趣，我们将扩展身份验证规则以拒绝不在正确组织中的用户。使用GithubAPI可以很容易地找到有关用户的更多信息，因此我们只需要正确地将其插入身份验证过程的部分。幸运的是，对于这样一个简单的用例，Spring Boot提供了一个简单的扩展点:如果我们声明一个类型为`AuthoritiesExtractor`的`@Bean`，它将被用来构造经过身份验证的用户的权限(通常是“角色”)。我们可以使用钩子函数判断用户正处于正确组织中，如果不是，则抛出异常:

    SocialApplication.java
    
    @Bean
    public AuthoritiesExtractor authoritiesExtractor(OAuth2RestOperations template) {
      return map -> {
        String url = (String) map.get("organizations_url");
        @SuppressWarnings("unchecked")
        List<Map<String, Object>> orgs = template.getForObject(url, List.class);
        if (orgs.stream()
            .anyMatch(org -> "spring-projects".equals(org.get("login")))) {
          return AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_USER");
        }
        throw new BadCredentialsException("Not in Spring Projects origanization");
      };
    }

请注意，我们已将`OAuth2RestOperations`自动注入到此方法中，因此可以使用它代表已验证的用户访问GithubAPI。我们这样做，并循环遍历organizations，寻找与“Spring-projects”匹配的组织(这是用于存储Spring开源项目的组织)。如果你希望能够成功进行身份验证，并且不在Spring工程团队中，则可以在此处替换自己的值。如果没有匹配项，我们将抛出`BadCredentialsException`，Spring Security将接受该异常并转换为401响应。    

`OAuth2RestOperations`也必须作为bean创建(从Spring Boot 1.4开始)，但这很简单，因为使用`@Enableoauthso`后，其成分都是可自动生成的:

    @Bean
    public OAuth2RestTemplate oauth2RestTemplate(OAuth2ProtectedResourceDetails resource, OAuth2ClientContext context) {
    	return new OAuth2RestTemplate(resource, context);
    }
    
显然，上面的代码可以推广到其他身份验证规则，有些适用于Github，有些适用于其他oauth2提供程序。你只需要知道`OAuth2RestOperations`和验证服务器提供方API的一些知识。

## 总结
我们已经看到了如何使用Spring Boot和Spring Security来构建多种样式的应用程序，而不需要太多代码。贯穿所有示例的主要主题是使用外部OAuth2提供程序的“社交”登录。最终的示例甚至可以用于“内部”提供这种服务，因为它具有与外部提供商相同的基本特性。所有示例应用程序都可以轻松扩展和重新配置，以满足更具体的使用情形，通常只需更改配置文件即可。请记住，如果你使用自己服务器中的示例版本向Facebook或Github(或类似的)注册，并获取自己主机地址的客户端凭据。记住不要将这些凭据放在公开的代码管理工具中！

如果想要编写新指南或对现有指南作出贡献？查看我们的[贡献指南](https://github.com/spring-guides/getting-started-guides/wiki)。

发布的所有指南都有ASLv2与 [Attribution NoDerivatives creative commons license](https://creativecommons.org/licenses/by-nd/3.0/)许可证。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



