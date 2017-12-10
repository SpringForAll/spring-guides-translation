# 把一个Spring Boot JAR程序转为WAR程序

> 原文：[Converting a Spring Boot JAR Application to a WAR](https://spring.io/guides/gs/convert-jar-to-war/) 
>
> 译者：[JohnHello](https://github.com/JohnHello)
>
> 校对：[Filoy](https://github.com/feilangrenM)


Spring Boot自带两个强大的插件：

* spring-boot-gradle-plugin
* spring-boot-maven-plugin

这两个插件本质上提供相同的功能，都可以在命令行启动Spring Boot程序或直接打包成可运行的jar包。几乎每一个Spring Boot新手指南上都会提到这一点。

除了上述两种方式外，还有很多人习惯于生成WAR文件部署到容器中。当然，这两个插件同样支持生成WAR包。基本上两步就可以搞定，首先需要修改最终生成的程序文件类型为WAR，同时声明嵌入式容器的依赖为"provided"类型。这样就可以保证相关的内嵌式容器依赖文件不会被放入生成的WAR文件中。

如何配置你的项目打包成可被容器使用的WAR文件，更多详细步骤可以参考：
* [Packaging executable jar and war files with Maven](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#build-tool-plugins-maven-packaging)
* [Packaging executable jar and war files with Gradle](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#build-tool-plugins-gradle-packaging)

>  Spring Boot 程序需要运行在符合servlet 3.0规范的容器中。

### 更多
下面这篇指南可能对你有所帮助：
* [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)







> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。



