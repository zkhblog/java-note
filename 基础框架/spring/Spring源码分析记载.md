ImportBeanDefinitionRegistrar：实现该接口将自定义的组件添加到IOC容器中。


```
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
```


