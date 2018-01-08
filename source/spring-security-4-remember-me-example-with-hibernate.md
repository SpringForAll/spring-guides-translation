# Spring Security 4 Remember Me Example with Hibernate

原文：[Spring Security 4 Remember Me Example with Hibernate](http://websystique.com/spring-security/spring-security-4-remember-me-example-with-hibernate/)

译者：

校对：

This tutorial shows you **Remember-Me authentication** using Spring Security 4 with Hibernate. Let’s get going.

In `Remember-me` or **persistent-login authentication**, Applications remember the identity of user between sessions. Basically, during login, when you ask for Remember-Me support, application will **send a cookie to the browser** during login. This cookie will be stored at browser side and will remain there for certain period(defined by cookie lifetime). Next time when you try to access the application, browser will **detect the cookie** (if still valid) and user will be automatically logged in, without providing userid/password e.g.

Spring Security provides two implementations for Remember-Me :

- Simple Hash-Based Token Approach : It uses hashing to preserve the security of cookie-based tokens
- Persistent Token Approach : It uses a database or other persistent storage mechanism to store the generated tokens

In this post, we will be discussing about **Persistent Token Approach**

**Modifications compare to normal login posts:**

**1. **With Persistent Token Approach, the database should contain a **persistent_logins** table, created using the following SQL (or equivalent):

```
CREATE TABLE persistent_logins (
    username VARCHAR(64) NOT NULL,
    series VARCHAR(64) NOT NULL,
    token VARCHAR(64) NOT NULL,
    last_used TIMESTAMP NOT NULL,
    PRIMARY KEY (series)
);

```

This table contains the username, last_used timestamp of active remember-me, security token and series information which are internals of Spring Bcrypt implementation. For further details, please refer [here](http://jaspan.com/improved_persistent_login_cookie_best_practice).

**2. **Configure Remember-Me in Spring Security

```
package com.websystique.springsecurity.configuration;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;
import org.springframework.security.web.authentication.rememberme.PersistentTokenRepository;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Autowired
	@Qualifier("customUserDetailsService")
	UserDetailsService userDetailsService;
	
	@Autowired
	DataSource dataSource;
	
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
	  	.and().rememberMe().rememberMeParameter("remember-me").tokenRepository(persistentTokenRepository()).tokenValiditySeconds(86400)
	  	.and().csrf()
	  	.and().exceptionHandling().accessDeniedPage("/Access_Denied");
	}
	
	@Bean
	public PersistentTokenRepository persistentTokenRepository() {
		JdbcTokenRepositoryImpl tokenRepositoryImpl = new JdbcTokenRepositoryImpl();
		tokenRepositoryImpl.setDataSource(dataSource);
		return tokenRepositoryImpl;
	}
}

```

Notice how we called **rememberMe()** to configure Remember-Me authenticationm providing HTTP parameter name attached with remember-me checkbox from view(we will see in view). We have also specified the tokenRepository(where the token will be stored) to be used, and how long (in seconds) this token remains valid ( we have provided 1 day). To configure Repository itself we have injected a DataSource here.

This is all you need to activate Remember-Me in your Spring Security based application.

Optionally, you can use [Spring Security Built-in expressions](http://docs.spring.io/spring-security/site/docs/4.0.1.RELEASE/reference/htmlsingle/#el-common-built-in) along with Spring security tags in your view to customize/show/hide specific logic based on Remember-Me or full-authentication.

**Note :** In case you are not willing to use JDBC, may be because rest of your application is Hibernate based, have a look at [Spring MVC 4 + Spring Security 4 + Hibernate Example](http://websystique.com/springmvc/spring-mvc-4-and-spring-security-4-integration-example/) post, where i’ve created a Hibernate based implementation of PersistentTokenRepository, named as HibernateTokenRepositoryImpl.

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
     
    <remember-me 
        remember-me-parameter="remember-me"
        remember-me-cookie="remember-me"
        token-validity-seconds="86400"
	data-source-ref="dataSource" />

    <beans:bean id="customUserDetailsService" class="com.websystique.springsecurity.service.CustomUserDetailsService" />
    
</beans:beans>

```

Complete code example is shown below.

------

#### Complete Example

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
![SpringSecurityRememberMeAnnotationExample_img00](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img00.png)
![SpringSecurityRememberMeAnnotationExample_img01](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img01.png)

#### Step 2: Update pom.xml to include required dependencies

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.websystique.springsecurity</groupId>
  <artifactId>SpringSecurityRememberMeAnnotationExample</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>

  <name>SpringSecurityRememberMeAnnotationExample</name>

 	<properties>
		<springframework.version>4.1.6.RELEASE</springframework.version>
		<springsecurity.version>4.0.1.RELEASE</springsecurity.version>
		<hibernate.version>4.3.6.Final</hibernate.version>
		<mysql.version>5.1.31</mysql.version>
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
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-taglibs</artifactId>
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
			<version>${mysql.version}</version>
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
						<warName>SpringSecurityRememberMeAnnotationExample</warName>
						<failOnMissingWebXml>false</failOnMissingWebXml>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
		<finalName>SpringSecurityRememberMeAnnotationExample</finalName>
	</build>
</project>

```

## Database Schema Part

#### Step 3: Create Database Schema & Populate dummy data

```
/* For Remember-Me token storage purpose */
CREATE TABLE persistent_logins (
    username VARCHAR(64) NOT NULL,
    series VARCHAR(64) NOT NULL,
    token VARCHAR(64) NOT NULL,
    last_used TIMESTAMP NOT NULL,
    PRIMARY KEY (series)
);


/*All User's gets stored in APP_USER table*/
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

/* Populate one Admin User. We need only one user to demonstrate this example. You can add more as done in previous posts*/
INSERT INTO APP_USER(sso_id, password, first_name, last_name, email, state)
VALUES ('sam','abc125', 'Sam','Smith','samy@xyz.com', 'Active');

/* Populate JOIN Table */
INSERT INTO APP_USER_USER_PROFILE (user_id, user_profile_id)
  SELECT user.id, profile.id FROM app_user user, user_profile profile
  where user.sso_id='sam' and profile.type='ADMIN';

```

## Security Part

#### Step 4: Add Spring Security Configuration Class

```
package com.websystique.springsecurity.configuration;

import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;
import org.springframework.security.web.authentication.rememberme.PersistentTokenRepository;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	@Autowired
	@Qualifier("customUserDetailsService")
	UserDetailsService userDetailsService;
	
	@Autowired
	DataSource dataSource;
	
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
	  	.and().rememberMe().rememberMeParameter("remember-me").tokenRepository(persistentTokenRepository()).tokenValiditySeconds(86400)
	  	.and().csrf()
	  	.and().exceptionHandling().accessDeniedPage("/Access_Denied");
	  	
	}
	
	@Bean
	public PersistentTokenRepository persistentTokenRepository() {
		JdbcTokenRepositoryImpl tokenRepositoryImpl = new JdbcTokenRepositoryImpl();
		tokenRepositoryImpl.setDataSource(dataSource);
		return tokenRepositoryImpl;
	}
}

```

#### Step 5: Register the springSecurityFilter with war

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

#### Step 6: Define UserDetailsService implementation

This service is responsible for providing authentication details to Authentication Manager.

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

#### Step 7: Add Controller

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

#### Step 8: Add SpringMVC Configuration Class

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

#### Step 9: Add Initializer class

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

## Hibernate Configuration Part

#### Step 10: Create Hibernate Configuration

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

## DAO, Model & Service Part

#### Step 11: Create Model classes

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

#### Step 12: Create Dao Layer

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

#### Step 13: Create Service Layer

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

## Views Part

#### Step 14: Add Views

`login.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
		<title>HelloWorld Login page</title>
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
                            <div class="input-group input-sm">
                              <div class="checkbox">
                                <label><input type="checkbox" id="rememberme" name="remember-me"> Remember Me</label>  
                              </div>
							</div>
							<input type="hidden" name="${_csrf.parameterName}"
								value="${_csrf.token}" />
								
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

`admin.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>

<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Admin page</title>
</head>
<body>
	Dear <strong>${user}</strong>, Welcome to Admin Page.
	
	<sec:authorize access="isFullyAuthenticated()">
		<label><a href="#">Create New User</a> | <a href="#">View existing Users</a></label>
	</sec:authorize>
	<sec:authorize access="isRememberMe()">
		<label><a href="#">View existing Users</a></label>
	</sec:authorize>
 
	<a href="<c:url value="/logout" />">Logout</a>
</body>
</html>

```

Below mentioned views are not required for this post, just mentioned here for completion.

`welcome.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>Welcome page</title>
	<link href="<c:url value='/static/css/bootstrap.css' />" rel="stylesheet"></link>
	<link href="<c:url value='/static/css/app.css' />" rel="stylesheet"></link>
</head>
<body>
	<div class="success">
		Greeting : ${greeting}
		This is a welcome page.
	</div>
</body>
</html>

```

`dba.jsp`

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"  pageEncoding="ISO-8859-1"%>
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

#### Step 15: Build and Deploy the application

Now build the war (either by eclipse/m2eclipse) or via maven command line( `mvn clean install`). Deploy the war to a Servlet 3.0 container . Since here i am using Tomcat, i will simply put this war file into `tomcat webapps folder` and click on `start.bat` inside tomcat bin directory.

Run the application
Open browser and goto **localhost:8080/SpringSecurityRememberMeAnnotationExample/admin**
you will be prompted for login.

![SpringSecurityRememberMeAnnotationExample_img2](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img2.png)

Login with Remember-Me checked in. You will get admin view

![SpringSecurityRememberMeAnnotationExample_img3](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img3.png)

Verify Cookies. Open Google Chrome Settings/Show advanced settings/Content Setting/All cookies and site data, fill localhost in serachbox. You will find two cookies, one for current session [JSESSIONID], and one for Remember-Me authentication [remember-me].

![SpringSecurityRememberMeAnnotationExample_img4](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img4.png)

Click on remember-me cookie to view the details. You can see that cookie will expire in one day(as was set in Security Configuration)

![SpringSecurityRememberMeAnnotationExample_img5](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img5.png)

Now verify persistent_logins table in database.There should be an entry in there for current user.

![SpringSecurityRememberMeAnnotationExample_img6](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img6.png)

Now logout.

![SpringSecurityRememberMeAnnotationExample_img7](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img7.png)

Try to goto **localhost:8080/SpringSecurityRememberMeAnnotationExample/admin** again, You should not be prompted for login.

![SpringSecurityRememberMeAnnotationExample_img8](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img8.png)

You can also see that ‘Create New User’ option is not available as that one is only available when user is full authenticated(using login/password e.g.)

Now remove the remember-me cookie from chrome(as shown before), and try to access **localhost:8080/SpringSecurityRememberMeAnnotationExample/admin** again, You should be prompted for login.

![SpringSecurityRememberMeAnnotationExample_img9](http://websystique.com/wp-content/uploads/2015/07/SpringSecurityRememberMeAnnotationExample_img9.png)

That’s it. [Next post](http://websystique.com/spring-security/spring-security-4-method-security-using-preauthorize-postauthorize-secured-el/) shows Method level security using Spring Security @PreAuthorize, @PostAuthorize, @Secured and Spring EL expressions.

#### *Download Source Code*

[Download Now!](http://websystique.com/?smd_process_download=1&download_id=1462)

> 本文由spring4all.com翻译小分队创作，采用[知识共享-署名-非商业性使用-相同方式共享 4.0 国际 许可](http://creativecommons.org/licenses/by-nc-sa/4.0/) 协议进行许可。