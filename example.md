`BookmarkRepository`有一个类似finder的方法，但这个方法依赖`Bookmark`实体对应`Account`关系的`username`属性，最终它需要一种连接，它会生成对应的查询：

```sql
SELECT b from Bookmark b WHERE b.account.username = :username
```

我们的应用程序将使用Spring Boot，Spring Boot的应用最少会有一个`public static void main`作为主程序入口，并且该入口被`@SpringBootApplication`注解。不论如何，它都会告诉Spring Boot这些信息，我们的`Application`类也是一个很好的地方作为其他零碎模块的支点，和`@Bean`注解定义一样，下边是我们的示例：

`Application.java`的类就像这样：

```java
package bookmarks;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.Arrays;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    CommandLineRunner init(AccountRepository accountRepository,
            BookmarkRepository bookmarkRepository) {
        return (evt) -> Arrays.asList(
                "jhoeller,dsyer,pwebb,ogierke,rwinch,mfisher,mpollack,jlong".split(","))
                .forEach(a -> {
                        Account account = accountRepository.save(
                            new Account(a,"password"));
                        bookmarkRepository.save(
                            new Bookmark(account,"http://bookmark.com/1/" + a, "A description"));
                        bookmarkRepository.save(
                            new Bookmark(account,"http://bookmark.com/2/" + a, "A description"));
                }
            );
    }

}
```



