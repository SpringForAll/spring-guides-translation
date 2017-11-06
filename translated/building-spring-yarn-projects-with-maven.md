# Maven构建Spring YARN 项目

> 原文：[Building Spring YARN Projects with Maven](https://spring.io/guides/gs/maven-yarn/)
>
> 译者：[UniKrau](https://github.com/UniKrau)
>
> 校对: [dyc87112](https://github.com/dyc87112)


学会用`Maven`构建一个简单的Spring YARN 工程

## 你将学到点什么

创建一个简单的app然后用`Maven`构建之

这篇文章不准备创建一个完整可用的YARN app，而是关注项目和构建模型

## 需要你准备些什么
*  差不多15分钟搞定
*  文本编辑器或者IDE都行
*  支持JDK6以上

## 怎样完成指南

像大多数[Spring入门](https://spring.io/guides)文章一样，即新手按部就班学习或者如果有基础可以跳过这些基本步骤，不过最后，程序是可以跑的.

**如果从基础开始** ，请移步[配置项目](#setup_with_maven)


**如果已经熟悉跳过一些基本步骤** 你可以这样：
 可以通过[下载](https://github.com/spring-guides/gs-maven-yarn/archive/master.zip)并解压，或者使用[Git](https://spring.io/understanding/Git)的命令拷贝这个项目
git clone  [`https://github.com/spring-guides/gs-maven-yarn.git`](https://github.com/spring-guides/gs-maven-yarn.git)
* 使用cd 命令跳到 `gs-maven-yarn/initial`目录

* 跳转到 [通过Spring YARN项目理解Maven的使用规则](#spring_yarn_id)

**当以上步骤完成**，可以到`gs-maven-yarn/complete`目录检查代码


如果不熟悉gradle工具，可以参考[ Building Java Projects with Maven](https://spring.io/guides/gs/maven)

<h2 id="setup_with_maven">配置项目</h2>

首先用Maven构建一个java 工程，当然这个项目尽可能简单，因为重点是在Maven上

### 创建工程目录结构

选定工程文件夹以后，建立子文件夹：

```
├── gs-maven-yarn-appmaster
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── appmaster
├── gs-maven-yarn-container
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── container
├── gs-maven-yarn-client
│   └── src
│       └── main
│           ├── resources
│           └── java
│               └── hello
│                   └── client
└── gs-maven-yarn-dist
```

举个例子，在`Unix`或`Linux`系统下，使用`mkdir -p` 命令创建以下目录

```bash
mkdir -p gs-maven-yarn-appmaster/src/main/resources
mkdir -p gs-maven-yarn-appmaster/src/main/java/hello/appmaster
mkdir -p gs-maven-yarn-container/src/main/resources
mkdir -p gs-maven-yarn-container/src/main/java/hello/container
mkdir -p gs-maven-yarn-client/src/main/resources
mkdir -p gs-maven-yarn-client/src/main/java/hello/client
mkdir  gs-maven-yarn-dist
```

创建一个`ContainerApplication` 类

`gs-maven-yarn-container/src/main/java/hello/container/ContainerApplication.java`

```java
package hello.container;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class ContainerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ContainerApplication.class, args);
	}

}
```

创建一个 `AppmasterApplication` 类

`gs-maven-yarn-appmaster/src/main/java/hello/appmaster/AppmasterApplication.java`

```java
package hello.appmaster;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class AppmasterApplication {

	public static void main(String[] args) {
		SpringApplication.run(AppmasterApplication.class, args);
	}

}
```

创建一个 `ClientApplication` 类


`gs-maven-yarn-client/src/main/java/hello/client/ClientApplication.java`

```java
package hello.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class ClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(ClientApplication.class, args);
	}

}
```

创建一个application的yaml[什么是yaml](https://www.google.co.th/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwi9tfm73OXWAhVMOo8KHSGpBLcQFggmMAE&url=https%3A%2F%2Flearn.getgrav.org%2Fadvanced%2Fyaml&usg=AOvVaw0uo63vWO2fwfqu40VHZFUZ)配置文件供子项目使用

`gs-maven-yarn-container/src/main/resources/application.yml`

```yaml
spring:
    yarn:
        appName: gs-yarn-maven
```

<h4 id="spring_yarn_id">通过Spring YARN项目理解Maven的使用规则</h4>

父级目录的pom.xml文件结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-maven-yarn</artifactId>
    <version>0.1.0</version>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.3.RELEASE</version>
    </parent>

    <modules>
        <module>gs-maven-yarn-container</module>
        <module>gs-maven-yarn-appmaster</module>
        <module>gs-maven-yarn-client</module>
        <module>gs-maven-yarn-dist</module>
    </modules>

    <dependencies>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-yarn-boot</artifactId>
            <version>2.4.0.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <descriptors>
                        <descriptor>assembly.xml</descriptor>
                    </descriptors>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-release</id>
            <url>http://repo.spring.io/libs-release</url>
            <snapshots><enabled>false</enabled></snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-release</id>
            <url>http://repo.spring.io/libs-release</url>
            <snapshots><enabled>false</enabled></snapshots>
        </pluginRepository>
    </pluginRepositories>

</project>
```

创建了appmaster，container，client 三个独立子项目。另外，构建一个完整的项目的时候还要创建dist项目用来集成所有的artifacts，以便管理其子项目

appmaster项目的pom.xml文件

`gs-maven-yarn-appmaster/pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-maven-yarn-appmaster</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework</groupId>
        <artifactId>gs-maven-yarn</artifactId>
        <version>0.1.0</version>
    </parent>

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

container子项目的pom.xml文件

`gs-maven-yarn-container/pom.xml` 

pom.xml文件路径

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-maven-yarn-container</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework</groupId>
        <artifactId>gs-maven-yarn</artifactId>
        <version>0.1.0</version>
    </parent>

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

client子项目的pom.xml文件

`gs-maven-yarn-client/pom.xml` 

pom.xml文件路径

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-maven-yarn-client</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework</groupId>
        <artifactId>gs-maven-yarn</artifactId>
        <version>0.1.0</version>
    </parent>

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


父目录pom.xml文件

`gs-maven-yarn-dist/pom.xml` 

pom.xml文件路径

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-maven-yarn-dist</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework</groupId>
        <artifactId>gs-maven-yarn</artifactId>
        <version>0.1.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>gs-maven-yarn-client</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>gs-maven-yarn-appmaster</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>gs-maven-yarn-container</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-yarn-boot-test</artifactId>
            <version>2.4.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-library</artifactId>
            <version>1.2.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <executions>
                    <execution>
                        <id>distro-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <finalName>${project.name}</finalName>
                            <appendAssemblyId>false</appendAssemblyId>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <artifactId>maven-failsafe-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```



集成配置文件

`gs-maven-yarn-dist/assembly.xml`

集成配置文件路径

```yaml
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">

    <id>dist</id>
    <formats>
        <format>dir</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <moduleSets>
        <moduleSet>
            <useAllReactorProjects>true</useAllReactorProjects>
            <includes>
                <include>org.springframework:gs-maven-yarn-client</include>
                <include>org.springframework:gs-maven-yarn-appmaster</include>
                <include>org.springframework:gs-maven-yarn-container</include>
            </includes>
            <binaries>
                <unpack>false</unpack>
            </binaries>
        </moduleSet>
    </moduleSets>
</assembly>
```

### 解决Hadoop依赖

通过`Maven POM`项目形式Spring可以很好解决Apache Hadoop的依赖问题。默认使用的是 Apache Hadoop 2.6.x 但是也可以使用其他发行商的Hadoop比如 Pivotal HD, Hortonworks Data Platform or Cloudera CDH。指定好版本，Maven 会自动找到正确的Hadoop发行版本的依赖关系。例如以下案例

` Apache Hadoop 2.6.x`
````
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-yarn-boot</artifactId>
        <version>2.1.0.RELEASE</version>
    </dependency>

 </dependencies>
````
 `Pivotal HD 2.1`
````
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-yarn-boot</artifactId>
        <version>2.1.0.RELEASE-phd21</version>
    </dependency>
</dependencies>
````
`Hortonworks Data Platform 2.2` 

````
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-yarn-boot</artifactId>
        <version>2.1.0.RELEASE-hdp22</version>
    </dependency>
</dependencies>
````

 `Cloudera CDH 5.x `
 
````
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-yarn-boot</artifactId>
        <version>2.1.0.RELEASE-cdh5</version>
    </dependency>
</dependencies>
````
 `Apache Hadoop 2.4.x`
````
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-yarn-boot</artifactId>
        <version>2.1.0.RELEASE-hadoop24</version>
    </dependency>
</dependencies>
````
`Apache Hadoop 2.5.x`
```
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-yarn-boot</artifactId>
        <version>2.1.0.RELEASE-hadoop25</version>
    </dependency>
</dependencies>
```
### 编译应用程序包

运行编译命令:

```bash
mvn clean package
```

编译成功后会产生三个jar包

```groovy
gs-maven-yarn-dist/target/gs-maven-yarn-dist/gs-maven-yarn-client-0.1.0.jar
gs-maven-yarn-dist/target/gs-maven-yarn-dist/gs-maven-yarn-container-0.1.0.jar
gs-maven-yarn-dist/target/gs-maven-yarn-dist/gs-maven-yarn-appmaster-0.1.0.jar
```

运行项目

```bash
java -jar gs-maven-yarn-dist/target/dist/gs-maven-yarn-client-0.1.0.jar
```
但是事实上还不是一个可以提交的YARN应用程序，所以会有错误信息。因此要参考Spring YARN 入门的文章，构建一个完整可以提交到YARN上的应用程序

## 总结

恭喜了，现在可以通过高效的Maven构建一个简单Spring YARN工程了


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



