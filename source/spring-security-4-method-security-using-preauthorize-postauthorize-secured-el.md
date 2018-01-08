# Spring Security 4 Method security using @PreAuthorize,@PostAuthorize, @Secured, EL

原文：[Spring Security 4 Method security using @PreAuthorize,@PostAuthorize, @Secured, EL](http://websystique.com/spring-security/spring-security-4-method-security-using-preauthorize-postauthorize-secured-el/)

译者：

校对：

This post shows Method level security in Spring Security 4 with **@PreAuthorize, @PostAuthorize, @Secured and Spring EL expressions**. Let’s get going.

In order to enable Spring Method level Security, we need to annotate a **@Configuration** class with `@EnableGlobalMethodSecurity`, as shown below:

```
package com.websystique.springsecurity.configuration;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
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
	  	.antMatchers("/", "/home").access("hasRole('USER') or hasRole('ADMIN') or hasRole('DBA')")
	  	.and().formLogin().loginPage("/login")
	  	.usernameParameter("ssoId").passwordParameter("password")
	  	.and().exceptionHandling().accessDeniedPage("/Access_Denied");
	}
}

```

**@EnableGlobalMethodSecurity** enables Spring Security global method security similar to the in XML configurations like shown below:

```
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
    http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-4.0.xsd">
     
    <http auto-config="true" >
        <intercept-url pattern="/"     access="hasRole('USER') or hasRole('ADMIN') and hasRole('DBA')" />
        <intercept-url pattern="/home" access="hasRole('USER') or hasRole('ADMIN') and hasRole('DBA')" />
        <form-login  login-page="/login" 
                     username-parameter="ssoId" 
                     password-parameter="password" 
                     authentication-failure-url="/Access_Denied" />
    </http>
    
    <global-method-security pre-post-annotations="enabled"/>

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

Note that @EnableGlobalMethodSecurity can take several arguments, some are shown below:

- **prePostEnabled** :Determines if Spring Security’s pre post annotations [@PreAuthorize,@PostAuthorize,..] should be enabled.
- **secureEnabled** : Determines if Spring Security’s secured annotation [@Secured] should be enabled.
- **jsr250Enabled** : Determines if [JSR-250 annotations](https://en.wikipedia.org/wiki/JSR_250) [@RolesAllowed..] should be enabled.

You can enable more than one type of annotation in the same application, but only one type should be used for any interface or class as the behavior will not be well-defined otherwise. If two annotations are found which apply to a particular method, then only one of them will be applied.

We will explore first two of above mentioned in detail.

#### @Secured

@Secured annotation is used to define a list of security configuration attributes for business methods. You can specify the security requirements[roles/permission etc] on a method using @Secured, and than only the user with those roles/permissions can invoke that method. If anyone tries to invoke a method and does not possess the required roles/permissions, an AccessDenied exception will be thrown.

@Secured is coming from previous versions of Spring. It has a limitation that it does not support Spring EL expressions. Consider following example:

```
package com.websystique.springsecurity.service;

import org.springframework.security.access.annotation.Secured;


public interface UserService {

	List<User> findAllUsers();

	@Secured("ROLE_ADMIN")
	void updateUser(User user);

	@Secured({ "ROLE_DBA", "ROLE_ADMIN" })
	void deleteUser();
	
}

```

In above example, updateUser method can be invoked by someone with ADMIN role while deleteUser can be invoked by anyone with DBA **OR** ADMIN role. If anyone tries to invoke a method and does not possess the required role, an AccessDenied exception will be thrown.

What if you want to specify an **‘AND’** condition. I mean , you want deleteUser method to be invoked by a user who have both ADMIN & DBA role. **This is not possible straight-away with @Secured annotation.**

This can however be done using Spring’s new @PreAuthorize/@PostAuthorize annotations which supports Spring EL, that means possibilities are unlimited.

#### @PreAuthorize / @PostAuthorize

Spring’s `@PreAuthorize/@PostAuthorize` annotations are preferred way for applying method-level security, and supports Spring Expression Language out of the box, and provide expression-based access control.

**@PreAuthorize** is suitable for verifying authorization before entering into method. @PreAuthorize can take into account, the roles/permissions of logged-in User, argument passed to the method etc.

**@PostAuthorize** , not often used though, checks for authorization after method have been executed, so it is suitable for verifying authorization on returned values. Spring EL provides **returnObject** object that can be accessed in expression language and reflects the actual object returned from method.

Please refer to [Common Built-In Expressions](http://docs.spring.io/spring-security/site/docs/4.0.1.RELEASE/reference/htmlsingle/#el-common-built-in) to get the complete list of supported expressions.

Let’s get back to our example, this time using @PreAuthorize / @PostAuthorize.

```
package com.websystique.springsecurity.service;

import org.springframework.security.access.prepost.PostAuthorize;
import org.springframework.security.access.prepost.PreAuthorize;

import com.websystique.springsecurity.model.User;


public interface UserService {

	List<User> findAllUsers();

	@PostAuthorize ("returnObject.type == authentication.name")
	User findById(int id);

	@PreAuthorize("hasRole('ADMIN')")
	void updateUser(User user);
	
	@PreAuthorize("hasRole('ADMIN') AND hasRole('DBA')")
	void deleteUser(int id);

}

```

Since **@PreAuthorize** can use Spring Expression Language, any condition can easily be expressed using EL. Method deleteUser is now configured to be invoked by a user who have both ADMIN & DBA roles.

Additionally, we have added a method findById() with **@PostAuthorize** annotation. With @PostAuthorize, the returned value from the method(User object) will be accessible with `returnObject` in Spring Expression Language, and individual properties of return user object can be used to apply some security rules. In this example we are making sure that a logged-in user can only get it’s own User type object.

That’s all about basic usage of @Secured, @PreAuthorize, @PostAuthorize and EL.

Below mentioned is service implementation, User model class & Controller, used in this example.

```
package com.websystique.springsecurity.service;


import java.util.ArrayList;
import java.util.List;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.websystique.springsecurity.model.User;

@Service("userService")
@Transactional
public class UserServiceImpl implements UserService{

	static List<User> users = new ArrayList<User>();
	
	static{
		users = populateUser();
	}
	
	public List<User> findAllUsers(){
		return users;
	}
	
	public User findById(int id){
		for(User u : users){
			if(u.getId()==id){
				return u;
			}
		}
		return null;
	}
	
	public void updateUser(User user) {
		System.out.println("Only an Admin can Update a User");
		User u = findById(user.getId());
		users.remove(u);
		u.setFirstName(user.getFirstName());
		u.setLastName(user.getLastName());
		u.setType(user.getType());
		users.add(u);
	}
	
	public void deleteUser(int id){
		User u = findById(id);
		users.remove(u);
	}
	
	private static List<User> populateUser(){
		List<User> users = new ArrayList<User>();
		users.add(new User(1,"Sam","Disilva","admin"));
		users.add(new User(2,"Kevin","Brayn","admin"));
		users.add(new User(3,"Nina","Conor","dba"));
		users.add(new User(4,"Tito","Menz","dba"));
		return users;
	}

}

```

```
public class User {

	private int id;
	
	private String firstName;
	
	private String lastName;
	
	private String type;

//getters/setters
}

```

```
package com.websystique.springsecurity.controller;

import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.websystique.springsecurity.model.User;
import com.websystique.springsecurity.service.UserService;

@Controller
public class HelloWorldController {

	@Autowired
	UserService service;
	
    @RequestMapping(value = { "/", "/list" }, method = RequestMethod.GET)
    public String listAllUsers(ModelMap model) {
 
        List<User> users = service.findAllUsers();
        model.addAttribute("users", users);
        return "allusers";
    }
	
    @RequestMapping(value = { "/edit-user-{id}" }, method = RequestMethod.GET)
    public String editUser(@PathVariable int id, ModelMap model) {
        User user  = service.findById(id);
        model.addAttribute("user", user);
        model.addAttribute("edit", true);
        return "registration";
    }
    
    @RequestMapping(value = { "/edit-user-{id}" }, method = RequestMethod.POST)
    public String updateUser(User user, ModelMap model, @PathVariable int id) {
        service.updateUser(user);
        model.addAttribute("success", "User " + user.getFirstName()  + " updated successfully");
        return "success";
    }
    
    @RequestMapping(value = { "/delete-user-{id}" }, method = RequestMethod.GET)
    public String deleteUser(@PathVariable int id) {
        service.deleteUser(id);
        return "redirect:/list";
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

Complete code for this example is attached at the end of this post.

#### Deploy & Run

Download complete code example attached at the end of post. Deploy it on Servlet 3.0 container(Tomcat 8.0.21 e.g.).
Open browser and goto **http://localhost:8080/SpringSecurityMethodLevelSecurityAnnotationExample/**, you will be prompted for login. Fill in USER role credentials.

![SpringSecurityMethodLevelSecurityAnnotationExample_img1](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img1.png)

Submit, you will see list of users.

![SpringSecurityMethodLevelSecurityAnnotationExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img2.png)

Now try to edit or delete a users, you should see accessDenied page, because USER role does not have access to these functions.

![SpringSecurityMethodLevelSecurityAnnotationExample_img3](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img3.png)

Logout. Login with ADMIN role credentials.

![SpringSecurityMethodLevelSecurityAnnotationExample_img4](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img4.png)

Submit, you will see list of users.

![SpringSecurityMethodLevelSecurityAnnotationExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img2.png)

Now click on edit for first row [with type='admin']. Edit page should be presented.

![SpringSecurityMethodLevelSecurityAnnotationExample_img5](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img5.png)

Now go back to list of item and click on third row [with type = 'dba']

![SpringSecurityMethodLevelSecurityAnnotationExample_img6](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img6.png)

You got accessDenied because during edit, function findById gets called which is annotated with @PostAuthorize annotation with EL restricting the returned object to be only with type['dba'] same as logged-in user name.

Now click on any delete row, accessDenied should be shown as deleteUser is only allowed for role ‘DBA’.

![SpringSecurityMethodLevelSecurityAnnotationExample_img7](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img7.png)

Now logout, login with DBA role [dba,root123], and click on delete link of first row. Row should be deleted successfully.

![SpringSecurityMethodLevelSecurityAnnotationExample_img8](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityMethodLevelSecurityAnnotationExample_img8.png)

That’s it.

#### *Download Source Code*

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=1475)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。