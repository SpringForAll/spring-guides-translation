# 使用 Vaadin 创建 CRUD UI


> 原文：[Creating CRUD UI with Vaadin](https://spring.io/guides/gs/crud-with-vaadin/)
>
> 译者：[chenzhijun](http://github.com/chenzhijun)
>
> 校对：

本指南会教你如何在Spring Data JPA的基础上使用基于[Vaadin](https://vaadin.com/)来构建应用的UI：

## 你会构建什么？

你会编写一个使用JPA的Vaadin UI。得到一个完整的具有CRUD(Create，Read，Update，Delete)功能的应用和一个使用自定义的仓储方法实现的过滤实例。

你可以从两种方式开始，可以从已有的"initial"项目开始，或者从一个新的实例开始。他们之间的不同点在下面会讨论。

## 开始之前你需要准备

大约15分钟时间

一个喜欢的文本编辑器或者IDE

[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

[Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

你也可以直接导入代码到IDE:

[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)


## 如何完成指南？  

像大多数 `Spring` [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#miaodian)。

**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/gs-crud-with-vaadin/archive/master.zip)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/gs-accessing-mongodb-data-rest.git`
* 进入 `gs-accessing-mongodb-data-rest/initial`目录
* 跳过前面的部分[创建后台服务](#https://spring.io/guides/gs/crud-with-vaadin/#initial)

**当你完成之后**，你可以在`gs-crud-with-vaadin/complete`根据代码检查下结果。

<h2 id="miaodian"> 使用Gradle构建 </h2>

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建， 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

### 创建以下目录结构

在你的项目根目录，创建如下的子目录结构; 例如，如果你使用的是\*nix系统，你可以使用`mkdir -p src/main/java/hello`:

```
└── src
    └── main
        └── java
            └── hello
```

### 创建Gradle构建文件

下面是一份[初始化Gradle构建文件](https://github.com/spring-guides/gs-crud-with-vaadin/blob/master/initial/build.gradle).

`build.gradle`

```gradle
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'gs-accessing-data-jpa'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
    maven { url "https://repository.jboss.org/nexus/content/repositories/releases" }
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
    testCompile("junit:junit")
}
```

[Spring Boot gradle 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin) 提供了很多非常方便的功能：

* 将 classpath 里面所有用到的 jar 包构建成一个可执行的 JAR 文件，使得运行和发布你的服务变得更加便捷。
* 搜索 `public static void main()` 方法并且将它标记为可执行类。
* 提供了将内部依赖的版本都去匹配 [Spring Boot 依赖](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml) 的版本。你可以根据你的需要来重写版本，但是它默认提供给了 Spring Boot 依赖的版本。

## 使用Maven构建

首先，你需要设置一个基本的构建脚本。当使用 Spring 构建应用程序时，你可以使用任何你喜欢的构建系统，但是使用 [Maven](https://maven.apache.org/) 构建的代码如下所示。如果您不熟悉Maven，请参阅[使用Maven构建Java项目](https://spring.io/guides/gs/maven)。

### 创建目录结构

在你选择的项目目录中，创建以下子目录结构;例如, 在Linux/Unix系统中使用如下命令: `mkdir -p src/main/java/hello`

```
└── src
    └── main
        └── java
            └── hello
```

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-crud-with-vaadin</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

[Spring Boot Maven 插件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-maven-plugin) 提供了很多便捷的特性:

- 它收集类路径上的所有jar包，并构建一个可运行的jar包，这样可以更方便地执行和发布你的服务。
- 它寻找`public static void main()` 方法来将其标记为一个可执行的类。
- 它提供了一个内置的依赖解析器将应用与Spring Boot依赖的版本号进行匹配。你可以修改成任意的版本，但它将默认为 Boot所选择了一组版本。  

## 使用你的IDE构建

- 阅读如何将本指南直接导入 [Spring Tool Suite](https://spring.io/guides/gs/sts/)。
- 阅读如何使用 [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea) 来构建。

## 创建后台服务

本示例是 [Accessing Data with JPA](https://spring.io/gs/accessing-data-jpa)的扩展部分。唯一的不同是entity类有get/set方法，并且对于终端用户来说，可以更加优雅的在repository中自定义搜索方法。你并不需要先去阅读Access Data with JPA就可以完成本示例。

如果你是用一个全新的项目开始本示例，请将下面的entity和repository类加入到项目中。这样的目的是为了方便你接下来更好的完成。如果你的项目是从"初始化"开始的，这些类就都已经在项目中了。

`src/main/java/hello/Customer.java`

```java
package hello;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Customer {

	@Id
	@GeneratedValue
	private Long id;

	private String firstName;

	private String lastName;

	protected Customer() {
	}

	public Customer(String firstName, String lastName) {
		this.firstName = firstName;
		this.lastName = lastName;
	}

	public Long getId() {
		return id;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	@Override
	public String toString() {
		return String.format("Customer[id=%d, firstName='%s', lastName='%s']", id,
				firstName, lastName);
	}

}
```

`src/main/java/hello/CustomerRepository.java`

```java
package hello;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface CustomerRepository extends JpaRepository<Customer, Long> {

	List<Customer> findByLastNameStartsWithIgnoreCase(String lastName);
}
```

你可以保留整个Spring Boot应用，然后使用一些初始数据来填充数据库。

`src/main/java/hello/Application.java`

```java
package hello;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

	private static final Logger log = LoggerFactory.getLogger(Application.class);

	public static void main(String[] args) {
		SpringApplication.run(Application.class);
	}

	@Bean
	public CommandLineRunner loadData(CustomerRepository repository) {
		return (args) -> {
			// save a couple of customers
			repository.save(new Customer("Jack", "Bauer"));
			repository.save(new Customer("Chloe", "O'Brian"));
			repository.save(new Customer("Kim", "Bauer"));
			repository.save(new Customer("David", "Palmer"));
			repository.save(new Customer("Michelle", "Dessler"));

			// fetch all customers
			log.info("Customers found with findAll():");
			log.info("-------------------------------");
			for (Customer customer : repository.findAll()) {
				log.info(customer.toString());
			}
			log.info("");

			// fetch an individual customer by ID
			Customer customer = repository.findOne(1L);
			log.info("Customer found with findOne(1L):");
			log.info("--------------------------------");
			log.info(customer.toString());
			log.info("");

			// fetch customers by last name
			log.info("Customer found with findByLastNameStartsWithIgnoreCase('Bauer'):");
			log.info("--------------------------------------------");
			for (Customer bauer : repository
					.findByLastNameStartsWithIgnoreCase("Bauer")) {
				log.info(bauer.toString());
			}
			log.info("");
		};
	}

}
```

## Vaadin 依赖

如果你是从“initial”创建的项目，你已经将需要的依赖添加到项目里了。但如果你从一个全新的Spring项目中，你需要添加Vasdin依赖。Vaadin的Spring集成包中有Spring boot starter，你所需要做的，就是将它加入到Maven的pom文件中，或者在Gradle中做类似的配置：

```xml
<dependency>
    <groupId>com.vaadin</groupId>
    <artifactId>vaadin-spring-boot-starter</artifactId>
</dependency>
```

本示例使用的是较新版本的Vaadin，比starter默认模块中的版本要新一些。如果要使用新的版本，你可以定义Vaadin的一些参数，像下面这样：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.vaadin</groupId>
            <artifactId>vaadin-bom</artifactId>
            <version>8.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Gradle 默认不支持"BOMs"，但是这里有一个非常遍历的[插件](https://plugins.gradle.org/plugin/io.spring.dependency-management)。下载完整的[build.gradle](https://github.com/spring-guides/gs-crud-with-vaadin/blob/master/complete/build.gradle)。

## 定义 UI 类

UI 类是 Vaadin UI逻辑入口点。在 Spring Boot应用中，你仅仅需要使用`@SpringUI`注解，然后Spring就会自动注入它。一个简单的"hello world"示例就像下面这样：

```java

package hello;

import com.vaadin.annotations.Theme;
import com.vaadin.server.VaadinRequest;
import com.vaadin.spring.annotation.SpringUI;
import com.vaadin.ui.Button;
import com.vaadin.ui.Notification;
import com.vaadin.ui.UI;

@SpringUI
@Theme("valo")
public class VaadinUI extends UI {

    @Override
    protected void init(VaadinRequest request) {
	    setContent(new Button("Click me", e -> Notification.show("Hello Spring+Vaadin user!")));
    }
}

```

## 数据展示

使用一个好的布局，如使用`Grid`组件。下面的实体`CustomerRepository`是从构造函数注入的,它是用[BeanItemContainer](https://vaadin.com/book/-/page/datamodel.container.html)包装起来，[BeanItemContainer](https://vaadin.com/book/-/page/datamodel.container.html)为Grid组建提供数据。`VaadinUI`的主体部分展开就像下面这样：

```java
CustomerRepository repo;
Grid<Customer> grid;

@Autowired
public VaadinUI(CustomerRepository repo) {
    this.repo = repo;
    this.grid = new Grid<>(Customer.class);
}

@Override
protected void init(VaadinRequest request) {
    setContent(grid);
    listCustomers();
}

private void listCustomers() {
    grid.setItems(repo.findAll());
}
```

> 如果你有大量的表或者大量的同步用户，你不应该绑定整个数据集到你的UI组件。

尽管许多Vaadin组件从服务器到浏览器的过程是懒加载的，上面的方案是将整个数据列表存储在服务器内存中。为了节省一些内存，你应该仅仅展示最重要的数据，然后使用分页或者提供懒加载数据来展示其它数据。你可以使用`setDataProvider(FetchItemsCallback<T>, SerializableSupplier<Integer>)`方法。

## 数据过滤

在大量的数据集成为你服务器的一个问题前，它造成了一个对想编辑相应数据的人头疼的问题。使用`TextField`组件来创建一个过滤入口。首先，修改`listCustomer()`方法让它支持过滤：

```java
void listCustomers(String filterText) {
	if (StringUtils.isEmpty(filterText)) {
		grid.setItems(repo.findAll());
	}
	else {
		grid.setItems(repo.findByLastNameStartsWithIgnoreCase(filterText));
	}
}
```

> 这是Spring Data的声明查询的便利方法。在`CustomerRepository`定义一个新方法`findByLastNameStartsWithIgnoringCase`

给`TextField`组建添加一个钩子，并且将它的值插入到该过滤器方法中。当文本被改变时，`TextChangeListener`会被延迟调用：

```java

TextField filter = new TextField();
filter.setPlaceholder("Filter by last name");
filter.setValueChangeMode(ValueChangeMode.LAZY);
filter.addValueChangeListener(e -> listCustomers(e.getValue()));
VerticalLayout mainLayout = new VerticalLayout(filter, grid);
setContent(mainLayout);

```

## 定义编辑组建

因为Vaadin UI都是一些平常的Java代码，没有理由不从一开始就写一些可重复使用的代码。为你的Customer实体定一个编辑组件。你需要将它定义成一个让Spring能够管理的Bean，然后你可以直接将`CustomerRepostiry`注入到编辑器中，构成C，U和D的部分，或者我们的CRUD功能设计。

`src/main/java/hello/CustomerEditor.java`

```java

package hello;

import org.springframework.beans.factory.annotation.Autowired;

import com.vaadin.data.Binder;
import com.vaadin.event.ShortcutAction;
import com.vaadin.server.FontAwesome;
import com.vaadin.spring.annotation.SpringComponent;
import com.vaadin.spring.annotation.UIScope;
import com.vaadin.ui.Button;
import com.vaadin.ui.CssLayout;
import com.vaadin.ui.TextField;
import com.vaadin.ui.VerticalLayout;
import com.vaadin.ui.themes.ValoTheme;

/**
 * A simple example to introduce building forms. As your real application is probably much
 * more complicated than this example, you could re-use this form in multiple places. This
 * example component is only used in VaadinUI.
 * <p>
 * In a real world application you'll most likely using a common super class for all your
 * forms - less code, better UX. See e.g. AbstractForm in Viritin
 * (https://vaadin.com/addon/viritin).
 */
@SpringComponent
@UIScope
public class CustomerEditor extends VerticalLayout {

	private final CustomerRepository repository;

	/**
	 * The currently edited customer
	 */
	private Customer customer;

	/* Fields to edit properties in Customer entity */
	TextField firstName = new TextField("First name");
	TextField lastName = new TextField("Last name");

	/* Action buttons */
	Button save = new Button("Save", FontAwesome.SAVE);
	Button cancel = new Button("Cancel");
	Button delete = new Button("Delete", FontAwesome.TRASH_O);
	CssLayout actions = new CssLayout(save, cancel, delete);

	Binder<Customer> binder = new Binder<>(Customer.class);

	@Autowired
	public CustomerEditor(CustomerRepository repository) {
		this.repository = repository;

		addComponents(firstName, lastName, actions);

		// bind using naming convention
		binder.bindInstanceFields(this);

		// Configure and style components
		setSpacing(true);
		actions.setStyleName(ValoTheme.LAYOUT_COMPONENT_GROUP);
		save.setStyleName(ValoTheme.BUTTON_PRIMARY);
		save.setClickShortcut(ShortcutAction.KeyCode.ENTER);

		// wire action buttons to save, delete and reset
		save.addClickListener(e -> repository.save(customer));
		delete.addClickListener(e -> repository.delete(customer));
		cancel.addClickListener(e -> editCustomer(customer));
		setVisible(false);
	}

	public interface ChangeHandler {

		void onChange();
	}

	public final void editCustomer(Customer c) {
		if (c == null) {
			setVisible(false);
			return;
		}
		final boolean persisted = c.getId() != null;
		if (persisted) {
			// Find fresh entity for editing
			customer = repository.findOne(c.getId());
		}
		else {
			customer = c;
		}
		cancel.setVisible(persisted);

		// Bind customer properties to similarly named fields
		// Could also use annotation or "manual binding" or programmatically
		// moving values from fields to entities before saving
		binder.setBean(customer);

		setVisible(true);

		// A hack to ensure the whole form is visible
		save.focus();
		// Select all text in firstName field automatically
		firstName.selectAll();
	}

	public void setChangeHandler(ChangeHandler h) {
		// ChangeHandler is notified when either save or delete
		// is clicked
		save.addClickListener(e -> h.onChange());
		delete.addClickListener(e -> h.onChange());
	}

}
```

在一个大应用中你可以在多个地方使用这个编辑器组件。当然主要提出的是，在一些大的应用中，你应该提供一些通用的格式比如MVP来使你的UI代码具有结构性(当然这已经超出本教程的范围了)。

## 完整示例

在先前的步骤中，你已经知道了一些基础的组建该如果编写代码。使用`Button`并且监听`Grid`，你可以完全的集成我们的编辑器到主UI中，最终版本的`VaadinUi`类像下面这样：

`src/main/java/hello/VaadinUI.java`

```java
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.StringUtils;

import com.vaadin.server.FontAwesome;
import com.vaadin.server.VaadinRequest;
import com.vaadin.shared.ui.ValueChangeMode;
import com.vaadin.spring.annotation.SpringUI;
import com.vaadin.ui.Button;
import com.vaadin.ui.Grid;
import com.vaadin.ui.HorizontalLayout;
import com.vaadin.ui.TextField;
import com.vaadin.ui.UI;
import com.vaadin.ui.VerticalLayout;

@SpringUI
public class VaadinUI extends UI {

	private final CustomerRepository repo;

	private final CustomerEditor editor;

	final Grid<Customer> grid;

	final TextField filter;

	private final Button addNewBtn;

	@Autowired
	public VaadinUI(CustomerRepository repo, CustomerEditor editor) {
		this.repo = repo;
		this.editor = editor;
		this.grid = new Grid<>(Customer.class);
		this.filter = new TextField();
		this.addNewBtn = new Button("New customer", FontAwesome.PLUS);
	}

	@Override
	protected void init(VaadinRequest request) {
		// build layout
		HorizontalLayout actions = new HorizontalLayout(filter, addNewBtn);
		VerticalLayout mainLayout = new VerticalLayout(actions, grid, editor);
		setContent(mainLayout);

		grid.setHeight(300, Unit.PIXELS);
		grid.setColumns("id", "firstName", "lastName");

		filter.setPlaceholder("Filter by last name");

		// Hook logic to components

		// Replace listing with filtered content when user changes filter
		filter.setValueChangeMode(ValueChangeMode.LAZY);
		filter.addValueChangeListener(e -> listCustomers(e.getValue()));

		// Connect selected Customer to editor or hide if none is selected
		grid.asSingleSelect().addValueChangeListener(e -> {
			editor.editCustomer(e.getValue());
		});

		// Instantiate and edit new Customer the new button is clicked
		addNewBtn.addClickListener(e -> editor.editCustomer(new Customer("", "")));

		// Listen changes made by the editor, refresh data from backend
		editor.setChangeHandler(() -> {
			editor.setVisible(false);
			listCustomers(filter.getValue());
		});

		// Initialize listing
		listCustomers(null);
	}

	// tag::listCustomers[]
	void listCustomers(String filterText) {
		if (StringUtils.isEmpty(filterText)) {
			grid.setItems(repo.findAll());
		}
		else {
			grid.setItems(repo.findByLastNameStartsWithIgnoreCase(filterText));
		}
	}
	// end::listCustomers[]

}
```

## 总结

恭喜，你已经编写了一个完整的具有CURD功能的UI应用并且它使用Spring Data来持久化。你完成了它并且不需要暴露任何REST服务，也不需要编写一行JavaScript或者HTML代码。

## 了解更多

下面的指南可能对你有用:

- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
- [Accessing Data with JPA](https://spring.io/guides/gs/accessing-data-jpa/)
- [Accessing Data with MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/)
- [Accessing Data with GemFire](https://spring.io/guides/gs/accessing-data-gemfire/)
- [Accessing Data with Neo4j](https://spring.io/guides/gs/accessing-data-neo4j/)
- [Accessing data with MySQL](https://spring.io/guides/gs/accessing-data-mysql/)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。