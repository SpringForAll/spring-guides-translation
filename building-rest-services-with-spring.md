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

有什么好处？

