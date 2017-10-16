# Spring官方Guides

随着微服务的流行，Spring Boot/Cloud的崛起，Spring Source几乎再一次要成为Java的代名词。那么我们如何才能快速的学习和入门Spring呢？除了很多国内高手编写的一些教程之外，有没有更为官方的指导呢？实际上，在Spring官方网站中是有非常优秀的教程页面的：[https://spring.io/guides。](https://spring.io/guides。)

但是由于该教程内容均是英文的，所以只有少部分人会关注这里。所以，我们[SpringForAll社区](http://spring4all.com)计划开始组织对这部分高质量内容的翻译工作，以促进Spring这样优秀的框架在国内的发展！由衷的希望Spring大生态变得越来越强大！

## 协作参与

* 交流社区：[SpringForAll社区](http://spring4all.com)
* Github：[https://github.com/SpringForAll/spring-guides-translation](https://github.com/SpringForAll/spring-guides-translation)
* 组织人：[程序猿DD](https://github.com/dyc87112/)
* 管理组成员：[maskwang520](https://github.com/maskwang520)、[strongant](https://github.com/strongant)、[chenzhijun](https://github.com/chenzhijun)、[lexburner](https://github.com/lexburner)、[Jitianyu](https://github.com/Jitianyu)
* 校对组头目：[maskwang520](https://github.com/maskwang520)

### 协作流程

* [翻译和校对的流程](https://github.com/SpringForAll/spring-guides-translation/blob/master/translate-readme.md)
* [待校对的翻译](https://github.com/SpringForAll/spring-guides-translation/pulls)
* [翻译操作指南](https://github.com/SpringForAll/spring-guides-translation/blob/master/translate-step.md)
* [校对流程指南](https://github.com/SpringForAll/spring-guides-translation/blob/master/proofread.md)

### 翻译内容

* [待翻译内容](https://github.com/SpringForAll/spring-guides-translation/tree/master/source)
* [已完成内容](https://github.com/SpringForAll/spring-guides-translation/tree/master/translated)

## 翻译规范

关于翻译的一些规范如下：

* 翻译文件格式：markdown

* 文件名格式：原文标题.md （文件已经预创建，直接编辑即可）

* 文章摘要部分采用如下的固定格式：（已经预创建，请勿删除该部分）

  > 原文：\[Securing a Web Application\]\([https://spring.io/guides/gs/securing-web/\](https://spring.io/guides/gs/securing-web/%29\)
  >
  > 译者：\[xxx\]\(\)
  >
  > 校对：\[yyy\]\(\)

* 文章末尾使用统一的版权声明：（已经预创建，请勿删除该部分）
  > 本文由spring4all.com翻译小分队创作，采用\[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可\]\([http://creativecommons.org/licenses/by-nc-sa/4.0/\)协议进行许可。](http://creativecommons.org/licenses/by-nc-sa/4.0/%29协议进行许可。)

* 正文标题按层次结构 从\# 开始

* 代码片段\`\`\`之后需要写明语言类型

* 如有图片更静态资源保存在static目录下，每篇文章建立自己的目录存储（目录名使用文章编号，文章编号见README）

* 文章锚点的跳转处理，不要跳转到英文原文，而应该采用文章内部锚点的跳转，在Markdown中的实现示例：假设有一个链接`xxx`是跳转到标题`Gradle`的，那么链接和标题需要下面这样写：
  * 链接这样写 ：`[xxx](#maodian)`
  * 要跳转到的标题这样写：`<h2 id="maodian">Gradle</h2>` ， 不能用传统的写法：`## Gradle`

* 尊重原作、不修改、不删减内容

* 每篇文章翻译完成之后提交PR，并在翻译交流群中找校对人员完成review，最后由管理员完成Merge

* 若译者与校对有不同建议，可以将争议部分发到交流群中一起讨论确定结果

## 致谢

将最诚挚的谢意给SpringForAll社区每一位翻译小队成员，他们都是Spring的忠实爱好者，这里的所有内容都是他们牺牲了业余时间所创造出来的。

下面是所有参与翻译与校对的小伙伴名单（排名不分先后）：

* 待补充

## License

本站作品由SpringForAll翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/)协议进行许可。

