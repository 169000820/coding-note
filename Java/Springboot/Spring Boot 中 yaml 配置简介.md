# Spring Boot 中 yaml 配置简介

在 Spring Boot 中， 配置文件一般有两种形式,，properties -> .properties 或者 yaml -> .yml 格式, 一般情况下， 两者可以任意使用。

在 Spring Boot 中，配置文件可以放在不同的位置：

* 项目根目录下的 config 目录中
* 项目根目录下
* classpath 下的 config 根目录下
* classpath 目录下

这几个位置的优先级别顺序由上往下，如果一个项目中出现了多个配置文件，则以优先级别高的为准。当然了，application.yml 也并不是一定要叫 application.yml，这个是可以进行配置的。命令如下：

```java
java -jar app.jar --spring.config.name=app
```

这种配置就会告诉系统启动后按照优先级别位置顺序从上往下去找 app.yml 文件。当然， 这个配置文件存放的位置也是可以改变的，示例如下：

```java
spring.config.name=classpath:/myconfig/ // 这种写法表示重新定义配置文件的位置，会覆盖系统默认位置
spirng.config.additional -location  // 表示再添加存放配置文件的位置，不会覆盖原位置，且优先级高于原位置
```

优点：

* yaml 内配置文件有序，支持多种语法规则 

缺点：

* 不支持 @PropertySource 注解等。