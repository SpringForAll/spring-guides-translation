# React.js和Spring Data REST

本教程展示了一组使用Spring Data REST的应用程序，以及其强大的后端功能与React的复杂功能相结合，构建了一个易于使用的UI。

- [Spring Data REST](https://www.youtube.com/watch?v=TgCr7v9tdKM)提供了一种构建超媒体驱动的存储库的快速方法。
- [React](https://facebook.github.io/react/index.html)是Facebook在JavaScript领域高效，快速和易于使用的解决方案的解决方案。

## 第1部分 - 基本功能

欢迎Spring社区，

在本节中，您将看到如何快速启动并运行一个简单的Spring Data REST应用程序。然后，您将使用Facebook的React.js工具集在其上构建一个简单的UI。

### 第0步 - 设置您的环境

随意从这个仓库中[获取代码](https://github.com/spring-guides/tut-react-and-spring-data-rest/tree/master/basic)，并遵循。

如果你想自己做，请访问[http://start.spring.io](https://start.spring.io/)并选择这些项目：

- Rest Repositories
- Thymeleaf
- JPA
- H2
- Lombok（可能想确保你的IDE也支持这个）

本演示使用Java 8，Maven Project和Spring Boot的最新稳定版本。它也使用[ES6](http://es6-features.org/)编码的React.js 。这将给你一个干净，空的项目。从那里，您可以添加本节中明确显示的各种文件，和/或从上面列出的存储库借用。

### 一开始...

一开始有数据。这很好。但是人们希望通过各种方式访问数据。多年来，人们一起拼凑了大量的MVC控制器，许多人使用Spring的强大的REST支持。但一遍又一遍地花费很多时间。

Spring Data REST 指出，如果做出一些假设这个问题是多么简单：

- 开发人员使用支持存储库模型的Spring Data项目。
- 该系统使用公认的行业标准协议，如HTTP动词，标准化媒体类型和IANA认可的链接名称。

#### 声明你的域名

任何基于Spring Data REST的应用程序的基石都是域对象。对于本节，您将构建一个应用程序来跟踪公司的员工。通过创建一个像这样的数据类型来开始：

src/main/java/com/greglturnquist/payroll/Employee.java

```Java
@Data
@Entity
public class Employee {

	private @Id @GeneratedValue Long id;
	private String firstName;
	private String lastName;
	private String description;

	private Employee() {}

	public Employee(String firstName, String lastName, String description) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.description = description;
	}
}
```

- `@Entity` 是一个JPA注释，它表示整个类在关系表中的存储。
- `@Id`并且`@GeneratedValue`是JPA注释来记录主键，并在需要时自动生成。
- `@Data`是一个Project Lombok注释，用于自动生成getter，setter，构造函数，toString，hash，equals和其他东西。它减少了样板代码的书写。

该实体用于记录员工信息。例如，他们的名字和工作描述。

| **   | Spring Data REST不局限于JPA。它支持许多NoSQL数据存储，但是你不会在这里覆盖这些数据存储。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 定义存储库

Spring Data REST应用程序的另一个关键部分是创建相应的存储库定义。

src/main/java/com/greglturnquist/payroll/EmployeeRepository.java

```Java
public interface EmployeeRepository extends CrudRepository<Employee, Long> {

}
```

- EmployeeRepository继承了Spring Data Commons的`CrudRepository`并且插入了域对象的类型及其主键

这就是所需要做的全部！事实上，如果它是顶级并且是可见的，你甚至不需要对它进行注解。如果你使用IDE并打开`CrudRepository`，你会发现一系列已经定义好的并且预先构建过的方法。

| **   | 如果你愿意，你可以定义[你自己的仓库](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.definition)。Spring Data REST也支持这一点。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 预加载示例

要使用这个应用程序，你需要预先加载一些像这样的数据：

src/main/java/com/greglturnquist/payroll/DatabaseLoader.java

```java
@Component
public class DatabaseLoader implements CommandLineRunner {

	private final EmployeeRepository repository;

	@Autowired
	public DatabaseLoader(EmployeeRepository repository) {
		this.repository = repository;
	}

	@Override
	public void run(String... strings) throws Exception {
		this.repository.save(new Employee("Frodo", "Baggins", "ring bearer"));
	}
}
```

- 这个类是用Spring的`@Component`注解标记的，所以它被`@SpringBootApplication`自动拾取。
- 它实现了Spring Boot的`CommandLineRunner`，以便在所有的Bean被创建和注册后运行。
- 它使用构造函数注入和自动装配来获取Spring Data自动创建的`EmployeeRepository`。
- `run()`方法通过命令行参数来调用，从而加载你的数据。

Spring Data最大最强大的功能之一就是它能够为你编写JPA查询。这不仅减少了开发时间，而且减少了错误和错误的风险。Spring Data [查看](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.details)存储库类[中方法的名称，](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.details)并计算出所需的操作，包括保存，删除和查找。

这就是我们如何编写一个空的接口并继承已经建立的保存，查找和删除操作。

### 调整根URI

默认情况下，Spring Data REST托管链接的根集合`/`。由于您将在同一路径上托管Web UI，因此您需要更改根URI。

src/main/resources/application.properties

```properties
spring.data.rest.base-path=/api
```

### 启动后端

为了获得完全可操作的REST API，最后一步是编写一个使用Spring Boot的main方法`public static void main`：

src/main/java/com/greglturnquist/payroll/ReactAndSpringDataRestApplication.java

```Java
@SpringBootApplication
public class ReactAndSpringDataRestApplication {

	public static void main(String[] args) {
		SpringApplication.run(ReactAndSpringDataRestApplication.class, args);
	}
}
```

假设先前的类以及Maven构建文件是从[http://start.spring.io](https://start.spring.io/)生成的，那么现在可以通过`main()`在IDE中运行该方法来启动它，或者在命令行上键入`./mvnw spring-boot:run`。（Windows用户使用mvnw.bat）。

| **   | 如果您对Spring Boot没有及时更新，以及它是如何工作的，您应该考虑观看[Josh Long的介绍性演讲](https://www.youtube.com/watch?v=sbPSjI4tt10)。做到了？按下！ |
| ---- | ---------------------------------------- |
|      |                                          |

### 参观您的REST服务

随着应用程序的运行，你可以使用[cURL](https://curl.haxx.se/)（或任何其他你喜欢的工具）在命令行上把事情搞定。

```javascript
$ curl localhost：8080 / api
{
  “_links”：{
    “雇员” ： {
      “href”：“http：// localhost：8080 / api / employees”
    },
    “个人资料”：{
      “href”：“http：// localhost：8080 / api / profile”
    }
  }
}
```

当您ping根节点时，您将获取包含在[HAL格式的JSON文档中](http://stateless.co/hal_specification.html)的链接集合。

- **_links**是可用链接的集合。
- **employees**指向由`EmployeeRepository`接口定义的员工对象的根路径。
- **profile**是IANA标准关系，并指向有关整个服务的可发现元数据。我们将在后面的章节中进行探讨。

您可以通过浏览**employees**链接进一步挖掘此服务。

```js
$ curl localhost：8080 / api / employees
{
  “_embedded”：{
    “雇员” ： [ {
      “firstName”：“Frodo”，
      “lastName”：“Baggins”，
      “描述”：“持戒者”，
      “_links”：{
        “自我”：{
          “href”：“http：// localhost：8080 / api / employees / 1”
        }
      }
    } ]
  }
}
```

在这个阶段，你正在查看整个员工的集合。

与之前预先加载的数据一起包含的是带有**self**链接的**_links**属性。这是特定员工的规范链接。什么是规范？这意味着没有上下文。例如，同一个用户可以通过像/ api / orders / 1 / processor这样的链接获取，其中员工与处理特定顺序相关联。这里和其他实体没有任何关系。****

| **   | 链接是REST的一个关键方面。他们提供导航到相关项目的能力。这使得其他各方可以浏览您的API，而无需每次改变时重写。当客户端硬编码资源路径时，客户端更新是一个常见问题。重组资源可能会在代码中造成巨大的动荡。如果使用链接而不是导航路线，那么做出这样的调整变得容易和灵活。 |
| ---- | ---------------------------------------- |
|      |                                          |

如果您愿意，您可以决定查看该员工。

```js
$ curl localhost:8080/api/employees/1
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "description" : "ring bearer",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/api/employees/1"
    }
  }
}
```

除了没有必要使用**_embedded**包装，因为只有域对象，所以这里几乎没有变化。

这一切都很好，但你可能很想创造一些新的条目。

```js
$ curl -X POST localhost:8080/api/employees -d "{\"firstName\": \"Bilbo\", \"lastName\": \"Baggins\", \"description\": \"burglar\"}" -H "Content-Type:application/json"
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "description" : "burglar",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/api/employees/2"
    }
  }
}
```

您也可以按照[本相关指南中](https://spring.io/guides/gs/accessing-data-rest/)所示的PUT，PATCH和DELETE 。但是，我们不要深究这一点。您已经花费太多时间手动与此REST服务进行交互。你不想要建立一个平滑的用户界面吗？

### 设置一个自定义的UI控制器

Spring Boot使得建立自定义网页变得非常简单。首先，你需要一个Spring MVC控制器。

src/main/java/com/greglturnquist/payroll/HomeController.java

```java
@Controller
public class HomeController {

	@RequestMapping(value = "/")
	public String index() {
		return "index";
	}

}
```

- `@Controller` 将这个类标记为Spring MVC控制器。
- `@RequestMapping`标记`index()`以支持`/`路径的方法。
- 它返回`index`来作为模板的名称,此时Spring Boot默认的视图解析器会将其映射到页面`src/main/resources/templates/index.html`。

### 定义一个HTML模板

你正在使用Thymeleaf，虽然你不会真的使用它的许多功能。

src/main/resources/templates/index.html

```Html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8"/>
    <title>ReactJS + Spring Data REST</title>
    <link rel="stylesheet" href="/main.css" />
</head>
<body>

    <div id="react"></div>

    <script src="built/bundle.js"></script>

</body>
</html>
```

这个模板中的关键部分是`<div id="react"></div>`中间的组件。这是你将引导React插入渲染输出的地方。

### 加载JavaScript模块

本教程不会详细介绍如何使用[webpack](https://webpack.github.io/)加载JavaScript模块。但多亏了**frontend-maven-plugin**插件，你不需要安装任何Node.js工具来构建和运行代码。

以下JavaScript模块将被使用：

- webpack
- babel
- react.js
- rest.js

用babel的力量，JavaScript是用ES6编写的。

如果您有兴趣，可以在[webpack.config.js](https://github.com/spring-guides/tut-react-and-spring-data-rest/blob/master/basic/webpack.config.js)中定义JavaScript模块的路径。然后这个被webpack用来生成一个JavaScript包，它被加载到模板中。

| **   | 希望自动查看您的JavaScript更改？运行`npm run-script watch`将webpack放入监视模式。它将在您编辑源代码时重新生成bundle.js。 |
| ---- | ---------------------------------------- |
|      |                                          |

所有这一切，你可以专注于加载DOM后获取的React位。它分为以下几部分：

由于您正在使用webpack来组装东西，请继续并取出您需要的模块：

src/main/js/app.js

```react
const React = require('react');
const ReactDOM = require('react-dom');
const client = require('./client');
```

- `React` 是来自Facebook的构建这个应用程序的主要代码库。
- `client`是配置rest.js以包含对HAL，URI模板和其他事物的支持的自定义代码。它还将默认的**Accept**请求标头设置为**application / hal + json**。你可以[在这里阅读代码](https://github.com/spring-guides/tut-react-and-spring-data-rest/blob/master/basic/src/main/js/client.js)。

### 深入React

React基于定义组件。通常情况下，一个组件可以在父子关系中保存另一个组件的多个实例。这个概念很容易扩展到几个层次。

要开始，所有组件都有一个顶级容器是非常方便的。（当你在本系列的代码中展开时，这将变得更加明显。）现在，你只有员工名单。但是您稍后可能需要一些其他相关的组件，所以我们先从这个开始：

src/main/js/app.js - App component

```react
class App extends React.Component {

	constructor(props) {
		super(props);
		this.state = {employees: []};
	}

	componentDidMount() {
		client({method: 'GET', path: '/api/employees'}).done(response => {
			this.setState({employees: response.entity._embedded.employees});
		});
	}

	render() {
		return (
			<EmployeeList employees={this.state.employees}/>
		)
	}
}
```

- `class Foo extends React.Component{…}` 是创建一个React组件的方法。
- `componentDidMount` 是React在DOM中呈现组件后调用的API。
- `render` 是在屏幕上“绘制”组件的API。

| **   | 在React中，大写字母是命名组件的惯例。 |
| ---- | --------------------- |
|      |                       |

在**App**组件中，从Spring Data REST后端获取一个员工数组并存储在该组件的**状态**数据中。

React组件有两种类型的数据：**状态**和**属性**。

**状态**是组件需要处理的数据。这也是可以波动和变化的数据。要阅读状态，请使用`this.state`。要更新它，请使用`this.setState()`。每次`this.setState()`调用时，React都会更新状态，计算前一状态和新状态之间的差异，并在页面上向DOM注入一组更改。这样可以快速高效地更新您的用户界面。

通用约定是在构造函数中将所有的属性都清空，以初始化状态。然后，您使用服务器查找数据`componentDidMount`并填充您的属性。从那里开始，更新可以由用户操作或其他事件驱动。

**属性**包含传入组件的数据。属性不会改变，而是固定的值。要设置它们，在创建新组件时将它们分配给属性，您很快就会看到。

| **   | JavaScript不像其他语言那样锁定数据结构。您可以尝试通过分配值来颠覆属性，但这不适用于React的差异引擎，应该避免。 |
| ---- | ---------------------------------------- |
|      |                                          |

在这段代码中，函数通过rest.js `client`的[Promise兼容](https://promisesaplus.com/)实例加载数据。当它完成检索时`/api/employees`，它会调用里面的函数，`done()`并根据它的HAL文件（`response.entity._embedded.employees`）设置状态。你可能还记得的结构`curl /api/employees` [较早](https://spring.io/guides/tutorials/react-and-spring-data-rest/#_touring_your_rest_service)，看看它是如何映射到这个结构。

当状态被更新时，该`render()`功能被框架调用。员工状态数据包含在创建`<EmployeeList />`React组件作为输入参数。

下面是一个的定义`EmployeeList`。

src/main/js/app.js - EmployeeList component

```react
class EmployeeList extends React.Component{
	render() {
		var employees = this.props.employees.map(employee =>
			<Employee key={employee._links.self.href} employee={employee}/>
		);
		return (
			<table>
				<tbody>
					<tr>
						<th>First Name</th>
						<th>Last Name</th>
						<th>Description</th>
					</tr>
					{employees}
				</tbody>
			</table>
		)
	}
}
```

使用JavaScript的映射函数，`this.props.employees`从一个员工记录数组转换成一个`<Element />`React组件数组（你将会看到更多的内容）。

```
<Employee key={employee._links.self.href} data={employee} />
```

这显示了一个新的React组件（注意大写格式）以及两个属性：**键**和**数据**。这些是从`employee._links.self.href`和提供的值`employee`。

| **   | 无论何时使用Spring Data REST，**self**链接都是给定资源的关键。React需要一个独特的子节点标识符，并且`_links.self.href`是完美的。 |
| ---- | ---------------------------------------- |
|      |                                          |

最后，你返回一个HTML表格，包裹在`employees`带映射的内建数组中。

```html
<table>
    <tr>
        <th>First Name</th>
        <th>Last Name</th>
        <th>Description</th>
    </tr>
    {employees}
</table>
```

这种简单的状态，属性和HTML布局显示了React如何让你创建一个简单而易于理解的组件。

此代码是否包含HTML *和* JavaScript？是的，这是[JSX](https://facebook.github.io/js/)。没有要求使用它。React可以使用纯JavaScript编写，但JSX语法非常简洁。由于Babel.js的快速工作，转译器同时提供了对JSX和ES6的支持

JSX还包括[ES6的一些](http://es6-features.org/#Constants)零件。代码中使用的是[箭头函数](http://es6-features.org/#ExpressionBodies)。它避免了创建一个嵌套函数（）有自己的作用域**此**，以及避免一个[**self**变量](https://stackoverflow.com/a/962040/28214)。

担心将逻辑与你的结构混在一起？React的API鼓励结合状态和属性的良好声明结构。React不是混合一堆不相关的JavaScript和HTML，而是鼓励用简单的组件来构建一些相关的状态和属性，这些组件可以很好地协同工作。它可以让你看一个单一的组件，并理解设计。然后他们很容易结合在一起的更大的结构。

接下来，你需要真正定义什么是`<Employee />`。

src/main/js/app.js - Employee component

```js
class Employee extends React.Component{
	render() {
		return (
			<tr>
				<td>{this.props.employee.firstName}</td>
				<td>{this.props.employee.lastName}</td>
				<td>{this.props.employee.description}</td>
			</tr>
		)
	}
}
```

这个组件非常简单。它有一个单独的HTML表格行缠绕在员工的三个属性中。财产本身是`this.props.employee`。请注意，如何传入JavaScript对象可以轻松地传递从服务器获取的数据？

由于这个组件不管理任何状态，也不处理用户输入，所以没有别的办法。这可能会诱使你把它塞进`<EmployeeList />`上面。不要这样做！相反，将您的应用程序分解成各自完成一项工作的小型组件，将使未来构建功能变得更加容易。

最后一步是渲染。

src/main/js/app.js - rendering code

```js
ReactDOM.render(
	<App />,
	document.getElementById('react')
)
```

`React.render()`接受两个参数：您定义的React组件以及注入它的DOM节点。还记得你是如何`<div id="react"></div>`从HTML页面看到的？这是它被拿起和插入的地方。

有了这一切，重新运行应用程序（`./mvnw spring-boot:run`）并访问[http：// localhost：8080](http://localhost:8080/)。

![基本1](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/basic/images/basic-1.png)

您可以看到系统加载的初始员工。

记得使用cURL来创建新的条目？再次这样做。

```bash
curl -X POST localhost：8080 / api / employees -d“{\”firstName \“：\”Bilbo \“，\”lastName \“：\”Baggins \“，\”description \“：\”burglar \“ }“-H”Content-Type：application / json“
```

刷新浏览器，你应该看到新的条目：

![基本2](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/basic/images/basic-2.png)

现在你可以看到他们在网站上列出。

### 回顾

在这个部分：

- 您定义了一个域对象和一个相应的存储库。
- 您让Spring Data REST使用全面的超媒体控件导出它。
- 您在父子关系中创建了两个简单的React组件。
- 您获取服务器数据并将其呈现为简单的静态HTML结构。

问题是什么？

- 网页不是动态的。您必须刷新浏览器才能获取新记录。
- 该网页没有使用任何超媒体控件或元数据。相反，它是硬编码来从中获取数据`/api/employees`。
- 它是只读的。虽然您可以使用cURL来更改记录，但是网页并不提供这些信息。

这些是我们在下一节中可以解决的问题。

## 第2部分 - 超媒体控件

在[前面的章节中](https://spring.io/guides/tutorials/react-and-spring-data-rest/#react-and-spring-data-rest-part-1)，您了解了如何使用Spring Data REST来支持后端薪资服务以存储员工数据。它缺乏的一个重要功能是使用超媒体控件和链接导航。相反，它硬编码寻找数据的路径。

随意从这个仓库中[获取代码](https://github.com/spring-guides/tut-react-and-spring-data-rest/tree/master/hypermedia)，并遵循。本部分是基于上一部分的应用程序添加额外的东西。

### 一开始有数据...然后是REST

> 我感到沮丧的人调用任何基于HTTP的接口REST API的人数。今天的例子是SocialSite REST API。这是RPC。它尖叫着RPC ......需要做些什么才能使REST架构风格清晰地表明超文本是一种约束？换句话说，如果应用程序状态（因此API）的引擎没有被超文本驱动，那么它不能是RESTful，并且不能是REST API。期。是否有某些需要修复的手册中断？

\- Roy T. Fielding 
http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven

那么，什么是超媒体控制，即超文本，你如何使用它们？为了找到答案，让我们退一步看看REST的核心任务。

REST的概念是借鉴使网络如此成功的思想，并将它们应用于API。尽管网络庞大，动态性强，客户端（即浏览器）更新率低，网络却是一个了不起的成功。Roy Fielding试图利用它的一些限制和特点，看看这样做是否能够支持API生产和消费的类似扩张。

其中一个约束是限制动词的数量。对于REST，主要的是GET，POST，PUT，DELETE和PATCH。还有其他的，但我们不会在这里进入他们。

- GET - 在不改变系统的情况下获取资源的状态
- POST - 创建一个新的资源，而不用说在哪里
- PUT - 替换现有的资源，覆盖其他已有的资源（如果有的话）
- 删除 - 删除现有的资源
- 修补程序 - 部分修改现有的资源

这些都是标准的HTTP动词，写得很好。通过提取和使用已经创建的HTTP操作，我们不必发明一种新的语言，并教育行业。

REST的另一个约束是使用媒体类型来定义数据的格式。不是每个人都用自己的方言来交流信息，而是开发一些媒体类型是审慎的。其中最受欢迎的是HAL，媒体类型应用程序/ hal + json。这是Spring Data REST的默认媒体类型。值得一提的是REST没有集中的单一媒体类型。相反，人们可以开发媒体类型并插入。尝试一下。随着不同的需求，行业可以灵活移动。

REST的一个重要功能是包含相关资源的链接。例如，如果您正在查看订单，则RESTful API将包含指向相关客户的链接，指向商品目录的链接以及指向订单的商店的链接。在本节中，您将介绍分页，并了解如何使用导航分页链接。

### 打开后端页面

要开始使用前端超媒体控件，您需要打开一些额外的控件。Spring Data REST提供分页支持。要使用它，只需调整存储库定义：

src/main/java/com/greglturnquist/payroll/EmployeeRepository.java

```Java
public interface EmployeeRepository extends PagingAndSortingRepository<Employee, Long> {

}
```

现在`PagingAndSortingRepository`，您的界面已经扩展，增加了额外的选项来设置页面大小，并且还添加了导航链接，以便逐页跳转。后端的其余部分是相同的（一些[额外的预加载数据的](https://github.com/spring-guides/tut-react-and-spring-data-rest/blob/master/hypermedia/src/main/java/com/greglturnquist/payroll/DatabaseLoader.java)例外，使事情变得有趣）。

重新启动应用程序（`./mvnw spring-boot:run`）并查看它是如何工作的。

```bash
$ curl“localhost：8080 / api / employees？size = 2”
{
  “_links”：{
    “第一”：{
      “href”：“http：// localhost：8080 / api / employees？page = 0＆size = 2”
    },
    “自我”：{
      “href”：“http：// localhost：8080 / api / employees”
    },
    “下一个” ： {
      “href”：“http：// localhost：8080 / api / employees？page = 1＆size = 2”
    },
    “last”：{
      “href”：“http：// localhost：8080 / api / employees？page = 2＆size = 2”
    }
  },
  “_embedded”：{
    “雇员” ： [ {
      “firstName”：“Frodo”，
      “lastName”：“Baggins”，
      “描述”：“持戒者”，
      “_links”：{
        “自我”：{
          “href”：“http：// localhost：8080 / api / employees / 1”
        }
      }
    }, {
      “firstName”：“Bilbo”，
      “lastName”：“Baggins”，
      “描述”：“窃贼”，
      “_links”：{
        “自我”：{
          “href”：“http：// localhost：8080 / api / employees / 2”
        }
      }
    } ]
  },
  “page”：{
    “大小”：2，
    “totalElements”：6，
    “totalPages”：3，
    “数字”：0
  }
}
```

默认页面大小是20，所以看到它的行动，`?size=2`应用。正如预期的那样，只有两名员工被列出。另外，还有**第一个**，**第二个**和**最后一个**链接。还有**自我**链接，没有上下文，*包括页面参数*。

如果你导航到**下一个**链接，你会看到一个**prev**链接：

```
$ curl“http：// localhost：8080 / api / employees？page = 1＆size = 2”
{
  “_links”：{
    “第一”：{
      “href”：“http：// localhost：8080 / api / employees？page = 0＆size = 2”
    },
    “prev”：{
      “href”：“http：// localhost：8080 / api / employees？page = 0＆size = 2”
    },
    “自我”：{
      “href”：“http：// localhost：8080 / api / employees”
    },
    “下一个” ： {
      “href”：“http：// localhost：8080 / api / employees？page = 2＆size = 2”
    },
    “last”：{
      “href”：“http：// localhost：8080 / api / employees？page = 2＆size = 2”
    }
  },
...
```

| **   | 在URL查询参数中使用“＆”时，命令行认为是换行符。将整个URL用引号括起来。 |
| ---- | --------------------------------------- |
|      |                                         |

这看起来很整洁，但是当你更新前端以利用它的时候会更好。

### 通过关系导航

就是它！Spring Data REST提供了开箱即用的功能，不需要对后端进行任何更改。你可以切换到前端工作。（这是Spring Data REST美丽的一部分，没有凌乱的控制器更新！）

| **   | 需要指出的是，这个应用程序不是“Spring Data REST特有的”。而是使用[HAL](http://stateless.co/hal_specification.html)，[URI模板](https://tools.ietf.org/html/rfc6570)和其他标准。这就是如何使用rest.js是一个单元：该库带有HAL支持。 |
| ---- | ---------------------------------------- |
|      |                                          |

在前面的章节中，您将路径硬编码为`/api/employees`。相反，你应该硬编码的唯一路径是根。

```js
...
var root = '/api';
...
```

用一个方便的小[`follow()`功能](https://github.com/spring-guides/tut-react-and-spring-data-rest/blob/master/hypermedia/src/main/js/follow.js)，你现在可以从根开始，并导航到你需要的地方！

```js
componentDidMount() {
	this.loadFromServer(this.state.pageSize);
}
```

在上一节中，加载是直接在内部完成的`componentDidMount()`。在本节中，我们可以在更新页面大小时重新加载整个员工列表。要做到这一点，我们已经把事情转移到了`loadFromServer()`。

```js
loadFromServer(pageSize) {
	follow(client, root, [
		{rel: 'employees', params: {size: pageSize}}]
	).then(employeeCollection => {
		return client({
			method: 'GET',
			path: employeeCollection.entity._links.profile.href,
			headers: {'Accept': 'application/schema+json'}
		}).then(schema => {
			this.schema = schema.entity;
			return employeeCollection;
		});
	}).done(employeeCollection => {
		this.setState({
			employees: employeeCollection.entity._embedded.employees,
			attributes: Object.keys(this.schema.properties),
			pageSize: pageSize,
			links: employeeCollection.entity._links});
	});
}
```

`loadFromServer`与上一节非常相似，但是如果使用`follow()`：

- follow（）函数的第一个参数是`client`用于进行REST调用的对象。
- 第二个参数是从头开始的根URI。
- 第三个参数是一组要导航的关系。每一个都可以是一个字符串或一个对象。

关系数组可以很简单`["employees"]`，也就是说，在第一次调用时，查看名为**employees**的关系（或**rel**）的**_links**。找到它的**href**并导航到它。如果阵列中存在另一种关系，则冲洗并重复。************

有时候，一个rel本身是不够的。在这个代码片段，它也塞在查询参数**？大小= <pageSize的>**。还有其他的选项可以提供，你会看到更多的。

### 抓取JSON模式元数据

在用基于大小的查询导航到**员工**之后，**employeeCollection**就在您的指尖。在上一节中，我们称它为“日”，并显示`<EmployeeList />`中的数据。现在，您正在执行另一个调用，以便在`/api/profile/employees/`找到的一些[JSON模式元数据](http://json-schema.org/)。

您可以自己查看数据：

```Bash
$ curl http：// localhost：8080 / api / profile / employees -H“Accept：application / schema + json”
{
  “标题”：“员工”，
  “属性”：{
    “名字” ： {
      “标题”：“名”，
      “readOnly”：假，
      “type”：“string”
    },
    “姓” ： {
      “标题”：“姓氏”，
      “readOnly”：假，
      “type”：“string”
    },
    “description”：{
      “标题描述”，
      “readOnly”：假，
      “type”：“string”
    }
  },
  “定义”：{}，
  “type”：“object”，
  “$ schema”：“http://json-schema.org/draft-04/schema#”
}
```

| **   | / profile / employees中的默认元数据形式是ALPS。但是，在这种情况下，您正在使用内容协商来获取JSON模式。 |
| ---- | ---------------------------------------- |
|      |                                          |

通过在“<App />”组件的状态中捕获这些信息，在构建输入表单时，可以稍后使用它。

### 创建新记录

配备这个元数据，你现在可以添加一些额外的控制到用户界面。创建一个新的React组件`<CreateDialog />`。

```react
class CreateDialog extends React.Component {

	constructor(props) {
		super(props);
		this.handleSubmit = this.handleSubmit.bind(this);
	}

	handleSubmit(e) {
		e.preventDefault();
		var newEmployee = {};
		this.props.attributes.forEach(attribute => {
			newEmployee[attribute] = ReactDOM.findDOMNode(this.refs[attribute]).value.trim();
		});
		this.props.onCreate(newEmployee);

		// clear out the dialog's inputs
		this.props.attributes.forEach(attribute => {
			ReactDOM.findDOMNode(this.refs[attribute]).value = '';
		});

		// Navigate away from the dialog to hide it.
		window.location = "#";
	}

	render() {
		var inputs = this.props.attributes.map(attribute =>
			<p key={attribute}>
				<input type="text" placeholder={attribute} ref={attribute} className="field" />
			</p>
		);

		return (
			<div>
				<a href="#createEmployee">Create</a>

				<div id="createEmployee" className="modalDialog">
					<div>
						<a href="#" title="Close" className="close">X</a>

						<h2>Create new employee</h2>

						<form>
							{inputs}
							<button onClick={this.handleSubmit}>Create</button>
						</form>
					</div>
				</div>
			</div>
		)
	}

}
```

这个新的组件既有`handleSubmit()`功能也有预期的`render()`功能。

让我们以相反的顺序挖掘这些功能，并首先看看这个`render()`功能。

#### 渲染

您的代码映射到**attributes**属性中的JSON模式数据，并将其转换为`<p><input></p>`元素数组。

- React再次需要**key**来区分多个子节点。
- 这是一个简单的基于文本的输入字段。
- **placeholder**是我们可以显示用户的字段是哪个。
- 您可能习惯于拥有一个**name**属性，但这不是必需的。使用React，**ref**是获取特定DOM节点的机制（就像你将会看到的那样）。

这表示组件的动态特性，由从服务器加载数据驱动。

在这个组件的顶层`<div>`是一个锚标签和另一个`<div>`。锚标记是打开对话框的按钮。嵌套`<div>`是隐藏的对话框本身。在这个例子中，你使用纯HTML5和CSS3。根本没有JavaScript！您可以[看到](https://github.com/spring-guides/tut-react-and-spring-data-rest/blob/master/hypermedia/src/main/resources/static/main.css)用于显示/隐藏对话框[的CSS代码](https://github.com/spring-guides/tut-react-and-spring-data-rest/blob/master/hypermedia/src/main/resources/static/main.css)。我们不会在这里深究。

位于里面的`<div id="createEmployee">`是一个窗体，在窗体中输入动态的输入字段列表，之后是**创建**按钮。该按钮有一个`onClick={this.handleSubmit}`事件处理程序。这是注册事件处理程序的React方法。

| **   | React不会在每个DOM元素上创建一堆事件处理程序。相反，它有一个[更高性能和复杂的](https://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html#under-the-hood-autobinding-and-event-delegation)解决方案。重要的是你不必管理这个基础结构，而是可以专注于编写功能代码。 |
| ---- | ---------------------------------------- |
|      |                                          |

#### 处理用户输入

该`handleSubmit()`函数首先阻止该事件进一步向上冒泡。然后，它使用相同的JSON模式属性来查找每个`<input>`使用的属性`React.findDOMNode(this.refs[attribute])`。

`this.refs`是一种通过名称获取特定React组件的方法。从这个意义上说，你只是获得虚拟DOM组件。获取您需要使用的实际DOM元素`React.findDOMNode()`。

遍历每个输入并建立`newEmployee`对象后，我们调用一个回调给`onCreate()`新员工。这个函数在最上面，`App.onCreate`并作为另一个属性提供给这个React组件。看看顶级功能如何运作：

```react
onCreate(newEmployee) {
	follow(client, root, ['employees']).then(employeeCollection => {
		return client({
			method: 'POST',
			path: employeeCollection.entity._links.self.href,
			entity: newEmployee,
			headers: {'Content-Type': 'application/json'}
		})
	}).then(response => {
		return follow(client, root, [
			{rel: 'employees', params: {'size': this.state.pageSize}}]);
	}).done(response => {
		if (typeof response.entity._links.last != "undefined") {
			this.onNavigate(response.entity._links.last.href);
		} else {
			this.onNavigate(response.entity._links.self.href);
		}
	});
}
```

再次使用该`follow()`功能导航到执行POST操作的**员工**资源。在这种情况下，不需要应用任何参数，所以基于字符串的rels数组没有问题。在这种情况下，POST调用返回。这允许下一个`then()`子句处理POST的结果。

新记录通常会添加到数据集的末尾。由于您正在查看某个页面，因此期望新员工记录不在当前页面上是合乎逻辑的。要处理这个问题，您需要使用相同的页面大小获取新的一批数据。这个承诺是在最后的条款里面返回的`done()`。

由于用户可能希望看到新创建的员工，因此可以使用超媒体控件并导航到**最后一个**条目。

这在我们的UI中引入了分页的概念。接下来我们来解决这个问题！

第一次使用基于承诺的API？[Promise](https://promisesaplus.com/)是启动异步操作的一种方式，然后注册一个函数来完成任务。承诺被设计为链接在一起，以避免“回拨地狱”。看下面的流程：

```react
when.promise(async_func_call())
	.then(function(results) {
		/* process the outcome of async_func_call */
	})
	.then(function(more_results) {
		/* process the previous then() return value */
	})
	.done(function(yet_more) {
		/* process the previous then() and wrap things up */
	});
```

有关更多详细信息，请参阅[承诺上的本教程](http://know.cujojs.com/tutorials/promises/consuming-promises)。

需要记住的秘密是`then()`函数*需要*返回一些东西，不管是价值还是其他承诺。`done()`函数不会返回任何东西，而且也不会链接任何东西。如果你还没有注意到`client`（这是`rest`rest.js的一个实例）以及`follow`函数return promise。

### 通过数据进行分页

您在后台设置分页，并在创建新员工时已经开始利用分页功能。

在[上一节中](https://spring.io/guides/tutorials/react-and-spring-data-rest/#creating-new-records)，你所使用的页面控件跳转到**最后**一页。动态地将其应用到UI并让用户按需要导航将非常方便。动态调整控件基于可用的导航链接将是很好的。

首先，我们来看看`onNavigate()`你使用的功能。

```react
onNavigate(navUri) {
	client({method: 'GET', path: navUri}).done(employeeCollection => {
		this.setState({
			employees: employeeCollection.entity._embedded.employees,
			attributes: this.state.attributes,
			pageSize: this.state.pageSize,
			links: employeeCollection.entity._links
		});
	});
}
```

这是在顶部，里面定义的`App.onNavigate`。再次，这是为了允许管理顶层组件中的UI状态。传递`onNavigate()`给`<EmployeeList />`React组件后，以下处理程序将被编码以处理单击某些按钮：

```react
handleNavFirst(e){
	e.preventDefault();
	this.props.onNavigate(this.props.links.first.href);
}

handleNavPrev(e) {
	e.preventDefault();
	this.props.onNavigate(this.props.links.prev.href);
}

handleNavNext(e) {
	e.preventDefault();
	this.props.onNavigate(this.props.links.next.href);
}

handleNavLast(e) {
	e.preventDefault();
	this.props.onNavigate(this.props.links.last.href);
}
```

这些函数中的每一个拦截默认事件，并阻止它冒泡。然后它调用`onNavigate()`适当的超媒体链接的功能。

现在有条件地显示基于哪些链接出现在超媒体链接中的控件`EmployeeList.render`：

```react
render() {
	var employees = this.props.employees.map(employee =>
		<Employee key={employee._links.self.href} employee={employee} onDelete={this.props.onDelete}/>
	);

	var navLinks = [];
	if ("first" in this.props.links) {
		navLinks.push(<button key="first" onClick={this.handleNavFirst}>&lt;&lt;</button>);
	}
	if ("prev" in this.props.links) {
		navLinks.push(<button key="prev" onClick={this.handleNavPrev}>&lt;</button>);
	}
	if ("next" in this.props.links) {
		navLinks.push(<button key="next" onClick={this.handleNavNext}>&gt;</button>);
	}
	if ("last" in this.props.links) {
		navLinks.push(<button key="last" onClick={this.handleNavLast}>&gt;&gt;</button>);
	}

	return (
		<div>
			<input ref="pageSize" defaultValue={this.props.pageSize} onInput={this.handleInput}/>
			<table>
				<tbody>
					<tr>
						<th>First Name</th>
						<th>Last Name</th>
						<th>Description</th>
						<th></th>
					</tr>
					{employees}
				</tbody>
			</table>
			<div>
				{navLinks}
			</div>
		</div>
	)
}
```

如前一节所述，它仍然转换`this.props.employees`成一个`<Element />`组件数组。然后它建立一个数组`navLinks`，一个HTML按钮的数组。

| **   | 由于React基于XML，因此不能在`<button>`元素中放置“<” 。您必须改用编码版本。 |
| ---- | ---------------------------------------- |
|      |                                          |

然后你可以看到`{navLinks}`插入到返回的HTML的底部。

### 删除现有的记录

删除条目要容易得多。获取其基于HAL的记录，并将**DELETE**应用于其**自我**链接。

```react
class Employee extends React.Component {

	constructor(props) {
		super(props);
		this.handleDelete = this.handleDelete.bind(this);
	}

	handleDelete() {
		this.props.onDelete(this.props.employee);
	}

	render() {
		return (
			<tr>
				<td>{this.props.employee.firstName}</td>
				<td>{this.props.employee.lastName}</td>
				<td>{this.props.employee.description}</td>
				<td>
					<button onClick={this.handleDelete}>Delete</button>
				</td>
			</tr>
		)
	}
}
```

Employee组件的更新版本在行的末尾显示一个额外的条目，一个删除按钮。`this.handleDelete`点击时会被注册。`handleDelete()`然后该函数可以在提供上下文重要`this.props.employee`记录时调用传递的回调函数。

| **   | 这再次表明，在一个地方管理顶级组件中的状态是最容易的。这可能并不*总是*如此，但通常情况下，在一个地方管理状态可以更容易保持直线和简单。通过使用特定于组件的细节（`this.props.onDelete(this.props.employee)`）调用回调，组件之间的行为编排非常容易。 |
| ---- | ---------------------------------------- |
|      |                                          |

将`onDelete()`函数追溯到顶端`App.onDelete`，您可以看到它是如何工作的：

```react
onDelete(employee) {
	client({method: 'DELETE', path: employee._links.self.href}).done(response => {
		this.loadFromServer(this.state.pageSize);
	});
}
```

使用基于页面的UI删除记录后应用的行为有点棘手。在这种情况下，它将从服务器重新加载整个数据，并应用相同的页面大小。然后显示第一页。

如果删除最后一页上的最后一条记录，则会跳到第一页。

### 调整页面大小

看超媒体真正发光的一种方法是更新页面大小。Spring Data REST根据页面大小流畅地更新导航链接。

在顶部有一个HTML元素`ElementList.render`：`<input ref="pageSize" defaultValue={this.props.pageSize} onInput={this.handleInput}/>`。

- `ref="pageSize"` 通过this.refs.pageSize可以轻松获取该元素。
- `defaultValue`用状态的**pageSize**初始化它。
- `onInput` 注册一个处理程序，如下所示。

```react
handleInput(e) {
	e.preventDefault();
	var pageSize = ReactDOM.findDOMNode(this.refs.pageSize).value;
	if (/^[0-9]+$/.test(pageSize)) {
		this.props.updatePageSize(pageSize);
	} else {
		ReactDOM.findDOMNode(this.refs.pageSize).value =
			pageSize.substring(0, pageSize.length - 1);
	}
}
```

它阻止事件冒泡。然后使用React的辅助函数来查找DOM节点的**ref**属性`<input>`并提取它的值`findDOMNode()`。它通过检查它是否是一串数字来测试输入是否真的是一个数字。如果是这样，它调用回调，发送新的页面大小到`App`React组件。如果不是，则刚输入的字符被剥离输入。

这是什么`App`做的，当它得到一个`updatePageSize()`？一探究竟：

```react
updatePageSize(pageSize) {
	if (pageSize !== this.state.pageSize) {
		this.loadFromServer(pageSize);
	}
}
```

由于新的页面大小会导致所有导航链接的更改，因此最好重新提取数据并从头开始。

### 把它放在一起

随着所有这些不错的增加，你现在有一个真正的虚拟的UI。

![超媒体1](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/hypermedia/images/hypermedia-1.png)

您可以看到顶部的页面大小设置，每行的删除按钮以及底部的导航按钮。导航按钮说明了超媒体控件的强大功能。

在下面，你可以看到`CreateDialog`元数据插入HTML输入占位符。

![超媒体2](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/hypermedia/images/hypermedia-2.png)

这确实显示了使用超媒体与域驱动的元数据（JSON模式）结合使用的强大功能。网页不必知道哪个字段是哪个。相反，用户可以*看到*它并知道如何使用它。如果您向`Employee`域对象添加了另一个字段，则此弹出窗口会自动显示该字段。

### 回顾

在这个部分：

- 您打开了Spring Data REST的分页功能。
- 您抛出硬编码的URI路径，并开始使用根URI与关系名称或“rels”相结合。
- 您更新了UI以动态使用基于页面的超媒体控件。
- 您添加了创建和删除员工的功能，并根据需要更新UI。
- 您可以更改页面大小并使UI灵活响应。

问题是什么？

你使网页动态。但打开另一个浏览器选项卡，并指向同一个应用程序。一个选项卡中的更改不会更新其他任何内容。

这是我们在下一节中可以解决的问题。

## 第3部分 - 条件操作

在上[一节中](https://spring.io/guides/tutorials/react-and-spring-data-rest/#react-and-spring-data-rest-part-2)，您了解了如何打开Spring Data REST的超媒体控件，通过分页导航UI，并基于更改页面大小动态调整大小。您添加了创建和删除员工并调整页面的功能。但是，考虑到其他用户对当前正在编辑的同一位数据所做的更新，没有任何解决方案是完整的。

随意从这个仓库中[获取代码](https://github.com/spring-guides/tut-react-and-spring-data-rest/tree/master/conditional)，并遵循。本部分是基于上一部分的应用程序添加额外的东西。

### 放弃或不放弃，这是一个问题

当您获取资源时，如果其他人更新资源，可能会失效。为了解决这个问题，Spring Data REST集成了两种技术：资源版本化和ETags。

通过在后端版本化资源并在前端使用ETags，可以有条件地放弃更改。换句话说，你可以检测资源是否已经改变，并防止PUT（或PATCH）跺脚别人的更新。让我们来看看。

### 版本控制REST资源

要支持资源的版本控制，请为需要此类保护的域对象定义一个版本属性。

src/main/java/com/greglturnquist/payroll/Employee.java

```Java
@Data
@Entity
public class Employee {

	private @Id @GeneratedValue Long id;
	private String firstName;
	private String lastName;
	private String description;

	private @Version @JsonIgnore Long version;

	private Employee() {}

	public Employee(String firstName, String lastName, String description) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.description = description;
	}
}
```

- 该**version**字段被`javax.persistence.Version`所注解。它会导致一个值被自动存储和更新，每一行插入和更新。

当获取一个单独的资源（而不是一个集合资源）时，Spring Data REST会自动添加一个[ETag响应头](https://tools.ietf.org/html/rfc7232#section-2.3)，其中包含这个字段的值。

### 获取个人资源及其标题

在上[一节中，](https://spring.io/guides/tutorials/react-and-spring-data-rest/#react-and-spring-data-rest-part-2)您使用集合资源来收集数据并填充UI的HTML表格。使用Spring Data REST，**_embedded**数据集被认为是数据预览。虽然用于浏览数据，为了得到像ETags的标题，你需要单独获取每个资源。

在此版本中，`loadFromServer`更新为获取集合，然后使用URI检索每个单独的资源。

src/main/js/app.js - 获取每个资源

```js
loadFromServer(pageSize) {
	follow(client, root, [
		{rel: 'employees', params: {size: pageSize}}]
	).then(employeeCollection => {
		return client({
			method: 'GET',
			path: employeeCollection.entity._links.profile.href,
			headers: {'Accept': 'application/schema+json'}
		}).then(schema => {
			this.schema = schema.entity;
			this.links = employeeCollection.entity._links;
			return employeeCollection;
		});
	}).then(employeeCollection => {
		return employeeCollection.entity._embedded.employees.map(employee =>
				client({
					method: 'GET',
					path: employee._links.self.href
				})
		);
	}).then(employeePromises => {
		return when.all(employeePromises);
	}).done(employees => {
		this.setState({
			employees: employees,
			attributes: Object.keys(this.schema.properties),
			pageSize: pageSize,
			links: this.links
		});
	});
}
```

1. 该`follow()`函数转到**employees**集合。
2. 该`then(employeeCollection ⇒ …)`子句创建一个调用来获取JSON模式数据。这有一个sub-then子句来存储`<App/>`组件中的元数据和导航链接。
   - 注意，这个嵌入的promise将返回employeeCollection。这样，收集可以传递给下一个电话，让你抓住元数据。
3. 第二个`then(employeeCollection ⇒ …)`子句将员工集合转换为GET promise的数组，以获取每个单独的资源。**这就是你需要为每个员工获取一个ETag头。**
4. 该`then(employeePromises ⇒ …)`子句采用GET promise的数组，并将它们合并成一个单一的promise `when.all()`，解决所有的GET promise时解决。
5. `loadFromServer``done(employees ⇒ …)`用这种数据合并来包装更新UI状态的地方。

这个链条也在其他地方实施。例如，`onNavigate()`哪个用于跳转到不同的页面，已被更新以获取单个资源。由于它与上面显示的大部分相同，所以不在本节中。

### 更新现有资源

在本节中，您将添加一个`UpdateDialog`React组件来编辑现有员工记录。

src / main / js / app.js - UpdateDialog组件

```react
class UpdateDialog extends React.Component {

	constructor(props) {
		super(props);
		this.handleSubmit = this.handleSubmit.bind(this);
	}

	handleSubmit(e) {
		e.preventDefault();
		var updatedEmployee = {};
		this.props.attributes.forEach(attribute => {
			updatedEmployee[attribute] = ReactDOM.findDOMNode(this.refs[attribute]).value.trim();
		});
		this.props.onUpdate(this.props.employee, updatedEmployee);
		window.location = "#";
	}

	render() {
		var inputs = this.props.attributes.map(attribute =>
				<p key={this.props.employee.entity[attribute]}>
					<input type="text" placeholder={attribute}
						   defaultValue={this.props.employee.entity[attribute]}
						   ref={attribute} className="field" />
				</p>
		);

		var dialogId = "updateEmployee-" + this.props.employee.entity._links.self.href;

		return (
			<div key={this.props.employee.entity._links.self.href}>
				<a href={"#" + dialogId}>Update</a>
				<div id={dialogId} className="modalDialog">
					<div>
						<a href="#" title="Close" className="close">X</a>

						<h2>Update an employee</h2>

						<form>
							{inputs}
							<button onClick={this.handleSubmit}>Update</button>
						</form>
					</div>
				</div>
			</div>
		)
	}

};
```

这个新的组件有一个`handleSubmit()`功能，以及预期的`render()`功能，类似于`<CreateDialog />`组件。

让我们以相反的顺序挖掘这些功能，并首先看看这个`render()`功能。

#### 渲染

该组件使用相同的CSS / HTML策略来显示和隐藏对话框`<CreateDialog />`。

它将JSON模式属性数组转换为一个HTML输入数组，包装在样式的段落元素中。这也`<CreateDialog />`与一个区别是一样的：字段是加载**this.props.employee**。在CreateDialog组件中，这些字段是空的。

该**ID**字段不同建造。整个用户界面上只有一个CreateDialog链接，但显示的每一行都有单独的UpdateDialog链接。因此，**id**字段是基于**自**链接的URI。这用于<div>元素的React **键**以及HTML锚点标记和隐藏的弹出窗口。

#### 处理用户输入

提交按钮链接到组件的`handleSubmit()`功能。这很方便地`React.findDOMNode()`使用[React refs](https://facebook.github.io/react/docs/more-about-refs.html)来提取弹出窗口的细节。

在输入值被提取并加载到`updatedEmployee`对象中之后，`onUpdate()`调用顶级方法。这继续了React单向绑定的风格，要调用的函数被从上层组件推送到下层组件。这样，国家仍然处于顶端。

### 条件PUT

所以你已经花费了所有的努力在数据模型中嵌入版本。Spring Data REST已经提供了ETag响应头的值。这里是你得到它使用得很好的地方！

src/main/js/app.js - onUpdate函数

```react
onUpdate(employee, updatedEmployee) {
	client({
		method: 'PUT',
		path: employee.entity._links.self.href,
		entity: updatedEmployee,
		headers: {
			'Content-Type': 'application/json',
			'If-Match': employee.headers.Etag
		}
	}).done(response => {
		this.loadFromServer(this.state.pageSize);
	}, response => {
		if (response.status.code === 412) {
			alert('DENIED: Unable to update ' +
				employee.entity._links.self.href + '. Your copy is stale.');
		}
	});
}
```

PUT与[If-Match请求标头](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.24)会导致Spring Data REST检查相对于当前版本的值。如果传入的**If-Match**值与数据存储的版本值不匹配，则Spring Data REST将失败，**HTTP 412 Precondition Failed**。

| **   | [Promises / A +](https://promisesaplus.com/)的规范实际上将它们的API定义为`then(successFunction, errorFunction)`。到目前为止，您只能看到它与成功功能一起使用。在上面的代码片段中，有两个函数。`loadFromServer`当错误函数显示关于陈旧数据的浏览器警报时，成功函数调用。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 把它放在一起

在`UpdateDialog`定义了React组件并很好地链接到顶层`onUpdate`函数后，最后一步是将其连接到现有的组件布局。

在`CreateDialog`上一节中创建在的上面放`EmployeeList`，因为只有一个实例。但是，`UpdateDialog`直接绑定到特定的员工。所以你可以看到它在下面的`Employee`React组件中插入：

src/main/js/app.js - 具有UpdateDialog的员工

```react
class Employee extends React.Component {

	constructor(props) {
		super(props);
		this.handleDelete = this.handleDelete.bind(this);
	}

	handleDelete() {
		this.props.onDelete(this.props.employee);
	}

	render() {
		return (
			<tr>
				<td>{this.props.employee.entity.firstName}</td>
				<td>{this.props.employee.entity.lastName}</td>
				<td>{this.props.employee.entity.description}</td>
				<td>
					<UpdateDialog employee={this.props.employee}
								  attributes={this.props.attributes}
								  onUpdate={this.props.onUpdate}/>
				</td>
				<td>
					<button onClick={this.handleDelete}>Delete</button>
				</td>
			</tr>
		)
	}
}
```

在本节中，您将使用收集资源切换到单个资源。现在可以找到员工记录的字段`this.props.employee.entity`。它使我们能够在`this.props.employee.headers`哪里找到ETags。

Spring Data REST还支持其他头文件（如[Last-Modified](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29)），它们不是本系列的一部分。所以用这种方式来构建数据是非常方便的。

| **   | 当使用[rest.js](https://github.com/cujojs/rest)作为选择的REST库时，`.entity`和的结构`.headers`是唯一相关的。如果您使用不同的库，则必须根据需要进行调整。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 见证结果

1. 启动应用程序（`./mvnw spring-boot:run`）。

2. 打开一个选项卡并导航到[http：// localhost：8080](http://localhost:8080/)。

   ![条件1](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/conditional/images/conditional-1.png)

3. 拉起Frodo的编辑对话框。

4. 在浏览器中打开另一个选项卡，并拉起相同的记录。

5. 在第一个标签中更改记录。

6. 尝试在第二个选项卡中进行更改。

   ![条件2](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/conditional/images/conditional-2.png)

![有条件的3](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/conditional/images/conditional-3.png)

有了这些MODS，您可以通过避免冲突来提高数据完整性。

### 回顾

在这个部分：

- 您为域模型配置了一个`@Version`基于JPA的乐观锁定的字段。
- 您调整了前端以获取单个资源。
- 您将ETag头从单个资源中插入到**If-Match**请求头中，以使PUT成为有条件的。
- 您为列表中显示的每个员工编写了一个新的UpdateDialog。

插入这个插件后，很容易避免与其他用户发生冲突，或者简单地覆盖他们的编辑。

问题是什么？

当你编辑一个不好的记录时，这当然是很好的。但是最好等到你点击“提交”找出来？

获取资源的逻辑是在这两个非常相似的`loadFromServer`和`onNavigate`。你看到避免重复代码的方法吗？

您将JSON Schema元数据用于构建`CreateDialog`和`UpdateDialog`输入。你看到其他地方使用元数据来使事物更通用？想象一下，你想添加五个字段`Employee.java`。如何更新UI？

## 第4部分 - 事件

在[上一节](https://spring.io/guides/tutorials/react-and-spring-data-rest/#react-and-spring-data-rest-part-3)，您引入有条件的更新编辑相同数据时避免与其他用户的冲突。您还学习了如何在乐观锁定的情况下在后端版本化数据。如果某人编辑了相同的记录，则可以获得一个提示，以便刷新页面并获取更新。

那很好。但是你知道甚至更好吗？当其他人更新资源时让UI动态响应。

在本节中，您将学习如何使用Spring Data REST的内置事件系统来检测后端的变化，并通过Spring的WebSocket支持向所有用户发布更新。然后，您将能够随着数据更新动态调整客户端。

随意从这个仓库中[获取代码](https://github.com/sprig-guides/tut-react-and-spring-data-rest/tree/master/events)，并遵循。本部分是基于上一部分的应用程序添加额外的东西。

### 将Spring WebSocket支持添加到项目中

在进行之前，你需要添加一个依赖项到你的项目的pom.xml文件中：

```Xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

这带来了Spring Boot的WebSocket启动器。

### 用Spring配置WebSockets

[Spring带有强大的WebSocket支持](https://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#websocket)。有一点要认识到，WebSocket是一个非常低层的协议。它只是提供了在客户端和服务器之间传输数据的手段。建议使用一个子协议（本节的STOMP）来实际编码数据和路由。

以下代码用于在服务器端配置WebSocket支持：

```java
@Component
@EnableWebSocketMessageBroker
public class WebSocketConfiguration extends AbstractWebSocketMessageBrokerConfigurer {

	static final String MESSAGE_PREFIX = "/topic";

	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/payroll").withSockJS();
	}

	@Override
	public void configureMessageBroker(MessageBrokerRegistry registry) {
		registry.enableSimpleBroker(MESSAGE_PREFIX);
		registry.setApplicationDestinationPrefixes("/app");
	}
}
```

- `@EnableWebSocketMessageBroker` 打开WebSocket支持。
- `AbstractWebSocketMessageBrokerConfigurer` 提供了一个方便的基类来配置基本功能。
- **MESSAGE_PREFIX**是您将添加到每条消息路由的前缀。
- `registerStompEndpoints()`用于在后端为客户端和服务器配置端点以链接（`/payroll`）。
- `configureMessageBroker()` 用于配置用于在服务器和客户端之间中继消息的代理。

通过这个配置，现在可以使用Spring Data REST事件并通过WebSocket发布它们。

### 订阅Spring Data REST事件

Spring Data REST 根据存储库上发生的操作生成多个[应用程序事件](https://docs.spring.io/spring-data/rest/docs/current/reference/html/#events)。以下代码显示了如何订阅以下某些事件：

```java
@Component
@RepositoryEventHandler(Employee.class)
public class EventHandler {

	private final SimpMessagingTemplate websocket;

	private final EntityLinks entityLinks;

	@Autowired
	public EventHandler(SimpMessagingTemplate websocket, EntityLinks entityLinks) {
		this.websocket = websocket;
		this.entityLinks = entityLinks;
	}

	@HandleAfterCreate
	public void newEmployee(Employee employee) {
		this.websocket.convertAndSend(
				MESSAGE_PREFIX + "/newEmployee", getPath(employee));
	}

	@HandleAfterDelete
	public void deleteEmployee(Employee employee) {
		this.websocket.convertAndSend(
				MESSAGE_PREFIX + "/deleteEmployee", getPath(employee));
	}

	@HandleAfterSave
	public void updateEmployee(Employee employee) {
		this.websocket.convertAndSend(
				MESSAGE_PREFIX + "/updateEmployee", getPath(employee));
	}

	/**
	 * Take an {@link Employee} and get the URI using Spring Data REST's {@link EntityLinks}.
	 *
	 * @param employee
	 */
	private String getPath(Employee employee) {
		return this.entityLinks.linkForSingleResource(employee.getClass(),
				employee.getId()).toUri().getPath();
	}

}
```

- `@RepositoryEventHandler(Employee.class)`标记这个类来捕获基于**员工的**事件。
- `SimpMessagingTemplate`并`EntityLinks`从应用程序上下文自动装配。
- 该`@HandleXYZ`注释标志需要听的方法。这些方法必须是公开的。

这些处理程序方法中的每一个都调用`SimpMessagingTemplate.convertAndSend()`通过WebSocket传输消息。这是一个pub-sub方法，因此一条消息被传递给每个连接的消费者。

每个消息的路由是不同的，允许多个消息发送到客户端上不同的接收者，而只需要一个打开的WebSocket，一个资源高效的方法。

`getPath()`使用Spring Data REST `EntityLinks`来查找给定的类类型和id的路径。为了满足客户的需求，这个`Link`对象被转换为Java URI，其路径被提取。

| **   | `EntityLinks` 带有几种实用方法来编程地查找各种资源的路径，无论是单个还是集合。 |
| ---- | ---------------------------------------- |
|      |                                          |

本质上，您正在侦听创建，更新和删除事件，并在完成后发送通知给所有客户。在它们发生之前拦截这些操作也是可能的，也许可以记录它们，出于某种原因阻止它们，或者用额外的信息装饰域对象。（在下一节中，我们将看到一个非常方便的使用！）

### 配置一个JavaScript WebSocket

下一步是编写一些客户端代码来使用WebSocket事件。在他们的主要应用程序中的后续块拉入模块。

```js
var stompClient = require('./websocket-listener')
```

该模块如下所示：

```Js
'use strict';

var SockJS = require('sockjs-client'); (1)
require('stompjs'); (2)

function register(registrations) {
	var socket = SockJS('/payroll'); (3)
	var stompClient = Stomp.over(socket);
	stompClient.connect({}, function(frame) {
		registrations.forEach(function (registration) { (4)
			stompClient.subscribe(registration.route, registration.callback);
		});
	});
}

module.exports.register = register;
```

| ****1** | 您可以使用SockJS JavaScript库来通过WebSockets进行通话。 |
| ------- | ---------------------------------------- |
| ****2** | 您可以使用stomp-websocket JavaScript库来使用STOMP子协议。 |
| ****3** | 这里是WebSocket指向应用程序`/payroll`端点的地方。       |
| ****4** | 遍历所`registrations`提供的数组，每个可以订阅消息到达时的回调。  |

每个注册条目都有一个`route`和一个`callback`。在下一节中，您可以看到如何注册事件处理程序。

### 注册WebSocket事件

在React中，组件`componentDidMount()`是在DOM中呈现后调用的函数。这也是注册WebSocket事件的正确时机，因为组件现在已经联机并准备就绪。检出以下代码：

```Js
componentDidMount() {
	this.loadFromServer(this.state.pageSize);
	stompClient.register([
		{route: '/topic/newEmployee', callback: this.refreshAndGoToLastPage},
		{route: '/topic/updateEmployee', callback: this.refreshCurrentPage},
		{route: '/topic/deleteEmployee', callback: this.refreshCurrentPage}
	]);
}
```

第一行和以前一样，所有员工都是使用页面大小从服务器获取的。第二行显示了为WebSocket事件注册的JavaScript对象数组，每个对象都有一个`route`和一个`callback`。

创建新员工时，行为是刷新数据集，然后使用分页链接导航到**最后**一页。为什么刷新数据之前导航到最后？添加新记录可能会导致创建新页面。虽然有可能计算这是否会发生，但它颠覆了超媒体的观点。如果有性能驱动的原因，那么最好是使用现有的链接，而不是一起下去。

员工更新或删除时，行为是刷新当前页面。更新记录时，会影响您正在查看的页面。当您删除当前页面上的记录时，下一页的记录将被拉入当前页面，因此也需要刷新当前页面。

| **   | 这些WebSocket消息不需要开始`/topic`。这只是一个表示pub-sub语义的常用约定。 |
| ---- | ---------------------------------------- |
|      |                                          |

在下一节中，您可以看到执行这些操作的实际操作。

### 反应到WebSocket事件和更新UI状态

以下代码块包含用于在收到WebSocket事件时更新UI状态的两个回调。

```js
refreshAndGoToLastPage(message) {
	follow(client, root, [{
		rel: 'employees',
		params: {size: this.state.pageSize}
	}]).done(response => {
		if (response.entity._links.last !== undefined) {
			this.onNavigate(response.entity._links.last.href);
		} else {
			this.onNavigate(response.entity._links.self.href);
		}
	})
}

refreshCurrentPage(message) {
	follow(client, root, [{
		rel: 'employees',
		params: {
			size: this.state.pageSize,
			page: this.state.page.number
		}
	}]).then(employeeCollection => {
		this.links = employeeCollection.entity._links;
		this.page = employeeCollection.entity.page;

		return employeeCollection.entity._embedded.employees.map(employee => {
			return client({
				method: 'GET',
				path: employee._links.self.href
			})
		});
	}).then(employeePromises => {
		return when.all(employeePromises);
	}).then(employees => {
		this.setState({
			page: this.page,
			employees: employees,
			attributes: Object.keys(this.schema.properties),
			pageSize: this.state.pageSize,
			links: this.links
		});
	});
}
```

`refreshAndGoToLastPage()`使用熟悉的`follow()`功能导航到应用的**大小**参数的**员工**链接，插入。接收到响应后，您将从上一节调用相同的函数，然后跳转到**最后一个**页面，即查找新记录的页面。****`this.state.pageSize``onNavigate()`****

`refreshCurrentPage()`还采用了`follow()`功能，但适用`this.state.pageSize`于**尺寸**和`this.state.page.number`到**页面**。这会获取您当前正在查看的相同页面，并相应地更新状态。

| **   | 此行为通知每个客户端在发送更新或删除消息时刷新其当前页面。当前页面可能与当前事件无关。但是，弄清楚这一点可能非常棘手。如果被删除的记录在第二页，而你在看第三页？每一个条目都会改变。但是，这是所需的行为呢？也许，也许不是。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 将状态管理移出本地更新

在完成本节之前，有一些事情要认识。你刚刚添加了一个新的方式在UI中的状态得到更新：当一个WebSocket消息到达。但是更新国家的旧方法仍然存在。

为了简化你的代码的状态管理，如果你删除旧的方法，它简化了事情。换句话说，提交您的**POST**，**PUT**和**DELETE**调用，但不要使用它们的结果来更新UI的状态。相反，等待WebSocket事件回圈，然后进行更新。

下面的代码块显示了与`onCreate()`上一节相同的功能，只是简化了：

```js
onCreate(newEmployee) {
	follow(client, root, ['employees']).done(response => {
		client({
			method: 'POST',
			path: response.entity._links.self.href,
			entity: newEmployee,
			headers: {'Content-Type': 'application/json'}
		})
	})
}
```

在这里，该`follow()`函数用于到达**员工**链接，然后应用**POST**操作。注意`client({method: 'GET' …})`有没有`then()`或`done()`以前有过？现在可以找到`refreshAndGoToLastPage()`您刚刚查看的用于侦听更新的事件处理程序。

### 把它放在一起

随着所有这些MODS，启动应用程序（`./mvnw spring-boot:run`），并与它捅。打开两个浏览器选项卡并调整大小，以便可以看到它们。开始在一个更新，看看他们如何即时更新其他选项卡。打开手机并访问相同的页面。找一个朋友，问他或她做同样的事情。你可能会发现这种动态更新更加敏锐。

想要挑战？尝试从前一节中的练习，在两个不同的浏览器选项卡中打开相同的记录。尝试在一个更新它，而不是看到它在另一个更新。如果可能的话，有条件的PUT代码仍应该保护你。但是这可能会更棘手！

### 回顾

在这个部分：

- 你配置了Spring的WebSocket suport和SockJS fallback。
- 您订阅了从Spring Data REST创建，更新和删除事件以动态更新UI。
- 您发布了受影响的REST资源的URI以及上下文消息（“/ topic / newEmployee”，“/ topic / updateEmployee”等）。
- 您在UI中注册了WebSocket侦听器来侦听这些事件。
- 您将监听器连接到处理程序以更新UI状态。

通过所有这些功能，可以轻松地并行运行两个浏览器，并查看如何更新一个涟漪到另一个涟漪。

问题是什么？

虽然多个显示很好地更新，抛光确切的行为是有保证的。例如，创建新用户将导致所有用户跳到最后。任何想法如何处理？

分页是有用的，但提供了一个棘手的状态来管理。这个示例应用程序的成本很低，并且在更新DOM时非常高效地响应，而不会在UI中引起太多的闪烁。但是有了更复杂的应用程序，并不是所有这些方法都适合。

在使用分页进行设计时，必须确定客户端之间的预期行为以及是否需要更新。根据您的要求和系统的性能，现有的导航超媒体可能是足够的。

## 第5部分 - 保护用户界面和API

在[上一节](https://spring.io/guides/tutorials/react-and-spring-data-rest/#react-and-spring-data-rest-part-4)，你所做的应用程序动态响应通过Spring数据REST内置的事件处理程序和Spring框架的WebSocket的支持来自其他用户的更新。但是，没有保护整个事物的应用程序是不完整的，只有适当的用户才能访问UI及其背后的资源。

随意从这个仓库中[获取代码](https://github.com/spring-guides/tut-react-and-spring-data-rest/tree/master/security)，并遵循。本部分是基于上一部分的应用程序添加额外的东西。

### 将Spring Security添加到项目中

在进行之前，你需要添加一些依赖项到你的项目的pom.xml文件中：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
	<groupId>org.thymeleaf.extras</groupId>
	<artifactId>thymeleaf-extras-springsecurity4</artifactId>
</dependency>
```

这引入了Spring Boot的Spring Security启动器以及一些额外的Thymeleaf标签来在网页中进行安全性查找。

### 定义安全模型

在过去的部分，你已经有了一个很好的工资系统。在后端声明事物并且让Spring Data REST完成繁重的工作非常方便。下一步是对需要建立安全控制的系统进行建模。

如果这是一个工资系统，那么只有管理人员可以访问它。所以通过建模一个`Manager`对象来解决问题：

```Java
@Data
@ToString(exclude = "password")
@Entity
public class Manager {

	public static final PasswordEncoder PASSWORD_ENCODER = new BCryptPasswordEncoder();

	private @Id @GeneratedValue Long id;

	private String name;

	private @JsonIgnore String password;

	private String[] roles;

	public void setPassword(String password) {
		this.password = PASSWORD_ENCODER.encode(password);
	}

	protected Manager() {}

	public Manager(String name, String password, String... roles) {

		this.name = name;
		this.setPassword(password);
		this.roles = roles;
	}

}
```

- `PASSWORD_ENCODER` 是加密新的密码或采取密码输入并加以比较之前的手段。
- `id`，`name`，`password`，和`roles`定义来限制访问所需的参数。
- 定制`setPassword()`确保密码永远不会被清除。

设计安全层时需要记住一个关键要点。保护正确的数据位（如密码），不要让它们打印到控制台，日志或通过JSON序列化输出。

- `@ToString(exclude = "password")` 确保Lombok生成的toString（）方法不会打印出密码。
- `@JsonIgnore` 应用于密码字段保护从Jackson序列化该字段。

### 创建一个经理的仓库

Spring Data在管理实体方面非常出色。为什么不创建一个存储库来处理这些经理？

```java
@RepositoryRestResource(exported = false)
public interface ManagerRepository extends Repository<Manager, Long> {

	Manager save(Manager manager);

	Manager findByName(String name);

}
```

而不是延长惯常`CrudRepository`，你不需要太多的方法。相反，您需要保存数据（也用于更新），您需要查找现有用户。因此，你可以使用Spring Data Common的最小`Repository`标记接口。它没有预定义的操作。

Spring Data REST默认情况下会导出它找到的任何库。您不希望此存储库公开REST操作！应用`@RepositoryRestResource(exported = false)`注释以阻止其导出。这可以防止存储库以及任何元数据。

### 将员工与经理联系起来

建模安全的最后一点是将员工与经理联系起来。在这个领域，一个员工可以有一个经理，而一个经理可以有多个员工：

```java
@Data
@Entity
public class Employee {

	private @Id @GeneratedValue Long id;
	private String firstName;
	private String lastName;
	private String description;

	private @Version @JsonIgnore Long version;

	private @ManyToOne Manager manager;

	private Employee() {}

	public Employee(String firstName, String lastName, String description, Manager manager) {
		this.firstName = firstName;
		this.lastName = lastName;
		this.description = description;
		this.manager = manager;
	}
}
```

- 经理属性通过JPA链接`@ManyToOne`。经理不需要，`@OneToMany`因为你没有定义需要查看。
- 实用程序构造函数调用更新为支持初始化。

### 保护员工的经理

Spring Security在定义安全策略方面支持多种选项。在本节中，您想限制管理人员只能查看员工工资单数据，而保存，更新和删除操作仅限于员工经理。换句话说，任何经理都可以登录和查看数据，但只有给定的员工经理可以进行任何更改。

```java
@PreAuthorize("hasRole('ROLE_MANAGER')")
public interface EmployeeRepository extends PagingAndSortingRepository<Employee, Long> {

	@Override
	@PreAuthorize("#employee?.manager == null or #employee?.manager?.name == authentication?.name")
	Employee save(@Param("employee") Employee employee);

	@Override
	@PreAuthorize("@employeeRepository.findOne(#id)?.manager?.name == authentication?.name")
	void delete(@Param("id") Long id);

	@Override
	@PreAuthorize("#employee?.manager?.name == authentication?.name")
	void delete(@Param("employee") Employee employee);

}
```

`@PreAuthorize`在界面的顶部限制了用**ROLE_MANAGER**访问的人。

在上`save()`，员工的经理是空的（当没有经理分配时，最初创建新员工），或者员工的经理姓名与当前认证的用户姓名相匹配。这里您使用[Spring Security的SpEL表达式](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#el-access)来定义访问权限。它带有一个方便的“？”。属性导航器来处理空的检查。注意使用`@Param(…)`参数来链接HTTP操作和方法也很重要。

上`delete()`，该方法既可以访问员工或事件，它只有一个ID，那么它必须找到**employeeRepository**在应用程序上下文，执行`findOne(id)`，然后检查经理人对当前认证的用户。

### 写一个`UserDetails`服务

与安全性整合的一个共同点是定义一个`UserDetailsService`。这是将用户的数据存储连接到Spring Security界面的方法。Spring Security需要一种方法来查找用户的安全检查，这是一个桥梁。值得庆幸的是，Spring Data的工作量非常小：

```java
@Component
public class SpringDataJpaUserDetailsService implements UserDetailsService {

	private final ManagerRepository repository;

	@Autowired
	public SpringDataJpaUserDetailsService(ManagerRepository repository) {
		this.repository = repository;
	}

	@Override
	public UserDetails loadUserByUsername(String name) throws UsernameNotFoundException {
		Manager manager = this.repository.findByName(name);
		return new User(manager.getName(), manager.getPassword(),
				AuthorityUtils.createAuthorityList(manager.getRoles()));
	}

}
```

`SpringDataJpaUserDetailsService`实现Spring Security的`UserDetailsService`。界面有一个方法：`loadUserByUsername()`。这个方法是为了返回一个`UserDetails`对象，所以Spring Security可以询问用户的信息。

因为您有一个`ManagerRepository`，所以不需要编写任何SQL或JPA表达式来获取所需的数据。在这个类中，它是通过构造函数注入自动装配的。

`loadUserByUsername()`进入你刚刚写的定制发现者`findByName()`。然后填充一个`User`实现了`UserDetails`接口的Spring Security 实例。您也正在使用Spring Securiy `AuthorityUtils`从一个基于字符串的角色数组转换为Java `List`的`GrantedAuthority`。

### 连接你的安全策略

`@PreAuthorize`应用于您的存储库的表达式是**访问规则**。这些规则是没有安全策略的。

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Autowired
	private SpringDataJpaUserDetailsService userDetailsService;

	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth
			.userDetailsService(this.userDetailsService)
				.passwordEncoder(Manager.PASSWORD_ENCODER);
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/built/**", "/main.css").permitAll()
				.anyRequest().authenticated()
				.and()
			.formLogin()
				.defaultSuccessUrl("/", true)
				.permitAll()
				.and()
			.httpBasic()
				.and()
			.csrf().disable()
			.logout()
				.logoutSuccessUrl("/");
	}

}
```

这个代码有很多复杂性，所以我们先来看看注释和API。然后我们将讨论它定义的安全策略。

- `@EnableWebSecurity`告诉Spring Boot删除其自动配置的安全策略，然后使用这个策略。对于快速演示，自动配置的安全性是好的。但是对于任何真实的事情，你应该自己写这个政策。
- `@EnableGlobalMethodSecurity`使用Spring Security复杂的[@Pre和@Post标注](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#el-pre-post-annotations)打开方法级别的安全性。
- 它扩展`WebSecurityConfigurerAdapter`了一个方便的基类来编写策略。
- 它`SpringDataJpaUserDetailsService`通过现场注入自动装配，然后通过该`configure(AuthenticationManagerBuilder)`方法将其插入。在`PASSWORD_ENCODER`从`Manager`也设置。
- 关键的安全策略是使用纯Java编写的`configure(HttpSecurity)`。

安全策略表示使用之前定义的访问规则来授权所有请求。

- 列出的路径`antMatchers()`被授予无条件访问权限，因为没有理由阻止静态Web资源。
- 任何不符合`anyRequest().authenticated()`要求的内容都需要认证。
- 通过这些访问规则设置，Spring Security被告知使用基于表单的认证，成功时默认为“/”，并授予访问登录页面的权限。
- BASIC登录也被配置为禁用CSRF。这主要是用于演示，不经过仔细分析，不建议用于生产系统。
- 注销配置为使用户“/”。

| **   | BASIC认证是方便的，当你正在试验curl。使用curl访问基于表单的系统令人望而生畏。认识到通过HTTP（而不是HTTPS）通过任何机制进行身份验证非常重要，这使得您有可能通过网络嗅探凭据。CSRF是一个完好无损的协议。它只是简单地禁用与BASIC的交互和卷曲更容易。在生产中，最好离开它。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 自动添加安全细节

良好的用户体验是应用程序可以自动应用上下文。在这个例子中，如果一个登录管理器创建一个新的员工记录，这个管理者拥有它是有意义的。使用Spring Data REST的事件处理程序，用户不需要明确地链接它。这也确保用户不会意外地记录到错误的管理者。

```Java
@Component
@RepositoryEventHandler(Employee.class)
public class SpringDataRestEventHandler {

	private final ManagerRepository managerRepository;

	@Autowired
	public SpringDataRestEventHandler(ManagerRepository managerRepository) {
		this.managerRepository = managerRepository;
	}

	@HandleBeforeCreate
	public void applyUserInformationUsingSecurityContext(Employee employee) {

		String name = SecurityContextHolder.getContext().getAuthentication().getName();
		Manager manager = this.managerRepository.findByName(name);
		if (manager == null) {
			Manager newManager = new Manager();
			newManager.setName(name);
			newManager.setRoles(new String[]{"ROLE_MANAGER"});
			manager = this.managerRepository.save(newManager);
		}
		employee.setManager(manager);
	}
}
```

`@RepositoryEventHandler(Employee.class)`将此事件处理程序标记为仅应用于`Employee`对象。该`@HandleBeforeCreate`注解给你一个机会来改变输入`Employee`记录之前，它被写入到数据库中。

在这种情况下，您可以查找当前用户的安全上下文来获取用户名。然后查找相关的经理使用`findByName()`并应用到经理。如果系统中不存在一个额外的胶水代码，可以创建一个新的管理器。但是这主要是为了支持数据库的初始化。在真正的生产系统中，应该删除该代码，而是依靠DBA或Security Ops团队来正确维护用户数据存储。

### 预加载管理器数据

加载管理者并将这些管理者连接到这些管理者是相当直接的

```java
@Component
public class DatabaseLoader implements CommandLineRunner {

	private final EmployeeRepository employees;
	private final ManagerRepository managers;

	@Autowired
	public DatabaseLoader(EmployeeRepository employeeRepository,
						  ManagerRepository managerRepository) {

		this.employees = employeeRepository;
		this.managers = managerRepository;
	}

	@Override
	public void run(String... strings) throws Exception {

		Manager greg = this.managers.save(new Manager("greg", "turnquist",
							"ROLE_MANAGER"));
		Manager oliver = this.managers.save(new Manager("oliver", "gierke",
							"ROLE_MANAGER"));

		SecurityContextHolder.getContext().setAuthentication(
			new UsernamePasswordAuthenticationToken("greg", "doesn't matter",
				AuthorityUtils.createAuthorityList("ROLE_MANAGER")));

		this.employees.save(new Employee("Frodo", "Baggins", "ring bearer", greg));
		this.employees.save(new Employee("Bilbo", "Baggins", "burglar", greg));
		this.employees.save(new Employee("Gandalf", "the Grey", "wizard", greg));

		SecurityContextHolder.getContext().setAuthentication(
			new UsernamePasswordAuthenticationToken("oliver", "doesn't matter",
				AuthorityUtils.createAuthorityList("ROLE_MANAGER")));

		this.employees.save(new Employee("Samwise", "Gamgee", "gardener", oliver));
		this.employees.save(new Employee("Merry", "Brandybuck", "pony rider", oliver));
		this.employees.save(new Employee("Peregrin", "Took", "pipe smoker", oliver));

		SecurityContextHolder.clearContext();
	}
}
```

一个问题就是，当这个装载器运行时，Spring Security在访问规则的激活下是完全有效的。因此，为了保存员工数据，您必须使用Spring Security的`setAuthentication()`API来使用正确的名称和角色对此加载器进行身份验证。最后，安全上下文被清除。

### 浏览您的安全REST服务

使用所有这些mod，你可以启动应用程序（`./mvnw spring-boot:run`）并使用cURL检查mods。

```bash
$ curl -v -u greg:turnquist localhost:8080/api/employees/1
*   Trying ::1...
* Connected to localhost (::1) port 8080 (#0)
* Server auth using Basic with user 'greg'
> GET /api/employees/1 HTTP/1.1
> Host: localhost:8080
> Authorization: Basic Z3JlZzp0dXJucXVpc3Q=
> User-Agent: curl/7.43.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: Apache-Coyote/1.1
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< Set-Cookie: JSESSIONID=E27F929C1836CC5BABBEAB78A548DF8C; Path=/; HttpOnly
< ETag: "0"
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Tue, 25 Aug 2015 15:57:34 GMT
<
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "description" : "ring bearer",
  "manager" : {
    "name" : "greg",
    "roles" : [ "ROLE_MANAGER" ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/api/employees/1"
    }
  }
}
```

这显示了比第一节更多的细节。首先，Spring Security打开几个HTTP协议来防止各种攻击媒介（Pragma，Expires，X-Frame-Options等）。您也正在发出BASIC证书，`-u greg:turnquist`用于呈现Authorization标头。

在所有标题中，您可以从版本化资源中看到**ETag**标题。

最后，在数据本身内部，您可以看到一个新的属性：**manager**。您可以看到它包含名称和角色，但不包括密码。这是由于`@JsonIgnore`在该领域使用。由于Spring Data REST没有导出该存储库，因此它的值在此资源中内联。在下一部分中更新UI时，您会很好地使用它。

### 在UI上显示管理员信息

随着所有这些mod在后端，你现在可以转移到更新前端的东西。首先，在`<Employee />`React组件中显示一名员工的经理：

```js
class Employee extends React.Component {

	constructor(props) {
		super(props);
		this.handleDelete = this.handleDelete.bind(this);
	}

	handleDelete() {
		this.props.onDelete(this.props.employee);
	}

	render() {
		return (
			<tr>
				<td>{this.props.employee.entity.firstName}</td>
				<td>{this.props.employee.entity.lastName}</td>
				<td>{this.props.employee.entity.description}</td>
				<td>{this.props.employee.entity.manager.name}</td>
				<td>
					<UpdateDialog employee={this.props.employee}
								  attributes={this.props.attributes}
								  onUpdate={this.props.onUpdate}/>
				</td>
				<td>
					<button onClick={this.handleDelete}>Delete</button>
				</td>
			</tr>
		)
	}
}
```

这只是添加一列`this.props.employee.entity.manager.name`。

### 筛选出JSON模式元数据

如果数据输出中显示一个字段，则可以安全地假定它在JSON模式元数据中具有条目。你可以在下面的摘录中看到它：

```js
{
	...
    “经理” ： {
      “readOnly”：假，
      “$ ref”：“＃/ descriptors / manager”
    },
    ...
  },
  ...
  “$ schema”：“http://json-schema.org/draft-04/schema#”
}
```

经理人字段不是你想要人们直接编辑的东西。由于它是内联的，因此应将其视为只读属性。要从`CreateDialog`and中过滤内联条目`UpdateDialog`，只需在获取JSON Schema元数据后删除这些条目`loadFromServer()`。

```js
/**
 * Filter unneeded JSON Schema properties, like uri references and
 * subtypes ($ref).
 */
Object.keys(schema.entity.properties).forEach(function (property) {
	if (schema.entity.properties[property].hasOwnProperty('format') &&
		schema.entity.properties[property].format === 'uri') {
		delete schema.entity.properties[property];
	}
	else if (schema.entity.properties[property].hasOwnProperty('$ref')) {
		delete schema.entity.properties[property];
	}
});

this.schema = schema.entity;
this.links = employeeCollection.entity._links;
return employeeCollection;
```

这段代码修剪了URI关系以及$ ref条目。

### 陷入未经授权的访问

通过在后端配置安全检查，添加一个处理程序，以防有人尝试更新记录时未经授权：

```js
onUpdate(employee, updatedEmployee) {
	client({
		method: 'PUT',
		path: employee.entity._links.self.href,
		entity: updatedEmployee,
		headers: {
			'Content-Type': 'application/json',
			'If-Match': employee.headers.Etag
		}
	}).done(response => {
		/* Let the websocket handler update the state */
	}, response => {
		if (response.status.code === 403) {
			alert('ACCESS DENIED: You are not authorized to update ' +
				employee.entity._links.self.href);
		}
		if (response.status.code === 412) {
			alert('DENIED: Unable to update ' + employee.entity._links.self.href +
				'. Your copy is stale.');
		}
	});
}
```

您有代码来捕获HTTP 412错误。这会捕获HTTP 403状态码并提供合适的警报。

对于删除操作也是这样做的：

```js
onDelete(employee) {
	client({method: 'DELETE', path: employee.entity._links.self.href}
	).done(response => {/* let the websocket handle updating the UI */},
	response => {
		if (response.status.code === 403) {
			alert('ACCESS DENIED: You are not authorized to delete ' +
				employee.entity._links.self.href);
		}
	});
}
```

这与编写的错误消息类似地编码。

### 添加一些安全细节到用户界面

这个版本的应用程序的最后一件事是显示谁登录以及提供一个注销按钮，通过将index.html中的新`<div>`包含在`react``<div>`之前：

```html
<div>
    Hello, <span th:text="${#authentication.name}">user</span>.
    <form th:action="@{/logout}" method="post">
        <input type="submit" value="Log Out"/>
    </form>
</div>
```

### 把它放在一起

通过前端的这些更改，重新启动应用程序并导航到[http：// localhost：8080](http://localhost:8080/)。

您立即重定向到登录表单。这个表格是由Spring Security提供的，尽管你可以[创建你自己的](https://spring.io/guides/gs/securing-web/)。以greg / turnquist登录。

![安全1](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/security/images/security-1.png)

您可以看到新添加的管理员列。浏览几页，直到找到**oliver**拥有的员工。

![安全2](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/security/images/security-2.png)

点击**更新**，进行一些更改，然后点击**更新**。它会失败，出现以下弹出窗口：

![安全3](https://github.com/spring-guides/tut-react-and-spring-data-rest/raw/master/security/images/security-3.png)

如果您尝试**删除**，则会失败，并显示类似的消息。创建一个新员工，它应该被分配给你。

### 回顾

在这个部分：

- 您定义了经理的模型，并通过一对多的关系将其与员工关联起来。
- 您为经理创建了一个存储库，并告诉Spring Data REST不导出。
- 您为empoyee存储库编写了一组访问规则，并编写了安全策略。
- 您编写了另一个Spring Data REST事件处理程序，以便在创建事件发生之前将其捕获，以便将当前用户分配为员工的经理。
- 您更新了UI以显示员工的经理，并在未经授权的情况下显示错误弹出窗口。

问题是什么？

网页变得相当复杂。但是管理关系和内联数据呢？创建/更新对话框并不适合于此。它可能需要一些自定义的书面形式。

经理可以访问员工数据。员工应该有机会吗？如果你要添加更多的细节，如电话号码和地址，你将如何建模？您如何授予员工访问系统的权限，以便他们可以更新这些特定的字段？是否有更多的超媒体控件可以方便地放在页面上？
