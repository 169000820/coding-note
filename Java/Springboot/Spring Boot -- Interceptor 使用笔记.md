# Spring Boot -- Filter 使用笔记

* 自定义一个 MyFilter 实现 Filter 接口，实现`init()` , `doFilter()`, `destory()` 方法

  ```java
  /**
  * @Slf4j => Logger log = LoggerFactory.getLogger(this.getClass())
  * @Component => 启动自动注入 IOC 容器，成为 Spring 上下文中的一个 Bean
  * @WebFilter(urlPatterns = "/*") 匹配所有请求，即所有请求都会进入到这个拦截器
  */
  @Slf4j                          
  @Component                     
  @WebFilter(urlPatterns = "/*") 
  public class MyFilter implents Filter {
      
      @Override
      pulic void init(FilterConfig filterConfig) throws ServletException {
          log.info("MyFilter 初始化");
      }
      
      @Override
      pulic void doFilter(ServletRequest req, ServletResponse res, FilterChain filterChain) throws 
          IOException, ServletException {
      	log.info("doFilter"); 
          HttpServletResponse httpRes = (HttpServletResponse) res;
          HttpServletRequest httpReq = (HttpServletRequest) req;
           File file = new File("log/111.jpg");
              byte[] bytes = new byte[(int) file.length()];
              int len;
              try (InputStream is = new FileInputStream(file)){
                  while ((len = is.read(bytes)) != -1) {
                      System.out.println(len);
                  }
  
                  // 设置返回类型，MediaType 包含了基本的返回类型, 这个是二进制流，提供下载文件
                  // 详情参考菜鸟教程 Http 相关内容
                  httpRes.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
                  // 客户端弹出对话框,是否保存等
                  httpRes.setHeader("Content-Disposition", "attachment;fileName=" + file.getName());
                  httpRes.setContentLength((int) file.length());
                  httpRes.getOutputStream().write(bytes);
              }
          // 代表当前过滤器已处理完毕，传递给下一个处理
          filterChain(req, res);
      }
      
      @Override
      public void destory() {
          log.info("destory ==> 过滤器销毁");
      }
  }
  ```

* 继承 `HttpFilter`

  ```java
  public class MyHttpFilter extends HttpFilter {
  
      public MyHttpFilter() {
          super();
      }
  
      @Override
      protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
  		super(request, response, chain);
      }
  
      @Override
      public void destroy() {
  
      }
  }
  ```

  