# 使用Spring构建REST服务

> 原文：[Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) （这里为示例，译者需根据具体文章修改）
>
> 译者：戒子猪
>
> 校对：

在此处编写正文，基本翻译规范：

* 正文标题按层次结构 从 \#\# 开始
* 代码片段\`\`\`之后需要写明语言类型
* 如有图片更静态资源保存在static目录下，每篇文章建立自己的目录存储
* 尊重原作、不修改、不删减内容
* 每篇文章翻译完成之后提交PR，并在翻译交流群中找校对人员完成review，最后由管理员完成Merge
* 若译者与校对有不同建议，可以将争议部分发到交流群中一起讨论确定结果

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。

## 术语表

* REST/RESTful：统一使用REST，不翻译。

## 正文

因为REST易于构建和使用，它很快成为网络中构建Web服务应用的标准。

关于REST如何在微服务的世界使用会有更广的讨论，但对于本教程，我们来看看构建REST服务。

为什么选择REST？借鉴[Martin Fowler](http://martinfowler.com/)的说法，[REST实践](https://www.amazon.com/gp/product/0596805829?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0596805829)中指出：“网络是大规模可扩展分布式系统的一个存在证据，它的工作原理很好，我们借鉴它更容易构建集成系统。”我认为这是一个很好的理由：REST包含了自身的网络规则、架构、优势以及所有。

有什么好处？主要因为HTTP自身是一个免费的平台，应用程序安全性（加密和验证）解决方案的数量是已知的，缓存直接内置在协议中，通过DNS的服务路由有良好的弹性，并且大部分系统都支持。

因此，REST无处不在，它并不是一种标准，而是一种方法，是一种基于HTTP约束的架构风格。它的实施方式可能会有所不同。作为API消费者，这可能是一个令人沮丧的经历，因为REST服务的质量差异会很大。

Leonard Richardson博士将REST总结汇集了一个成熟度模型，解释了系统遵循RESTful原则的程度，并对其进行了评分。它描述了从0级开始的四个级别，Martin Fowler在成熟度模型上写出了明确的解释：

* **0级（Level 0）**：POX沼泽——在这个级别，我们仅使用HTTP作为传输。您可以将SOAP称为0级技术，它使用HTTP协议，但主要是作为传输。值得一提的是，您也可以在没有HTTP的情况下使用SOAP，如JMS。因此，SOAP不是RESTful的，它只是HTTP感知的。
* **1级（Level 1）**：资源——在此级别，服务可能会使用HTTP URI来区分系统中的名词、实体。例如，您可以将请求路由到`/customers，/users`等。XML-RPC是1级技术的代表：它使用HTTP，并且可以使用URI来区分端点（Endpoint）。最终，尽管有这些定义，XML-RPC实际也不是Restful的：它使用HTTP做了其他东西（如RPC远程过程调用）的传输。
* **2级（Level 2）**：HTTP动词——这就是您想要的级别。如果您用Spring MVC时所有事都做错了，您将仍然在这个地方。在这个级别，服务利用了HTTP的原生术语如标题、状态代码、不同的URI等。这也将是我们旅程的起点。
* 


