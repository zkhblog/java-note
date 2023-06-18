# SpringMVC工作流程
当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给
中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者  
1 DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求  
2 HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler  
3 返回处理器执行链，根据url查找控制器，并且将解析后的信息传递给DispatcherServlet  
4 HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler  
5 执行handler找到具体的处理器  
6 Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView  
7 HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet  
8 DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名  
9 视图解析器将解析的逻辑视图名传给DispatcherServlet  
10 DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图，进行试图渲染  
11 将响应数据返回给客户端

# 获取请求参数的方式
① 将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象，然后通过ServletAPI获取  
② 通过控制器方法的形参获取请求参数  
在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参  
③ @RequestParam是将请求参数和控制器方法的形参创建映射关系  
④ @RequestHeader是将请求头信息和控制器方法的形参创建映射关系  
⑤ @CookieValue是将cookie数据和控制器方法的形参创建映射关系  
⑥ RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，
可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

# HttpMessageConverter：报文信息转换器
将请求报文转换为Java对象，或将Java对象转换为响应报文  

# HandlerMapping
根据request找到相应的处理器。Handler（Controller）有两种形式，一种是基于类的Handler，另一种是基于Method的Handler（也就是我们常用的）

# HandlerAdapter
调用Handler的适配器。如果把Handler（Controller）当做工具的话，那么HandlerAdapter就相当于干活的工人  
MappingJackson2HttpMessageConverter，它可以将响应到浏览器的Java对象转换为Json格式的字符串

# HandlerExceptionResolver
对异常的处理

# ViewResolver
用来将String类型的视图名和Locale解析为View类型的视图

# RequestToViewNameTranslator
有的Handler（Controller）处理完后没有设置返回类型，比如是void方法，这是就需要从request中获取viewName

# LocaleResolver
从request中解析出Locale。Locale表示一个区域，比如zh-cn，对不同的区域的用户，显示不同的结果，这就是i18n（SpringMVC中有具体的拦截器LocaleChangeInterceptor）

# ThemeResolver
主题解析，这种类似于我们手机更换主题，不同的UI，css等

# MultipartResolver
处理上传请求，将普通的request封装成MultipartHttpServletRequest

# FlashMapManager
用于管理FlashMap，FlashMap用于在redirect重定向中传递参数