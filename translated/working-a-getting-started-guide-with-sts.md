# 使用STS的入门指南  
> 原文：[Working a Getting Started guide with STS](https://spring.io/guides/gs/sts/)
>
> 译者：[hanbin](https://github.com/hanbin)
>
> 校对：[Mr.lzc](https://github.com/cleverlzc)

这个指南引导您使用 Spring Tool Suite (STS) 去构建入门程序。

## 你将构建什么

您将选择一个Spring入门程序并将其导入到Spring Tool Suite中。 接下来，您可以阅读指南，然后编写代码并运行项目。

## 你需要什么

- 大概15分钟
- [Spring Tool Suite (STS)](https://spring.io/tools/sts/all)
- [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或者更高版本

## 安装STS

如果您尚未安装STS，请访问上面的链接。 从那里，您可以下载您的平台的副本。 只需解压缩下载的存档就可以完成安装。

安装完成后，打开STS。

## 导入入门工程

启动STS后, 从File菜单打开 **Import Spring Getting Started Content** 向导框。

![Import Getting Started Content](static/1033/1_open_wizard.png)

会有一个弹出向导提供从Spring网站搜索并选择已经发布的入门指南。 您可以点击列表，也可以输入搜索字词，以便即时过滤选项。

> 进行搜索时，标题和描述都支持通配符，并且会实时给出搜索结果。


您可以选择 [Maven](https://spring.io/guides/gs/maven) 或者 [Gradle](https://spring.io/guides/gs/gradle) 作为系统的构建方式。

您还可以决定是获取**初始**代码集还是**完成**代码集，或者两者都获取。 对于大多数项目，**初始**代码集是一个空项目，使您可以通过指南复制和粘贴代码完成工程。 **完整**代码集是已经包含了指南中的所有代码。 如果你同时获取了这两个，你可以和指南对比下，看看差异。

最后，您可以通过STS的浏览器选项卡打开网站上的指南。 这将让您方便查看指导，而不必离开STS。

为了完成本指南的目标,  在搜索框中输入**rest**. 然后选择 [Consuming Rest](https://spring.io/guides/gs/consuming-rest). 选择 **Maven** 作为构建方式, 同时选择 **initial** 和 **complete** 代码集. 还可以同时勾选最下面的打开项目首页的网页， 如下图所示:

![Pick a guide](static/1033/3_wizard.png)

STS将在您的工作空间中创建两个新项目, 并同时导入 [Consuming Rest](https://spring.io/guides/gs/consuming-rest) 的代码 (包括 初始集和完整集), 打开一个STS内置的浏览器选项卡，如下图所示：

![View the code and the guide](static/1033/4_after-import.png)

到这儿，您就可以开始浏览指南并且查看代码文件了。

## 总结

恭喜！ 您已经安装了Spring Tool Suite，导入了“Consuming Rest入门项目”，并打开了一个浏览器选项卡来浏览它的说明。

## 了解更多

下面的指南也很有用哦：

- [Working a Getting Started guide with IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
