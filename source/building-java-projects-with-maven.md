# 基于Maven来构建Java项目

原文：[Building Java Projects with Maven](https://spring.io/guides/gs/maven/)

译者： liqiangatongoingdotme      

校对：william-hyx 



这个教程可以教会你通过Maven来构建一个简单的Java项目


###你会收获什么？
你可以在短时间内用Maven构建你的应用程序

###你需要准备什么？
  
  * 15分钟
  * 一个顺手的文本编辑器或者IDE
  * [JDK6](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或者更高版本 
  
###如何学习本教程？

像大多数 Spring[ Getting Started 手册]()一样，你可以从头开始，完成每一步，或者你可以绕过已经熟悉的基本设置步骤。 

从头开始，移步[建立项目](#Setuptheproject)

跳过基础步骤，可以直接执行以下操作：

* [下载](https://github.com/spring-guides/gs-maven/archive/master.zip)和解压本教程的源代码，或者使用[Git](https://spring.io/understanding/Git)的clone命令 
* 进入 gs-maven/initial 目录
* 提前跳转到[初始化]()

当你完成后，可以将你的结果与 gs-maven/complete中的代码进行对比。

###建立项目
<span id="Setuptheproject"> </span>


首先，你需要新建一个Java项目用于构建后续的Maven工程。为了着重展示Maven的使用，尽可能新建一个简单的java项目。

创建目录结构

在你选择的项目文件夹中，新建如下目录结构。 
若在*nix系统中，可以使用mkdir -p src/main/java/hello命令创建该目录结构。

~~~
└── src
    └── main
        └── java
            └── hello
~~~

在这个目录下 src/main/java/hello，你可以创建你想要的任何Java类。为了保持与本教程其余部分的一致性，创建这两个类: HelloWorld.java 和  Greeter.java.

>src/main/java/hello/HelloWorld.java
 
~~~
package hello;

public class HelloWorld {
    public static void main(String[] args) {
        Greeter greeter = new Greeter();
        System.out.println(greeter.sayHello());
    }
}
~~~

>src/main/java/hello/Greeter.java

~~~
package hello;

public class Greeter {
    public String sayHello() {
        return "Hello world!";
    }
}
~~~

现在用于构建Maven工程的Java项目已经准备完毕，下一步需要你安装Maven。

通过http://maven.apache.org/download.cgi 可以下载到zip格式的Maven文件。

下载了zip文件后，将其解压缩到您的计算机上。然后将bin文件夹添加到环境变量path中。

测试Maven安装是否正确，从命令行运行mvn命令

~~~
mvn -v
~~~

如果配置无误，你应该看到一些关于Maven的安装信息。看起来与以下内容类似(尽管可能略有不同)：

~~~
Apache Maven 3.0.5 (r01de14724cdef164cd33c7c8c2fe155faf9602da; 2013-02-19 07:51:28-0600)
Maven home: /usr/share/maven
Java version: 1.7.0_09, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.7.0_09.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.3", arch: "x86_64", family: "mac"
~~~

到此，Maven配置成功。


###配置Maven

现在Maven已经安装，你需要创建一个Maven项目的定义。Maven项目通过pom.xml文件进行定义，在文件中定义了项目的名称 版本和依赖包，以及一些扩展类库。

在项目的根目录下创建pom.xml文件（与src文件夹同一级目录）

`pom.xml`

~~~

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-maven</artifactId>
    <packaging>jar</packaging>
    <version>0.1.0</version>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>hello.HelloWorld</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>


~~~

除了可选的 `<packaging>`元素之外，这是构建Java项目所需最简单的pom.xml文件。构建Java项目所需的xml文件。它包括项目配置的以下细节:

* `<modelVersion>`  pom model 版本 通常都是4.0.0
*  `<groupId>`  项目所属的组织或机构。通常用一个反向域名来表示。
* ` <artifactId>`  项目的artifact名称(例如，它的JAR或WAR文件的名称)。
*  `<version>` 构建的项目版本
*  `<packaging> `  工程的打包方式。默认为“jar”文件。也可以用“war”打包成war文件

至此，你创建了一个最小可运行的Maven工程。

>在选择版本命名规范的时候，Spring推荐 [semantic versioning](http://semver.org) 
>
>

###Build Java代码

Maven现在已经可以构建项目了。现在可以使用Maven执行几个构建生命周期的命令，包括编译project的代码、创建library package(比如JAR文件)，并添加到本地库中

如果要用maven编译，可以运行下面的代码：

>mvn compile

这是Maven执行编译命令。当编译完成时，你可以在target/classes目录下找相应的class文件

你应该不太可能会直接部署.class文件，你应该运行打包命令：

>mvn package

package命令将会编译你的Java代码，运行所有测试类，最后在target目录下打包生产JAR文件。JAR文件的命名会基于项目的 `<artifactId>` 和  `<version>` 。用前面提到的pom.xml文件打包会得到 gs-maven-0.1.0.jar

>若pom.xml文件中元素从"jar"改成"war",target中JAR文件将变成WAR文件。

Maven还在本机上维护一个依赖库（通常在你的用户目录下的.m2/repository目录下），你可以快速访问依赖的项目。如果你想要将project的JAR文件安装到本地仓库，你应该试用`install` 命令

>mvn install

install命令将会编译，测试和打包你的项目代码，然后copy到本地仓库，以便被其余的项目依赖。

讲到依赖，现在是时候声明Maven中的依赖了。


###声明依赖
简单的Hello World程序是完全自包含，不依赖于任何类库。但对于大多数应用来说，或多或少的依赖于外部的类库来处理通用的和复杂的功能。

举例，假设除了说“Hello World” ，你还想应用程序打印当前日期和事件。你可以使用Java中的日期和时间库，但可以使用Joda时间库，这样更有趣。

首先，改变HelloWorld.java 如下：

`src/main/java/hello/HelloWorld.java`

~~~
package hello;

import org.joda.time.LocalTime;

public class HelloWorld {
	public static void main(String[] args) {
		LocalTime currentTime = new LocalTime();
		System.out.println("The current local time is: " + currentTime);
		Greeter greeter = new Greeter();
		System.out.println(greeter.sayHello());
	}
}

~~~

这个HelloWorld 使用 Joda类库中的 `LocalTime` 类来打印当前时间。

假如你现在运行 `mvn compile` 编译project ，会编译失败。因为你没有定义Joda
时间库作为依赖。你可以增加下面的配置在pom.xml中修复这个问题（在`<project>`元素里）：

~~~
<dependencies>
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.2</version>
		</dependency>
</dependencies>

~~~
这块XML声明了项目的依赖列表。具体的来说，他声明了对Joda时间库的单一依赖。在`<dependency>` 元素中，依赖项由三要素组成：

* `<groupId>`  这个依赖属于的组织或机构
* `<artifactId>` 该依赖的名称
* `<version>` 必须指定的依赖库的版本号

默认的所有的依赖作用域全部为 `compile`依赖。在编译时有效。（如果是编译WAR文件，包含/WEB-INF/libs文件夹下的JAR）此外，您可以指定一个`<scope>`元素来指定下列范围之一:

* `provided` 编译项目代码所需的依赖项，但在运行时由运行代码的容器提供(比如Java的Servlet Api)
* `test`  用于编译和运行测试的依赖项，但不需要构建或运行项目的运行时代码。

现在你可以使用 `mvn compile` 或者 `mvn package`  Maven已经可以从中央库解析Joda时间库依赖，并成功构建。



###编写测试用例
首先增加Junit依赖在pom.xml文件中，test作用域：

~~~
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
~~~

新建测试用例代码如下：

`src/test/java/hello/GreeterTest.java` 

~~~
package hello;

import static org.hamcrest.CoreMatchers.containsString;
import static org.junit.Assert.*;

import org.junit.Test;

public class GreeterTest {

	private Greeter greeter = new Greeter();

	@Test
	public void greeterSaysHello() {
		assertThat(greeter.sayHello(), containsString("Hello"));
	}

}

~~~

Maven使用叫做“surefire”的插件运行测试用例。这个插件默认会把`/src/test/java` 下的名字匹配`*Test`的类编译运行。你可以运行如下命令运行所有测试用例：

~~~
mvn test
~~~

或者只运行前述所示的mvn install命令（maven的“install”生命周期中已包含了“test”）

下面是完整的pom.xml文件：

~~~
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.springframework</groupId>
	<artifactId>gs-maven</artifactId>
	<packaging>jar</packaging>
	<version>0.1.0</version>

	<dependencies>
		<!-- tag::joda[] -->
		<dependency>
			<groupId>joda-time</groupId>
			<artifactId>joda-time</artifactId>
			<version>2.9.2</version>
		</dependency>
		<!-- end::joda[] -->
		<!-- tag::junit[] -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<!-- end::junit[] -->
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
				<version>2.1</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
							<transformers>
								<transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
									<mainClass>hello.HelloWorld</mainClass>
								</transformer>
							</transformers>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
~~~

>上面的pom.xml文件使用[Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/) 制作简单的可执行JAR文件。本教程重点是在Maven，不使用独特的插件。


###概要

你已经可以创建一个简单而有效的maven项目来构建java工程。


### 参考

下面的文章可能对你有帮助：

* [Building Java Projects with Gradle](https://spring.io/guides/gs/gradle/)

想写一个新的教程或者捐献已经存在的教程？请检出我们的[contribution guidelines.]()

>所有的手册发布遵循 ASLv2 license，遵循[Attribution, NoDerivatives creative commons license](https://creativecommons.org/licenses/by-nd/3.0/) 进行编写。

