# Spring Boot -- Interceptor 使用笔记

* 创建类实现 `HandleInterceptor` 实现拦截器功能，示例:

  ```java
  /**
  * 拦截器可以做更多的事情,功能比过滤器强大, 过滤器不能获取方法和异常等信息
  * 过滤器在拦截器之前执行, 在拦截器后销毁
  */
  @Component
  pulic class MyInterceptor implents HandleInterceptor {
      @Override
      public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
          log.info(">>>>>>>>>>  " + "处理拦截之前" + "  <<<<<<<<<<");
          if (handler instanceof HandlerMethod) {
              HandlerMethod handlerMethod = (HandlerMethod) handler;
              // 可利用反射注解做更多操作
          }
          if (request.getParameter("hhh") == null) {
              response.setCharacterEncoding("utf-8");
              return false;
          }
          return true;
      }
  
      @Override
      public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
          log.info("开始处理拦截");
      }
  
      @Override
      public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
          log.info("处理拦截后");
      }
  }
  ```

* 注册拦截器

  ```java
  public class MyWebConfig implents WebMvcConfiguer {
      @autowired
      private MyInterceptor myInterceptor;
      
      public void addInterceptor(InteceptorRegistry registry) {
          registry.addInterceptor(myInterceptor);
      }
  }
  ```

  

