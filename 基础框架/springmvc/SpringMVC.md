# 获取请求参数
①将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象，然后通过ServletAPI获取  
②通过控制器方法的形参获取请求参数  
在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参  
③@RequestParam是将请求参数和控制器方法的形参创建映射关系  
④@RequestHeader是将请求头信息和控制器方法的形参创建映射关系  
⑤@CookieValue是将cookie数据和控制器方法的形参创建映射关系  

# RequestEntity
RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，
可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

# HttpMessageConverter：报文信息转换器
将请求报文转换为Java对象，或将Java对象转换为响应报文  

# SpringMVC处理json
此时，会在HandlerAdaptor中自动装配一个消息转换器：MappingJackson2HttpMessageConverter，它可以将响应到浏览器的Java对象转换为Json格式的字符串

# WebApplicationInitializer
/**
 * 此实现类相当于配置了SpringMVC的DispatcherServlet
 * Tomcat启动时就会加载此实现类
 * 1）、创建容器，指定主配置类，此时IOC、AOP等Spring的功能就准备好了
 * 2）、注册了DispatcherServlet，访问Tomcat部署下的应用都会被此类处理，DispatcherServlet就会进入强大的基于注解的mvc处理流程
 *
 */
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {          // Spring传入servletContext

        // Load Spring web application configuration
        // 创建IOC容器
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);                          // 注册主配置类，（用注解版的SpringMVC配置替代配置文件）

        // Create and register the DispatcherServlet
        // 配置了DispatcherServlet，并且保存IOC容器
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);   // 利用servlet规范添加DispatcherServlet
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");                          // 指定好映射
    }
}




