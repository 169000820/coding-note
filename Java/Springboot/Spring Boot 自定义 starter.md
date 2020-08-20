# Spring Boot 自定义starter

Spring Boot 之所以能够快速开发就是因为有各种 starter。starter 帮我们完成了许多自动化配置。

## 核心知识

stater 的核心就是条件注解@Conditional， 当 classpath 下存在某个 class 时，某个配置才会生效。

## 定义一个自己的 starter

### 定义 starter

平时里见到的所谓 starter，其实就是一个普普通通的 Maven 项目。所以定义一个 starter，首先得创建一个 Maven 项目，项目创建完成后，需要添加 starter 得自动化配置类，即：

```xml
<denpendency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
    <!-- 版本号自己定义 -->
</denpendency>
```

配置完成后，创建一个配置类，接受 application.properties 中注入的值，代码如下：

```java
@ConfigurationProperties(prefix="book") // 如果没加 @EnableConfigurationProperties注解，编译会报错，不用担心，继续跟着走
public class Book {
    private String name;
    private String author;
    private Double price;
    // 省略 getter/setter
}
```

然后再定义一个 BookService，定义一个 info 方法：

```java
public class BookService {
    private String name;
    private String author;
    private Double price;
    
    // 构造方法
    public BookService(String name, String author, Double price) {
        this.name = name;
        this.author = author;
        this.price = price;
    }
    // 省略 getter/setter
    public String info() {
        return "Book(name="+name+",age="+age+",price="+price+")";
    }
}
```

**接下来就是重点了，便是创建自定义配置类**：

```java
@Configuration
@EnableConfigurationProperties(Book.class)
@ConditionalOnClass(BookService.class)
public class BookServiceAutoConfiguration {
    @Autowired
    Book book;
    
    @Bean
    BookService bookService() {
        return new BookService(Book.getName, Book.getAge, Book.getPrice);
    }
}
```

下面一一对其中的注解进行解释：

* <code>@Configuration</code> - 这个注解代表了这个类是配置类。

* `@EnableConfigurationProperties` - 这个注解代表了让之前 Book 类上的注解生效， 能够让配置的属性注入到 Book 中。

* `@ConditionalOnClass` - 表示当前项目 classpath 下存在 BookService 时， 后面的配置才会生效

* 自动配置类首先会注入 Book，数据来源于 application.properties
* 然后提供一个 BookService 对象，将 Book 的值注入进去

经过一顿操作后，自动化配置类也就算是完成了，接下来便创建一个 spring.factories 文件，这个文件乍一看没听说过，但是却是非常有必要的。Spring Boot 项目的启动类都有一个 @SpringBootApplication 注解，这是一个组合注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    
}
```

其中便有一个 @EnableAutoConfiguration 注解，其定义如下： 

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

它会自动导入 AutoConfigurationImportSelector 类， 这个类会自动去读取一个名 spring.factories 的文件， 而在这个文件中则需要定义需要加载的自动化配置类。任意一个 starter 下都会有这个文件。所以我们定义一个 starter 也会需要这么一个文件， 首先在 resources 目录下创建 META-INF 文件夹， 然后再文件夹下面创建 spring.factories 文件：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.siji.mystarter.BookServiceAutoConfiguration // 指定创建的自动化配置类路径
```

### 本地安装

运行 mvn install 命令或者右击 IDEA 右边窗口的 Maven Projects 即可，这样这个 starter 就安装到我们的本地仓库了。

### 使用 starter

创建一个普通的 Spring Boot 工程，加入自定义的 starter 依赖：

```xml
<dependency>
    <groupId>com.siji</groupId>
    <artifactId>mystarter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

由于上面引入了自定义 starter 依赖，所以会默认提供一个 BookService 的实例可以使用，并且可以在application.properties 中配置 Book 类相关信息：

```yml
book:
 name: 三国演义
 author: 罗贯中
 price: 100.00
```

然后即可进行单元测试：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTest {
    
    @Autowired
    BookService bookService;
    @Test
    pubic void myStarterTest() {
        System.out.println(bookService.info())
        // 输出 Book(name="三国演义", author="罗贯中", price: 100.00)
    }
}
```



