想要以安全的方式运行？

```bash
$ cd security
$ mvn clean spring-boot:run
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::

2016-11-04 11:54:21.135  INFO 73989 --- [           main] bookmarks.Application                    : Starting Application on retina with PID 73989 (/Users/gturnquist/src/tutorials/tut-bookmarks/security/target/classes started by gturnquist in /Users/gturnquist/src/tutorials/tut-bookmarks/security)
2016-11-04 11:54:21.137  INFO 73989 --- [           main] bookmarks.Application                    : No active profile set, falling back to default profiles: default
2016-11-04 11:54:21.181  INFO 73989 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@3db919e6: startup date [Fri Nov 04 11:54:21 CDT 2016]; root of context hierarchy
2016-11-04 11:54:22.116  WARN 73989 --- [           main] o.s.c.a.ConfigurationClassPostProcessor  : Cannot enhance @Configuration bean definition 'org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerEndpointsConfiguration$TokenKeyEndpointRegistrar' since its singleton instance has been created too early. The typical cause is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor return type: Consider declaring such methods as 'static'.
2016-11-04 11:54:22.392  INFO 73989 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [class org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration$$EnhancerBySpringCGLIB$$240a3410] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
...
```

使用Spring Security OAuth，我们可以对API进行一些修改，让用户标识贯穿在整个API的URI没有任何意义。毕竟，用户上下文已经暗示了它在我们端点的安全访问中，安全Spring MVC处理程序方法不需要在路径中使用`userId`，而是希望Spring Security为我们注入`javax.security.Principal`。这个类提供了一个可以替代userId的`javax.security.Principal#getName`方法。

## 结论

和无处不在、不同的客户端进行通信，REST是最自然的方式，它是基于HTTP协议工作的。一旦您了解REST如何在应用程序中实现，就可以设想一个具有众多单独关注统一接口的API架构，这些API通过REST的方式暴露出来，并且像任何其他HTTP服务一样实现负载均衡。因此，REST（通常但不一定）是微服务的关键并不奇怪，我们将在未来的教程中做更深入的研究。



