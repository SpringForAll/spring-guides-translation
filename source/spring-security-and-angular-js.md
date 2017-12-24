# Spring Security 和 Angular

> 原文：[Spring Security and Angular JS](https://spring.io/guides/tutorials/spring-security-and-angular-js/)
>
> 译者：strongant
>
> 校对：maskwang



## 一个安全的单页应用程序

在本教程中，我们将展示 Spring Security，Spring Boot 和 Angular 的一些不错的功能，共同提供令人愉快和安全的用户体验。这篇文章是面向 Spring 和 Angular 初学者，但是也有许多细节可供专家使用。这实际上是 Spring Security和 Angular 的一系列章节中的第一章，每个章节都有新的特征。我们将在 [第二章节](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii) 和后续章节中对应用程序进行改进，但之后的主要更改是架构性而非功能性的内容。



### Spring 和单页面应用程序

HTML5，基于浏览器丰富的功能以及“单页面应用程序”对于现代开发人员来说是非常有价值的工具，但任何有意义的交互都将涉及到后端服务器，以及我们将要涉及的静态内容（HTML，CSS和JavaScript）需要一个后端服务器。后端服务器可以扮演任何或所有角色：提供静态内容，有时（但不是经常这样）呈现动态HTML，认证用户，保护对受保护资源的访问，以及（最后但并非最不重要的）与JavaScript交互在浏览器中通过HTTP和JSON（有时也称为REST API）。

Spring一直是构建后端功能（特别是在企业中）的流行技术，而随着 [Spring Boot](https://projects.spring.io/spring-boot) 的到来，事情从未如此简单。让我们来看看如何使用Spring Boot，Angular和Twitter Bootstrap从零开始构建一个新的单页面应用程序。没有什么特别的理由可以选择这个特定的技术栈，但是它非常受欢迎，特别是在企业级Java商店中的核心Spring选区，所以这是一个有价值的起点。

### 创建一个新项目

我们将逐步详细地创建这个应用程序，以便任何没有任何 Spring 和 Angular 使用经验的人都可以跟随我们的。如果你喜欢跳过这一章节，你可以 [跳到文章的最后](https://spring.io/guides/tutorials/spring-security-and-angular-js/#how-does-it-work) 有一份可工作的应用 DEMO，看看它们是如何组合在一起的。创建新项目有多种选择：

- [使用命令行 curl ](https://spring.io/guides/tutorials/spring-security-and-angular-js/#using-curl)
- [使用 Spring Boot CLI](https://spring.io/guides/tutorials/spring-security-and-angular-js/#using-spring-boot-cli)
- [使用  Spring 初始化网站](https://spring.io/guides/tutorials/spring-security-and-angular-js/#using-the-initializr-website)
- [使用 Spring Tool Suite](https://spring.io/guides/tutorials/spring-security-and-angular-js/#using-spring-tool-suite)


我们要建立的完整项目的源代码在 [Github](https://github.com/dsyer/spring-security-angular/tree/master/basic), 所以你可以直接克隆项目并直接从那里工作。然后跳转到 [下一节](https://spring.io/guides/tutorials/spring-security-and-angular-js/#add-a-home-page)。

#### 使用 Curl

创建一个新项目最简单的方法就是通过 [Spring Boot Initializr](https://start.spring.io/)。 比如： 在一个类似 UN*X 系统使用 curl :

```shell
$ mkdir ui && cd ui
$ curl https://start.spring.io/starter.tgz -d style=web \
-d style=security -d name=ui | tar -xzvf -
```

然后，您可以将该项目（默认情况下是普通的Maven Java项目）导入到您最喜欢的IDE中，或者仅在命令行中使用这些文件和“mvn”。然后跳转到 [下一章](https://spring.io/guides/tutorials/spring-security-and-angular-js/#add-a-home-page)。

#### 使用 Spring Boot CLI

你可以使用 [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#getting-started-installing-the-cli) 创建项目, 就像这样:

```Shell
$ spring init --dependencies web,security ui/ && cd ui
```

跳转到 [下一章](https://spring.io/guides/tutorials/spring-security-and-angular-js/#add-a-home-page) 。

#### 使用 Initializr 网站

如果您愿意，也可以直接从[Spring Boot Initializr](https://start.spring.io/) .zip 文件中获取相同的代码 。只需在浏览器中打开它并选择“Web”和“Security”依赖关系，然后点击“Generate Project”。 .zip文件在根目录中包含一个标准的Maven或Gradle项目，因此您可能需要在解压缩之前创建一个空目录。跳转到 [下一章](https://spring.io/guides/tutorials/spring-security-and-angular-js/#add-a-home-page)。

#### 使用 Spring Tool Suite

在 [Spring Tool Suite](https://spring.io/tools/sts) (一组 Eclipse 插件) 中，您还可以使用File-> New-> Spring Starter Project中的向导来创建和导入项目。 跳转到 [下一章](https://spring.io/guides/tutorials/spring-security-and-angular-js/#add-a-home-page)。IntelliJ IDEA和 NetBeans 也有类似的功能。



### 添加一个主页

单页面应用程序的核心是一个静态的“index.html”，所以让我们继续创建一个（在“src / main / resources / static”或“src / main / resources / public”中）

index.html

```html
<!doctype html>
<html>
<head>
<title>Hello Angular</title>
<link rel="stylesheet" type="text/css"
  href="/webjars/bootstrap/css/bootstrap.min.css" />
<script type="text/javascript" src="/webjars/jquery/jquery.min.js"></script>
<script type="text/javascript"
  src="/webjars/bootstrap/js/bootstrap.min.js"></script>
</head>

<body>
  <div class="container">
    <h1>Greeting</h1>
    <div><app/></div>
  </div>
  <script type="text/javascript" src="webjars/core-js/client/shim.min.js"></script>
  <script type="text/javascript" src="webjars/rxjs/bundles/Rx.min.js"></script>
  <script type="text/javascript" src="webjars/zone.js/dist/zone.min.js"></script>
  <script type="text/javascript" src="webjars/reflect-metadata/Reflect.js"></script>
  <script type="text/javascript" src="webjars/angular__core/bundles/core.umd.js"></script>
  <script type="text/javascript" src="webjars/angular__common/bundles/common.umd.js"></script>
  <script type="text/javascript" src="webjars/angular__compiler/bundles/compiler.umd.js"></script>
  <script type="text/javascript"
    src="webjars/angular__platform-browser/bundles/platform-browser.umd.js"></script>
  <script type="text/javascript"
    src="webjars/angular__platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js"></script>
  <script type="text/javascript" src="webjars/angular__http/bundles/http.umd.js"></script>
  <script type="text/javascript" src="js/hello.js"></script>
  <script type="text/javascript">
    document.addEventListener('DOMContentLoaded', function() {
      ng.platformBrowserDynamic.platformBrowserDynamic().bootstrapModule(AppModule);
    });
  </script>
</body>
</html>
```

这很短暂，很亲切，因为它只是说“Hello World”。

#### 主页的功能

显著的功能包括：

- 所有的CSS类都来自 [Twitter Bootstrap](https://getbootstrap.com/)。一旦我们设置了正确的样式，它会让页面看起来很漂亮。
- 问候语中的内容是我们尚未定义的一个Angular组件（<app />）。
- 所有的Angular库和依赖项（以及Twitter Bootstrap）都包含在<body>的底部，以便浏览器在加载HTML之前引入所有的依赖。
- 我们还包含一个单独的“hello.js”，这是我们要定义应用程序行为的地方。
- `AppModule`在“hello.js”中定义，但在HTML中显式初始化，因为它使得“hello.js”可测试。

我们将在一分钟内添加脚本和样式表资源，但现在事实上我们可以忽略它们不存在。



#### 运行应用

一旦主页文件被添加，您的应用程序将在浏览器中加载（尽管它还没有做太多）。在命令行上，你可以做到这一点:

```
$ mvn spring-boot:run
```

并通过 http://localhost:8080 访问浏览器。当你加载主页时，你应该得到一个浏览器对话框，询问用户名和密码（用户名是“user”，密码是在启动时在控制台日志中打印的）。实际上还没有内容，所以一旦成功通过验证，您应该会看到一个带有“问候”标题的空白页面。

>如果你不喜欢把密码打印到控制台日志中，只需将其`security.user.password = password`（并选择你自己的密码）添加到“application.properties”（在“src/main/resources”）中 。我们在示例代码中使用“application.yml”。

在IDE中，只需在应用程序类中运行`main()`方法（只有一个类，如果使用上面的“curl”命令则称为`UiApplication`）。

要打包并作为独立的JAR运行，可以这样做：
```shell
$ mvn package
$ java -jar target/*.jar
```

### 前端资源

Angular的大多数现代教程倾向于使用node.js工具链下载和重新打包所有的库及其依赖关系。 Angular和其他前端技术的一些演示通常只包括直接来自互联网的库资源。我们将使用 [Webjars](https://webjars.org/) 并在应用程序类路径中安装第三方依赖，而不是使用node.js工具链其中的任何一个。

即使是最基本的Angular应用程序，也有不少需要的webjars：

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>${jquery.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>core-js</artifactId>
    <version>${corejs.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>zone.js</artifactId>
    <version>${zone.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>reflect-metadata</artifactId>
    <version>${reflect.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>rxjs</artifactId>
    <version>${rxjs.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>angular__common</artifactId>
    <version>${angular.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>angular__compiler</artifactId>
    <version>${angular.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>angular__platform-browser</artifactId>
    <version>${angular.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>angular__http</artifactId>
    <version>${angular.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>angular__platform-browser-dynamic</artifactId>
    <version>${angular.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars.npm</groupId>
    <artifactId>angular__core</artifactId>
    <version>${angular.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>${bootstrap.version}</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator</artifactId>
</dependency>
```

示例应用程序中的版本目前是：

```xml
<angular.version>4.4.4</angular.version>
<bootstrap.version>3.3.7-1</bootstrap.version>
<corejs.version>2.5.1</corejs.version>
<jquery.version>2.2.4</jquery.version>
<reflect.version>0.1.10</reflect.version>
<rxjs.version>5.4.3</rxjs.version>
<zone.version>0.8.16</zone.version>
```

你可以把这个逐字复制到你的POM中，或者只是在 [Github的源代码](https://github.com/dsyer/spring-security-angular/tree/master/basic/pom.xml#L43) 中进行查看复制。 Twitter Bootstrap对jQuery有依赖性，所以我们也包括这个。一个没有使用Bootstrap的Angular应用程序不需要这样做，因为Angular拥有jQuery本身所需的功能。

如果您现在运行应用程序，您应该看到CSS生效，所有资源将成功加载，但业务逻辑和导航仍然缺失。

> 使用webjars是一个切合实际的选择，具有强大的教学影响力。大多数Spring应用程序开发人员不熟悉node.js工具链或TypeScript，这正是所有官方的Angular文档现在使用的。因此，Webjar和普通的旧JavaScript（ES5）是让您开始使用Java开发人员熟悉的工具的一种简单方法，但是从长远来看，它们可能会受到限制。您可能想要使用`npm`和Angular CLI来查看比最基本的应用程序更大的东西：如果您只是将这些工具链的输出放在“src/main/resources/static”或“target/classes/static”那么它将全部工作。


### 创建 Angular 应用

让我们创建"hello"应用(在"src/main/resources/static/js/hello.js")，这样在"index.html"的页面底部在页面加载的时候会被正常加载。

一个小巧的 Angular 应用就像下面这样:

hello.js

```javascript
var AppComponent = ng.core.Component({
    selector : 'app',
    template : '<p>The ID is {{greeting.id}}</p><p>The content is {{greeting.content}}</p>'
}).Class({
    constructor : function() {
        this.greeting = {id:'XYZ', content:'Hello World'};
    }
});

var AppModule = ng.core.NgModule({
    imports: [ng.platformBrowser.BrowserModule],
    declarations: [AppComponent],
    bootstrap: [AppComponent]
  }).Class({constructor : function(){}})

document.addEventListener('DOMContentLoaded', function() {
    ng.platformBrowserDynamic.platformBrowserDynamic().bootstrapModule(AppModule);
});
```

这个Javascript中的大部分代码都是样板片段 - 它只是在那里让Angular运行起来。有趣的东西都在`AppComponent`中，我们定义了“选择器”（HTML元素的名称）和HTML呈现的片段。

如果你在“src/main/resources/static/js”下面添加了这个文件，你的应用程序现在应该是安全的并且可以运行，并且会显示“Hello World！”。问候语是由HTML中的使用占位符`{{greeting.id}}`和`{{greeting.content}}`被Angular渲染。

### 添加动态内容

到目前为止，我们有一个硬编码问候的应用程序。这对于学习如何融合在一起非常有用，但实际上我们希望内容来自后端服务器，因此我们创建一个HTTP端点，用于获取问候语。在您的[应用程序类](https://github.com/dsyer/spring-security-angular/blob/master/basic/src/main/java/demo/UiApplication.java)（在“src/main/java/demo”）中，添加`@RestController`注解并定义一个新的`@RequestMapping`：

UiApplication.java

```java
@SpringBootApplication
@RestController
public class UiApplication {

  @RequestMapping("/resource")
  public Map<String,Object> home() {
    Map<String,Object> model = new HashMap<String,Object>();
    model.put("id", UUID.randomUUID().toString());
    model.put("content", "Hello World");
    return model;
  }

  public static void main(String[] args) {
    SpringApplication.run(UiApplication.class, args);
  }

}
```

> 根据您创建新项目的方式，可能不会将其命名为`UiApplication`。   

运行该应用并尝试访问"resource"端点，你会发现默认情况下它是安全的。

```
$ curl localhost:8080/resource
{"timestamp":1420442772928,"status":401,"error":"Unauthorized","message":"Full authentication is required to access this resource","path":"/resource"}
```

#### 从Angular 加载动态资源

所以让我们在浏览器中抓取这个消息。修改`AppComponent`使用XHR加载受保护的资源：


hello.js

```Javascript
var AppComponent = ng.core.Component({
    selector : 'app',
    template : '<p>The ID is {{greeting.id}}</p><p>The content is {{greeting.content}}</p>'
}).Class({
    constructor : [ng.http.Http, function(http) {
        var self = this;
        self.greeting = {id:'', content:''};
        http.get("/resource").subscribe(response => self.greeting = response.json());
    }]
});
```

我们通过 Angular `http`模块注入了一个由Angular提供的[http服务](https://angular.io/guide/http)，并使用它来获取我们的资源。Angular将响应传递给我们，我们将JSON取出并将其分配给`greeting`。

> 遵循一个通用的约定，我们引入了一个“self”变量作为“this”的别名，以便回调回调内部的控制器。

为了将`http`服务依赖注入到我们的自定义组件中，我们需要在`AppModule`包含组件的地方声明它（`imports`与初始代码相比，它只是变化了一行）：

hello.js

```javascript
---
var AppModule = ng.core.NgModule({
    imports: [ng.platformBrowser.BrowserModule, ng.http.HttpModule],
    declarations: [AppComponent],
    bootstrap: [AppComponent]
  }).Class({constructor : function(){}})
---
```

再次运行应用程序（或者只是在浏览器中重新加载主页），您将看到具有唯一ID的动态消息。因此，即使资源受到保护，而且无法直接调整资源，浏览器也能够访问该内容。我们有一个安全的单页应用程序在不到一百行的代码！

> 您可能需要强制浏览器在更改静态资源后重新加载静态资源。在Chrome（和带有插件的Firefox）中，您可以使用“开发人员工具”（F12），这可能就足够了。或者你可能不得不使用CTRL + F5。


### 它是如何工作的？

如果您使用某些开发工具（通常是F12打开此工具，默认情况下在Chrome中运行，可能需要Firefox中的插件），则可以在浏览器中看到浏览器与后端之间的交互。这里有一个总结：

| 动作   | 路径           | 状态码  | 响应               |
| ---- | ------------ | ---- | ---------------- |
| GET  | /            | 401  | 浏览器提示进行身份验证      |
| GET  | /            | 200  | index.html       |
| GET  | /webjars/**  | 200  | 从 webjars加载第三方资源 |
| GET  | /js/hello.js | 200  | 应用程序逻辑           |
| GET  | /resource    | 200  | JSON greeting    |

您可能看不到401，因为浏览器将主页加载视为单个交互，您可能会看到2个“/ resource”请求，因为存在CORS协商。

仔细查看请求，你会看到他们都有一个“授权”头，如下所示：

```Authorization: Basic dXNlcjpwYXNzd29yZA==```

浏览器正在发送每个请求的用户名和密码（所以请记住在生产中专门使用HTTPS）。这里没有什么“Angular”，所以它适用于你的JavaScript框架或非框架的选择。



#### 这是什么错？

从表面上看，我们似乎做得非常好，简洁，易于实现，我们所有的数据都被一个秘钥密码保护，如果我们改变了前端或后端技术，它仍然可以工作。但是有一些问题。

- 基本身份验证仅限于用户名和密码身份验证。
- 验证用户界面无处不在（丑陋的浏览器）。
- [跨站点请求伪造](https://en.wikipedia.org/wiki/Cross-site_request_forgery)（CSRF）没有保护。

因为它只需要获取后端资源（即服务器中没有状态改变），所以CSRF对于我们的应用来说并不是真正的问题。只要你在你的应用程序中有一个POST，PUT或DELETE，就不会再有任何合理的现代措施。

在[本系列的下一节中，](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)我们将扩展应用程序以使用基于表单的身份验证，这比HTTP Basic灵活得多。一旦我们有了一个表单，我们需要CSRF保护，Spring Security和Angular都有一些不错的开箱即用功能来帮助实现这一点。剧透：我们将需要使用`HttpSession`。

感谢：我要感谢所有帮助我开发这个系列的人，特别是[Rob Winch](https://spring.io/team/rwinch)和[Thorsten Spaeth](https://twitter.com/thspaeth)对文本和源代码的仔细审查，并且教给我一些我甚至不知道的部分的技巧我倍感亲切。

## 登录页面

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)如何在“单页面应用程序”中使用[Angular JS和](https://angularjs.org/)[Spring Security](https://projects.spring.io/spring-security)。在这里，我们展示了如何使用Angular JS通过表单对用户进行身份验证，并获取在UI中呈现的安全资源。这是一系列章节的第二部分，您可以通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)，从头开始构建应用程序的基本构建块，或者直接转到[Github](https://github.com/dsyer/spring-security-angular/tree/master/single)的[源代码](https://github.com/dsyer/spring-security-angular/tree/master/single)。在第一部分中，我们构建了一个使用HTTP基本身份验证来保护后端资源的简单应用程序。在这一个我们添加一个登录表单，给用户一些控制是否验证，并修复第一次迭代（主要是缺乏CSRF保护）的问题。

> 提醒：如果您正在使用示例应用程序完成本节，请务必清除Cookie和HTTP Basic凭据的浏览器缓存。在Chrome中，为单个服务器执行此操作的最佳方式是打开一个新的隐身窗口。

### 将导航添加到主页

单页面应用程序的核心是一个静态的“index.html”。我们已经有了一个真正的基础，但是对于这个应用程序，我们需要提供一些导航功能（登录，注销，主页），所以我们来修改它（在“src/main/resources/static”中）：

的index.html

```html
<html>
<head>
...
<base href="/">
</head>
<body>
    <app/>
    <script type="text/javascript" src="webjars/core-js/client/shim.min.js"></script>
    ...
	<script src="js/hello.js"></script>
</body>
</html>
```

实际上，它与原始文件没有多大区别，但是我们用一个`<html/>`文本替换了主体内容（除了脚本之外的所有内容）`<app/>`，并向（后面将需要的路由器组件）添加了一个`<base/>`元素`<head/>`。

的`<app/>`选择器，反过来，需要在角应用到进行布线的部件，同时包含导航元素和主体内容。这将比基本更复杂一点，`AppComponent`但它有一些相同的功能。这是实现：

hello.js

```
var AppComponent = ng.core.Component({
        templateUrl: 'app.html',
        selector: 'app',
        providers: [AppService]
    }).Class({constructor : [AppService, ng.http.Http, ng.router.Router, function(app, http, router){

        this.logout = function() {
            http.post('logout', {}).finally(function() {
                app.authenticated = false;
                router.navigateByUrl('/login')
            }).subscribe();
        }

        app.authenticate();
    }]
});
```

突出功能：

- 还有一些更多的依赖注入，以及`http`我们正在注入的一个自定义服务的服务`app`（将用于集中认证信息）和Angular `router`。
- 有一个注销函数作为组件的一个属性公开，我们稍后可以使用它来向后端发送注销请求。它在`app`服务中设置一个标志，并将用户发送回登录屏幕（并通过`finally()`回调无条件执行）。
- 我们正在使用`templateUrl`将模板HTML外部化为一个单独的文件。
- `authenticate()`当控制器被加载时，该函数被调用以查看用户是否实际上已经被认证（例如，如果他在会话中间刷新了浏览器）。我们需要这个`authenticate()`功能来进行远程调用，因为实际的认证是由服务器完成的，我们不希望信任浏览器来跟踪它。

这里是`app.html`：

app.html

```
<div class="container">
  <ul class="nav nav-pills">
    <li><a routerLinkActive="active" routerLink="/home">Home</a></li>
    <li><a routerLinkActive="active" routerLink="/login">Login</a></li>
    <li><a (click)="logout()">Logout</a></li>
  </ul>
</div>
<div class="container">
  <router-outlet></router-outlet>
</div>
```

- 有一个`<ul>`导航栏。有2个使用Angular路由器的“导航”链接，一个`logout()`从`AppComponent`上面调用函数。
- 所有主要内容将被添加为指令中的“partials” `<router-outlet>`（在Angular Router中定义）。

`app`我们上面注入的服务看起来需要一个布尔标志，所以我们可以判断用户当前是否已经通过身份验证，并且`authenticate()`可以使用一个函数来对后端服务器进行身份验证，或者只是查询用户详细信息：

hello.js

```javascript
var AppService = ng.core.Injectable({}).Class({constructor: [ng.http.Http, function(http) {

    var self = this;
    this.authenticated = false;
    this.authenticate = function(credentials, callback) {

        var headers = credentials ? {
            authorization : "Basic " + btoa(credentials.username + ":" + credentials.password)
        } : {};
        http.get('user', {headers: headers}).subscribe(function(response) {
            if (response.json().name) {
                self.authenticated = true;
            } else {
                self.authenticated = false;
            }
            callback && callback();
        });

    }

}]})
```

该`authenticated`标志是简单的。`authenticate()`如果提供了HTTP基本认证凭证，则该功能发送，否则不会。它还有一个可选`callback`参数，如果认证成功，我们可以使用它来执行一些代码。

### 前端用于路由和表单绑定的资源

正如在[第一部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)，前端资产被添加为webjars。我们在这里做同样的事情，对于这个样本，我们需要一些新的：

index.html

```html
<html>
...
<body>
  <app/>
  ...
  <script type="text/javascript" src="webjars/angular__router/bundles/router.umd.js"></script>
  <script type="text/javascript" src="webjars/angular__forms/bundles/forms.umd.js"></script>
  <script type="text/javascript" src="js/hello.js"></script>
</body>
</html>
```

与相应的条目在`pom.xml`：

的pom.xml

```Xml
<dependency>
  <groupId>org.webjars.npm</groupId>
  <artifactId>angular__forms</artifactId>
  <version>${angular.version}</version>
</dependency>
<dependency>
  <groupId>org.webjars.npm</groupId>
  <artifactId>angular__router</artifactId>
  <version>${angular.version}</version>
</dependency>
```

### 将导航添加到Angular应用程序

让我们修改应用程序（在“src/main/resources/public/js/hello.js”中）来添加一些导航功能。我们可以开始为路由添加一些配置，以便主页中的链接实际上做一些事情。例如

hello.js

```javascript
var routes = [
    { path: '', pathMatch: 'full', redirectTo: 'home'},
    { path: 'home', component: HomeComponent},
    { path: 'login', component: LoginComponent}
];

var AppModule = ng.core.NgModule({
    imports: [ng.platformBrowser.BrowserModule, ng.http.HttpModule,
            ng.router.RouterModule.forRoot(routes), ng.forms.FormsModule],
    declarations: [HomeComponent, LoginComponent, AppComponent],
    bootstrap: [AppComponent]
  }).Class({constructor : function(){}});
```

我们添加了一个名为[“RouterModule”](https://angular.io/guide/router)的Angular模块的依赖，这使我们能够`router`在构造函数中注入魔法`AppComponent`。在`routes`使用的进口内`AppModule`设置链接到“/”（"home" controller）和“/login”（以下简称“login” controller）。

我们也偷偷的`FormsModule`在那里，因为稍后需要将数据绑定到我们想要在用户登录时提交的表单。

#### 打招呼

来自旧主页的问候内容可以在“home.html”（在“src/main/resources/static”中的“index.html”旁边）：

home.html的

```javascript
<h1>Greeting</h1>
<div [hidden]="!authenticated()">
	<p>The ID is {{greeting.id}}</p>
	<p>The content is {{greeting.content}}</p>
</div>
<div [hidden]="authenticated()">
	<p>Login to see your greeting</p>
</div>
```

由于用户现在可以选择是否登录（在全部由浏览器控制之前），我们需要在UI之间区分安全内容和不安全内容。我们已经通过添加对（现在还不存在的）`authenticated()`函数的引用来预期这一点。

该`HomeComponent`再去问候，同时还提供了`authenticated()`拉动旗出的效用函数`AppService`：

hello.js

```javascript
var HomeComponent = ng.core.Component({
    templateUrl : 'home.html'
}).Class({
    constructor : [AppService, ng.http.Http, function(app, http) {
        var self = this;
        this.greeting = {id:'', msg:''};
        http.get('resource').map(response => response.json()).subscribe(data => self.greeting = data);
        this.authenticated = function() { return app.authenticated; };
    }]
});
```

#### 登录表单

登录表单进入“login.html”：

login.html

```
<div class="alert alert-danger" [hidden]="!error">
	There was a problem logging in. Please try again.
</div>
<form role="form" (submit)="login()">
	<div class="form-group">
		<label for="username">Username:</label> <input type="text"
			class="form-control" id="username" name="username" [(ngModel)]="credentials.username"/>
	</div>
	<div class="form-group">
		<label for="password">Password:</label> <input type="password"
			class="form-control" id="password" name="password" [(ngModel)]="credentials.password"/>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
```

这是一个非常标准的登录表单，带有2个用户名和密码输入以及一个通过Angular事件处理程序提交表单的按钮`(submit)`。您不需要在表单标签上执行任何操作，因此最好不要放置一个表单标签。还有一个错误消息，只有在角模型包含一个`error`。表单控件使用`ngModel`从[Angular Forms](https://angular.io/guide/reactive-forms)的HTML和角控制器之间传递数据，并且在这种情况下，我们使用的是`credentials`对象来保存用户名和pasword。

### 身份验证过程

为了支持我们刚刚添加的登录表单，我们需要添加更多的功能。在客户端，这些将在“导航”控制器中实现，在服务器上将是Spring Security配置。

#### 提交登录表单

要提交表单，我们需要定义`login()`已经在表单via中引用的函数`ng-submit`，以及`credentials`我们通过引用的对象`ng-model`。让我们在“hello.js”中省略“navigation”控制器（省略路由配置和“home”控制器）：

我们还需要`LoginComponent`通过`login()`我们在下面引用的函数来处理表单提交`login.html`：

hello.js

```
var LoginComponent = ng.core.Component({
    templateUrl : 'login.html'
}).Class({
    constructor : [AppService, ng.router.Router, function(app, router) {
        var self = this;
        this.credentials = {username:'', password:''};
        this.login = function() {
            app.authenticate(self.credentials, function() {
                router.navigateByUrl('/')
            });
            return false;
        };
    }]
});
```

“Navigation” controller 中的所有代码都将在页面加载时执行，因为`<div>`包含菜单栏的内容是可见的并且被装饰`ng-controller="navigation"`。除了初始化`credentials`对象之外，它还定义了2个函数，`login()`我们需要在表单中，还有一个本地帮助函数`authenticate()`，它试图从后端加载“user”资源。

该`authenticate()`函数设置一个应用程序范围标志，称为`authenticated`我们已经在我们的“home.html”中使用的标志来控制页面的哪些部分被渲染。我们这样做[`$rootScope`](https://docs.angularjs.org/api/ng/service/$rootScope)是因为它方便易用，我们需要`authenticated`在“导航”和“主页”控制器之间共享标志。Angular的专家可能更喜欢通过共享的用户定义的服务来共享数据（但它最终是相同的机制）。

在`authenticate()`发出GET一个相对资源（相对于你的应用程序的部署根）“/user”。当从`login()`函数中调用时，它会在头文件中添加Base64编码的凭据，所以在服务器上进行身份验证并接受cookie。当我们获得认证的结果时，`login()`函数也会相应地设置一个本地`$scope.error`标志，用于控制在登录表单上方显示错误信息。

#### 当前认证的用户

为了服务这个`authenticate()`功能，我们需要添加一个新的端点到后端：

UiApplication.java

```java
@SpringBootApplication
@RestController
public class UiApplication {

  @RequestMapping("/user")
  public Principal user(Principal user) {
    return user;
  }

  ...

}
```

这在Spring Security应用程序中是一个有用的技巧。如果“/ user”资源是可访问的，那么它将返回当前通过身份验证的用户（an [`Authentication`](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/Authentication.java)），否则Spring Security将拦截请求并通过一个[`AuthenticationEntryPoint`](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/AuthenticationEntryPoint.java)。

#### 处理服务器上的登录请求

Spring Security可以轻松处理登录请求。我们只需要添加一些配置到我们的[主应用程序类](https://github.com/dsyer/spring-security-angular/blob/master/single/src/main/java/demo/UiApplication.java)（例如作为一个内部类）：

UiApplication.java

```java
@SpringBootApplication
@RestController
public class UiApplication {

  ...

  @Configuration
  @Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
  protected static class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
      http
        .httpBasic()
      .and()
        .authorizeRequests()
          .antMatchers("/index.html", "/home.html", "/login.html", "/", "/home", "/login").permitAll()
          .anyRequest().authenticated();
    }
  }

}
```

这是Spring Security定制的标准Spring Boot应用程序，只允许匿名访问静态（HTML）资源（默认情况下CSS和JS资源已经可以访问）。这些HTML资源需要匿名用户使用，Spring Security不会忽略它，原因很明显。

### 添加默认的HTTP请求头

如果您在此时运行应用程序，您会发现浏览器会弹出一个基本身份验证对话框（用于用户和密码）。它这样做是因为它看到一个401效应初探从XHR请求`/user`，并`/resource`以“WWW身份验证”标头。抑制这个弹出窗口的方法是压制来自Spring Security的头文件。而抑制响应头的方法是发送一个特殊的，传统的请求头“X-Requested-With = XMLHttpRequest”。它曾经是Angular的默认版本，但是它们[在1.3.0版本中将其取出](https://github.com/angular/angular.js/issues/1004)。所以这里是如何在Angular XHR请求中设置默认请求头。

首先扩展`RequestOptions`Angular HTTP模块提供的默认值：

hello.js

```javascript
var RequestOptionsService = ng.core.Class({
    extends: ng.http.BaseRequestOptions,
    constructor : function() {},
    merge: function(opts) {
        opts.headers = new ng.http.Headers(opts.headers ? opts.headers : {});
        opts.headers.set('X-Requested-With', 'XMLHttpRequest');
        return opts.merge(opts);
    }
});
```

这里的语法是愚蠢的（它实际上在TypeScript中更好），但是样板。该`extends`属性`Class`是它的基类，除了构造函数之外，我们真正需要做的就是覆盖`merge()`Angular总是调用的函数，并且可以用来添加额外的头文件。基类有一个`merge`对传递给`http`服务的参数进行一些有用的验证，所以我们通过`opts.merge(opts)`在那里调用来重用它。

要安装这个新的`RequestOptions`工厂，我们需要声明它在`providers`的`AppModule`：

hello.js

```javascript
var AppModule = ng.core.NgModule({
    imports: [ng.platformBrowser.BrowserModule, ng.http.HttpModule,
            ng.router.RouterModule.forRoot(routes), ng.forms.FormsModule],
    declarations: [HomeComponent, LoginComponent, AppComponent],
    providers : [{ provide: ng.http.RequestOptions, useClass: RequestOptionsService }],
    bootstrap: [AppComponent]
  }).Class({constructor : function(){}});
```

### 登出

应用程序几乎在功能上完成。我们需要做的最后一件事就是实现我们在主页上绘制的注销功能。如果用户被认证，那么我们会显示一个“logout”链接，并将其挂接到一个`logout()`函数中`AppComponent`。请记住，它发送一个HTTP POST到“/logout”，我们现在需要在服务器上实现。这很简单，因为它已经被Spring Security加入了我们（也就是说我们不需要为这个简单的用例做任何事情）。为了更好的控制注销的行为，你可以使用`HttpSecurity`你的`WebSecurityAdapter`to 回调，例如注销后执行一些业务逻辑。

### CSRF保护

应用程序几乎可以使用了，事实上，如果你运行它，你会发现，我们迄今为止建立的所有东西实际上都工作，除了注销链接。尝试使用它并查看浏览器中的响应，您将看到原因：

```http
POST /logout HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded

username=user&password=password

HTTP/1.1 403 Forbidden
Set-Cookie: JSESSIONID=3941352C51ABB941781E1DF312DA474E; Path=/; HttpOnly
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
...

{"timestamp":1420467113764,"status":403,"error":"Forbidden","message":"Expected CSRF token not found. Has your session expired?","path":"/login"}
```

这很好，因为这意味着Spring Security内置的CSRF保护措施已经启动，以防止我们自己底层攻击。它只需要一个名为“X-CSRF”的头部发送给它的令牌。CSRF令牌的值是在`HttpRequest`加载主页的初始请求的属性中可用的服务器端。为了得到它到客户端，我们可以使用服务器上的动态HTML页面来呈现它，或者通过自定义端点来展示它，否则我们可以把它作为一个cookie发送出去。最后的选择是最好的，因为Angular 基于cookie [建立了对CSRF](https://angular.io/guide/http#security-xsrf-protection)（它称为“XSRF”）的支持。

所以在服务器上我们需要一个自定义的过滤器来发送cookie。Angular希望cookie名称为“XSRF-TOKEN”，Spring Security默认将它作为请求属性提供，所以我们只需要将值从请求属性转移到cookie。幸运的是，Spring Security（从4.1.0开始）提供了一个特殊的`CsrfTokenRepository`功能：

UiApplication.java

```java
@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
protected static class SecurityConfiguration extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
      ...
      .and().csrf()
        .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
  }
}
```

随着这些变化，我们不需要在客户端做任何事情，登录表单现在正在工作。

### 它是如何工作的？

如果您使用某些开发工具（通常是F12打开此工具，默认情况下在Chrome中运行，可能需要Firefox中的插件），则可以在浏览器中看到浏览器与后端之间的交互。这里有一个总结：

| 动作   | 路径             | 状态   | 响应                |
| ---- | -------------- | ---- | ----------------- |
| GET  | /              | 200  | index.html        |
| GET  | / CSS / **     | 200  | Twitter引导CSS      |
| GET  | / webjars / ** | 200  | Bootstrap和Angular |
| GET  | /js/hello.js   | 200  | 应用程序逻辑            |
| GET  | /user          | 401  | 未经授权（忽略）          |
| GET  | /home.html     | 200  | 主页                |
| GET  | /app.html      | 200  | 主页                |
| GET  | /login.html    | 200  | 角度登录表单部分          |
| GET  | /user          | 401  | 未经授权（忽略）          |
| GET  | /resource      | 401  | 未经授权（忽略）          |
| GET  | /user          | 200  | 发送凭据并获取JSON       |
| GET  | /resource      | 200  | JSON问候            |

上面标记为“忽略”的响应是由Angular在XHR调用中收到的HTML响应，由于我们没有处理这些数据，所以HTML被放在地板上。我们在“/ user”资源的情况下寻找一个经过认证的用户，但是由于它在第一次调用中不存在，那么这个响应就会被丢弃。

仔细查看请求，你会看到他们都有cookie。如果你从一个干净的浏览器开始（例如在Chrome浏览器中进行隐身），第一个请求没有发送到服务器的cookie，但是服务器发回“JSESSIONID”（常规`HttpSession`）和“X-XSRF -TOKEN“（我们上面设置的CRSF cookie）。随后的请求都有这些cookie，而且它们很重要：没有它们，应用程序就不能工作，它们提供了一些非常基本的安全功能（认证和CSRF保护）。当用户进行身份验证（POST之后）时，cookie的值会改变，这是另一个重要的安全功能（防止[会话修复攻击](https://en.wikipedia.org/wiki/Session_fixation)）。

> 对于CSRF保护来说，依靠cookie被发送回服务器是不够的，因为即使您不在从您的应用程序（跨站点脚本攻击，也称为[XSS](https://en.wikipedia.org/wiki/Cross-site_scripting)）加载的页面中，浏览器也会自动发送它。标题不会自动发送，所以原点在控制之下。您可能会看到，在我们的应用程序中，CSRF令牌作为cookie发送给客户端，所以我们将看到它由浏览器自动发回，但它是提供保护的头。

### 帮助，我的应用程序如何扩展？

“但是等等...”你说：“是不是在单页面应用程序中使用会话状态真的很差？” 这个问题的答案将是“大部分”，因为使用会话进行认证和CSRF保护是非常明智的。这个状态必须存储在某个地方，如果你把它从会话中取出，你将不得不把它放在别的地方，并在服务器和客户端自己手动管理它。这只是更多的代码，可能更多的维护，通常重新发明一个完美的轮子。

“但是，但是......”你会回应，“我现在如何横向扩展我的应用程序？这是上面提到的“真正的”问题，但往往会缩短为“会话状态不好，我必须是无状态的”。不要惊慌。这里采取的主要观点是安全*是*有状态的。你不能有一个安全的，无状态的应用程序。那么你要在哪里存储状态？这里的所有都是它的。[Rob Winch](https://spring.io/team/rwinch)在[Spring Exchange 2014上](https://skillsmatter.com/skillscasts/5398-the-state-of-securing-restful-apis-with-spring)做了非常有用和深入的讨论，解释了对状态的需求（以及它的普遍性 - TCP和SSL是有状态的，所以无论你是否知道你的系统是有状态的），这可能值得一看如果你想深入研究这个话题。

好消息是你有一个选择。最简单的选择是将会话数据存储在内存中，并依靠负载均衡器中的粘性会话将来自同一会话的请求路由回同一个JVM（它们都以某种方式支持）。这是够好让你掉在地上，并会为工作*真正*大量的使用案例。另一种选择是在应用程序的实例之间共享会话数据。只要你是严格的，只存储安全数据，它很小，不经常变化（只有当用户登录或退出，或他们的会话超时），所以不应该有任何重大的基础设施问题。[Spring Session](https://github.com/spring-projects/spring-session/)也很容易做到。我们将在本系列的下一部分中使用Spring Session，因此不需要详细介绍如何在这里设置它，但它实际上是几行代码和一个Redis服务器，它是超快的。

> 设置共享会话状态的另一个简单方法是将应用程序作为WAR文件部署到Cloud Foundry [Pivotal Web Services，](https://run.pivotal.io/)并将其绑定到Redis服务。

### 但是，我的自定义令牌实现（无状态，锁）呢？

如果这是你对上一节的回应，那么再读一遍，因为也许你第一次没有得到它。如果您将令牌存储在某个地方，那么这可能不是无状态的，但即使您没有（例如，使用JWT编码的令牌），您将如何提供CSRF保护？这一点很重要。下面是一个经验法则（归功于Rob Winch）：如果您的应用程序或API将被浏览器访问，您需要CSRF保护。不是没有会话就无法做到这一点，只需要自己编写所有的代码就可以了，因为它已经被实现并且在`HttpSession`（反过来，它是你使用的容器的一部分，并从一开始就支持）？即使你决定不需要CSRF，并且有一个完美的“无状态”（基于非会话的）标记实现，你仍然必须在客户端编写额外的代码来消费和使用它，你可以委托给浏览器和服务器自己的内置功能：浏览器总是发送cookie，服务器总是有一个会话（除非你把它关闭）。这些代码不是商业逻辑，它不会让你赚钱，这只是一个开销，所以更糟的是，它会花费你的钱。

### 结论

我们现在拥有的应用程序接近于用户在实际环境中的“真实”应用程序中可能期望的内容，并且可能将其用作构建具有该架构的功能更丰富的应用程序的模板（带有静态内容和JSON资源）。我们正在使用`HttpSession`存储安全数据，依靠我们的客户尊重和使用我们发送给他们的cookies，我们对此感到满意，因为它让我们专注于我们自己的业务领域。在[下一节中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)我们将体系结构扩展为单独的身份验证和UI服务器，以及JSON的独立资源服务器。这显然很容易推广到多个资源服务器。我们也将介绍Spring Session到堆栈中，并展示如何共享认证数据。

## 资源服务器

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)如何在“单页面应用程序”中使用[Spring Security](https://projects.spring.io/spring-security) 和 [Angular](http://angular.io/)。在这里，我们首先将我们用作应用程序中动态内容的“问候语”资源分解为单独的服务器，首先将其作为不受保护的资源，然后用不透明的令牌进行保护。这是一系列部分的第三部分，您可以通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)，从头开始构建应用程序的基本构建块，或者直接转到Github中的源代码两部分：[资源不受保护的部分](https://github.com/dsyer/spring-security-angular/tree/master/vanilla)，以及[受令牌保护的部分](https://github.com/dsyer/spring-security-angular/tree/master/spring-session)。

> 如果您正在处理示例应用程序的这一部分，请务必清除Cookie和HTTP基本凭据的浏览器缓存。在Chrome中，为单个服务器执行此操作的最佳方式是打开一个新的隐身窗口。

### 独立的资源服务器

#### 客户端变化

在客户端，将资源移动到不同的后端并不需要太多操作。下面是[最后一节中](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/single/src/main/resources/static/js/hello.js)的“home”组件：

hello.js

```javascript
var HomeComponent = ng.core.Component({
    templateUrl : 'home.html'
}).Class({
    constructor : [AppService, ng.http.Http, function(app, http) {
        var self = this;
        this.greeting = {id:'', msg:''};
        http.get('resource')
            .subscribe(response => self.greeting = response.json());
        this.authenticated = function() { return app.authenticated; };
    }]
});
```

我们所需要做的就是改变网址。例如，如果我们要在本地主机上运行新的资源，它可能看起来像这样：

hello.js

```javascript
        http.get('http://localhost:9000')
            .map(response => self.greeting = response.json());
```

#### 服务器端更改

该[UI服务器](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/vanilla/ui/src/main/java/demo/UiApplication.java)是微不足道的改变：我们只需要删除`@RequestMapping`的问候资源（这是“/资源”）。然后我们需要创建一个新的资源服务器，我们可以像使用[Spring Boot Initializr](https://start.spring.io/)在[第一部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)那样做。例如，在UN * X系统上使用卷曲：

```Shell
$ mkdir resource && cd resource
$ curl https://start.spring.io/starter.tgz -d style=web \
-d name=resource | tar -xzvf -
```

然后，您可以将该项目（默认情况下是普通的Maven Java项目）导入到您最喜欢的IDE中，或者仅在命令行中使用这些文件和“mvn”。

只需添加一个`@RequestMapping`到[主应用程序类](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/vanilla/resource/src/main/java/demo/ResourceApplication.java)，从[旧的UI](https://github.com/dsyer/spring-security-angular/blob/master/single/src/main/java/demo/UiApplication.java)复制实现：

ResourceApplication.java

```java
@SpringBootApplication
@RestController
class ResourceApplication {

  @RequestMapping("/")
  public Message home() {
    return new Message("Hello World");
  }

  public static void main(String[] args) {
    SpringApplication.run(ResourceApplication.class, args);
  }

}

class Message {
  private String id = UUID.randomUUID().toString();
  private String content;
  public Message(String content) {
    this.content = content;
  }
  // ... getters and setters and default constructor
}
```

一旦完成，您的应用程序将在浏览器中加载。在命令行上，你可以做到这一点

```shell
$ mvn spring-boot:run -Dserver.port=9000
```

并通过[http//localhost:9000](http://localhost:9000/)访问浏览器，您将看到带有问候语的JSON。您可以在`application.properties`（“src /main/resources”）中更改端口：

application.properties

```properties
server.port: 9000
```

如果您尝试在浏览器中从UI（端口8080）加载该资源，您将发现它不起作用，因为浏览器将不允许XHR请求。

### CORS谈判

浏览器尝试与我们的资源服务器进行协商，以确定是否允许根据[跨源资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)协议访问它。这不是一个AngularJS的责任，所以就像cookie合同一样，它将在浏览器中使用所有JavaScript。这两个服务器没有声明它们有一个共同的来源，所以浏览器拒绝发送请求，UI被破坏。

为了解决这个问题，我们需要支持CORS协议，这个协议涉及到一个“pre-flight”选项请求和一些头文件来列出调用者允许的行为。Spring 4.2有一些很好[的细粒度的CORS支持](https://jira.spring.io/browse/SPR-9278)，所以我们可以添加一个注释到我们的控制器映射，例如：

ResourceApplication.java

```java
@RequestMapping("/")
@CrossOrigin(origins="*", maxAge=3600)
public Message home() {
  return new Message("Hello World");
}
```

> 巧妙地使用`origins=*`是快速和肮脏的，它的工作原理，但它不是不安全的，不以任何方式推荐。                                        

### 保护资源服务器

太棒了！我们有一个新的架构工作的应用程序。唯一的问题是资源服务器没有安全性。

#### 添加Spring安全性

我们也可以看看如何在资源服务器上添加安全性作为过滤器层，就像在UI服务器中一样。第一步非常简单：只需将Spring Security添加到Maven POM中的类路径中即可：

的pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  ...
</dependencies>
```

重新启动资源服务器，嘿！这是安全的：

```shell
$ curl -v localhost:9000
< HTTP/1.1 302 Found
< Location: http://localhost:9000/login
...
```

我们正在重定向到（白标签）登录页面，因为curl不会发送与我们的Angular客户端相同的请求头。修改命令发送更多类似的请求头：

```
$ curl -v -H "Accept: application/json" \
    -H "X-Requested-With: XMLHttpRequest" localhost:9000
< HTTP/1.1 401 Unauthorized
...
```

所以我们需要做的就是教客户发送每个请求的凭证。

### 令牌认证

互联网和人们的Spring后端项目，都是基于自定义令牌的认证解决方案。Spring Security提供了一个准系统的`Filter`实现来让你自己开始（参见例子[`AbstractPreAuthenticatedProcessingFilter`](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/authentication/preauth/AbstractPreAuthenticatedProcessingFilter.java)和[`TokenService`](https://github.com/spring-projects/spring-security/blob/master/core/src/main/java/org/springframework/security/core/token/TokenService.java)）。虽然Spring Security没有规范的实现，但其中一个原因可能就是更简单一些。

请记住，在本系列的[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)中，Spring Security `HttpSession`默认使用存储验证数据。它不会直接与会话交互：有一个抽象层（[`SecurityContextRepository`](https://github.com/spring-projects/spring-security/blob/master/web/src/main/java/org/springframework/security/web/context/SecurityContextRepository.java)），您可以使用它来更改存储后端。如果我们可以在我们的资源服务器中将该存储库指向一个经过我们的用户界面验证的认证的存储，那么我们就可以在这两个服务器之间共享认证。UI服务器已经有了这样一个存储（所`HttpSession`），所以如果我们可以分发这个存储并将其打开到资源服务器，我们就拥有了大部分解决方案。

#### Spring Session

[Spring Session的](https://github.com/spring-projects/spring-session/)这个部分非常简单。我们所需要的只是一个共享的数据存储（Redis和JDBC是开箱即用的），并在服务器上配置几行配置来设置一个`Filter`。

在UI应用程序中，我们需要添加一些依赖到我们的[POM](https://github.com/dsyer/spring-security-angular/blob/master/spring-session/ui/pom.xml)：

的pom.xml

```Xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

Spring Boot和Spring Session一起工作来连接到Redis并集中存储会话数据。

使用这一行代码和在本地主机上运行的Redis服务器，您可以运行UI应用程序，使用一些有效的用户凭证登录，会话数据（身份验证）将存储在redis中。

> 如果你没有在本地运行的Redis服务器，你可以很容易地使用[Docker来](https://www.docker.com/)启动一个服务器（在Windows或MacOS上，这需要一个虚拟机）。[在Github](https://github.com/dsyer/spring-security-angular/tree/master/spring-session/docker-compose.yml)[`docker-compose.yml`](https://docs.docker.com/compose/)的[源代码中](https://github.com/dsyer/spring-security-angular/tree/master/spring-session/docker-compose.yml)有一个文件，你可以很容易地在命令行上运行`docker-compose up`。如果在虚拟机中执行此操作，则Redis服务器将在与本地主机不同的主机上运行，因此您需要将其隧道到本地主机，或者将应用程序配置为指向`spring.redis.host`您的本地主机`application.properties`。

### 从UI发送自定义令牌

唯一缺少的部分是商店中数据密钥的传输机制。关键是`HttpSession`ID，所以如果我们能够在UI客户端中获得这个键，我们可以把它作为一个自定义头部发送到资源服务器。所以“家”控制器将需要改变，以便它发送标题作为问候资源的HTTP请求的一部分。例如：

hello.js

```Javascript
var HomeComponent = ng.core.Component({
    templateUrl : 'home.html'
}).Class({
    constructor : [AppService, ng.http.Http, function(app, http) {
        var self = this;
        this.greeting = {id:'', msg:''};
        http.get('token').subscribe(response => {
            var token = response.json().token;
            http.get('http://localhost:9000', {
                headers: {'X-Auth-Token': token}
            }).subscribe(response => self.greeting =response.json());
        })
        this.authenticated = function() { return app.authenticated; };
    }]
});

```

（更优雅的解决方案可能是根据需要获取令牌，并使用我们`RequestOptionsService`的头添加到资源服务器的每个请求。

我们已经在调用UI服务器上新的自定义端点的“/ token”的成功回调中封装了这个调用，而不是直接到“http://:localhost:9000 [ [http://localhost:9000](http://localhost:9000/) ] 这个实现是微不足道的：

UiApplication.java

```java
@SpringBootApplication
@RestController
public class UiApplication {

  public static void main(String[] args) {
    SpringApplication.run(UiApplication.class, args);
  }

  ...

  @RequestMapping("/token")
  public Map<String,String> token(HttpSession session) {
    return Collections.singletonMap("token", session.getId());
  }

}

```

所以UI应用程序已经准备就绪，并将所有对后端的调用都称为“X-Auth-Token”。

### 资源服务器中的身份验证

资源服务器有一个微小的变化，它可以接受自定义标题。CORS配置必须把这个头指定为来自远程客户端的头，例如

ResourceApplication.java

```Java
@RequestMapping("/")
@CrossOrigin(origins = "*", maxAge = 3600,
    allowedHeaders={"x-auth-token", "x-requested-with", "x-xsrf-token"})
public Message home() {
  return new Message("Hello World");
}

```

浏览器的前置检查现在将被Spring MVC处理，但是我们需要告诉Spring Security它允许它通过：

ResourceApplication.java

```java
public class ResourceApplication extends WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.cors().and().authorizeRequests()
      .anyRequest().authenticated();
  }

  ...

```

> 没有必要`permitAll()`访问所有资源，并且可能有一个处理程序无意中发送敏感数据，因为它不知道请求是在飞行前。该`cors()`配置实用程序通过处理的过滤层的所有飞行前请求减轻这一点。                                        

剩下的就是在资源服务器中选取自定义标记并使用它来认证我们的用户。事实证明，这非常简单，因为我们只需告诉Spring Security会话存储库的位置，以及在传入请求中查找令牌（会话ID）的位置。首先我们需要添加Spring Session和Redis依赖关系，然后我们可以设置`Filter`：

ResourceApplication.java

```java
@SpringBootApplication
@RestController
class ResourceApplication {

  ...

  @Bean
  HeaderHttpSessionStrategy sessionStrategy() {
    return new HeaderHttpSessionStrategy();
  }

}

```

这`Filter`是在UI服务器中创建的镜像，因此它将Redis建立为会话存储。唯一的区别是它使用了一个自定义`HttpSessionStrategy`的头部（默认为“X-Auth-Token”），而不是默认的（名为“JSESSIONID”的Cookie）。我们还需要防止浏览器在未经身份验证的客户端弹出一个对话框 - 应用程序是安全的，但`WWW-Authenticate: Basic`默认情况下会发送一个401 ，所以浏览器响应一个用户名和密码的对话框。有不止一种方法来实现这一点，但是我们已经让Angular发送了“X-Requested-With”头文件，所以Spring Security默认为我们处理。

资源服务器有一个最后的改变，使它与我们的新的认证方案一起工作。Spring Boot的默认安全是无状态的，我们希望在会话中存储身份验证，所以我们需要在`application.yml`（或`application.properties`）

application.yml

```Yaml
security:
  sessions: NEVER

```

这对Spring Security来说“永远不会创建一个会话，而是使用一个会话”（由于UI中的身份验证，它已经在那里了）。

重新启动资源服务器并在新的浏览器窗口中打开UI。

### 为什么不全部使用Cookie？

我们不得不使用自定义的头文件，并在客户端编写代码来填充头文件，这并不复杂，但似乎与[第II部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)的建议相矛盾，只能尽可能地使用cookie和会话。有人认为不这样做会引入额外的不必要的复杂性，而且我们现在所执行的是目前为止最复杂的一种：解决方案的技术部分远远超过了业务逻辑（这是公认的微小）。这绝对是一个公平的批评（我们打算在本系列的下一部分中讨论），但是让我们简单地看看为什么它不像使用cookies和会话一样简单。

至少我们还在使用这个会话，这是有道理的，因为Spring Security和Servlet容器知道如何做到这一点，我们不费吹灰之力。但是，我们不能继续使用cookie来传输身份验证令牌吗？这本来不错，但有一个原因，它不会工作，那就是浏览器不会让我们。您可以从JavaScript客户端浏览器的Cookie存储区中浏览，但是有一些限制，并且有很好的理由。特别是你没有权限访问服务器发送的“HttpOnly”（你将会看到默认情况下会话cookie的情况）。你也不能在传出的请求中设置cookie，所以我们不能设置一个“SESSION”cookie（这是Spring Session默认的cookie名称），我们必须使用一个自定义的“X-Session” 头。这两个限制都是为了您自己的保护，所以恶意脚本无法在没有适当授权的情况下访问资源。

UI和资源服务器没有共同的来源，所以他们不能共享cookie（尽管我们可以使用Spring Session来强制他们共享会话）。

### 结论

我们在[本系列的第二部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)已经复制了应用程序的功能：主页包含从远程后端获取的问候语，在导航栏中提供登录和注销链接。不同之处在于问候来自独立的资源服务器，而不是嵌入在UI服务器中。这增加了实现的复杂性，但好消息是我们主要是基于配置（实际上是100％的声明）解决方案。我们甚至可以通过将所有新代码抽取到库中（Spring配置和Angular自定义指令）来使解决方案100％声明。在接下来的几部分之后，我们将推迟这个有趣的任务。在[下一节中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv) 我们将看到一种不同的真正好方法来减少当前实现中的所有复杂性：API网关模式（客户端将其所有请求发送到一个地方，并在那里处理认证）。

> 我们在这里使用Spring会话来分享两个不是逻辑上相同的应用程序的服务器之间的会话。这是一个巧妙的伎俩，而且这是不可能的“常规”JEE分布式会议。

## API网关

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)如何在“单页面应用程序”中使用[Spring Security](https://projects.spring.io/spring-security) 和 [Angular](http://angular.io/)。在这里，我们展示了如何构建一个API网关来控制使用[Spring Cloud](https://projects.spring.io/spring-cloud/)的身份验证和对后端资源的访问。这是一系列章节的第四部分，您可以通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)从头开始构建应用程序的基本构建块，或者直接转到[Github](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/proxy)的[源代码](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/proxy)。在[最后一节中，](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)我们构建了一个使用[Spring Session](https://github.com/spring-projects/spring-session/)的简单分布式应用程序验证后端资源。在这个例子中，我们把UI服务器变成了后端资源服务器的反向代理，解决了最后一个实现（由自定义令牌认证引入的技术复杂性）的问题，给了我们很多新的选项来控制来自浏览器客户端的访问。

> 提醒：如果您正在使用示例应用程序完成本节，请务必清除Cookie和HTTP Basic凭据的浏览器缓存。在Chrome中，为单个服务器执行此操作的最佳方式是打开一个新的隐身窗口。

### 创建一个API网关

API网关是前端客户端的单点入口（和控制），可以是基于浏览器的（如本节中的示例）或移动端。客户端只需要知道一个服务器的URL，后端可以随意重构，不会有任何改变，这是一个显着的优势。在集中和控制方面还有其他优势：限速，认证，审计和记录。[Spring Cloud](https://projects.spring.io/spring-cloud/)实现简单的反向代理非常简单。

如果你在代码中，你会知道[上一节](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)结束时的应用程序实现有点复杂，所以它不是一个好的地方。然而，还有一个中间点，我们可以更容易地开始，后端资源还没有被Spring Security保护。这个源代码是[Github中的](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/vanilla)一个单独的项目[，](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/vanilla)所以我们将从那里开始。它有一个UI服务器和一个资源服务器，他们正在交谈。资源服务器还没有Spring Security，所以我们可以先使系统工作，然后添加该层。

#### 使用一行声明反向代理

为了把它变成一个API网关，UI服务器需要一个小的调整。在Spring配置中的某处，我们需要添加一个`@EnableZuulProxy`注释，例如在主（仅）[应用程序类中](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/proxy/ui/src/main/java/demo/UiApplication.java)：

UiApplication.java

```java
@SpringBootApplication
@RestController
@EnableZuulProxy
public class UiApplication {
  ...
}

```

而在外部配置文件中，我们需要将UI服务器中的本地资源映射到[外部配置](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/proxy/ui/src/main/resources/application.yml)（“application.yml”）中的远程资源：

application.yml

```yaml
security:
  ...
zuul:
  routes:
    resource:
      path: /resource/**
      url: http://localhost:9000

```

这就是说：“在本地服务器中将pattern/resource/**映射到远程服务器localhost:9000中的相同路径”。简单而有效（OK，所以包括YAML在内的6行，但你并不总是需要）。

所有我们需要做的工作是在类路径上的正确的东西。为了这个目的，我们在Maven POM中有一些新的行：

的pom.xml

```Xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>Dalston.SR4</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
  </dependency>
  ...
</dependencies>

```

请注意“spring-cloud-starter-zuul”的使用 - 它像Spring Boot那样是一个初学者POM，但是它支配了我们需要的这个Zuul代理的依赖关系。我们也在使用，`<dependencyManagement>`因为我们希望能够依赖所有的传递依赖是正确的版本。

#### 在客户端使用代理

随着这些变化，我们的应用程序仍然有效，但是直到我们修改客户端，我们还没有实际使用新的代理。幸运的是，这是微不足道的。我们只需要将[上一节中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)从“单一”到“香草”样本的变化恢复为：

hello.js

```
var HomeComponent = ng.core.Component({
    templateUrl : 'home.html'
}).Class({
    constructor : [AppService, ng.http.Http, function(app, http) {
        var self = this;
        this.greeting = {id:'', msg:''};
        http.get('resource')
            .subscribe(response => self.greeting = response.json());
        this.authenticated = function() { return app.authenticated; };
    }]
});

```

现在，当我们启动服务器时，一切正常，请求正在通过UI（API网关）代理到资源服务器。

#### 进一步的简化

更好的是：我们在资源服务器中不再需要CORS过滤器。无论如何，我们很快就把这个问题扔到了一边，应该是一个红灯，我们不得不做任何事情，从技术角度来看，尤其是在涉及安全的地方。幸运的是，它现在是多余的，所以我们可以扔掉它，直到晚上睡觉！

### 保护资源服务器

你可能记得在中间状态，我们从资源服务器没有安全的地方开始。

> 另外：如果你的网络体系结构反映了应用程序的体系结构，那么缺乏软件安全性可能不会成为一个问题（你可以让资源服务器在物理上不能访问除UI服务器以外的任何人）。作为一个简单的演示，我们可以使资源服务器只能在本地主机上访问。只需将其添加到`application.properties`资源服务器中：

application.properties

```properties
server.address: 127.0.0.1

```

> 哇，那很容易！通过只在数据中心中可见的网络地址执行此操作，并且您拥有适用于所有资源服务器和所有用户桌面的安全解决方案。

假设我们决定在软件层面确实需要安全性（很可能有很多原因）。这不会是一个问题，因为我们只需要添加Spring Security作为依赖项（在[资源服务器POM中](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/proxy/resource/pom.xml)）：

的pom.xml

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>

```

这足以让我们成为一个安全的资源服务器，但它不会让我们的工作应用程序，同样的原因，它没有在[第三部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)：两个服务器之间没有共享的身份验证状态。

### 共享认证状态

我们可以使用相同的机制来共享身份验证（和CSRF）状态，就像我们在上一次（即[Spring Session](https://github.com/spring-projects/spring-session))中所做的一样。我们像以前一样向两台服务器添加依赖关系：

的pom.xml

```Xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-redis</artifactId>
</dependency>

```

但是这一次配置要简单得多，因为我们可以`Filter`在两者中添加相同的声明。首先是UI服务器，明确声明我们希望所有头文件被转发（即没有“敏感”）：

application.yml

```Yaml
zuul:
  routes:
    resource:
      sensitive-headers:

```

然后我们可以转到资源服务器。有两个小的更改：一个是显式禁用资源服务器中的HTTP Basic（以防止浏览器弹出验证对话框）：

ResourceApplication.java

```Java
@SpringBootApplication
@RestController
class ResourceApplication extends WebSecurityConfigurerAdapter {

  ...

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.httpBasic().disable();
    http.authorizeRequests().anyRequest().authenticated();
  }

}

```

> 另外：另一个也会阻止验证对话框的方法是保持HTTP Basic，但将401挑战改为“Basic”以外的其他方法。你可以`AuthenticationEntryPoint`通过`HttpSecurity`配置回调中的单行实现来实现。

另一个是明确要求在非国家的会话创建政策`application.properties`：

application.properties

```properties
security.sessions: NEVER

```

只要redis仍在后台运行（使用[`docker-compose.yml`](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/proxy/docker-compose.yml)如果你想启动它），那么系统将工作。在[http://localhost:8080](http://localhost:8080/)加载用户界面的主页并登录，你会看到主页上呈现的后端消息。

### 它是如何工作的？

现在幕后发生了什么？首先，我们可以查看UI服务器（和API网关）中的HTTP请求：

| 动作   | 路径             | 状态   | 响应                   |
| ---- | -------------- | ---- | -------------------- |
| GET  | /              | 200  | index.html           |
| GET  | / webjars / ** | 200  | Bootstrap和Angular JS |
| GET  | /js/hello.js   | 200  | 应用程序逻辑               |
| GET  | /app.html      | 200  | 主版面的角度模板             |
| GET  | /login.html    | 200  | 角度登录表单部分             |
| GET  | /user          | 401  | 未经授权的用户访问            |
| GET  | /resource      | 401  | 未经验证的对资源的访问          |
| GET  | /user          | 200  | JSON身份验证的用户          |
| GET  | /resource      | 200  | （代理）JSON问候           |

这与[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)末尾的序列是一样的，除了cookie名称稍有不同（“SESSION”而不是“JSESSIONID”），因为我们使用的是Spring Session。但是体系结构是不同的，最后一次对“/ resource”的请求是特殊的，因为它被代理到资源服务器。

通过查看UI服务器中的“/trace”端点（来自Spring Boot Actuator，我们添加了Spring Cloud依赖项），我们可以看到反向代理正在运行。在一个新的浏览器中访问[http://localhost:8080/trace](http://localhost:8080/trace)（如果你没有一个已经获得一个JSON插件的浏览器，使其更好和可读）。您将需要使用HTTP Basic（浏览器弹出窗口）进行身份验证，但是与登录表单相同的凭据是有效的。在开始或接近开始时，您应该看到一对像这样的请求：

> 尝试使用不同的浏览器，这样就不存在身份验证交叉的可能（例如，如果使用Chrome浏览器来测试用户界面，则使用Firefox） - 它不会阻止应用程序正常工作，但是如果它们包含来自同一浏览器的认证混合。                                   

/trace

```
{
  "timestamp": 1420558194546,
  "info": {
    "method": "GET",
    "path": "/",
    "query": ""
    "remote": true,
    "proxy": "resource",
    "headers": {
      "request": {
        "accept": "application/json, text/plain, */*",
        "x-xsrf-token": "542c7005-309c-4f50-8a1d-d6c74afe8260",
        "cookie": "SESSION=c18846b5-f805-4679-9820-cd13bd83be67; XSRF-TOKEN=542c7005-309c-4f50-8a1d-d6c74afe8260",
        "x-forwarded-prefix": "/resource",
        "x-forwarded-host": "localhost:8080"
      },
      "response": {
        "Content-Type": "application/json;charset=UTF-8",
        "status": "200"
      }
    },
  }
},
{
  "timestamp": 1420558200232,
  "info": {
    "method": "GET",
    "path": "/resource/",
    "headers": {
      "request": {
        "host": "localhost:8080",
        "accept": "application/json, text/plain, */*",
        "x-xsrf-token": "542c7005-309c-4f50-8a1d-d6c74afe8260",
        "cookie": "SESSION=c18846b5-f805-4679-9820-cd13bd83be67; XSRF-TOKEN=542c7005-309c-4f50-8a1d-d6c74afe8260"
      },
      "response": {
        "Content-Type": "application/json;charset=UTF-8",
        "status": "200"
      }
    }
  }
},

```

第二项是从客户端向“/resource”上的网关请求，你可以看到cookie（由浏览器添加）和CSRF头（由Angular添加，见[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/second)）。第一个条目有`remote: true`，这意味着它跟踪到资源服务器的调用。你可以看到它出去了一个URI路径“/”，你可以看到（关键）饼干和CSRF头也被发送了。如果没有Spring Session，这些头文件对于资源服务器来说是没有意义的，但是现在我们已经设置了它，现在可以使用这些头文件来重新组成一个具有认证和CSRF令牌数据的会话。所以这个请求是被允许的，我们在做生意！

### 结论

我们在这一节中介绍了很多，但是我们得到了一个非常好的地方，那就是在我们的两台服务器中有最少量的样板代码，它们都非常安全，用户体验也不会受到影响。仅此一项就是使用API网关模式的一个原因，但实际上我们只是抓住了可能用到的东西的表面（Netflix用它来做[很多事情](https://github.com/Netflix/zuul/wiki/How-We-Use-Zuul-At-Netflix)）。在[Spring Cloud](https://projects.spring.io/spring-cloud/)上阅读，了解更多关于如何使网关易于添加更多功能的信息。本系列的[下一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_sso_with_oauth2_angular_js_and_spring_security_part_v)将通过将身份验证职责提取到单独的服务器（单点登录模式）来扩展应用程序体系结构。

## 单点登录使用OAuth2

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)如何在“单页面应用程序”中使用[Spring Security](https://projects.spring.io/spring-security) 和 [Angular](http://angular.io/)。在这里，我们将演示如何将[Spring Security OAuth](https://projects.spring.io/spring-security-oauth/)与[Spring Cloud](https://projects.spring.io/spring-cloud/)一起使用，以扩展我们的API网关，以便将Single Sign On和OAuth2令牌身份验证扩展到后端资源。这是一系列章节的第五部分，您可以通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)从头开始构建应用程序的基本构建块，或者直接转到[Github](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2)的[源代码](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2)。在[最后一节中，](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)我们构建了一个使用[Spring Session](https://github.com/spring-projects/spring-session/)的小型分布式应用程序验证后端资源和[Spring Cloud](https://projects.spring.io/spring-cloud/)以在UI服务器中实现嵌入式API网关。在本节中，我们将身份验证职责提取到单独的服务器，以使我们的UI服务器成为授权服务器上第一个可能的单点登录应用程序。这在当今许多应用中是常见的模式，无论是在企业还是社会初创阶段。我们将使用OAuth2服务器作为身份验证器，以便我们还可以使用它来为后端资源服务器授予令牌。Spring Cloud会自动将访问令牌转发到我们的后端，并使我们能够进一步简化UI和资源服务器的实现。

> 提醒：如果您正在使用示例应用程序完成本节，请务必清除Cookie和HTTP Basic凭据的浏览器缓存。在Chrome中，为单个服务器执行此操作的最佳方式是打开一个新的隐身窗口。

### 创建一个OAuth2授权服务器

我们的第一步是创建一个新的服务器来处理认证和令牌管理。按照[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)的步骤，我们可以从[Spring Boot Initializr](https://start.spring.io/)开始。例如，在UN * X系统上使用卷曲：

```Shell
$ curl https://start.spring.io/starter.tgz -d style=web \
-d style=security -d name=authserver | tar -xzvf -

```

然后，您可以将该项目（默认情况下是普通的Maven Java项目）导入到您最喜欢的IDE中，或者仅在命令行中使用这些文件和“mvn”。

#### 添加OAuth2依赖关系

我们需要添加[Spring OAuth](https://projects.spring.io/spring-security-oauth)依赖关系，所以在我们的[POM中](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/oauth2/authserver/pom.xml)添加：

的pom.xml

```xml
<dependency>
  <groupId>org.springframework.security.oauth</groupId>
  <artifactId>spring-security-oauth2</artifactId>
</dependency>

```

授权服务器很容易实现。一个最小版本看起来像这样：

AuthserverApplication.java

```java
@SpringBootApplication
@EnableAuthorizationServer
public class AuthserverApplication extends WebMvcConfigurerAdapter {

  public static void main(String[] args) {
    SpringApplication.run(AuthserverApplication.class, args);
  }

}

```

我们只需要做1件事（添加之后`@EnableAuthorizationServer`）：

application.properties

```properties
---
...
security.oauth2.client.clientId: acme
security.oauth2.client.clientSecret: acmesecret
security.oauth2.client.authorized-grant-types: authorization_code,refresh_token,password
security.oauth2.client.scope: openid
---

```

这注册了一个“acme”客户与一个秘钥和一些授权的授权类型，包括“授权码”。

现在让我们把它运行在9999端口，用一个可预测的密码进行测试：

application.properties

```properties
server.port=9999
security.user.password=password
server.contextPath=/uaa
...

```

我们还设置上下文路径，以便它不使用默认（“/”），否则您可以将本地主机上其他服务器的cookie发送到错误的服务器。所以让服务器运行，我们可以确保它正在工作：

```shell
$ mvn spring-boot:run

```

或者`main()`在IDE中启动该方法。

#### 测试授权服务器

我们的服务器正在使用Spring Boot的默认安全设置，所以和[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)的服务器一样，它将受到HTTP基本认证的保护。要启动[授权码代金券授予](https://tools.ietf.org/html/rfc6749#section-1.3.1)您访问的授权端点，例如在[HTTP://本地主机:9999/UAA/的OAuth/授权RESPONSE_TYPE =代码＆CLIENT_ID = ACME＆REDIRECT_URI = HTTP://示例.com](http://localhost:9999/uaa/oauth/authorize?response_type=code&client_id=acme&redirect_uri=http://example.com)一旦你验证你会得到一个重定向到带有授权码的example.com，例如<http://example.com/?code=jYWioI>。

> 为了这个示例应用程序的目的，我们创建了一个没有注册重定向的客户端“acme”，这使我们能够重定向example.com。在生产应用程序中，您应该始终注册重定向（并使用HTTPS）。
> 代码可以使用令牌端点上的“acme”客户端凭据交换访问令牌：

```shell
$ curl acme:acmesecret@localhost:9999/uaa/oauth/token  \
-d grant_type=authorization_code -d client_id=acme     \
-d redirect_uri=http://example.com -d code=jYWioI
{"access_token":"2219199c-966e-4466-8b7e-12bb9038c9bb","token_type":"bearer","refresh_token":"d193caf4-5643-4988-9a4a-1c03c9d657aa","expires_in":43199,"scope":"openid"}

```

访问令牌是UUID（“2219199c ...”），由服务器中的内存令牌存储支持。我们还得到了一个刷新标记，当前标记到期时我们可以使用它来获取新的访问标记。

> 由于我们为“acme”客户端授予了“密码”授权，因此我们也可以使用curl和用户凭证直接从令牌端点获取令牌，而不是授权代码。这不适合基于浏览器的客户端，但对测试非常有用。

如果你按照上面的链接，你会看到Spring OAuth提供的白标签UI。首先，我们将使用这个，我们可以稍后再回来，像我们在[第二部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)为自包含服务器做的那样。

### 更改资源服务器

如果我们从[第四部分开始](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)，我们的资源服务器正在使用[Spring Session](https://github.com/spring-projects/spring-session/)进行身份验证，所以我们可以把它取出来，并用Spring OAuth来代替它。我们还需要删除Spring会话和Redis依赖关系，因此请将其替换为：

的pom.xml

```Xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-redis</artifactId>
</dependency>

```

有了这个：

pom.xml

```xml
<dependency>
  <groupId>org.springframework.security.oauth</groupId>
  <artifactId>spring-security-oauth2</artifactId>
</dependency>

```

然后`Filter`从[主应用程序类中](https://github.com/spring-guides/tut-spring-security-and-angular-js/blob/master/vanilla-oauth2/resource/src/main/java/demo/ResourceApplication.java)删除会话，并使用方便的`@EnableResourceServer`注释（从Spring Security OAuth2）替换它：

ResourceApplication.java

```java
@SpringBootApplication
@RestController
@EnableResourceServer
class ResourceApplication {

  @RequestMapping("/")
  public Message home() {
    return new Message("Hello World");
  }

  public static void main(String[] args) {
    SpringApplication.run(ResourceApplication.class, args);
  }
}

```

有了这一改变，应用程序就可以挑战访问令牌而不是HTTP Basic了，但是我们需要一个配置更改来实际完成这个过程。我们将添加少量的外部配置（在“application.properties”中），以允许资源服务器解码给定的令牌并对用户进行身份验证：

application.properties

```properties
...
security.oauth2.resource.userInfoUri: http://localhost:9999/uaa/user

```

这告诉服务器它可以使用令牌来访问“/ user”端点并使用它来派生认证信息（这有点像Facebook API中的[“/me”端点](https://developers.facebook.com/docs/graph-api/reference/v2.2/user/?locale=en_GB)）。实际上，它为资源服务器提供了一种解释令牌的方法，如`ResourceServerTokenServices`Spring OAuth2中的接口所表示的。

运行应用程序并用命令行客户端打到主页：

```shell
$ curl -v localhost:9000
> GET / HTTP/1.1
> User-Agent: curl/7.35.0
> Host: localhost:9000
> Accept: */*
>
< HTTP/1.1 401 Unauthorized
...
< WWW-Authenticate: Bearer realm="null", error="unauthorized", error_description="An Authentication object was not found in the SecurityContext"
< Content-Type: application/json;charset=UTF-8
{"error":"unauthorized","error_description":"An Authentication object was not found in the SecurityContext"}

```

你会看到一个带有“WWW-Authenticate”头的401表示它需要一个不记名的令牌。

> 该`userInfoUri`是迄今为止不挂钩的资源服务器想出了一个办法，以凭证进行解码的唯一途径。事实上，这是一个最低的共同标准（而不是规范的一部分），但经常可以从OAuth2提供商（如Facebook，Cloud Foundry，Github）获得，还有其他选择。例如，您可以在令牌本身（例如使用[JWT](http://jwt.io/)）中编码用户身份验证，或者使用共享的后端存储。`/token_info`CloudFoundry中还有一个端点，它提供比用户信息端点更详细的信息，但需要更彻底的认证。不同的选择（自然地）提供不同的利益和折衷，但是对这些的全面讨论超出了本节的范围。                                       |

### 实现用户端点

在授权服务器上，我们可以轻松添加该端点

AuthserverApplication.java

```java
@SpringBootApplication
@RestController
@EnableAuthorizationServer
@EnableResourceServer
public class AuthserverApplication {

  @RequestMapping("/user")
  public Principal user(Principal user) {
    return user;
  }

  ...

}

```

我们添加了`@RequestMapping`与[第二部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)的UI服务器相同的[部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)，并且还添加了`@EnableResourceServer`Spring OAuth 的注释，默认情况下，除了“/oauth/ *”端点之外，所有这些注释都保存在授权服务器中。

有了这个端点，我们可以测试它和问候资源，因为它们现在都可以接受由授权服务器创建的承载令牌：

```shell
$ TOKEN=2219199c-966e-4466-8b7e-12bb9038c9bb
$ curl -H "Authorization: Bearer $TOKEN" localhost:9000
{"id":"03af8be3-2fc3-4d75-acf7-c484d9cf32b1","content":"Hello World"}
$ curl -H "Authorization: Bearer $TOKEN" localhost:9999/uaa/user
{"details":...,"principal":{"username":"user",...},"name":"user"}

```

（用您自己的授权服务器获得的访问令牌的值代替自己的工作）。

### UI服务器

我们需要完成的这个应用程序的最后一部分是UI服务器，提取认证部分并委托给授权服务器。因此，与[资源服务器一样](https://spring.io/guides/tutorials/spring-security-and-angular-js/#changing-the-resource-server)，我们首先需要删除Spring Session和Redis依赖关系，并将其替换为Spring OAuth2。因为我们在UI层使用Zuul我们实际使用`spring-cloud-starter-oauth2`，而不是`spring-security-oauth2`直接（这个参数设置一些自动配置为通过代理中继令牌）。

一旦完成，我们可以删除会话过滤器和“/用户”端点，并设置应用程序重定向到授权服务器（使用`@EnableOAuth2Sso`注释）：

UiApplication.java

```java
@SpringBootApplication
@EnableZuulProxy
@EnableOAuth2Sso
public class UiApplication {

  public static void main(String[] args) {
    SpringApplication.run(UiApplication.class, args);
  }

...

}

```

回想一下[第四部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)，UI服务器凭借`@EnableZuulProxy`这个API作为一个API网关，我们可以在YAML中声明路由映射。所以“/user”端点可以代理授权服务器：

application.yml

```yaml
zuul:
  routes:
    resource:
      path: /resource/**
      url: http://localhost:9000
    user:
      path: /user/**
      url: http://localhost:9999/uaa/user

```

最后，我们需要将应用程序更改为a，`WebSecurityConfigurerAdapter`因为现在它将用于修改SSO过滤器链中的默认设置`@EnableOAuth2Sso`：

SecurityConfiguration.java

```java
@SpringBootApplication
@EnableZuulProxy
@EnableOAuth2Sso
public class UiApplication extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception {
      http
          .logout().logoutSuccessUrl("/").and()
          .authorizeRequests().antMatchers("/index.html", "/app.html", "/")
          .permitAll().anyRequest().authenticated().and()
          .csrf()
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
    }

}

```

主要的变化（除了基类名外）是匹配者进入他们自己的方法，并且不再需要`formLogin()`。显式`logout()`配置显式添加一个不受保护的成功url，以便XHR请求`/logout`成功返回。

还有一些强制的外部配置属性，使`@EnableOAuth2Sso`注释能够与正确的授权服务器进行联系和验证。所以我们需要这个`application.yml`：

application.yml

```yaml
security:
  ...
  oauth2:
    client:
      accessTokenUri: http://localhost:9999/uaa/oauth/token
      userAuthorizationUri: http://localhost:9999/uaa/oauth/authorize
      clientId: acme
      clientSecret: acmesecret
    resource:
      userInfoUri: http://localhost:9999/uaa/user

```

大部分是关于OAuth2客户端（“acme”）和授权服务器的位置。还有一个`userInfoUri`（就像资源服务器一样），以便用户可以在UI应用程序本身进行身份验证。

> 如果您希望UI应用程序能够自动刷新过期的访问令牌，则必须`OAuth2RestOperations`注入进行中继的Zuul过滤器。你可以通过创建一个这种类型的bean来做到这一点（查看`OAuth2TokenRelayFilter`详细信息）：

```java
@Bean
protected OAuth2RestTemplate OAuth2RestTemplate(
    OAuth2ProtectedResourceDetails resource, OAuth2ClientContext context) {
  return new OAuth2RestTemplate(resource, context);
}

```

#### 在客户端

对前端UI应用程序进行了一些调整，我们仍然需要触发重定向到授权服务器。在这个简单的演示中，我们可以将Angular应用程序剥离到最基本的要领，以便更清楚地看到发生的事情。所以我们现在放弃使用表单或路线，我们回到一个单一的Angular组件：

hello.js

```javascript
var AppComponent = ng.core.Component({
    selector : 'app',
    templateUrl : 'app.html'
}).Class({
    constructor : [ng.http.Http, function(http) {
        var self = this;
        this.greeting = {id:'', msg:''};
        this.authenticated = false;
        this.authenticate = function() {

            http.get('user').subscribe(function(response) {
                if (response.json().name) {
                    self.authenticated = true;
                    http.get('resource/').subscribe(response => self.greeting = response.json());
                } else {
                    self.authenticated = false;
                }
            }, function() {self.authenticated = false;});

        }
        this.logout = function() {
            http.post('logout', {}).subscribe(function() {
                self.authenticated = false;
            });
        };
        this.authenticate();
    }]
});

var AppModule = ng.core.NgModule({
    imports: [ng.platformBrowser.BrowserModule, ng.http.HttpModule],
    declarations: [AppComponent],
    bootstrap: [AppComponent]
  }).Class({constructor : function(){}})

document.addEventListener('DOMContentLoaded', function() {
    ng.platformBrowserDynamic.platformBrowserDynamic().bootstrapModule(AppModule);
});

```

该`AppComponent`处理一切，获取用户的详细信息，如果成功，问候。它也提供了这个`logout`功能。

现在我们需要为这个新组件创建“app.html”模板：

app.html

```html
<div class="container">
  <ul class="nav nav-pills">
    <li><a>Home</a></li>
    <li><a href="login">Login</a></li>
    <li><a (click)="logout()">Logout</a></li>
  </ul>
</div>
<div class="container">
<h1>Greeting</h1>
<div [hidden]="!authenticated">
	<p>The ID is {{greeting.id}}</p>
	<p>The content is {{greeting.content}}</p>
</div>
<div [hidden]="authenticated">
	<p>Login to see your greeting</p>
</div>
</div>

```

并将其包含在主页中`<app/>`。

请注意，“登录”的导航链接是一个正常的链接`href`（不是一个角度路线）。这个“/login”端点是由Spring Security处理的，如果用户没有被认证，将导致重定向到授权服务器。

### 它是如何工作的？

现在一起运行所有的服务器，并通过浏览器访问[http://localhost:8080](http://localhost:8080/)的UI 。点击“登录”链接，您将被重定向到授权服务器进行身份验证（HTTP基本弹出窗口）并批准令牌授权（白标签HTML），然后重定向到用户界面中的主页，从OAuth2资源服务器使用相同的标记，因为我们使用UI进行身份验证。

如果您使用某些开发工具（通常是F12打开此工具，默认情况下在Chrome中运行，可能需要Firefox中的插件），则可以在浏览器中看到浏览器与后端之间的交互。这里有一个总结：

| 动作   | 路径                   | 状态   | 响应                |
| ---- | -------------------- | ---- | ----------------- |
| GET  | /                    | 200  | index.html        |
| GET  | / webjars / **       | 200  | Bootstrap和Angular |
| GET  | /js/hello.js         | 200  | 应用程序逻辑            |
| GET  | /app.html            | 200  | HTML部分内容          |
| GET  | /user                | 302  | 重定向到登录页面          |
| GET  | /login               | 302  | 重定向到auth服务器       |
| GET  | （uaa）/outh/authorize | 401  | （忽略）              |
| GET  | /login               | 302  | 重定向到auth服务器       |
| GET  | （uaa）/outh/authorize | 200  | HTTP基本身份验证发生在这里   |
| POST | （uaa）/outh/authorize | 302  | 用户批准授予，重定向到/登录    |
| GET  | /login               | 302  | 重定向到主页            |
| GET  | /user                | 200  | （Proxied）JSON认证用户 |
| GET  | /app.html            | 200  | HTML部分为主页         |
| GET  | /resource            | 200  | （代理）JSON问候        |

前缀（uaa）的请求是授权服务器。被标记为“忽略”的响应是Angular在XHR调用中收到的响应，由于我们没有处理这些数据，所以它们被放在地板上。我们在“/user”资源的情况下寻找一个认证的用户，但是由于它在第一个调用中不存在，所以这个响应被丢弃。

在用户界面的“/trace”端点中（向下滚动到底部），您将看到代理后端请求为“/user”和“/resource”，`remote:true`而不是cookie在[第四部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)）被用于认证。春季云安全一直照顾这对我们来说：通过认识到我们有`@EnableOAuth2Sso`和`@EnableZuulProxy`已经想通了，（默认），我们希望将令牌中继到代理后端。

> 正如在以前的部分，尝试使用不同的浏览器“/跟踪”，以便没有认证交叉的机会（例如，如果您使用Chrome浏览器测试用户界面使用Firefox）。

### 注销体验

如果您点击“注销”链接，您将看到主页更改（不再显示问候语），因此用户不再使用UI服务器进行身份验证。点击“登录”，但实际上*并不*需要通过授权服务器的认证和批准周期（因为您没有注销）。意见将分为是否是一个理想的用户体验，这是一个臭名昭着的棘手问题（单一登出：[科学直接文章](http://www.sciencedirect.com/science/article/pii/S2214212614000179)和[Shibboleth文档](https://wiki.shibboleth.net/confluence/display/SHIB2/SLOIssues)）。理想的用户体验可能在技术上是不可行的，而且有时用户确实需要他们所说的话，也必须保持警惕。“我希望”注销“注销我”听起来很简单，但显而易见的回应是，“退出什么？你想退出*所有*由这个SSO服务器控制的系统，或者只是你点击“注销”链接？如果您感兴趣，那么本教程[的后面部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_oauth2_logout_angular_js_and_spring_security_part_ix)将对其进行更深入的讨论。

### 结论

这几乎是我们通过Spring Security和Angular栈进行简单使用的结束。我们有一个很好的体系结构，在三个独立的组件（UI / API网关，资源服务器和授权服务器/令牌授权器）中有明确的责任。现在，所有图层中的非业务代码数量都是最小的，并且可以很容易地看到在哪里扩展，并通过更多的业务逻辑来改进实现。接下来的步骤是清理我们的授权服务器中的UI，并且可能会添加更多的测试，包括JavaScript客户端上的测试。另一个有趣的任务是提取所有的样板代码，并将其放在包含Spring Security和Spring Session自动配置的库（例如“spring-security-angular”）中，以及在Angular块中的导航控制器的一些webjars资源。[Spring Cloud](https://projects.spring.io/spring-cloud/)是新的，这些示例在写入时需要快照，但是有可用的候选版本和GA版本即将推出，因此请检查并[通过Github](https://github.com/spring-cloud)或[gitter.im](https://gitter.im/spring-cloud/spring-cloud)发送一些反馈。

本系列的[下一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_multiple_ui_applications_and_a_gateway_single_page_application_with_spring_and_angular_js_part_vi)是关于访问决策（不包括身份验证），并在同一代理背后采用多个UI应用程序。

### 附录：授权服务器的引导UI和JWT令牌

您将[在Github](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2)的[源代码中](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2)找到这个应用程序的另一个版本，它有一个漂亮的登录页面和用户批准页面，其实现方式类似于我们在[第二部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)的登录页面。它还使用[JWT](http://jwt.io/)对令牌进行编码，所以资源服务器不用使用“/user”端点，而是从令牌中抽取足够的信息来进行简单的认证。浏览器客户端仍然使用它，通过UI服务器进行代理，以便它可以确定用户是否被认证（与实际应用中对资源服务器的可能调用次数相比，它不需要经常进行认证）。

## 多个UI应用程序和一个网关

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_sso_with_oauth2_angular_js_and_spring_security_part_v)如何在“单页面应用程序”中使用[Spring Security](https://projects.spring.io/spring-security) 和 [Angular](http://angular.io/)。在这里，我们展示了如何将[Spring Session](https://projects.spring.io/spring-security-oauth/)与[Spring Cloud](https://projects.spring.io/spring-cloud/)一起使用，将我们在第二部分和第四部分中构建的系统的特性结合起来，实际上最终构建了3个具有不同职责的单页面应用程序。目标是构建一个网关（如[第四部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)），不仅用于API资源，而且还用于从后端服务器加载用户界面。我们简化了[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)的令牌扯皮通过使用网关将身份验证传递给后端。然后，我们扩展系统，以显示如何在后端进行本地，细粒度的访问决策，同时仍然控制网关上的身份和身份验证。这是构建分布式系统的一个非常强大的模型，并且在我们构建的代码中引入这些特性时，我们可以探索许多好处。

> 提醒：如果您正在使用示例应用程序完成本节，请务必清除Cookie和HTTP Basic凭据的浏览器缓存。在Chrome中，最好的方法是打开一个新的隐身窗口。

### 目标架构

下面是我们将要开始构建的基本系统的图片：

![系统的组件](https://raw.githubusercontent.com/spring-guides/tut-spring-security-and-angular-js/master/double/double-simple.png)

像本系列中的其他示例应用程序一样，它具有UI（HTML和JavaScript）和资源服务器。像[第四部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)的示例一样，它有一个网关，但这里是单独的，而不是用户界面的一部分。UI有效地成为后端的一部分，给我们更多的选择来重新配置和重新实现功能，并带来其他好处，我们将看到。

浏览器去了网关的一切，它不必知道后端的架构（根本上，它不知道有后端）。浏览器在这个网关上做的事情之一是身份验证，例如，它发送一个用户名和密码，就像在[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)，它得到一个cookie作为回报。在随后的请求中，它自动显示cookie，网关将其传递到后端。不需要在客户端上写入代码来启用cookie传递。后端使用cookie进行身份验证，并且由于所有组件共享一个会话，所以它们共享关于该用户的相同信息。与[第五部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_sso_with_oauth2_angular_js_and_spring_security_part_v)对比 Cookie必须在网关中转换为访问令牌，访问令牌必须由所有后端组件独立解码。

正如在[第四部分中一样](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)，网关简化了客户端和服务器之间的交互，并且提供了一个小而明确的表面来处理安全问题。例如，我们不需要担心[跨源资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)，因为容易出错，这是一个可喜的解决办法。

我们要建立的完整项目的源代码在[Github这里](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/double)，所以你可以直接克隆项目并直接从那里工作。在这个系统的最终状态（“双管理”）中有一个额外的组件，所以现在就忽略它。

### 建立后端

在这个架构中，后端与我们在[第三部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)构建的[“spring-session”](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/spring-session)示例非常相似，只是它实际上并不需要登录页面。最简单的方法去我们想要的这里可能是从第三部分复制“资源”服务器，并从走UI [“基本”](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/basic)的样本[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)。为了从“基本”的用户界面到我们想要的用户界面，我们只需要添加一些依赖关系（就像我们在第三部分中第一次使用[Spring Session](https://github.com/spring-projects/spring-session/)）：

的pom.xml

```xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-redis</artifactId>
</dependency>

```

由于这是一个用户界面，因此不需要“/ resource”端点。当你这样做的时候，你将会有一个非常简单的Angular应用（和“基本”样例一样），这大大简化了对它的行为的测试和推理。

最后，我们希望这个服务器作为后端运行，所以我们给它一个非默认端口来监听（in `application.properties`）：

application.properties

```properties
server.port: 8081
security.sessions: NEVER

```

如果这是*整个*内容，`application.properties`那么应用程序将是一个安全的，并且被一个名为“user”的用户所访问，密码是随机的，但是在启动时打印在控制台（日志级INFO）上。“security.sessions”设置意味着Spring Security将接受cookie作为认证令牌，但除非已经存在，否则不会创建它们。

### 资源服务器

资源服务器很容易从我们现有的一个样本中生成。它与[第三部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)的“spring-session”资源服务器相同：只是一个“/resource”端点Spring会话来获得分布式会话数据。我们希望这个服务器有一个非默认端口来监听，我们希望能够在会话中查找认证，所以我们需要这个（in `application.properties`）：

application.properties

```properties
server.port: 9000
security.sessions: NEVER

```

我们将在POST消息资源中进行更改，这是本教程中的一个新功能。这意味着我们需要在后端进行CSRF保护，而我们需要按照惯常的方法使Spring Security和Angular良好地配合：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
	http.csrf()
			.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
}

```

如果你想看[一下，](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/double/resource)完整的示例[在github中](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/double/resource)。

### 网关

对于一个网关的初始实现（最简单的事情，可能会工作），我们可以采取一个空的Spring Boot Web应用程序，并添加`@EnableZuulProxy`注释。正如我们在[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)看到的那样，有几种方法可以做到这一点，一种是使用[Spring Initializr](https://start.spring.io/)生成一个框架项目。更简单一点，就是使用[Spring Cloud Initializr](https://cloud-start.spring.io/)，这是同样的事情，但对于[Spring Cloud](https://cloud.spring.io/)应用程序来说。使用与第一部分相同的命令行操作序列：

```shell
$ mkdir gateway && cd gateway
$ curl https://cloud-start.spring.io/starter.tgz -d style=web \
  -d style=security -d style=cloud-zuul -d name=gateway \
  -d style=redis | tar -xzvf -

```

然后，您可以将该项目（默认情况下是普通的Maven Java项目）导入到您最喜欢的IDE中，或者仅在命令行中使用这些文件和“mvn”。[github上](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/double/gateway)有[一个版本](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/double/gateway)如果你想从那里去，但它有一些额外的功能，我们不需要。

从初始化的Initializr应用程序开始，我们添加Spring Session依赖项（就像上面的UI一样）。网关已经准备好运行了，但是它还不知道我们的后端服务，所以让我们把它设置在它的`application.yml`重命名（`application.properties`如果你做了上面的curl的话）：

application.yml

```yaml
zuul:
  routes:
    ui:
      url: http://localhost:8081
   resource:
      url: http://localhost:9000
security:
  user:
    password:
      password
  sessions: ALWAYS

```

代理中有两条路由，一条是UI和资源服务器，我们设置了一个默认密码和一个会话持久性策略（告诉Spring Security始终创建一个认证会话）。最后一点很重要，因为我们希望验证，因此会话在网关中进行管理。

### 正在运行

我们现在有三个组件，运行在三个端口上。如果您将浏览器指向[http//localhost:8080/ui/](http://localhost:8080/ui/)您应该获得HTTP Basic挑战，并且可以通过“用户/密码”（网关中的凭证）进行身份验证，一旦这样做，您应该看到在UI中通过代理到资源服务器的后台调用的问候语。

如果您使用某些开发工具（通常是F12打开此工具，默认情况下在Chrome中运行，可能需要Firefox中的插件），则可以在浏览器中看到浏览器与后端之间的交互。这里有一个总结：

| 动作   | 路径                            | 状态   | 响应                |
| ---- | ----------------------------- | ---- | ----------------- |
| GET  | /ui/                          | 401  | 浏览器提示进行身份验证       |
| GET  | /ui/                          | 200  | index.html        |
| GET  | /ui/css/angular-bootstrap.css | 200  | Twitter引导CSS      |
| GET  | /ui/js/angular-bootstrap.js   | 200  | Bootstrap和Angular |
| GET  | /ui/js/hello.js               | 200  | 应用程序逻辑            |
| GET  | /ui/user                      | 200  | 认证                |
| GET  | /resource/                    | 200  | JSON问候            |

您可能看不到401，因为浏览器将主页加载视为单个交互。所有的请求都是代理的（网关中没有内容，除了执行器管理端点之外）。

太好了，它在工作！您有两台后端服务器，其中一台是一个用户界面，每台服务器都具有独立的功能，可以独立进行测试，并且使用您控制的安全网关连接在一起，并为其配置了身份验证。如果后端无法访问浏览器，则无关紧要（实际上，这可能是一个优势，因为它可以更好地控制物理安全性）。

### 添加登录表单

就像在[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)的“基本”示例中一样，我们现在可以向网关添加一个登录表单，例如通过复制[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)的代码。当我们这样做时，我们也可以在网关中添加一些基本的导航元素，这样用户就不必知道代理中UI后端的路径。因此，首先将静态资产从“单个”UI复制到网关中，删除消息呈现并在我们的主页（在`<app/>`某处）插入登录表单：

app.html

```html
<div class="container" [hidden]="authenticated">
	<form role="form" (submit)="login()">
		<div class="form-group">
			<label for="username">Username:</label> <input type="text"
				class="form-control" id="username" name="username"
				[(ngModel)]="credentials.username" />
		</div>
		<div class="form-group">
			<label for="password">Password:</label> <input type="password"
				class="form-control" id="password" name="password"
				[(ngModel)]="credentials.password" />
		</div>
		<button type="submit" class="btn btn-primary">Submit</button>
	</form>
</div>

```

而不是消息渲染，我们将有一个不错的导航按钮：

的index.html

```Html
<div class="container" [hidden]="!authenticated">
	<a class="btn btn-primary" href="/ui/">Go To User Interface</a>
</div>

```

如果你正在看github中的示例，它也有一个最小的导航栏和一个“注销”按钮。以下是屏幕截图中的登录表单：

![登录页面](https://raw.githubusercontent.com/spring-guides/tut-spring-security-and-angular-js/master/double/login.png)

为了支持登录表单，我们需要一些带有实现`login()`我们在其中声明的函数的组件`<form/>`，并且我们需要设置这个`authenticated`标记，这样主页将根据用户是否被认证而呈现不同的形式。例如：

gateway.js

```javascript
var AppComponent = ng.core.Component({
        templateUrl: 'app.html',
        selector: 'app'
    }).Class({
        constructor : [ng.http.Http, function(http){
            var self = this;
            this.credentials = {username:'', password:''};
            this.authenticated = false;
            var authenticate = function(credentials) {
                var headers = credentials ? {
                    authorization : "Basic " + btoa(credentials.username + ":" + credentials.password)
                } : {};
                http.get('user', {headers: headers}).subscribe(function(response) {
                    var data =response.json();
                    self.authenticated = data && data.name;
                    self.user = self.authenticated ? data.name : '';
                });
            }
            this.login = function() {
                authenticate(self.credentials);
                return false;
            };
            this.logout = function() {
                http.post('logout', {}).subscribe(function() {
                    self.authenticated = false;
                });
            }
            authenticate();
        }]
    });

```

`authenticate()`功能的实施与[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)相似。

我们可以使用`self`存储`authenticated`标志，因为在这个简单的应用程序中只有一个组件。

如果我们运行这个增强的网关，而不必记住用户界面的URL，我们可以加载主页和链接。以下是经过身份验证的用户的主页：

![主页](https://raw.githubusercontent.com/spring-guides/tut-spring-security-and-angular-js/master/double/home.png)

### 在后端粒度访问决策

到目前为止，我们的应用程序在功能上与[第三部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)或[第四](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_api_gateway_pattern_angular_js_and_spring_security_part_iv)[部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_resource_server_angular_js_and_spring_security_part_iii)非常相似，但是增加了一个专用网关。额外层的优势可能还不明显，但是我们可以通过扩展系统来强调它。假设我们想要使用该网关来公开另一个后端UI，以便用户“管理”主UI中的内容，并且我们希望限制对具有特殊角色的用户的访问权限。因此，我们将在代理后面添加一个“Admin”应用程序，系统将如下所示：

![系统的组件](https://raw.githubusercontent.com/spring-guides/tut-spring-security-and-angular-js/master/double/double-components.png)

在网关中有一个新的组件（Admin）和一个新的路由`application.yml`：

application.yml

```yaml
zuul:
  routes:
    ui:
      url: http://localhost:8081
    admin:
      url: http://localhost:8082
    resource:
      url: http://localhost:9000

```

在“用户”角色中，用户可以使用现有用户界面的事实在网关框（绿色字体）中的上面的框图中指出，因为需要“管理员”角色才能转到管理应用程序。对于“ADMIN”角色的访问决策可以应用在网关中，在这种情况下，它将出现在a中`WebSecurityConfigurerAdapter`，或者可以在管理应用程序本身中应用（我们将在下面看到如何做）。

首先，创建一个新的Spring Boot应用程序，或复制UI并对其进行编辑。除了名称之外，您不需要在UI应用程序中进行任何更改。完成的应用程序在[这里Github](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/double/admin)。

假设在Admin应用程序中我们要区分“READER”和“WRITER”角色，这样我们可以允许（比方说）作为审计人员的用户查看主要管理员用户所做的更改。这是一个细粒度的访问决策，其中规则只有在后端应用程序中才是已知的，而且应该是已知的。在网关中，我们只需要确保我们的用户帐户具有所需的角色，并且这个信息是可用的，但是网关不需要知道如何解释它。在网关中，我们创建用户帐户以保持示例应用程序独立：

SecurityConfiguration.class

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

  @Autowired
  public void globalUserDetails(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
      .withUser("user").password("password").roles("USER")
    .and()
      .withUser("admin").password("admin").roles("USER", "ADMIN", "READER", "WRITER")
    .and()
      .withUser("audit").password("audit").roles("USER", "ADMIN", "READER");
  }

}

```

其中“admin”用户已增强了3个新角色（“ADMIN”，“READER”和“WRITER”），我们还添加了“ADMIN”访问权限的“审核”用户，而不是“WRITER”。

> 在生产系统中，用户帐户数据将在后端数据库（很可能是目录服务）中进行管理，而不是在Spring配置中进行硬编码。连接到这样的数据库的示例应用程序很容易在互联网上找到，例如在[Spring Security示例中](https://github.com/spring-projects/spring-security/tree/master/samples)。

访问决定进入管理员应用程序。对于“ADMIN”角色（这是后端全局所需的），我们在Spring Security中执行：

SecurityConfiguration.java

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

@Override
  protected void configure(HttpSecurity http) throws Exception {
    http
    ...
      .authorizeRequests()
        .antMatchers("/index.html", "/app.html", "/login", "/").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
    ...
  }

}

```

对于“READER”和“WRITER”角色，应用程序本身是分开的，由于应用程序是用JavaScript实现的，所以我们需要做出访问决定。一种方法是通过路由器在主页中嵌入一个计算视图：

app.html

```html
<div class="container">
	<h1>Admin</h1>
	<router-outlet></router-outlet>
</div>

```

当组件加载时，路由是混合的：

admin.js

```javascript
var AppComponent = ng.core.Component({
        templateUrl: 'app.html',
        selector: 'app',
        providers: [AppService]
    }).Class({
        constructor : [AppService, ng.http.Http, ng.router.Router, function(app, http, router){
            var self = this;
            self.user = {};
            app.authenticate(response => {
              self.user = response.json();
              if (!app.authenticated) {
                router.navigate(['/unauthenticated'])
              } else {
                if (app.writer) {
                  router.navigate(['/write'])
                } else {
                  router.navigate(['/read'])
                }
              }
            });
        }]
})

var routes = [
    { path: '', pathMatch: 'full', redirectTo: 'read'},
    { path: 'read', component: ReadComponent},
    { path: 'write', component: WriteComponent},
    { path: 'unauthenticated', component: UnauthenticatedComponent}
];
...

```

应用程序所做的第一件事就是检查用户是否已通过身份验证，并通过查看用户数据来计算路由。每个组件（每个路由一个组件）都必须单独实现。以下`ReadComponent`是一个例子：

admin.js

```javascript
var ReadComponent = ng.core.Component({
    templateUrl : 'read.html'
}).Class({
    constructor : [AppService, ng.http.Http, function(app, http) {
        var self = this;
        self.greeting = {id:'', content:''};
        http.get('/resource').subscribe(response => self.greeting =response.json());
    }]
});

```

read.html

```html
<div>
	<p>The ID is {{greeting.id}}</p>
	<p>The content is {{greeting.content}}</p>
</div>

```

这`WriteComponent`是相似的，但有一个形式来改变在后端的消息：

admin.js

```javascript
var WriteComponent = ng.core.Component({
  templateUrl : 'write.html'
}).Class({
  constructor : [AppService, ng.http.Http, function(app, http) {
      var self = this;
      this.greeting = {id:'', content:''};
      http.get('/resource').subscribe(response => self.greeting = response.json());
      self.update = function() {
        http.post('/resource', {content: self.greeting.content}).subscribe(function(response) {
          self.greeting = response.json()
        })
      }
  }]
});

```

read.html

```html
<form (submit)="update()">
	<p>The ID is {{greeting.id}}</p>
	<div class="form-group">
		<label for="username">Content:</label> <input type="text"
			class="form-control" id="content" name="content" [(ngModel)]="greeting.content"/>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>

```

在`AppService`还需要提供数据来计算的路线，所以在`authenticate()`函数中我们看到：

admin.js

```javascript
        http.get('/user').subscribe(function(response) {
            var user = response.json();
            if (user.name) {
                self.authenticated = true;
                self.writer = user.roles && user.roles.indexOf("ROLE_WRITER")>0;
            } else {
                self.authenticated = false;
                self.writer = false;
            }
            callback && callback(response);
        })

```

为了在后端支持这个功能，我们需要`/user`端点，例如在我们的主应用程序类中：

AdminApplication.java

```java
@SpringBootApplication
@RestController
public class AdminApplication {

  @RequestMapping("/user")
  public Map<String, Object> user(Principal user) {
    Map<String, Object> map = new LinkedHashMap<String, Object>();
    map.put("name", user.getName());
    map.put("roles", AuthorityUtils.authorityListToSet(((Authentication) user)
        .getAuthorities()));
    return map;
  }

  public static void main(String[] args) {
    SpringApplication.run(AdminApplication.class, args);
  }

}

```

> 角色名称从带有“ROLE_”前缀的“/user”端点返回，所以我们可以将它们与其他类型的权限区分开来（这是Spring Security的一个事情）。因此，在JavaScript中需要“ROLE_”前缀，但在Spring Security配置中不需要，前者从方法名称中明确指出“roles”是操作的重点。
### 支持管理界面的网关改变

我们也将使用这些角色在网关中做出访问决定（所以我们可以有条件地显示一个到管理界面的链接），所以我们应该在网关的“/user”端点添加“角色” 。一旦到位，我们可以添加一些JavaScript来设置一个标志来指示当前用户是“ADMIN”。在`authenticated()`函数中：

gateway.js

```Javascript
http.get('user', {headers: headers}).subscribe(function(response) {
    var data =response.json();
    self.admin = data && data.roles && data.roles.indexOf("ROLE_ADMIN")>-1;
    self.authenticated = data && data.name;
    self.user = self.authenticated ? data.name : '';
});

```

我们还需要将`admin`标志重置`false`为用户注销时的状态：

gateway.js

```javascript
this.logout = function() {
    http.post('logout', {}).subscribe(function() {
        self.authenticated = false;
        self.admin = false;
    });
}

```

然后在HTML中我们可以有条件地显示一个新的链接：

app.html

```Html
<div class="container" [hidden]="!authenticated">
	<a class="btn btn-primary" href="/ui/">Go To User Interface</a>
</div>
<br />
<div class="container" [hidden]="!authenticated || !admin">
	<a class="btn btn-primary" href="/admin/">Go To Admin Interface</a>
</div>

```

运行所有应用程序并转到[http://localhost:8080](http://localhost:8080/)查看结果。一切都应该工作正常，用户界面应该改变，取决于当前通过身份验证的用户。

### 我们为什么在这里？

现在我们有一个很好的小系统，有两个独立的用户界面和一个后端资源服务器，所有这些系统都受到网关中同一个认证的保护。Gateway作为一个微代理的事实使得后端安全问题的实现非常简单，并且可以自由地关注自己的业务问题。Spring Session的使用（再次）避免了大量的麻烦和潜在的错误。

一个强大的功能是，后端可以独立地拥有任何他们喜欢的身份验证（例如，如果您知道其物理地址和一组本地凭据，则可以直接进入UI）。只要网关能够对用户进行身份验证并为其分配满足后端访问规则的元数据，网关就会施加一系列完全无关的约束。这是能够独立开发和测试后端组件的极好设计。如果我们想要的话，我们可以回到外部的OAuth2服务器（如[第五部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_sso_with_oauth2_angular_js_and_spring_security_part_v)，甚至完全不同的东西）在网关的身份验证，后端将不需要接触。

该体系结构的一个额外功能（单个网关控制认证，以及所有组件的共享会话令牌）是“单一注销”，这是我们在[第五部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_sso_with_oauth2_angular_js_and_spring_security_part_v)难以实现的功能。更确切地说，单一注销的用户体验的一种特定方法是在我们完成的系统中自动提供的：如果用户注销了任何UI（网关，UI后端或管理后端），他将从所有的其他人，假设每个单独的UI以相同的方式实现“注销”功能（使会话失效）。

感谢：再次感谢所有帮助我开发本系列的人，特别是[Rob Winch](https://spring.io/team/rwinch)和[ThorstenSpäth](https://twitter.com/thspaeth)对细节和源代码的仔细审查。自从[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)出版以来，没有什么太大的变化，但其他所有部分都是为了回应读者的评论和见解而发展起来的，所以也要感谢任何读过这些部分的人，不厌其烦地参加讨论。

## 模块化的角度应用

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)如何在“单页面应用程序”中使用[Spring Security](https://projects.spring.io/spring-security) 和 [Angular](https://angularjs.org/)。这里我们展示如何模块化客户端代码，以及如何使用Angular默认使用的“好的”URL路径，但是Spring MVC并不知道这些路径。这是教程的第七部分，您可以通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)，从头开始构建应用程序的基本构建块，或者直接转到[Github中](https://github.com/dsyer/spring-security-angular/tree/master/modular)的[源代码](https://github.com/dsyer/spring-security-angular/tree/master/modular)。我们将能够清理本系列其余部分的JavaScript代码中的大量松散结尾，同时展示如何能够很好地适应Spring Security和Spring Boot构建的后端服务器。

### 打破应用程序

我们在本系列中使用的示例应用程序已经足够小了，以至于我们可以用一个单一的JavaScript源文件来完成整个过程。没有更大的应用程序会以这种方式结束，即使它开始像这样的生活，所以模仿现实生活中的样品，我们将打破。一个好的起点是从[第二部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)获取“单个”应用程序，并查看源代码中的结构。以下是静态内容的目录列表（不包括服务器上的“application.yml”）：

```Shell
static/
 js /
   hello.js
 app.html
 home.html的
 的login.html
 的index.html

```

这有几个问题。一个显而易见的是：所有的JavaScript都在一个文件（`hello.js`）中。另一个更微妙：我们的应用程序（“login.html”和“home.html”）中的视图有HTML“partials”，但它们都是平面结构，并且与使用它们的组件代码无关。

所有的Angular代码实际上已经被模块化为组件（“app”，“home”和“login”），并且组件很好地映射到部分（分别为“home.html”和“login.html”）。所以让我们把它们分解成这些部分：

```shell
static/
  js/
    app/
      app.js
      app.html
    home/
      home.js
      home.html的
    login/
      login.js
      的login.html
    hello.js
  的index.html

```

组件定义已经进入了他们自己的模块，以及他们需要运行的HTML - 好的和模块化的。如果我们需要图像或自定义样式表，我们也可以做到这一点。

> 所有的客户端代码都在一个单独的目录“js”下（除了`index.html`因为这是一个“欢迎”页面并从“静态”目录自动加载）。这是有意的，因为它可以很容易地将一个Spring Security访问规则应用于所有的静态资源。这些都是不安全的（因为`/js/**`Spring Boot应用程序默认是不安全的），但是对于其他应用程序可能需要其他规则，在这种情况下，您将选择不同的路径。

例如，这里是`home.js`：

```
var HomeComponent = ng.core.Component({
    templateUrl : 'js/home/home.html'
}).Class({
    constructor : [AppService, ng.http.Http, function(app, http) {
        var self = this;
        this.greeting = {id:'', msg:''};
        http.get('resource').subscribe(response => self.greeting =response.json());
        this.authenticated = function() { return app.authenticated; };
    }]
});

```

这是新的`hello.js`：

hello.js

```
var RequestOptionsService = ng.core.Class({
    extends: ng.http.BaseRequestOptions,
    constructor : function() {},
    merge: function(opts) {
        opts.headers = new ng.http.Headers(opts.headers ? opts.headers : {});
        opts.headers.set('X-Requested-With', 'XMLHttpRequest');
        return opts.merge(opts);
    }
});

var routes = [
    { path: '', pathMatch: 'full', redirectTo: 'home'},
    { path: 'home', component: HomeComponent},
    { path: 'login', component: LoginComponent}
];

var AppModule = ng.core.NgModule({
    imports: [ng.platformBrowser.BrowserModule, ng.http.HttpModule,
            ng.router.RouterModule.forRoot(routes), ng.forms.FormsModule],
    declarations: [HomeComponent, LoginComponent, AppComponent],
    providers : [{ provide: ng.http.RequestOptions, useClass: RequestOptionsService }],
    bootstrap: [AppComponent]
  }).Class({constructor : function(){}});

document.addEventListener('DOMContentLoaded', function() {
    ng.platformBrowserDynamic.platformBrowserDynamic().bootstrapModule(AppModule);
});

```

请注意，“hello”模块是如何*依靠*其他两个在初始声明中列出它们的。为了完成这个工作，你只需要按照正确的顺序加载模块定义`index.html`：

```
...
<script src="js/app/app.js" type="text/javascript"></script>
<script src="js/home/home.js" type="text/javascript"></script>
<script src="js/login/login.js" type="text/javascript"></script>
<script src="js/hello.js" type="text/javascript"></script>
...

```

这是在行动中的角度依赖管理系统。其他框架也有类似的功能。而且，在一个更大的应用程序中，可以使用构建时间步骤将所有JavaScript捆绑在一起，以便浏览器可以高效地加载它，但这几乎是一个挑剔的问题。

### 使用“自然的”路由

Angular `Router`默认使用与路由相匹配的URL路径，例如，登录页面`hello.js`以“/login”的形式指定为路由，并且在实际URL（在浏览器窗口中看到的那个）中翻译为“/login” 。这很尴尬，因为它不可收藏或可刷新。我们需要在后端执行的操作是`index.html`通过根路径“/”加载，在所有路由上保持活动状态。如果*只有*静态资源，则不能这样做，因为`index.html`只能以一种方式加载，但是如果在堆栈中有一些活动组件（代理服务器或一些服务器端逻辑），则可以安排它工作`index.html`从所有Angular路线加载。

在这个系列中你有Spring Boot，所以当然你有服务器端的逻辑，并且使用一个简单的Spring MVC控制器，你可以在你的应用程序中自然化路由。所有你需要的是一个方法来枚举服务器中的Angular路由。在这里，我们选择通过命名约定来做到这一点：所有不包含句点（并且已经没有明确映射）的路径是Angular路由，并且应该转发到主页：

```java
@RequestMapping(value = "/{path:[^\\.]*}")
public String redirect() {
  return "forward:/";
}

```

这个方法只需要在Spring应用程序的某个地方`@Controller`（而不是`@RestController`）。我们使用“forward”（而不是“redrect”），以便浏览器记住“真实”路由，这就是用户在URL中看到的内容。这也意味着Spring Security中任何保存请求的身份验证机制都可以开箱即用，尽管我们不会在这个应用程序中利用这个机制。

> 示例代码中的所有应用程序都已经有了这个请求映射。它一直在那里，如果你一直沿用和使用Github的代码。                                      |

再加上你需要`<base/>`HTML中头部的一个元素`index.html`，你需要确保菜单栏中的链接指向正确的路径：

的index.html

```html
<html>
<head>
<base href="/" />
...
</head>
<body ng-app="hello" ng-cloak class="ng-cloak">
...
</html>

```

app.html

```Html
<div class="container">
  <ul class="nav nav-pills">
    <li><a routerLinkActive="active" routerLink="/home">Home</a></li>
    <li><a routerLinkActive="active" routerLink="/login">Login</a></li>
    <li><a (click)="logout()">Logout</a></li>
  </ul>
</div>

```

Angular使用该`<base/>`元素来锚定路由并编写在浏览器中显示的URL。您正在运行Spring Boot应用程序，因此默认设置是从根路径“/”（在端口8080上）提供。如果您需要能够使用相同的应用程序从不同的根路径提供服务，那么您将需要使用服务器端模板将该路径呈现到HTML中（许多人更喜欢使用单页应用程序的静态资源，所以他们被固定的根路径）。

### 结论

在本节中，我们已经看到了如何模块化一个Angular应用程序（以本教程第二[部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_the_login_page_angular_js_and_spring_security_part_ii)的应用程序为出发点），如何使它重定向到登录页面，以及如何使用可以键入的“自然”路线或用户容易书签。我们从教程的最后几节中退后一步，重点介绍了客户端代码，并暂时抛弃了我们在第III-VI节中构建的分布式体系结构。这并不意味着这里的变化不能适用于其他应用程序（实际上这是相当平凡的） - 这只是为了简化服务器端代码，而我们正在学习如何在客户端上做事情。还有*人* 我们使用或简要讨论了一些服务器端的特性（例如，在Spring MVC中使用“forward”视图来实现“自然的”路由），所以我们继续了Angular和Spring共同工作的主题，显示他们在这里和那里做了很小的调整。

## 测试一个 Angular 的应用程序

在本节中，我们继续[讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_modular_angular_js_and_spring_security_part_vii)如何在“单页面应用程序”中使用[Spring Security](https://projects.spring.io/spring-security) 和 [Angular](http://angular.io/)。在这里，我们展示了如何使用JavaScript测试框架[Jasmine](https://jasmine.github.io/2.0/introduction.html)编写和运行客户端代码的单元测试。您可以通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)来了解应用程序的基本构建块，或者从头开始构建它，或者您可以直接转到[Github中](https://github.com/dsyer/spring-security-angular/tree/master/basic)的[源代码](https://github.com/dsyer/spring-security-angular/tree/master/basic)（与第一部分相同的源代码，但现在添加了测试）。本节实际上使用Spring或Spring Security的代码非常少，但它涵盖了客户端测试的方式，在通常的Javascript社区资源中可能不那么容易找到，而且我们认为这对大多数人来说是舒适的的Spring用户。

与本系列的其余部分一样，构建工具对于Spring用户来说是典型的，而不是那些经验丰富的前端开发人员。因此，我们寻找可以从Java IDE中使用的解决方案，并在命令行上使用熟悉的Java构建工具。如果您已经了解了Jasmine和Javascript测试，并且您很高兴使用基于Node.js的工具链（例如`npm`，`grunt`等等），那么您可能完全跳过本节。如果您在Eclipse或IntelliJ中更加舒适，并且希望为前端使用与后端相同的工具，那么本节将对此感兴趣。当我们需要一个命令行（例如持续集成）时，我们在这里的例子中使用了Maven，但是Gradle用户可能会发现相同的代码易于集成。

> 提醒：如果您正在使用示例应用程序完成本节，请务必清除Cookie和HTTP Basic凭据的浏览器缓存。在Chrome中，为单个服务器执行此操作的最佳方式是打开一个新的隐身窗口。

### 在Jasmine写一个 Specs

我们的“basic”应用程序中的“home" controller 非常简单，所以不需要太多的测试。这里是代码（`hello.js`）的提示：

```javascript
var AppComponent = ng.core.Component({
    selector : 'app',
    template : '<p>The ID is {{greeting.id}}</p><p>The content is {{greeting.content}}</p>'
}).Class({
    constructor : [ng.http.Http, function(http) {
        var self = this;
        self.greeting = {id:'', content:''};
        http.get("/resource").subscribe(function(response) {
          self.greeting =response.json()
        });
    }]
});

```

我们面临的主要挑战是`http`在测试中提供对象，所以我们可以对组件中的使用方式做出断言。实际上，即使在我们遇到这个挑战之前，我们也需要能够创建一个组件实例，所以我们可以测试它在加载时会发生什么。这是你如何做到这一点。

创建一个新文件，`spec.js`并把它放在“src/test/resources/static/js”中：

```javascript
var TestBed = ng.core.testing.TestBed;

ng.core.testing.getTestBed().initTestEnvironment(
    ng.platformBrowserDynamic.testing.BrowserDynamicTestingModule,
    ng.platformBrowserDynamic.testing.platformBrowserDynamicTesting()
  );

describe("AppComponent", function() {

  beforeEach(function() {
    TestBed.configureTestingModule({
      declarations: [
        AppComponent
      ]
    }).compileComponents();
  });

  it('should create the app', function() {
    const fixture = TestBed.createComponent(AppComponent);
    const app = fixture.debugElement.componentInstance;
    expect(app).toBeTruthy();
  });

});

```

在这个非常基本的测试套件中，我们有这些重要元素：

1. 我们`describe()`正在测试的东西（在这种情况下的“AppComponent”）与一个函数。
2. 在那个函数里面，我们提供了一个`beforeEach()`回调，它加载了Angular组件。
3. 行为是通过呼吁来表达的`it()`，在那里我们用语言陈述期望是什么，然后提供一个函数来作出断言。
4. 我们不使用ES6风格的函数声明（with `⇒`），因为尽管它们可能在浏览器中工作，但它们将不能在PhantomJS中工作，我们稍后要使用PhantomJS来自动化测试。
5. 在发生任何事情之前，测试环境将被初始化。这是大多数Angular应用程序的样板。

这里的测试函数是如此微不足道，实际上只是声明组件存在，所以如果失败，测试将失败。

>  “src/test/resources/static/js”是测试代码在Java应用程序中的一个合理位置，尽管可以为“src/test/javascript”做一个例子。我们稍后会看到，为什么把它放在测试类路径中是有意义的（尽管如果你习惯了Spring Boot约定，你可能已经看到了原因）。

现在我们需要一个用于这个Javascript代码的驱动程序，以我们在浏览器中加载的HTML页面的形式。创建一个名为“test.html”的文件，并将其放在“src / test / resources / static”中：

```Html
<!doctype html>
<html>
<head>

<title>Jasmine Spec Runner</title>
<link rel="stylesheet" type="text/css"
	href="webjars/jasmine/jasmine.css">
<script type="text/javascript" src="webjars/jasmine/jasmine.js"></script>
<script type="text/javascript" src="webjars/jasmine/jasmine-html.js"></script>
<script type="text/javascript" src="webjars/jasmine/boot.js"></script>

<!-- include source files here... -->
<script type="text/javascript" src="webjars/core-js/client/shim.min.js"></script>
<script type="text/javascript" src="webjars/rxjs/bundles/Rx.min.js"></script>
<script type="text/javascript" src="webjars/zone.js/dist/zone.min.js"></script>
<script type="text/javascript" src="webjars/reflect-metadata/Reflect.js"></script>
<script type="text/javascript"
	src="webjars/angular__core/bundles/core.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__common/bundles/common.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__compiler/bundles/compiler.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__platform-browser/bundles/platform-browser.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__http/bundles/http.umd.js"></script>
<script type="text/javascript" src="js/hello.js"></script>

<!-- include spec files here... -->
<script type="text/javascript"
	src="webjars/zone.js/dist/long-stack-trace-zone.min.js"></script>
<script type="text/javascript" src="webjars/zone.js/dist/proxy.min.js"></script>
<script type="text/javascript" src="webjars/zone.js/dist/sync-test.js"></script>
<script type="text/javascript"
	src="webjars/zone.js/dist/jasmine-patch.min.js"></script>
<script type="text/javascript" src="webjars/zone.js/dist/async-test.js"></script>
<script type="text/javascript"
	src="webjars/zone.js/dist/fake-async-test.js"></script>
<script type="text/javascript"
	src="webjars/angular__core/bundles/core-testing.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__common/bundles/common-testing.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__compiler/bundles/compiler-testing.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__platform-browser/bundles/platform-browser-testing.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__platform-browser-dynamic/bundles/platform-browser-dynamic-testing.umd.js"></script>
<script type="text/javascript"
	src="webjars/angular__http/bundles/http-testing.umd.js"></script>
<script type="text/javascript" src="js/spec.js"></script>

</head>

<body>
</body>
</html>

```

HTML内容不受约束的，但它加载了一些Javascript，并且一旦脚本全部运行，它就会有一个UI。

首先我们从中加载所需的Jasmine组件`/webjars/**`。我们加载的文件只是样板文件 - 您可以为任何应用程序做同样的事情。为了使这些在测试中的运行时可用，我们需要将Jasmine依赖添加到我们的“pom.xml”中：

```Xml
<dependency>
  <groupId>org.webjars</groupId>
  <artifactId>jasmine</artifactId>
  <version>2.0.0</version>
  <scope>test</scope>
</dependency>

```

然后我们来到应用程序特定的代码。我们前端的主要源代码是“hello.js”，所以我们必须像在“index.html”中一样加载它，还有它的依赖关系。

最后，我们需要我们刚刚编写的“spec.js”及其依赖项（任何尚未包含在其他脚本中的依赖项），对于Angular应用程序来说，它几乎总是包含与每个主要mondules。我们从webjar加载它们，就像主要的依赖关系一样。

### 运行 Specs

要运行我们的“test.html”代码，我们需要一个小应用程序（例如在“src/test/java/test”中）：

```java
@SpringBootApplication
@Controller
public class TestApplication {

	@RequestMapping("/")
	public String home() {
		return "forward:/test.html";
	}

	public static void main(String[] args) {
		new SpringApplicationBuilder(TestApplication.class).properties(
				"server.port=9999", "security.basic.enabled=false").run(args);
	}

}

```

这`TestApplication`是纯粹的样板：所有应用程序都可以以相同的方式运行测试。您可以在IDE中运行它并访问[http://localhost:9999](http://localhost:9999/)以查看运行的Javascript。在一个`@RequestMapping`我们提供的只是让主页上显示出HTML测试。所有（一个）测试应该是绿色的。

您的开发人员工作流程将从此处对Javascript代码进行更改，并在您的浏览器中重新加载测试应用程序以运行测试。很简单！

### 改进单元测试：模拟HTTP后端

为了提高规格到生产等级，我们需要实际上声明控制器加载时发生的情况。由于它调用了`$http.get()`我们需要模拟这个调用，以避免为了一个单元测试而运行整个应用程序。为此，我们使用Angular `$httpBackend`（在“spec.js”中）：

```javascript
var TestBed = ng.core.testing.TestBed;
var inject = ng.core.testing.inject;
var ResponseOptions = ng.http.ResponseOptions;
var Response = ng.http.Response;

ng.core.testing.getTestBed().initTestEnvironment(
    ng.platformBrowserDynamic.testing.BrowserDynamicTestingModule,
    ng.platformBrowserDynamic.testing.platformBrowserDynamicTesting()
  );

describe("AppComponent", function() {

  beforeEach(function() {
    TestBed.configureTestingModule({
      imports: [ng.http.HttpModule],
      declarations: [
        AppComponent
      ],
      providers: [
        { provide: ng.http.XHRBackend, useClass: ng.http.testing.MockBackend }
      ]
    }).compileComponents();
  });

  it('should create the app', function() {
    const fixture = TestBed.createComponent(AppComponent);
    const app = fixture.debugElement.componentInstance;
    expect(app).toBeTruthy();
  });

  it('should download greeting', inject([ng.http.XHRBackend], function(backend) {
    backend.connections.subscribe(function(connection) {
      connection.mockRespond(new Response(new ResponseOptions({
        body: JSON.stringify({id:'ABC',content:'Hello World'})
      })));
    });
    const fixture = TestBed.createComponent(AppComponent);
    const app = fixture.debugElement.componentInstance;
    expect(app.greeting.content).toEqual('Hello World');
  }));

});

```

这里的新功能是：

- `XHRBackend`在a中的声明`beforeEach()`。
- 在测试函数中，我们在创建组件之前设置了后端的期望，告诉它期望调用“resource/”，以及响应应该是什么。

无需启动和停止测试应用程序，此测试现在应该在浏览器中呈绿色。

### 在命令行上 Specs

能够在浏览器中运行规格是非常好的，因为现代浏览器中内置了优秀的开发工具（例如Chrome中的F12）。您可以设置断点并检查变量，以及能够刷新视图以在活动服务器中重新运行测试。但是这不会帮助你持续集成：为此你需要一种从命令行运行测试的方法。无论您喜欢使用哪种构建工具，都可以使用这些工具，但由于我们在这里使用的是Maven，我们将在“pom.xml”中添加一个插件：

```xml
<plugin>
  <groupId>com.github.searls</groupId>
  <artifactId>jasmine-maven-plugin</artifactId>
  <version>2.2</version>
  <executions>
    <execution>
      <goals>
        <goal>test</goal>
      </goals>
    </execution>
  </executions>
</plugin>

```

这个插件的默认设置将不适用于我们已经制作的静态资源布局，所以我们需要一些配置：

```Xml
<plugin>
  ...
  <configuration>
    <browserVersion>CHROME</browserVersion>
      <preloadSources>
          <source>/webjars/core-js/client/shim.min.js</source>
          <source>/webjars/rxjs/bundles/Rx.min.js</source>
          <source>/webjars/zone.js/dist/zone.min.js</source>
          <source>/webjars/reflect-metadata/${reflect.version}/Reflect.js</source>
          <source>/webjars/angular__core/bundles/core.umd.js</source>
          <source>/webjars/angular__common/bundles/common.umd.js</source>
          <source>/webjars/angular__compiler/bundles/compiler.umd.js</source>
          <source>/webjars/angular__platform-browser/bundles/platform-browser.umd.js</source>
          <source>/webjars/angular__platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js</source>
          <source>/webjars/angular__http/bundles/http.umd.js</source>
          <source>/webjars/zone.js/dist/long-stack-trace-zone.min.js</source>
          <source>/webjars/zone.js/dist/proxy.min.js</source>
          <source>/webjars/zone.js/dist/sync-test.js</source>
          <source>/webjars/zone.js/dist/jasmine-patch.min.js</source>
          <source>/webjars/zone.js/dist/async-test.js</source>
          <source>/webjars/zone.js/dist/fake-async-test.js</source>
          <source>/webjars/angular__core/bundles/core-testing.umd.js</source>
          <source>/webjars/angular__common/bundles/common-testing.umd.js</source>
          <source>/webjars/angular__compiler/bundles/compiler-testing.umd.js</source>
          <source>/webjars/angular__platform-browser/bundles/platform-browser-testing.umd.js</source>
          <source>/webjars/angular__platform-browser-dynamic/bundles/platform-browser-dynamic-testing.umd.js</source>
          <source>/webjars/angular__http/bundles/http-testing.umd.js</source>
      </preloadSources>
      <jsSrcDir>${project.basedir}/src/main/resources/static/js</jsSrcDir>
      <jsTestSrcDir>${project.basedir}/src/test/resources/static/js</jsTestSrcDir>
      <phantomjs>
          <version>2.1.1</version>
      </phantomjs>
  </configuration>
</plugin>

```

默认的Selenium网络驱动程序是`phantomjs`，如果您在受支持的平台上，将会为您下载，否则它需要`PATH`在运行时。例如，这可以在[Travis CI](https://travis-ci.org/)中使用。原则上，这里可以使用任何驱动程序，但PhantomJS可能是最适合用于Angular应用程序的驱动程序。

而已。所有样板再次（如果你想分享多个项目之间的代码，它可以进入一个父POM）。只需在命令行上运行它：

```shell
$ mvn jasmine:test

```

测试也作为Maven“测试”生命周期的一部分运行，所以你可以`mvn test`运行所有的Java测试以及Javascript测试，很顺利地进入你现有的构建和部署周期。这里是日志：

```Shell
$ mvn test
...
[INFO]
-------------------------------------------------------
 J A S M I N E   S P E C S
-------------------------------------------------------
[INFO]
App
  says Hello Test when controller loads

Results: 1 specs, 0 failures

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 21.064s
[INFO] Finished at: Sun Apr 26 14:46:14 BST 2015
[INFO] Final Memory: 47M/385M
[INFO] ------------------------------------------------------------------------

```

Jasmine Maven插件还带有一个`mvn jasmine:bdd`运行服务器的目标，您可以在浏览器中加载这个服务器来运行测试（作为上述选择`TestApplication`）。

### 结论

能够运行Javascript的单元测试在现代Web应用程序中是非常重要的，这是我们在本系列中忽略（或闪避）的一个主题。本文将介绍如何编写测试的基本组成部分，如何在开发时运行测试，以及如何在持续集成环境中运行测试。我们所采取的方法并不适合每个人，所以请不要以不同的方式做这件事，但要确保你有所有这些成分。我们在这里做的方式可能会让传统的Java企业开发人员感到舒适，并且与他们现有的工具和流程很好地集成在一起，所以如果你在这个范畴，我希望你会觉得它是有用的起点。在Angular和Jasmine中测试的更多例子可以在互联网上的很多地方找到，这个系列中的[“单一”样本](https://github.com/dsyer/spring-security-angular/tree/master/single)，现在有一些最新的测试代码，这些代码比我们在本教程中为“基本”样本编写的代码略少一些。

## 从OAuth2客户端应用程序注销

在这一节中我们将继续[我们的讨论](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_testing_angular_js_and_spring_security_part_viii)如何使用[Spring Security的](https://projects.spring.io/spring-security)有[角](http://angular.io/)在“单页面应用程序”中。在这里，我们展示了如何获取OAuth2示例并添加不同的注销体验。许多实施OAuth2单点登录的人发现，他们有一个难题来解决如何“干净地”注销？这个难题的原因在于没有一个正确的方法可以做到这一点，您选择的解决方案将取决于您正在寻找的用户体验以及您愿意承担的复杂性。造成这种复杂性的原因是因为系统中可能存在多个浏览器会话，所有这些会话都有不同的后端服务器，所以当用户从其中一个退出时，其他人应该发生什么？这是教程的第九部分，你可以追踪应用程序的基本构建块，或者通过阅读[第一部分](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_spring_and_angular_js_a_secure_single_page_application)，或者你可以直接去[Github](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2-logout)的[源代码](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2-logout)。

### 注销模式

`oauth2`在本教程中，注销示例的用户体验是您注销了UI应用程序，而不是从authserver注销，因此，当您重新登录UI应用程序时，authenticserver不会再次询问凭据。当authenticserver是外部的时候，这是完全可以预料的，正常的和可取的 - Google和其他外部authserver提供者既不希望也不允许你从不可信任的应用程序中注销他们的服务器，但是如果authserver是真的与UI相同的系统的一部分。

一般来说，有三种模式可以从UI应用程序中注销，并通过OAuth2客户端身份验证：

1. 外部Authserver（EA，原始示例）。用户将authserver视为第三方（例如使用Facebook或Google进行身份验证）。当应用程序会话结束时，您不想注销authserver。您确实希望获得所有赠款的批准。本教程的`oauth2`（和`oauth2-vanilla`）示例实现了这种模式。
2. 网关和内部Authserver（GIA）。您只需要注销2个应用程序，并且它们是用户所感知的同一个系统的一部分。通常你想自动批准所有的赠款。
3. 单一注销（SL）。一个authserver和多个UI应用程序都有自己的身份验证，当用户注销一个时，你希望他们都效仿。由于网络分区和服务器故障，很可能会失败，因此您基本上需要全局一致的存储。

有时候，即使你有一个外部的authserver，你想要控制认证并添加一个内部的访问控制层（例如authserver不支持的范围或角色）。那么使用EA进行身份验证是一个好主意，但是有一个内部authserver，可以将所需的附加细节添加到令牌中。在`auth-server`此另一个样本[的OAuth2教程](https://github.com/spring-guides/tut-spring-boot-oauth2)告诉您如何做，在一个非常简单的方法。然后，您可以将GIA或SL模式应用于包含内部authserver的系统。

这里有一些选项，如果你不想要EA：

- 从authserver以及浏览器客户端中的UI应用程序注销。简单的方法，并与一些小心的CRSF和CORS配置。没有SL。
- 一旦令牌可用，从authserver注销。很难在获取令牌的UI中实现，因为在那里没有authserver的会话cookie。[Spring OAuth中](https://github.com/spring-projects/spring-security-oauth/issues/140)有一个[功能请求，](https://github.com/spring-projects/spring-security-oauth/issues/140)它显示了一个有趣的方法：一旦生成授权码，authserver中的会话就会失效。Github问题包含一个实现会话失效的方面，但是它更容易做到`HandlerInterceptor`。没有SL。
- 代理服务器通过与UI相同的网关，希望一个cookie足以管理整个系统的状态。不起作用，因为除非共享会话在某种程度上破坏对象（否则authserver没有会话存储）。SL只有当会话在所有应用程序之间共享。
- 网关中的Cookie中继。您正在使用网关作为身份验证的真实来源，并且authserver具有所需的所有状态，因为网关管理Cookie而不是浏览器。浏览器永远不会有来自多个服务器的cookie。没有SL。
- 将该令牌用作全局身份验证，并在用户注销UI应用程序时使其无效。下行：需要令牌被客户端应用无效，这实际上并不是他们设计的目的。SL可能，但通常的限制适用。
- 创建和管理authserver中的全局会话令牌（除了用户令牌）。这是[OpenId Connect](https://openid.net/)所采用的方法，它确实为SL提供了一些选择，但需要一些额外的机器。没有一个选项不受通常分布式系统限制的影响：如果网络和应用程序节点不稳定，则不能保证在需要时在所有参与者之间共享注销信号。所有的注销规格仍然是草案形式，这里有一些指标的链接：[会话管理](https://openid.net/specs/openid-connect-session-1_0.html)，[前通道注销](https://openid.net/specs/openid-connect-frontchannel-1_0.html)和[后退通道注销](https://openid.net/specs/openid-connect-backchannel-1_0.html)。

请注意，在SL很难或不可能的情况下，将所有UI放在单个网关后面可能会更好。然后，您可以使用GIA（更简单）来控制整个地区的注销。

最简单的两个选项，在GIA模式中很好地应用，可以在教程示例中实现，如下所示（从中`oauth2`取样和工作）。

### 从浏览器注销这两个服务器

一旦UI应用程序被注销，向浏览器客户端添加几行代码就很容易从authserver注销。例如

```
this.logout = function() {
    http.post('logout', {}).finally(function() {
        self.authenticated = false;
        http.post('http://localhost:9999/uaa/logout', {}, {withCredentials:true}).subscribe(function() {
            console.log('Logged out');
        });
    }).subscribe();
};

```

在这个示例中，我们将authserver注销端点URL硬编码到JavaScript中，但是如果需要，可以很容易地将其外部化。它必须是一个POST直接到authserver，因为我们也希望会话cookie也一样。如果我们特别要求的话，XHR请求只会从浏览器中附带一个cookie `withCredentials:true`。

相反，在服务器上，我们需要一些CORS配置，因为请求来自不同的域。例如在`WebSecurityConfigurerAdapter`

```Java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .cors().configurationSource(configurationSource())
    ...
}

private CorsConfigurationSource configurationSource() {
  UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
  CorsConfiguration config = new CorsConfiguration();
  config.addAllowedOrigin("*");
  config.setAllowCredentials(true);
  config.addAllowedHeader("X-Requested-With");
  config.addAllowedHeader("Content-Type");
  config.addAllowedMethod(HttpMethod.POST);
  source.registerCorsConfiguration("/logout", config);
  return source;
}

```

“/注销”端点已经得到了一些特殊的处理。它可以从任何来源被调用，并且明确地允许发送证书（例如，cookie）。允许的头文件只是Angular在示例应用程序中发送的头文件。

除CORS配置外，我们还需要为注销端点禁用CSRF，因为Angular不会`X-XSRF-TOKEN`在跨域请求中发送标头。authserver在此之前不需要任何CSRF配置，但为注销端点添加忽略很容易：

```
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .csrf()
      .ignoringAntMatchers("/logout/**")
    ...
}

```

> 放弃CSRF保护并不是真的可取，但是你可能会准备好忍受这个限制用例。 

通过这两个简单的更改，一个在UI应用程序客户端，另一个在authserver中，您将发现一旦注销了UI应用程序，当您重新登录时，将始终提示您输入密码。

另一个有用的更改是将OAuth2客户端设置为自动批准，以便用户不必批准令牌授权。这在内部authserver中很常见，用户不会将其视为单独的系统。在`AuthorizationServerConfigurerAdapter`客户端初始化时，你只需要一个标志：

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
  clients.inMemory().withClient("acme")
    ...
  .autoApprove(true);
}

```

### 在Authserver中使会话失效

如果您不想放弃注销端点上的CSRF保护，则可以尝试另一种简单的方法，即一旦授予令牌即可在授权服务器中使用户会话无效（实际上只要授权码被生成）。这也是非常容易实现的：从`oauth2`样本开始，只需将其添加`HandlerInterceptor`到OAuth2端点即可。

```java
@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints)
    throws Exception {
  ...
  endpoints.addInterceptor(new HandlerInterceptorAdapter() {
    @Override
    public void postHandle(HttpServletRequest request,
        HttpServletResponse response, Object handler,
        ModelAndView modelAndView) throws Exception {
      if (modelAndView != null
          && modelAndView.getView() instanceof RedirectView) {
        RedirectView redirect = (RedirectView) modelAndView.getView();
        String url = redirect.getUrl();
        if (url.contains("code=") || url.contains("error=")) {
          HttpSession session = request.getSession(false);
          if (session != null) {
            session.invalidate();
          }
        }
      }
    }
  });
}

```

这个拦截器寻找一个`RedirectView`，这是一个信号，用户被重定向回到客户端应用程序，并检查该位置是否包含验证码或错误。如果您使用隐式授权，则可以添加“token =”。

有了这个简单的改变，只要你认证，authserver中的会话已经死了，所以没有必要从客户端尝试和管理它。当您退出UI应用程序，然后重新登录时，authserver将无法识别您并提示输入凭据。这个模式是由本教程`oauth2-logout`的[源代码中](https://github.com/spring-guides/tut-spring-security-and-angular-js/tree/master/oauth2-logout)的示例实现的模式。这种方法的缺点是，你不再真正的单一登录 - 任何其他的应用程序是你的系统的一部分会发现authserver会话已经死了，他们必须再次提示认证 - 这不是一个很好的用户体验，如果有多个应用程序。

### 结论

在本节中，我们已经看到了如何实现从OAuth2客户端应用程序注销的几种不同模式（以[第五部分中](https://spring.io/guides/tutorials/spring-security-and-angular-js/#_sso_with_oauth2_angular_js_and_spring_security_part_v)的应用程序的教程），并讨论了其他模式的一些选项。这些选项并不是详尽无遗的，但应该给你一个关于所涉及的权衡的好主意，以及一些思考你的用例的最佳解决方案的工具。本节中只有几行JavaScript代码，而Angular并不是特定的（它为XHR请求添加了一个标志），所以所有的课程和模式都可以应用到本指南示例应用程序的范围之外。一个反复出现的主题是，有多个UI应用程序和单个authserver的单一注销（SL）的所有方法往往存在一些缺陷：最好的办法是选择让用户最不舒服的方法。如果您有一个内部authserver和一个由许多组件组成的系统，

想写一个新的指南或贡献一个现有的？查看我们的[贡献准则](https://github.com/spring-guides/getting-started-guides/wiki)。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



