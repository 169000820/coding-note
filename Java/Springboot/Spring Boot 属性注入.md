# Spring Boot 属性注入

## 普通属性注入

Spring Boot 源于 Spring，所以 Spring Boot 中也有 Spring 的属性注入。在 Spring Boot 中，会默认加载 application.properties 文件，所以一些比较简单的配置可以写在这个配置文件中：

* 首先在 application.properties 配置文件中定义一些属性：

  ```properties
  book.name=西游记 # 这里获取的值就是你写入的值，如果你写了 “” 号，则获取的值也会有 “”， --注意
  book.author=吴承恩
  book.price=33.80
  ```

* 创建一个 Book 类，便可以进行属性注入：

  ```java
  @Component
  puclic class Book {
      @Value("${book.name}")
      private String name;
      @Value("${book.author}")
      private String author;
      @Value("${book.price}")
      private double price;
      // 省略 getter/setter
  }
  ```

**注意：** 需要在类上加上 @Component 注解，代表这个类对象会交给 Spring 容器去进行管理。如果没加这个注解，则无法从 Spring 容器中获取值。

但一般来说，application.properties 中存放的都是一些系统级别的配置，而像这种自定义的注解一般不会推荐放在这个文件中，解决办法：可以自定义 properties 文件来保存自定义配置，文件名一般见名知意。示例：在 resources/config 目录下定义一个 book.properties 文件：

```properties
book.name=西游记
book.author=吴承恩
book.price=33.80
```

在 Spring 中，可以通过 @PropertySource 来引入自定义配置：

```java
@Component
@PropertySource("classpath:config/book.properties")
public class Book {
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;
    @Value("${book.price}")
    private double price;
    // 省略 getter/setter
}
```

这样项目启动时便会自动加载 book.properties 文件。

## 类型安全的属性注入

在 Spring Boot 中引入了类型安全的属性注入，采用 Spring 中的配置方式，当属性较多时，工作量会比较大，容易出错。

使用类型安全的属性注入可以有效的解决这个问题：

```java
@Component
@PropertySource("classpath:config/book.properties")
@ConfigurationProperties(prefix = "book")
public class Book {
    private String name;
    private String author;
    private double price;
    // 省略 getter/setter
}
```

@ConfigurationProperties(prefix = "book") 这个注解配置了属性的前缀，这样会将 Spring 容器中对应的数据自动注入到对象对应的属性，就不用一个个通过 @Value 进行注解了。

