# Spring Security 4 Custom Login Form Annotation+XML Example

原文：[spring-security-4-custom-login-form-annotation-example](http://websystique.com/spring-security/spring-security-4-custom-login-form-annotation-example/)

译者：

校对：

This post shows you creating **custom login form** in Spring Security 4 and integrate it in Spring MVC web application.

![SpringSecurityCusotmLoginFormAnnotationExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img2.png)

In [Spring Security 4 Hello World Annotation+xml example](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/), we have seen the default login form provided by Spring Security in case we don’t specify one. In this post, we will create our own Custom login form. Let’s get going.

Basically, the idea is, in Security Configuration, attach a call to **loginPage(URL)** function with formLogin() like shown below

```
	  	.and().formLogin().loginPage("/login")

```

And then, Map this **‘/login’** URL in your Spring MVC Controller which will return the login view defined by you. Now, on login attempt, the specified login view will be displayed.Rest of the login functionality remains same.

**Following technologies being used:**

- Spring 4.1.6.RELEASE
- Spring Security 4.0.1.RELEASE
- Maven 3
- JDK 1.7
- Tomcat 8.0.21
- Eclipse JUNO Service Release 2

Let’s begin.

#### Step 1: Project directory structure

Following will be the final project structure:
![SpringSecurityCusotmLoginFormAnnotationExample_img0](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img0.png)

Let’s now add the content mentioned in above structure explaining each in detail.

#### Step 2: Update pom.xml to include required dependencies

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.websystique.springsecurity</groupId>
	<artifactId>SpringSecurityCusotmLoginFormAnnotationExample</artifactId>
	<version>1.0.0</version>
	<packaging>war</packaging>

	<name>SpringSecurityCusotmLoginFormAnnotationExample</name>

	<properties>
		<springframework.version>4.1.6.RELEASE</springframework.version>
		<springsecurity.version>4.0.1.RELEASE</springsecurity.version>
	</properties>

	<dependencies>
		<!-- Spring -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${springframework.version}</version>
		</dependency>

		<!-- Spring Security -->
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-web</artifactId>
			<version>${springsecurity.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-config</artifactId>
			<version>${springsecurity.version}</version>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet.jsp</groupId>
			<artifactId>javax.servlet.jsp-api</artifactId>
			<version>2.3.1</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
	</dependencies>

	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.2</version>
					<configuration>
						<source>1.7</source>
						<target>1.7</target>
					</configuration>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>2.4</version>
					<configuration>
						<warSourceDirectory>src/main/webapp</warSourceDirectory>
						<warName>SpringSecurityCusotmLoginFormAnnotationExample</warName>
						<failOnMissingWebXml>false</failOnMissingWebXml>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<finalName>SpringSecurityCusotmLoginFormAnnotationExample</finalName>
	</build>
</project>

```

#### Step 3: Add Spring Security Configuration Class

The first and foremost step to add spring security in our application is to create **Spring Security Java Configuration**. This configuration creates a Servlet Filter known as the `springSecurityFilterChain` which is responsible for all the security (protecting the application URLs, validating submitted username and passwords, redirecting to the log in form, etc) within our application.

```
package com.websystique.springsecurity.configuration;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	
	@Autowired
	public void configureGlobalSecurity(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().withUser("bill").password("abc123").roles("USER");
		auth.inMemoryAuthentication().withUser("admin").password("root123").roles("ADMIN");
		auth.inMemoryAuthentication().withUser("dba").password("root123").roles("ADMIN","DBA");
	}
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
	  
	  http.authorizeRequests()
	  	.antMatchers("/", "/home").permitAll()
	  	.antMatchers("/admin/**").access("hasRole('ADMIN')")
	  	.antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")
	  	.and().formLogin().loginPage("/login")
	  	.usernameParameter("ssoId").passwordParameter("password")
	  	.and().csrf()
	  	.and().exceptionHandling().accessDeniedPage("/Access_Denied");
	}
}


```

This class is same as the one in [previous post](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/). Only differences are shown below

```
.and().formLogin().loginPage("/login")
	  	.usernameParameter("ssoId").passwordParameter("password")
	  	.and().csrf()

```

This code creates a custom login page with **‘/login’** url, which will accept **ssoId** as username and **password** Http request parameters. We have also shown a call to `csrf()` which is optional as it is by default active in Spring Security 4. This call is, however, required if you want to disable [CSRF protection](http://docs.spring.io/spring-security/site/docs/3.2.x/reference/htmlsingle/#csrf) by using `csrf().disable()` although it is not a good idea to disable it.

**Above security configuration in XML configuration format would be:**

```
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-4.0.xsd">
     
    <http auto-config="true" >
        <intercept-url pattern="/" access="permitAll" />
        <intercept-url pattern="/home" access="permitAll" />
        <intercept-url pattern="/admin**" access="hasRole('ADMIN')" />
        <intercept-url pattern="/dba**" access="hasRole('ADMIN') and hasRole('DBA')" />
        <form-login  login-page="/login" username-parameter="ssoId" password-parameter="password" authentication-failure-url="/Access_Denied" />
        <csrf/>
    </http>
 
    <authentication-manager >
        <authentication-provider>
            <user-service>
                <user name="bill"  password="abc123"  authorities="ROLE_USER" />
                <user name="admin" password="root123" authorities="ROLE_ADMIN" />
                <user name="dba"   password="root123" authorities="ROLE_ADMIN,ROLE_DBA" />
            </user-service>
        </authentication-provider>
    </authentication-manager>
     
    
</beans:beans>

```

#### Step 4: Register the springSecurityFilter with war

Below specified initializer class registers the `springSecurityFilter` [created in Step 3] with application war.

```
package com.websystique.springsecurity.configuration;

import org.springframework.security.web.context.AbstractSecurityWebApplicationInitializer;

public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {

}

```

This class is exactly same as in [previous post](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/).

**Above setup in XML configuration format would be:**

```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

#### Step 5: Add Controller

```
package com.websystique.springsecurity.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class HelloWorldController {

	
	@RequestMapping(value = { "/", "/home" }, method = RequestMethod.GET)
	public String homePage(ModelMap model) {
		model.addAttribute("greeting", "Hi, Welcome to mysite");
		return "welcome";
	}

	@RequestMapping(value = "/admin", method = RequestMethod.GET)
	public String adminPage(ModelMap model) {
		model.addAttribute("user", getPrincipal());
		return "admin";
	}
	
	@RequestMapping(value = "/db", method = RequestMethod.GET)
	public String dbaPage(ModelMap model) {
		model.addAttribute("user", getPrincipal());
		return "dba";
	}

	@RequestMapping(value = "/Access_Denied", method = RequestMethod.GET)
	public String accessDeniedPage(ModelMap model) {
		model.addAttribute("user", getPrincipal());
		return "accessDenied";
	}

	@RequestMapping(value = "/login", method = RequestMethod.GET)
	public String loginPage() {
		return "login";
	}

	@RequestMapping(value="/logout", method = RequestMethod.GET)
	public String logoutPage (HttpServletRequest request, HttpServletResponse response) {
		Authentication auth = SecurityContextHolder.getContext().getAuthentication();
		if (auth != null){    
			new SecurityContextLogoutHandler().logout(request, response, auth);
		}
		return "redirect:/login?logout";
	}

	private String getPrincipal(){
		String userName = null;
		Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();

		if (principal instanceof UserDetails) {
			userName = ((UserDetails)principal).getUsername();
		} else {
			userName = principal.toString();
		}
		return userName;
	}

}

```

In comparison to [previous post](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/) , only changes are new **loginPage** method to handle **‘/login’** requests and adapting **logout**to redirect to login page on logout, as shown below:

```
	@RequestMapping(value = "/login", method = RequestMethod.GET)
	public String loginPage() {
		return "login";
	}

	@RequestMapping(value="/logout", method = RequestMethod.GET)
	public String logoutPage (HttpServletRequest request, HttpServletResponse response) {
		Authentication auth = SecurityContextHolder.getContext().getAuthentication();
		if (auth != null){    
			new SecurityContextLogoutHandler().logout(request, response, auth);
		}
		return "redirect:/login?logout";
	}


```

#### Step 6: Add SpringMVC Configuration Class

```
package com.websystique.springsecurity.configuration;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.JstlView;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.websystique.springsecurity")
public class HelloWorldConfiguration extends WebMvcConfigurerAdapter{
	
	@Bean
	public ViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		viewResolver.setViewClass(JstlView.class);
		viewResolver.setPrefix("/WEB-INF/views/");
		viewResolver.setSuffix(".jsp");

		return viewResolver;
	}

     /*
     * Configure ResourceHandlers to serve static resources like CSS/ Javascript etc...
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("/static/");
    }
}

```

Changes from [previous post](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/) are extend from **WebMvcConfigurerAdapter** [just a convenience class] and implementing method **addResourceHandlers** which handles static resources(CSS/images/..) to be used in views.

#### Step 7: Add Initializer class

```
package com.websystique.springsecurity.configuration;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpringMvcInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class[] { HelloWorldConfiguration.class };
	}
 
	@Override
	protected Class<?>[] getServletConfigClasses() {
		return null;
	}
 
	@Override
	protected String[] getServletMappings() {
		return new String[] { "/" };
	}

}

```

This class is exactly same as in [previous post](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/).

#### Step 8: Add Views

`login.jsp`
This view additionally contains CSS for login panel layout.

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
		<title>Login page</title>
		<link href="<c:url value='/static/css/bootstrap.css' />"  rel="stylesheet"></link>
		<link href="<c:url value='/static/css/app.css' />" rel="stylesheet"></link>
		<link rel="stylesheet" type="text/css" href="//cdnjs.cloudflare.com/ajax/libs/font-awesome/4.2.0/css/font-awesome.css" />
	</head>

	<body>
		<div id="mainWrapper">
			<div class="login-container">
				<div class="login-card">
					<div class="login-form">
						<c:url var="loginUrl" value="/login" />
						<form action="${loginUrl}" method="post" class="form-horizontal">
							<c:if test="${param.error != null}">
								<div class="alert alert-danger">
									<p>Invalid username and password.</p>
								</div>
							</c:if>
							<c:if test="${param.logout != null}">
								<div class="alert alert-success">
									<p>You have been logged out successfully.</p>
								</div>
							</c:if>
							<div class="input-group input-sm">
								<label class="input-group-addon" for="username"><i class="fa fa-user"></i></label>
								<input type="text" class="form-control" id="username" name="ssoId" placeholder="Enter Username" required>
							</div>
							<div class="input-group input-sm">
								<label class="input-group-addon" for="password"><i class="fa fa-lock"></i></label> 
								<input type="password" class="form-control" id="password" name="password" placeholder="Enter Password" required>
							</div>
							<input type="hidden" name="${_csrf.parameterName}" 	value="${_csrf.token}" />
								
							<div class="form-actions">
								<input type="submit"
									class="btn btn-block btn-primary btn-default" value="Log in">
							</div>
						</form>
					</div>
				</div>
			</div>
		</div>

	</body>
</html>

```

Notice the CSRF related line in above jsp:

```
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}" /></strong>

```

This is required to protect against [CSRF attacks](http://docs.spring.io/spring-security/site/docs/3.2.x/reference/htmlsingle/#csrf). As you can see, the CSRF parameters are accessed using EL Expressions in your JSP, you may additionally prefer to force EL expressions to be evaluated, by adding following to the top of your JSP:

```
<%@ page isELIgnored="false"%>

```

`welcome.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Welcome page</title>
</head>
<body>
	Greeting : ${greeting}
	This is a welcome page.
</body>
</html>

```

`admin.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Admin page</title>
</head>
<body>
	Dear <strong>${user}</strong>, Welcome to Admin Page.
	<a href="<c:url value="/logout" />">Logout</a>
</body>
</html>

```

`dba.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>DBA page</title>
</head>
<body>
	Dear <strong>${user}</strong>, Welcome to DBA Page.
	<a href="<c:url value="/logout" />">Logout</a>
</body>
</html>

```

`accessDenied.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>AccessDenied page</title>
</head>
<body>
	Dear <strong>${user}</strong>, You are not authorized to access this page
	<a href="<c:url value="/logout" />">Logout</a>
</body>
</html>

```

In case you need it, here is the quick & dirty CSS i used for this example
`app.css`

```
html{
	background-color:#2F2F2F;
}

body, #mainWrapper {
	height: 100%;
	background-image: -webkit-gradient(
	linear,
	right bottom,
	right top,
	color-stop(0, #EDEDED),
	color-stop(0.08, #EAEAEA),
	color-stop(1, #2F2F2F),
	color-stop(1, #AAAAAA)
);
background-image: -o-linear-gradient(top, #EDEDED 0%, #EAEAEA 8%, #2F2F2F 100%, #AAAAAA 100%);
background-image: -moz-linear-gradient(top, #EDEDED 0%, #EAEAEA 8%, #2F2F2F 100%, #AAAAAA 100%);
background-image: -webkit-linear-gradient(top, #EDEDED 0%, #EAEAEA 8%, #2F2F2F 100%, #AAAAAA 100%);
background-image: -ms-linear-gradient(top, #EDEDED 0%, #EAEAEA 8%, #2F2F2F 100%, #AAAAAA 100%);
background-image: linear-gradient(to top, #EDEDED 0%, #EAEAEA 8%, #2F2F2F 100%, #AAAAAA 100%);
}

body, #mainWrapper, .form-control{
	font-size:12px!important;
}

#mainWrapper {
	height: 100vh; 
	padding-left:10px;
	padding-right:10px;
	padding-bottom:10px;
}

#authHeaderWrapper{
	clear:both;
	width: 100%;
	height:3%;
	padding-top:5px;
	padding-bottom:5px;
}

.login-container {
    margin-top: 100px;
    background-color: floralwhite;
    width: 40%;
    left: 30%;
    position: absolute;
}

.login-card {
    width: 80%;
    margin: auto;
}
.login-form {
    padding: 10%;
}

```

#### Step 9: Build and Deploy the application

Now build the war (either by eclipse/m2eclipse) or via maven command line( `mvn clean install`). Deploy the war to a Servlet 3.0 container . Since here i am using Tomcat, i will simply put this war file into `tomcat webapps folder` and click on `start.bat` inside tomcat bin directory.

**Run the application**
Open browser and goto **localhost:8080/SpringSecurityCusotmLoginFormAnnotationExample/**

![SpringSecurityCusotmLoginFormAnnotationExample_img1](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img1.png)

Now try to access admin page on **localhost:8080/SpringSecurityCusotmLoginFormAnnotationExample/admin**, you will be prompted for login.

![SpringSecurityCusotmLoginFormAnnotationExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img2.png)

Provide credentials of a ‘USER’ role.

![SpringSecurityCusotmLoginFormAnnotationExample_img3](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img3.png)

Submit, you will see AccessDenied Page

![SpringSecurityCusotmLoginFormAnnotationExample_img5](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img5.png)

Now logout and try to access admin page again

![SpringSecurityCusotmLoginFormAnnotationExample_img7](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img7.png)

Provide wrong password

![SpringSecurityCusotmLoginFormAnnotationExample_img4](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img4.png)

Provide proper admin role credentials and login

![SpringSecurityCusotmLoginFormAnnotationExample_img8](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img8.png)

Now try to access db page on **localhost:8080/SpringSecurityCusotmLoginFormAnnotationExample/db**, you will get AccessDenied page.

![SpringSecurityCusotmLoginFormAnnotationExample_img9](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img9.png)

Logout.

![SpringSecurityCusotmLoginFormAnnotationExample_img6](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCusotmLoginFormAnnotationExample_img6.png)

That’s it. [Next post](http://websystique.com/spring-security/spring-security-4-logout-example/) shows you implementing custom logout method which plays well with browser back button as well.

#### *Download Source Code*

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=1363)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。