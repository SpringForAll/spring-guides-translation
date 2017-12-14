# 使用 STS 部署 Cloud Foundry

> 原文：[Deploying to Cloud Foundry from STS](https://spring.io/guides/gs/sts-cloud-foundry-deployment/)
>
> 译者：[wjtBird](https://github.com/wjtBird)
>
> 校对：


本指南,旨在引导您使用 Spring Tool Suite（STS) 向 Cloud Foundry 部署一个 “hello world” 的 Spring 应用程序的过程。

# 你将部署什么到 Cloud Foundry
您将部署一个接受 HTTP GET 请求的 Spring Boot 的应用程序到 Cloud Foundry

```
 http://gs-sts-cloud-foundry-deployment-myname.cfapps.io/greeting
```

这个 URL 将在本指南中根据您修改的 host 部分而有所不同。

然后，应用程序将响应展示一个问候的 web 页：

``` "Hello World!" ```

您可以通过在查询字符串中使用可选参数```name```来定义自己的问候

``` http://gs-sts-cloud-foundry-deployment-myname.cfapps.io/greeting?name=User ```

这个参数```name```的值将覆盖默认的 “World” ，并展示在响应中：

```"Hello, User!"```

>当您通过 STS 将应用程序部署到 Cloud Foundry 时，URL 的 myname 部分将是您要更改的内容，以避免在部署过程中发生 host-taken error。

此应用基于一个提供 web 内容服务的 Spring 服务。更多有关如何从头开始创建服务的信息，请参见使用 [Spring MVC]() ，或通过[导入 Spring 入门内容向导]()将其导入 STS 中。

## 你需要的准备
* 花费15分钟
* [Spring Tool Suite(STS)]()
* [JDK8]() 以上
* [Pivotal Web Services (PWS)]() 账号
* [Spring Boot Dashboard]()

# 安装 STS
如果您还没有安装 STS，请访问上面的链接。从那里，你可以下载您系统对应的版本。将下载后的压缩包解压安装。

完成后，继续并启动 STS。

# Spring Boot Dashboard
Spring Boot Dashboard 用于将您的应用程序部署到 Cloud Foundry 中，需要使用 STS 3.7.1或更高版本。

# 创建一个 Cloud Foundary Target
首先您需要为想要部署应用程序的 Cloud Foundry 组织和空间创建一个target。

要创建 Cloud Foundry target，首先打开 Boot Dashboard。

您可以在 STS 主工具栏中单击 Boot Dashboard 按钮：

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/boot_dashboard_view_main_toolbar.png?raw=true)

或者，您可以在 Eclipse 的 “Show View” 菜单中打开它：

Window → Show View → Other → Spring → Boot Dashboard

之后点击 Boot Dashboard 工具栏右上角的“+”按钮，打开 Cloud Foundry Target 向导。

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/boot_dashboard_view_basic.png?raw=true)

在该向导中，输入您的PWS凭据，然后单击 “Select Space” 选择 Cloud Foundry organization 和 space。之后点击 “Finish” 完成创建 target。

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/add_cf_target.png?raw=true)

target 将显示在 Boot Dashboard 中。

# 导入 Spring 应用程序
现在您可以导入一个提供 web 内容服务的 Spring Boot 应用。之后将其部署到 Cloud Foundry。

在 STS 中，打开 “Import Spring Getting Started Content” 向导：

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/import_gsg.png?raw=true)

在 search field 中，输入 “sts cloud foundry”，然后出现 sts-cloud-foundry-deployment 指南。

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/import_gsg_wizard.png?raw=true)

* 选择 Build Type。
* 选择 “default” code set。
* 点击 “Finish”。

导入向导将在您的工作空间中创建一个名为 “gs-sts-cloud-foundry-deployment” 的新 project。

# 部署到 Cloud Foundry
现在只需将 project 拖放到在 Boot Dashboard 中的 Cloud Foundry target 即可。

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/drag_drop.png?raw=true)

这将打开部署清单对话框。Boot Dashboard 使用Cloud Foundry manifest.yml 来指定应用程序的部署详细信息，包括要绑定的 application name，host，memory 和 services。

您可以使用已添加到 Spring Boot 项目中的现有 manifest.yml 文件，也可以选择 “manual” 使用对话框生成的默认值

在 manual 模式下，您的项目中不会创建 manifest.yml 文件。

>为确保应用程序的 URL 没有被其他应用程序使用，并避免在部署过程中发生 host-taken error，请在对话框的 manifest.yml 编辑器中指定一个不同的 host。

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/deployment_manifest.png?raw=true)

完成应用程序配置后，点击 “OK” 完成部署。

部署可能需要一些时间，但是随着应用程序的部署和启动，应用程序的 console 将自动打开并显示进度。 当应用成功启动运行，console 和 Boot Dashboard 都会显示 “SUCCEEDED”。 应用程序的 Boot Dashboard 图标将变成绿色的“向上”箭头。

![Deploying to Cloud Foundry from STS](https://github.com/wjtBird/spring-guides-translation/blob/master/translated/static/1052/console_application_running.png?raw=true)

# 测试应用程序

现在您的应用程序正在 Cloud Foundry 上运行，您可以通过双击 Boot Dashboard 中的应用程序在 STS 内打开应用的网站。这将会打开默认浏览器。

在浏览器中，追加：

/greeting

到应用程序的 URL，你会看到应用程序页面显示：

```"Hello World!"```

使用查询字符串参数 ```name``` 并将其追加到浏览器中的 URL：

/greeting?name=User.

注意消息是如何从“Hello，World“ 到 “Hello，User”：

```"Hello, User!"```

# 总结
恭喜！你已经将你的 Spring Boot 应用部署到了 Cloud Foundry 上。

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



