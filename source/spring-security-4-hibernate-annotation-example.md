# Spring Security 4 Hibernate Integration Annotation+XML Example

原文：[Spring Security 4 Hibernate Integration Annotation+XML Example](http://websystique.com/spring-security/spring-security-4-hibernate-annotation-example/)

译者：

校对：

This tutorial demonstrates integrating Hibernate with Spring Security 4 to perform database authentication, showing Annotation+XML configuration example in Spring 4 MVC application. Let’s get going.

In this post we will learn Spring Security `database authentication` using `Hibernate` annotation+xml based approach. Previous posts discussed about Spring Security **in-memory authentication**. But in real-world projects, credentials are often stored in database or LDAP. In this post we will go through a complete example of setting up Spring security and verifying credentials directly against database using Hibernate.

#### What changes compare to in-memory authentication discussed in previous posts?

Only major change is the authentication method itself. 

If you recall from previous post, below shown is the setup for **in-memory authentication**

```
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Autowired
	public void configureGlobalSecurity(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().withUser("bill").password("abc123").roles("USER");
		auth.inMemoryAuthentication().withUser("admin").password("root123").roles("ADMIN");
		auth.inMemoryAuthentication().withUser("dba").password("root123").roles("ADMIN","DBA");
	}
...
...

```

This will be changed to following to support **database authentication**:

```
@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Autowired
	@Qualifier("customUserDetailsService")
	UserDetailsService userDetailsService;
	
	@Autowired
	public void configureGlobalSecurity(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(userDetailsService);
	}


```

All the credentials are now stored in database, and will be accessible to Spring Security via `org.springframework.security.core.userdetails.UserDetailsService` implementations. We will provide an implementation of **UserDetailsService** which will eventually use our fully transactional user defined **userService** to access data from database.

That’s it. Rest of the setup for this post is (deja vu) usual Spring Security, Spring MVC and trivial Hibernate Setup which we have seen many times in previous tutorials.

Below is the full example code for this post. We have divided the responsibilities into separate layers(service/dao) to make it manageable.

------

Other interesting posts you may like

- [Secure Spring REST API using OAuth2](http://websystique.com/spring-security/secure-spring-rest-api-using-oauth2/)
- [AngularJS+Spring Security using Basic Authentication](http://websystique.com/spring-security/angularjs-basic-authentication-using-spring-security/)
- [Secure Spring REST API using Basic Authentication](http://websystique.com/spring-security/secure-spring-rest-api-using-basic-authentication/)
- [Spring MVC 4+Spring Security 4 + Hibernate Integration Example](http://websystique.com/springmvc/spring-mvc-4-and-spring-security-4-integration-example/)
- [Spring 4 Caching Annotations Tutorial](http://websystique.com/spring/spring-4-cacheable-cacheput-cacheevict-caching-cacheconfig-enablecaching-tutorial/)
- [Spring 4 Cache Tutorial with EhCache](http://websystique.com/spring/spring-4-cache-tutorial-with-ehcache/)
- [Spring 4 MVC+JPA2+Hibernate Many-to-many Example](http://websystique.com/springmvc/spring-4-mvc-jpa2-hibernate-many-to-many-example/)
- [Spring 4 Email Template Library Example](http://websystique.com/spring/spring-4-email-using-velocity-freemaker-template-library/)
- [Spring 4 Email With Attachment Tutorial](http://websystique.com/spring/spring-4-email-with-attachment-tutorial/)
- [Spring 4 Email Integration Tutorial](http://websystique.com/spring/spring-4-email-integration-tutorial/)
- [Spring MVC 4+JMS+ActiveMQ Integration Example](http://websystique.com/springmvc/spring-4-mvc-jms-activemq-annotation-based-example/)
- [Spring 4+JMS+ActiveMQ @JmsLister @EnableJms Example](http://websystique.com/spring/spring-4-jms-activemq-example-with-jmslistener-enablejms/)
- [Spring 4+JMS+ActiveMQ Integration Example](http://websystique.com/spring/spring-4-jms-activemq-example-with-annotations/)
- [Spring MVC 4+AngularJS Example](http://websystique.com/springmvc/spring-mvc-4-angularjs-example/)
- [Spring MVC 4+AngularJS Routing with UI-Router Example](http://websystique.com/springmvc/spring-4-mvc-angularjs-routing-example-using-ui-router/)
- [Spring MVC 4 HelloWorld – Annotation/JavaConfig Example](http://websystique.com/springmvc/spring-4-mvc-helloworld-tutorial-annotation-javaconfig-full-example/)
- [Spring MVC 4+Hibernate 4+MySQL+Maven integration example](http://websystique.com/springmvc/spring-4-mvc-and-hibernate4-integration-example-using-annotations/)
- [Spring 4 Hello World Example](http://websystique.com/spring/spring-4-hello-world-example-annotation-tutorial-full-example/)
- [Spring Security 4 Hello World Annotation+XML Example](http://websystique.com/spring-security/spring-security-4-hello-world-annotation-xml-example/)
- [Hibernate MySQL Maven Hello World Example (Annotation)](http://websystique.com/hibernate/hibernate-mysql-maven-hello-world-example-annotation/)
- [TestNG Hello World Example](http://websystique.com/java/testing/testng-hello-world-example/)
- [JAXB2 Helloworld Example](http://websystique.com/java/jaxb/jaxb-helloworld-example-marshal-unmarshal-example/)
- [Spring Batch- Read a CSV file and write to an XML file](http://websystique.com/springbatch/spring-batch-read-a-csv-file-and-write-to-an-xml-file/)

**Following technologies being used:**

- Spring 4.1.6.RELEASE
- Spring Security 4.0.1.RELEASE
- Hibernate 4.3.6.Final
- MySQL Server 5.6
- Maven 3
- JDK 1.7
- Tomcat 8.0.21
- Eclipse JUNO Service Release 2

Let’s begin.

#### Step 1: Project directory structure

Following will be the final project structure:

![SpringSecurityHibernateAnnoationExample_img00](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img00.png)
![SpringSecurityHibernateAnnoationExample_img01](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img01.png)

Let’s now add the content mentioned in above structure explaining each in detail.

#### Step 2: Update pom.xml to include required dependencies

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.websystique.springsecurity</groupId>
  <artifactId>SpringSecurityHibernateAnnotationExample</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <name>SpringSecurityHibernateAnnotationExample</name>
  
  	<properties>
		<springframework.version>4.1.6.RELEASE</springframework.version>
		<springsecurity.version>4.0.1.RELEASE</springsecurity.version>
		<hibernate.version>4.3.6.Final</hibernate.version>
		<mysql.connector.version>5.1.31</mysql.connector.version>
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
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
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

		<!-- Hibernate -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${hibernate.version}</version>
		</dependency>

		<!-- MySQL -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>${mysql.connector.version}</version>
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
						<warName>SpringSecurityHibernateAnnotationExample</warName>
						<failOnMissingWebXml>false</failOnMissingWebXml>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<finalName>SpringSecurityHibernateAnnotationExample</finalName>
	</build>
</project>

```

## Security Part

#### Step 3: Add Spring Security Configuration Class

The first and foremost step to add spring security in our application is to create **Spring Security Java Configuration**. This configuration creates a Servlet Filter known as the `springSecurityFilterChain` which is responsible for all the security (protecting the application URLs, validating submitted username and passwords, redirecting to the log in form, etc) within our application.

```
package com.websystique.springsecurity.configuration;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Autowired
	@Qualifier("customUserDetailsService")
	UserDetailsService userDetailsService;
	
	@Autowired
	public void configureGlobalSecurity(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(userDetailsService);
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

As mentioned before, only change is that instead of in-memory authentication, we are using **UserDetailService implementations for database-authentication**.

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
        <form-login  login-page="/login" 
                     username-parameter="ssoId" 
                     password-parameter="password" 
                     authentication-failure-url="/Access_Denied" />
        <csrf/>
    </http>
 
    <authentication-manager >
        <authentication-provider user-service-ref="customUserDetailsService"/>
    </authentication-manager>
     
    <beans:bean id="customUserDetailsService" class="com.websystique.springsecurity.service.CustomUserDetailsService" />
    
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

#### Step 5: Define UserDetailsService implementation

This service is responsible for providing authentication details to Authentication Manager. It implements Spring’s `UserDetailsService` interface, which contains only one method **loadUserByUsername**, taking username[ssoId in our example] and returns a `org.springframework.security.core.userdetails.User` object. We will populate this object using our own UserService which gets data from db using UserDao object.

```
package com.websystique.springsecurity.service;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.websystique.springsecurity.model.User;
import com.websystique.springsecurity.model.UserProfile;

@Service("customUserDetailsService")
public class CustomUserDetailsService implements UserDetailsService{

	@Autowired
	private UserService userService;
	
	@Transactional(readOnly=true)
	public UserDetails loadUserByUsername(String ssoId)
			throws UsernameNotFoundException {
		User user = userService.findBySso(ssoId);
		System.out.println("User : "+user);
		if(user==null){
			System.out.println("User not found");
			throw new UsernameNotFoundException("Username not found");
		}
			return new org.springframework.security.core.userdetails.User(user.getSsoId(), user.getPassword(), 
				 user.getState().equals("Active"), true, true, true, getGrantedAuthorities(user));
	}

	
	private List<GrantedAuthority> getGrantedAuthorities(User user){
		List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
		
		for(UserProfile userProfile : user.getUserProfiles()){
			System.out.println("UserProfile : "+userProfile);
			authorities.add(new SimpleGrantedAuthority("ROLE_"+userProfile.getType()));
		}
		System.out.print("authorities :"+authorities);
		return authorities;
	}
	
}


```

## SpringMVC Part

#### Step 6: Add Controller

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

#### Step 7: Add SpringMVC Configuration Class

```
package com.websystique.springsecurity.configuration;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.websystique.springsecurity")
public class HelloWorldConfiguration extends WebMvcConfigurerAdapter {
	
	@Bean(name="HelloWorld")
	public ViewResolver viewResolver() {
		InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
		viewResolver.setViewClass(JstlView.class);
		viewResolver.setPrefix("/WEB-INF/views/");
		viewResolver.setSuffix(".jsp");

		return viewResolver;
	}

	/*
     * Configure ResourceHandlers to serve static resources like CSS/ Javascript etc...
     *
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("/static/");
    }
}


```

#### Step 8: Add Initializer class

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

## DAO, Model & Service Part

#### Step 9: Create Model classes

A User can have multiple roles [DBA,ADMIN,USER]. And a Role can be assigned to more than one user. Hence there is a `Many-To-Many relationship` between a User and UserProfile[role]. We kept this relationship uni-directional [User to UserProfile] as we are only interested in finding Roles for a give user (and not vice-versa). We will be using Many-To-Many association using Join table.

```
package com.websystique.springsecurity.model;

import java.util.HashSet;
import java.util.Set;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;
import javax.persistence.Table;

@Entity
@Table(name="APP_USER")
public class User {

	@Id @GeneratedValue(strategy=GenerationType.IDENTITY)
	private int id;

	@Column(name="SSO_ID", unique=true, nullable=false)
	private String ssoId;
	
	@Column(name="PASSWORD", nullable=false)
	private String password;
		
	@Column(name="FIRST_NAME", nullable=false)
	private String firstName;

	@Column(name="LAST_NAME", nullable=false)
	private String lastName;

	@Column(name="EMAIL", nullable=false)
	private String email;

	@Column(name="STATE", nullable=false)
	private String state=State.ACTIVE.getState();

	@ManyToMany(fetch = FetchType.EAGER)
	@JoinTable(name = "APP_USER_USER_PROFILE", 
             joinColumns = { @JoinColumn(name = "USER_ID") }, 
             inverseJoinColumns = { @JoinColumn(name = "USER_PROFILE_ID") })
	private Set<UserProfile> userProfiles = new HashSet<UserProfile>();

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getSsoId() {
		return ssoId;
	}

	public void setSsoId(String ssoId) {
		this.ssoId = ssoId;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public String getState() {
		return state;
	}

	public void setState(String state) {
		this.state = state;
	}

	public Set<UserProfile> getUserProfiles() {
		return userProfiles;
	}

	public void setUserProfiles(Set<UserProfile> userProfiles) {
		this.userProfiles = userProfiles;
	}

	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + id;
		result = prime * result + ((ssoId == null) ? 0 : ssoId.hashCode());
		return result;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (!(obj instanceof User))
			return false;
		User other = (User) obj;
		if (id != other.id)
			return false;
		if (ssoId == null) {
			if (other.ssoId != null)
				return false;
		} else if (!ssoId.equals(other.ssoId))
			return false;
		return true;
	}

	@Override
	public String toString() {
		return "User [id=" + id + ", ssoId=" + ssoId + ", password=" + password
				+ ", firstName=" + firstName + ", lastName=" + lastName
				+ ", email=" + email + ", state=" + state + ", userProfiles=" + userProfiles +"]";
	}

	
}


```

```
package com.websystique.springsecurity.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="USER_PROFILE")
public class UserProfile {

	@Id @GeneratedValue(strategy=GenerationType.IDENTITY)
	private int id;	

	@Column(name="TYPE", length=15, unique=true, nullable=false)
	private String type = UserProfileType.USER.getUserProfileType();
	
	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getType() {
		return type;
	}

	public void setType(String type) {
		this.type = type;
	}


	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + id;
		result = prime * result + ((type == null) ? 0 : type.hashCode());
		return result;
	}

	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (!(obj instanceof UserProfile))
			return false;
		UserProfile other = (UserProfile) obj;
		if (id != other.id)
			return false;
		if (type == null) {
			if (other.type != null)
				return false;
		} else if (!type.equals(other.type))
			return false;
		return true;
	}

	@Override
	public String toString() {
		return "UserProfile [id=" + id + ",  type=" + type	+ "]";
	}
	

}

```

```
package com.websystique.springsecurity.model;

public enum UserProfileType {
	USER("USER"),
	DBA("DBA"),
	ADMIN("ADMIN");
	
	String userProfileType;
	
	private UserProfileType(String userProfileType){
		this.userProfileType = userProfileType;
	}
	
	public String getUserProfileType(){
		return userProfileType;
	}
	
}

```

```
package com.websystique.springsecurity.model;

public enum State {

	ACTIVE("Active"),
	INACTIVE("Inactive"),
	DELETED("Deleted"),
	LOCKED("Locked");
	
	private String state;
	
	private State(final String state){
		this.state = state;
	}
	
	public String getState(){
		return this.state;
	}

	@Override
	public String toString(){
		return this.state;
	}

	public String getName(){
		return this.name();
	}


}

```

#### Step 10: Create Dao Layer

```
package com.websystique.springsecurity.dao;

import java.io.Serializable;

import java.lang.reflect.ParameterizedType;

import org.hibernate.Criteria;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;

public abstract class AbstractDao<PK extends Serializable, T> {
	
	private final Class<T> persistentClass;
	
	@SuppressWarnings("unchecked")
	public AbstractDao(){
		this.persistentClass =(Class<T>) ((ParameterizedType) this.getClass().getGenericSuperclass()).getActualTypeArguments()[1];
	}
	
	@Autowired
	private SessionFactory sessionFactory;

	protected Session getSession(){
		return sessionFactory.getCurrentSession();
	}

	@SuppressWarnings("unchecked")
	public T getByKey(PK key) {
		return (T) getSession().get(persistentClass, key);
	}

	public void persist(T entity) {
		getSession().persist(entity);
	}

	public void delete(T entity) {
		getSession().delete(entity);
	}
	
	protected Criteria createEntityCriteria(){
		return getSession().createCriteria(persistentClass);
	}

}

```

```
package com.websystique.springsecurity.dao;

import com.websystique.springsecurity.model.User;

public interface UserDao {

	User findById(int id);
	
	User findBySSO(String sso);
	
}

```

```
package com.websystique.springsecurity.dao;

import org.hibernate.Criteria;
import org.hibernate.criterion.Restrictions;
import org.springframework.stereotype.Repository;

import com.websystique.springsecurity.model.User;

@Repository("userDao")
public class UserDaoImpl extends AbstractDao<Integer, User> implements UserDao {

	public User findById(int id) {
		return getByKey(id);
	}

	public User findBySSO(String sso) {
		Criteria crit = createEntityCriteria();
		crit.add(Restrictions.eq("ssoId", sso));
		return (User) crit.uniqueResult();
	}

	
}

```

#### Step 11: Create Service Layer

```
package com.websystique.springsecurity.service;

import com.websystique.springsecurity.model.User;

public interface UserService {

	User findById(int id);
	
	User findBySso(String sso);
	
}

```

```
package com.websystique.springsecurity.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.websystique.springsecurity.dao.UserDao;
import com.websystique.springsecurity.model.User;

@Service("userService")
@Transactional
public class UserServiceImpl implements UserService{

	@Autowired
	private UserDao dao;

	public User findById(int id) {
		return dao.findById(id);
	}

	public User findBySso(String sso) {
		return dao.findBySSO(sso);
	}

}

```

## Hibernate Configuration Part

#### Step 12: Create Hibernate Configuration

Hibernate configuration class contains @Bean methods for DataSource, SessionFactory & Transaction Manager. Datasource properties are taken from
application.properties file and contains connection details for MySQL database.

```
package com.websystique.springsecurity.configuration;

import java.util.Properties;

import javax.sql.DataSource;

import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.orm.hibernate4.HibernateTransactionManager;
import org.springframework.orm.hibernate4.LocalSessionFactoryBean;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableTransactionManagement
@ComponentScan({ "com.websystique.springsecurity.configuration" })
@PropertySource(value = { "classpath:application.properties" })
public class HibernateConfiguration {

    @Autowired
    private Environment environment;

    @Bean
    public LocalSessionFactoryBean sessionFactory() {
        LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
        sessionFactory.setDataSource(dataSource());
        sessionFactory.setPackagesToScan(new String[] { "com.websystique.springsecurity.model" });
        sessionFactory.setHibernateProperties(hibernateProperties());
        return sessionFactory;
     }
	
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(environment.getRequiredProperty("jdbc.driverClassName"));
        dataSource.setUrl(environment.getRequiredProperty("jdbc.url"));
        dataSource.setUsername(environment.getRequiredProperty("jdbc.username"));
        dataSource.setPassword(environment.getRequiredProperty("jdbc.password"));
        return dataSource;
    }
    
    private Properties hibernateProperties() {
        Properties properties = new Properties();
        properties.put("hibernate.dialect", environment.getRequiredProperty("hibernate.dialect"));
        properties.put("hibernate.show_sql", environment.getRequiredProperty("hibernate.show_sql"));
        properties.put("hibernate.format_sql", environment.getRequiredProperty("hibernate.format_sql"));
        return properties;        
    }
    
    @Bean
    @Autowired
    public HibernateTransactionManager transactionManager(SessionFactory s) {
       HibernateTransactionManager txManager = new HibernateTransactionManager();
       txManager.setSessionFactory(s);
       return txManager;
    }
}

```

`application.properties`

```
jdbc.driverClassName = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/websystique
jdbc.username = myuser
jdbc.password = mypassword
hibernate.dialect = org.hibernate.dialect.MySQLDialect
hibernate.show_sql = true
hibernate.format_sql = true

```

## Views Part

#### Step 13: Add Views

`login.jsp`

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
							<input type="hidden" name="${_csrf.parameterName}"  value="${_csrf.token}" />
								
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

As you can see, the CSRF parameters are accessed using EL expressions in your JSP, you may additionally prefer to force EL expressions to be evaluated, by adding following to the top of your JSP:

```
<%@ page isELIgnored="false"%>

```

`welcome.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"  pageEncoding="ISO-8859-1"%>
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

## Database Schema Part

#### Step 14: Create Database Schema & Populate dummy data

As already explained in Step 9, there is a Many-To-Many relationship between User and UserProfile. Many-To-Many relationship is maintained using Join table, and it is unidirectional(from User to UserProfile) as the other case (finding all users with Admin role e.g. does not make much sense in this example)

```
/*All User's are stored in APP_USER table*/
create table APP_USER (
   id BIGINT NOT NULL AUTO_INCREMENT,
   sso_id VARCHAR(30) NOT NULL,
   password VARCHAR(100) NOT NULL,
   first_name VARCHAR(30) NOT NULL,
   last_name  VARCHAR(30) NOT NULL,
   email VARCHAR(30) NOT NULL,
   state VARCHAR(30) NOT NULL, 	
   PRIMARY KEY (id),
   UNIQUE (sso_id)
);
 
/* USER_PROFILE table contains all possible roles */
create table USER_PROFILE(
   id BIGINT NOT NULL AUTO_INCREMENT,
   type VARCHAR(30) NOT NULL,
   PRIMARY KEY (id),
   UNIQUE (type)
);
 
/* JOIN TABLE for MANY-TO-MANY relationship*/ 
CREATE TABLE APP_USER_USER_PROFILE (
    user_id BIGINT NOT NULL,
    user_profile_id BIGINT NOT NULL,
    PRIMARY KEY (user_id, user_profile_id),
    CONSTRAINT FK_APP_USER FOREIGN KEY (user_id) REFERENCES APP_USER (id),
    CONSTRAINT FK_USER_PROFILE FOREIGN KEY (user_profile_id) REFERENCES USER_PROFILE (id)
);

/* Populate USER_PROFILE Table */
INSERT INTO USER_PROFILE(type)
VALUES ('USER');

INSERT INTO USER_PROFILE(type)
VALUES ('ADMIN');

INSERT INTO USER_PROFILE(type)
VALUES ('DBA');

/* Populate APP_USER Table */
INSERT INTO APP_USER(sso_id, password, first_name, last_name, email, state)
VALUES ('bill','abc123', 'Bill','Watcher','bill@xyz.com', 'Active');

INSERT INTO APP_USER(sso_id, password, first_name, last_name, email, state)
VALUES ('danny','abc124', 'Danny','Theys','danny@xyz.com', 'Active');

INSERT INTO APP_USER(sso_id, password, first_name, last_name, email, state)
VALUES ('sam','abc125', 'Sam','Smith','samy@xyz.com', 'Active');

INSERT INTO APP_USER(sso_id, password, first_name, last_name, email, state)
VALUES ('nicole','abc126', 'Nicole','warner','nicloe@xyz.com', 'Active');

INSERT INTO APP_USER(sso_id, password, first_name, last_name, email, state)
VALUES ('kenny','abc127', 'Kenny','Roger','kenny@xyz.com', 'Active');

/* Populate JOIN Table */
INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile  
  where user.sso_id='bill' and profile.type='USER';

INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile
  where user.sso_id='danny' and profile.type='USER';

INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile
  where user.sso_id='sam' and profile.type='ADMIN';

INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile
  where user.sso_id='nicole' and profile.type='DBA';

INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile  
  where user.sso_id='kenny' and profile.type='ADMIN';

INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile  
  where user.sso_id='kenny' and profile.type='DBA';


```

We have created tables for User, UserProfile and Join table(to manage Many-To-Many relation ship). We populated following users and roles:

```
Bill,Danny : USER
Sam        : ADMIN
Nicole     : DBA
Kenny      : ADMIN, DBA

```

Below are the snapshots of MySQL database at this moment.

![SpringSecurityHibernateAnnoationExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img2.png)
![SpringSecurityHibernateAnnoationExample_img3](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img3.png)
![SpringSecurityHibernateAnnoationExample_img4](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img4.png)

Now we will start our web application and try to login & access different part of applications, using different users.

#### Step 15: Build and Deploy the application

Now build the war (either by eclipse/m2eclipse) or via maven command line( `mvn clean install`). Deploy the war to a Servlet 3.0 container . Since here i am using Tomcat, i will simply put this war file into `tomcat webapps folder` and click on `start.bat` inside tomcat bin directory.

Run the application
Open browser and goto **localhost:8080/SpringSecurityHibernateAnnotationExample/**

![SpringSecurityHibernateAnnoationExample_img5](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img5.png)

Now try to access **localhost:8080/SpringSecurityHibernateAnnotationExample/admin**, you will be prompted for login

![SpringSecurityHibernateAnnoationExample_img6](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img6.png)

Provide credentials of a ‘USER’ role.

![SpringSecurityHibernateAnnoationExample_img7](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img7.png)

Submit, you will see AccessDenied Page

![SpringSecurityHibernateAnnoationExample_img8](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img8.png)

Now logout and try to access admin page again

![SpringSecurityHibernateAnnoationExample_img9](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img9.png)

Provide wrong password

![SpringSecurityHibernateAnnoationExample_img10](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img10.png)

Provide proper admin role credentials and login

![SpringSecurityHibernateAnnoationExample_img11](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img11.png)

Now try to access db page on **localhost:8080/SpringSecurityHibernateAnnoationExample/db**, you will get AccessDenied page.

![SpringSecurityHibernateAnnoationExample_img12](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img12.png)

Now logout, provide a DBA credentials, and access admin page on **localhost:8080/SpringSecurityHibernateAnnoationExample/admin**
![SpringSecurityHibernateAnnoationExample_img13](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img13.png)

Logout. Enough for today.

![SpringSecurityHibernateAnnoationExample_img14](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityHibernateAnnoationExample_img14.png)

#### *Download Source Code*

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=1408)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。