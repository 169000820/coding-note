# SpringMvc 基本概念

## SpringMvc 能让我们做什么(好处)

* 让我们能够非常简单的就设计出干净的 web 层
* 进行更简洁的 web 层开发
* 天生与 spring 框架集成(如 ioc 容器, aop容器等)
* 提供强大的约定大于配置的契约式编程支持
* 能简单的进行 web 层单元测试
* 支持灵活的 url 到页面控制器的映射
* 能够非常容易的和其它视图技术集成, 如 Velocity, FreeMaker 等
* 非常灵活的数据验证、格式化和数据绑定机制, 能使任何对象进行数据绑定, 不必实现特定框架的 Api
* 提供一套强大的 JSP 标签库, 简化 JSP 的开发
* 支持灵活的本地化、主题解析等
* 更加简单的异常处理
* 对静态资源的支持
* 支持 restful 风格

## SpringMvc 中的组件

1. DispatcherServlet: 前端控制器

   > 用户请求到达前端控制器, 它相当于 mvc 中的 c, DispatcherServlet 是整个流程控制的中心, 相当于是 SpringMvc 的大脑, 由它调用其它组件处理用户的请求, DispatcherServlet 的存在降低了组件之间的耦合性。
   >
   > DispatcherServlet 作用？
   >
   > * DispatcherServlet 是前端控制器设计模式的实现，提供 Spring Web Mvc 的集中访问点， 而且负责职责的分派，而且与 Spring Ioc 容器无缝集成，从而可以获得 Spring 的所有好处。DispatherServlet 主要用作职责的调度工作，本身主要用于控制流程，主要职责如下：
   >   1. 文件上传解析，如果请求类型是 multipart 将通过 MultipartResolver 进行文件上传解析；
   >   2. 通过 HandlerMapping， 将请求映射到处理器（返回一个 HandlerExecutionChain，它包括一个处理器，多个 HandlerInterceptor 拦截器）；
   >   3. 通过 HandlerAdapter 支持多种类型的处理器（HandlerExceptionChain 中的处理器）；
   >   4. 通过 ViewResolver 解析逻辑视图名到具体视图实现；
   >   5. 本地化解析；
   >   6. 渲染具体的视图等；
   >   7. 如果执行过程中遇到异常将交给 HandlerExceptionResolver 来解析

2. HandlerMapping: 处理器映射器

   > HandlerMapping 负责根据用户请求找到 Handler 即处理器(也是就我们所说的 Controller), SpringMvc 提供了不同的映射器实现不同的映射方式, 例如: 配置文件方式、实现接口方式、注解方式等， 在实际开发中， 我们常用的方式是注解方式。

3. handler: 处理器

   > Handler 是继 DispatcherServlet 前端控制器的后端控制器， 在 DispatcherServlet 的控制下 Handler 对具体的用户请求进行处理。 由于 Handler 涉及到具体的用户业务请求， 所以一般情况需要程序员根据业务需求开发 Handler。(这里所说的 Handler 就是指我们的 Controller)。

4. handlerAdapter: 处理器适配器

   > 通过 HandlerAdapter 对处理器进行执行， 这是适配器模式的应用， 通过扩展适配器可以对更多类型的处理器进行执行。

5. ViewResolver: 视图解析器

   > ViewResolver 负责将处理结果生成 View 视图， ViewResolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。SpringMvc 框架提供了很多的 View 视图类型， 包括 jstlView、freemarkerView、pdfView 等。一般情况下需要通过页面标签或页面模板技术将模型数据通过页面展示给用户。

   