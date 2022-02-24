# SpringMVC
## 获取请求参数
①将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象，然后通过ServletAPI获取  
②通过控制器方法的形参获取请求参数  
在控制器方法的形参位置，设置和请求参数同名的形参，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参  
③@RequestParam是将请求参数和控制器方法的形参创建映射关系  
④@RequestHeader是将请求头信息和控制器方法的形参创建映射关系  
⑤@CookieValue是将cookie数据和控制器方法的形参创建映射关系  

## RequestEntity
RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，
可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

## HttpMessageConverter：报文信息转换器
将请求报文转换为Java对象，或将Java对象转换为响应报文  

## SpringMVC处理json
此时，会在HandlerAdaptor中自动装配一个消息转换器：MappingJackson2HttpMessageConverter，它可以将响应到浏览器的Java对象转换为Json格式的字符串
