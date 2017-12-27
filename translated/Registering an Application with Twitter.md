# 使用Twitter注册一个应用

> 原文：[Registering an Application with Twitter](https://spring.io/guides/gs/register-twitter-app/)
>
> 译者： [maskwang520](https://github.com/maskwang520)
>
> 

1 ---标签：[]项目：[spring-social-twitter] ---：toc：：图标：font：source-highlighter：prettify：project_id：gs-register-twitter-app

本指南将引导您完成注册可以与Twitter集成的应用程序的步骤。 注册应用程序是开发集成到其用户社交图中的应用程序的第一步。

您可以在Twitter.com的Web浏览器上执行本指南中的步骤。 尽管您不会编写任何代码，但您将使用一个简单的实用程序项目来验证您是否正确执行了这些步骤。 获取和运行实用程序的说明在本指南的末尾。

## 注册新的应用程序

所有的Twitter用户都是Twitter应用程序开发人员。 只需访问http://dev.twitter.com并使用您的Twitter凭据登录。

转到https://apps.twitter.com/查看一个页面，一旦你创建它们，就会列出所有的Twitter应用程序。 如果你还没有创建任何应用程序，列表是空的。

点击顶部附近的创建新应用按钮。 创建申请表单的新页面需要填写有关您的应用程序的基本信息。
![Alt text](https://spring.io/guides/gs/register-twitter-app/images/tw-create-app.png)

在“名称”字段中，将您的应用程序命名为32个字符或更少。 当提示用户授权您的应用程序访问他们的Twitter信息时，会向用户显示此名称。 在说明字段中，以10到200个字符描述您的应用程序。 再次，这在授权页面上呈现给用户。

在“网站”字段中，输入一个URL，指示用户返回到您的应用程序，在那里他们可以下载或找到更多信息。 与名称和说明一样，该字段将显示在面向用户的授权页面上。

“回调URL”字段可以指定Twitter在成功授权后应该重定向的网址。 你可以先空着来完成本指南中的练习，但是通常你会希望这是一个有效的URL（即使Twitter允许你在运行时改变它）。

如果你建立一个使用Twitter的API的应用程序，开发者规则部分概述了你必须同意遵守的规则。
![Alt text](https://spring.io/guides/gs/register-twitter-app/images/tw-rules-of-road.png)

这些规则包括如何展示Twitter和注意事项的指南，而不是简单地重新创建Twitter自己的客户端的功能。 建议您仔细阅读这些规则，以确保不违反规定。

如果您同意这些规则，请选中“是的，我同意”。

验证码确保您不会通过自动化流程设置应用程序。
![Alt text](https://spring.io/guides/gs/register-twitter-app/images/tw-captcha.png)

点击“创建您的Twitter应用程序”来完成表单并进入应用程序设置页面。
![Alt text](https://spring.io/guides/gs/register-twitter-app/images/tw-app-details.png)

在这里，根据您计划构建的应用程序以及您希望执行的操作，配置有关应用程序的详细信息。

需要注意的是Consumer key和Consumer secret这两个域。 这些值是您的应用程序的Twitter凭据。您需要他们使用Twitter做任何事情，包括通过OAuth授权流程和使用Twitter的REST API。

## 验证注册
现在你可以使用你的 consumer key and secret 来访问Twitter的API。 GitHub中的示例( sample utility application in GitHub)[https://github.com/spring-guides/%7Bproject_id%7D]是一个可以使用 consumer key and secret 来搜索Twitter的应用程序示例。您可以从GitHub克隆实用程序项目：

```shell
git clone https://github.com/spring-guides/{project_id}.git
```
要运行该实用程序，只需使用Gradle从命令行运行，如下所示：
```shell
./gradlew clean build && java -jar build/libs/{project_id}-0.1.0.jar
```
或者如果您使用的是Maven，请像这样运行它：
```shell
mvn package && java -jar target/{project_id}-0.1.0.jar
```
你也可以像这样直接从Gradle运行应用程序：
```shell
./gradlew bootRun
```
> 注意用mvn，你可以运行`mvn spring-boot：run`

你会看到两个对话框提示。 第一个将要求申请者的Consumer ID，第二个将要求申请者的Consumer Secret。 复制并粘贴来自Twitter开发者网站的值。

然后应用程序查询Twitter的REST API，在文本中搜索具有“#springframework”的Tweets。 如果你已经正确地设置你的应用程序，你应该看到几个匹配推文的文本。

## 总结
恭喜！ 您现在已经通过Twitter注册了一个应用程序。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。
