# 中文标题

> 原文：[Creating a Multi Module Project](https://spring.io/guides/gs/multi-module/)
>
> 译者：[panhoucheng](https://github.com/panhoucheng)
>
> 校对：


在此处编写正文，基本翻译规范：

* 正文标题按层次结构 从 \#\# 开始
* 代码片段\`\`\`之后需要写明语言类型
* 如有图片更静态资源保存在static目录下，每篇文章建立自己的目录存储
* 尊重原作、不修改、不删减内容
* 每篇文章翻译完成之后提交PR，并在翻译交流群中找校对人员完成review，最后由管理员完成Merge
* 若译者与校对有不同建议，可以将争议部分发到交流群中一起讨论确定结果


###panhoucheng翻译中

本指南将给你介绍如何使用Spring Boot创建一个多模块的项目，。

## 你会得到什么？


## 你需要准备什么？

大约15分钟时间

一个喜欢的文本编辑器或者IDE

[JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 更高版本

[Gradle 2.3+](http://www.gradle.org/downloads) 或 [Maven 3.0+](https://maven.apache.org/download.cgi)

你也可以直接导入代码到IDE:

[Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)

[IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)



## 怎样完成指南？

像大多数 Spring [入门指南](https://spring.io/guides)一样, 你可以从头开始，完成每一步, 或者你也可以绕过你熟悉的基本步骤再开始。 不管通过哪种方式，你最后都会得到一份可执行的代码。

**如果从基础开始**，你可以往下查看[怎样使用 Gradle 构建项目](#scratch)。


**如果已经熟悉跳过一些基本步骤**，你可以：

* [下载](https://github.com/spring-guides/draft-gs-multi-module.git)并解压源码库，或者通过 [Git](https://spring.io/understanding/Git)克隆：

 `git clone https://github.com/spring-guides/draft-gs-multi-module.git`
* 进入 `draft-gs-multi-module/initial`目录
* 跳过前面的部分[创建一个库项目](https://spring.io/guides/gs/multi-module/#initial) 

**当你完成之后**，你可以在`draft-gs-multi-module/complete`根据代码检查下结果。

首先你需要编写基础构建脚本。在构建 Spring 应用的时候，你可以使用任何你喜欢的系统来构建， 这里提供一份你可能需要用 [Gradle](http://gradle.org/) 或者 [Maven](https://maven.apache.org/) 构建的代码。 如果你两者都不是很熟悉, 你可以先去参考[如何使用 Gradle 构建 Java 项目](https://spring.io/guides/gs/gradle)或者[如何使用 Maven 构建 Java 项目](https://spring.io/guides/gs/maven)。

## 创建一个根项目
####创建目录结构




> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



