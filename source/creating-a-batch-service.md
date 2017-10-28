# 创建批量服务

> 原文：[Creating a Batch Service](https://spring.io/guides/gs/batch-processing/)
>
> 译者：[Mr.lzc](https://github.com/cleverlzc)
>
> 校对：

本指南将引导您完成创建基本的批量驱动解决方案的过程。

## 你将构建什么

您将构建一个从CSV电子表格导入数据的服务，并使用自定义代码进行转换，并将最终结果存储在数据库中。

## 你需要准备什么

- 大约15分钟
- 一个自己喜欢的文本编辑器或者IDE
- [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或以上版本
- [Gradle 2.3+](https://gradle.org/install/) 或者 [Maven 3.0+](https://maven.apache.org/download.cgi)
- 你也可以直接将代码导入到本地的IDE中：
  - [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  - [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本指南

像大多数[Spring入门指南](https://spring.io/guides)一样，你可以从头开始，完成每一步，也可以绕过已经熟悉的基本设置步骤。不管采用哪种方式，最终都可以得到能够运行的代码。

如果是**从零开始**，则可以从[使用Gradle构建项目](#使用gradle构建项目)小节开始。

如果想**跳过基本的设置步骤**，可以按照以下步骤执行：

- [下载](https://github.com/spring-guides/gs-batch-processing/archive/master.zip) 并解压本指南相关的源文件，或者直接通过[Git](https://spring.io/understanding/Git)命令克隆到本地：

```bash
git clone https://github.com/spring-guides/gs-batch-processing.git
```

- 进入到 `gs-batch-processing/initial` 目录
- 直接跳到[创建业务类](#创建业务类)小节。

**当完成所有的编码以后**，可以将你的代码与`gs-batch-processing/complete`目录下的示例代码进行对比，以检查结果是否正确。

## 使用Gradle构建项目

首先需要设置一个基本的构建脚本。


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



