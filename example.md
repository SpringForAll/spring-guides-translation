## HTTP是一个平台

HTTP URI是描述层次结构或关系的自然方式，例如，我们可能会在账户一级启动我们的REST API，所有的URI都以账户的用户名开头。因此，对于名为bob的账户，我们可能会将该账户解析成`/users/bob`或者只是`/bob`。想要访问该账户的书签集合时，我们可以降一级（像文件系统一样），降到`/bob/bookmarks`资源。

REST不规定表述和编码。REST是Representational State Transfer（表述状态迁移）的缩写，适用于HTTP的内容协商机制，允许客户端和服务端在可能的情况下同意来自服务的数据相互理解的表述方式。处理内容协商有很多种方法，最简单的方式就是客户端发送一个`Accept`请求头，该`Accept`标头指定了可接受的MIME类型的逗号分隔列表（如：`application/json, application/xml`）。如果该服务可以生成和这些MIME想匹配的类型，它将以第一个解析的MIME类型的表述形式进行响应。（译者语：对于Accept的“第一个”的概念主要和参数q相关，q的范围描述了这个MIME类型的优先级，并不是书写顺序，如：`Accept: application/json;q=0.5,application/xml;q=1.0`这种）

