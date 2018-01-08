# Spring Security 4 Logout Example

原文：[Spring Security 4 Logout Example](http://websystique.com/spring-security/spring-security-4-logout-example/)

译者：

校对：

This post shows you how to **programatically logout** a user in Spring Security. This works well with browser back button too. Let’s get going.

![SpringSecurityCustomLogoutExample_img4](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCustomLogoutExample_img4.png)



Generally, In your views, you should be providing a simple **logout link** to logout a user, something like shown below:

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

Nothing fancy about it. Now , We just need to map this **/logout** link in our controller. Create a new method like shown below:

```
	@RequestMapping(value="/logout", method = RequestMethod.GET)
	public String logoutPage (HttpServletRequest request, HttpServletResponse response) {
		Authentication auth = SecurityContextHolder.getContext().getAuthentication();
		if (auth != null){    
			new SecurityContextLogoutHandler().logout(request, response, auth);
		}
		return "redirect:/login?logout";//You can redirect wherever you want, but generally it's a good practice to show login screen again.
	}


```

Here firstly we identified if user was authenticated before using **SecurityContextHolder.getContext().getAuthentication()**. If he/she was, then we called **SecurityContextLogoutHandler().logout(request, response, auth)** to logout user properly.

This **logout** call performs following:

- Invalidates HTTP Session ,then unbinds any objects bound to it.
- Removes the Authentication from the SecurityContext to prevent issues with concurrent requests.
- Explicitly clears the context value from the current thread.

That’s it. You don’t need anything else anywhere in your application to handle logout. Notice that you don’t even need to do anything special in your spring configuration(xml or annotation based), shown below just for information:

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
	  	.and().exceptionHandling().accessDeniedPage("/Access_Denied");
	}
}

```

There is no special logout handling mentioned in above configuration.

**Above security configuration in XML configuration format would be:**

```
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-4.0.xsd">
     
    <http auto-config="true" >
        <intercept-url pattern="/" access="hasRole('USER')" />
        <intercept-url pattern="/home" access="hasRole('USER')" />
        <intercept-url pattern="/admin**" access="hasRole('ADMIN')" />
        <intercept-url pattern="/dba**" access="hasRole('ADMIN') and hasRole('DBA')" />
        <form-login  login-page="/login" 
                     username-parameter="ssoId" 
                     password-parameter="password" 
                     authentication-failure-url="/Access_Denied" />
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

Rest of application code is same as mentioned in every post in this series. [This Post](http://websystique.com/spring-security/spring-security-4-custom-login-form-annotation-example/) (and actually every post in this series) shows this logout in action.

#### Deploy & Run

Download complete code of this project using download button shown at the bottom of this post. Build and deploy it on Servlet 3.0 container(Tomcat7/8).

Open your browser, go to **http://localhost:8080/SpringSecurityCustomLogoutExample/**, home page will be shown

![SpringSecurityCustomLogoutExample_img1](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCustomLogoutExample_img1.png)

Now go to **http://localhost:8080/SpringSecurityCustomLogoutExample/admin**, you will be prompted for login

![SpringSecurityCustomLogoutExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCustomLogoutExample_img2.png)

provide credentials, submit, you will see admin page.

![SpringSecurityCustomLogoutExample_img3](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCustomLogoutExample_img3.png)

click on logout, you will be taken to login page.

![SpringSecurityCustomLogoutExample_img4](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCustomLogoutExample_img4.png)

click on **browser back button**, you will remain at login screen.

![SpringSecurityCustomLogoutExample_img5](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityCustomLogoutExample_img5.png)

That’s it. [Next post](http://websystique.com/spring-security/spring-security-4-secure-view-layer-using-taglibs/) shows you how to show/hide parts of jsp/view based on logged-in user’s roles, using Spring Security tags.

#### *Download Source Code*

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=1375)


> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。