# 通过IntelliJ IDEA开始你的工作   

> 原文：[Working a Getting Started guide with IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
>
> 译者：[xiudongxu](github.com/xiudongxu)
>
> 校对：

本指南将仔细的一步步教会你使用IntelliJ IDEA去创建一个入门指南工程。

## 你会创建什么

你将挑选一个Spring的指南工程并将它导入到IntelliJ IDEA开发工具中。之后你就可以阅读这个指南工程，通过代码进行开发工作，最后将这个工程运行起来。

## 你需要准备什么

* 大约15分钟
* [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
* [JDK 6](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或者以上的版本

## 安装IntelliJ IDEA

如果你还没有安装IntelliJ IDEA工具的话，请查看上面的链接。从那个链接里面，你可以为你的平台下载一个副本。要安装它只需要解压缩下载的文件。

当你完成上面的步骤，那就打开IntelliJ IDEA吧。

## 导入一个入门指南

要导入现有的项目，你需要一些代码，因此克隆或者复制一份入门指南，例如，[REST Service](https://spring.io/guides/gs/rest-service/)指南：

```git
./gradlew build && java -jar build/libs/gs-spring-boot-0.1.0.jar
```

当IntelliJ IDEA启动和运行后，在**欢迎界面**点击**Import Project**，或者**File Open**在主菜单中：

![Import Project](https://spring.io/guides/gs/intellij-idea/images/spring_guide_welcome_import.png)

请确保在弹出的对话框中选择[Maven](https://spring.io/guides/gs/maven)的**pom.xml**或[Gradle](https://spring.io/guides/gs/gradle)的**build.gradle**文件在**完整**的文件夹路径下。

![Select File or Directory to Import](https://spring.io/guides/gs/intellij-idea/images/spring_guide_select_gradle_file.png)

IntelliJ IDEA将使用指南中的所有代码创建一个准备运行的项目。

## 从零开始创建一个项目

假如你喜欢通过一个空的工程或者粘贴复制的方式来开始的话，那么在**项目向导**界面创建一个**Maven**或者**Gradle**工程。

![Project Wizard](https://spring.io/guides/gs/intellij-idea/images/spring_guide_new_project.png)

## 你也可以看

下面的文档可能也是有帮助的：

* [通过STS开始一个入门指南](https://spring.io/guides/gs/sts/)

如果你想写一个新的教程或者对我们的的教程有任何建议，可以看我们的[贡献流程](https://github.com/SpringForAll/spring-guides-translation)

---



> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。