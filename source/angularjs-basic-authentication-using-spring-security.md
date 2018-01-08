# AngularJS+Spring Security using Basic Authentication

原文：[AngularJS+Spring Security using Basic Authentication](http://websystique.com/spring-security/angularjs-basic-authentication-using-spring-security/)

译者：

校对：

This post shows how an AngularJS application can consume a REST API which is secured with Basic authentication using Spring Security. 

Post [Secure Spring REST API with Basic Authentication](http://websystique.com/spring-security/secure-spring-rest-api-using-basic-authentication/) shows in great details how to secure a REST API using Basic authentication with Spring Security. That application will serve as a Back-end for this example. Although we will touch the main concepts here, complete code for the back-end will not be repeated here again. Please download, install and start that application locally in order to test the AngularJS app from this post.

We will mainly focus onto Front-end side which is pure AngularJS based application, communicating nicely with our REST-API. Let’s get started.
![AngularJSBasicAuth_img1](http://websystique.com/wp-content/uploads/2016/07/AngularJSBasicAuth_img1.png)



### What is Basic Authentication?

Traditional authentication approaches like login pages or session identification are good for web based clients involving human interaction but does not really fit well when communicating with [REST] clients which may not even be a web application. Think of an API over a server which tries to communicate with another API on a totally different server, without any human intervention.

[Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) provides a solution for this problem, although not very secure. With Basic Authentication, clients send it’s Base64 encoded credentials **with each request**, using HTTP [Authorization] header . That means each request is independent of other request and server may/does not maintain any state information for the client, which is good for scalability point of view.

### Front-end

#### 1. Send Authorization header with each request in AngularJS

Since Authentication header needs to be sent with Each request, Interceptors are a good choice to handle that instead of manually specifying the header in all $http methods.

`authInterceptor.js`

Notice how we have used `btoa()` function to get the Base64 encoded string from user-credentials. That’s all we need to enable basic authentication. Rest of the application is typical AngularJS app communicating with REST API on server. Now this interceptor needs to be registered with AngularJS application, as shown below.

#### 2. Application

`app.js`

#### 3. Service to communicate with REST API on server

Below service does not refer to any security related stuff. Thanks to interceptors, it is managed seperately.
`UserService.js`

#### 4. Controller

`user_controller.js`

#### 5. View

`index.html`

### Back-end

#### 1. REST API

Shown below is the REST API our Angular app will be communicating with.

#### 2. Enable Basic Authentication in Spring Security

With two steps, you can enable the Basic Authentication in Spring Security Configuration.
**1. Configure httpBasic** : Configures HTTP Basic authentication. [**http-basic** in XML]
**2. Configure authentication entry point with BasicAuthenticationEntryPoint** : In case the Authentication fails [invalid/missing credentials], this entry point will get triggered. It is very important, because we don’t want [Spring Security default behavior] of redirecting to a login page on authentication failure [ We don't have a login page]. Additionally, we want to remain stateless [no session information needs to be maintained on server]. Shown below is the complete Spring Security configuration with httpBasic and entry point setup.

### Running the application

**Back-end** : Build and deploy the Backend from Post [Secure Spring REST API with Basic Authentication](http://websystique.com/spring-security/secure-spring-rest-api-using-basic-authentication/).
**Front-end** : Download the AngularJS app from this post, deploy it [Put it in htdocs folder of your Apache server e.g, and start Apache]. You could have run FE, without even putting it behind Apache Server, but as your FE app grows with templates , it becomes a necessity to use a server.

Browse to **http://localhost/AngularClientWithBasicAuth/**

![AngularJSBasicAuth_img1](http://websystique.com/wp-content/uploads/2016/07/AngularJSBasicAuth_img1.png)

Open the Developer tools and verify the request, a Basic Authentication header is sent.

![AngularJSBasicAuth_img2](http://websystique.com/wp-content/uploads/2016/07/AngularJSBasicAuth_img2.png)

You might have notice that there is one more request towards server before the actual GET. It’s a OPTION request fired by browser itself, not under application control.
![AngularJSBasicAuth_img3](http://websystique.com/wp-content/uploads/2016/07/AngularJSBasicAuth_img3.png)
In order to allow this pre-flight request, we have adopted our Security configuration with `web.ignoring().antMatchers(HttpMethod.OPTIONS, "/**");`.

#### *Download Source Code*

Front-end Application:

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=2923)

Back-end Application:

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=2920)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。