![img.png](images/SpringSecurity流程图.png)

# 权限管理中的相关概念
1 用户认证：判断用户的身份是否合法的过程；  
2 用户授权：指的是验证某个用户是否有权限执行某个操作。  
> 授权的实现方式：基于角色的访问控制（按角色进行授权）；基于资源的访问控制（按资源/权限进行授权）

3 会话：会话就是系统为了为了避免每次操作都要进行认证，将用户的登录状态保存在会话中。
> 1 基于session的认证：  
> 
> 用户认证成功后，在服务端生成用户相关的数据保存在session中，发给客户端的session_id存放到cookie中，
> 这样用户客户端请求时带上 session_id 就可以验证服务器端是否存在 session 数 据，以此完成用户的合法校验，
> 当用户退出系统或session过期销毁时,客户端的session_id也就无效了。  
> 2 基于token的认证：  
> 用户认证成功后，服务端生成一个token发给客户端，客户端可以放到 cookie 或 localStorage 等存储中，
> 每次请求时带上 token，服务端收到token通过验证后即可确认用户身份。

# OncePerRequestFilter
通常被用于继承实现并在每次请求时只执行一次过滤，通过增加标记的方式来实现过滤器只被执行一次  
```java
// 获取当前filter的属性名称，该名称后面会被用于放到request当作key
String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();
// 检查当前请求是否已经有了该标记，如果有了该标记，则代表该过滤器已经执行过了
boolean hasAlreadyFilteredAttribute = request.getAttribute(alreadyFilteredAttributeName) != null;

if (skipDispatch(httpRequest) || shouldNotFilter(httpRequest)) {

	// Proceed without invoking this filter...
	filterChain.doFilter(request, response);
}
// 如果此过滤器已经执行过则执行下面的逻辑
else if (hasAlreadyFilteredAttribute) {

	if (DispatcherType.ERROR.equals(request.getDispatcherType())) {
		doFilterNestedErrorDispatch(httpRequest, httpResponse, filterChain);
		return;
	}

	// Proceed without invoking this filter...
	filterChain.doFilter(request, response);
}
// 该过滤器未被执行过
else {
    // 在当前请求里面设置标记，key就是前面拼接的那个变量，value是true
	request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);
	try {
        // 子类实现具体过滤逻辑
		doFilterInternal(httpRequest, httpResponse, filterChain);
	}
	finally {
        // 执行完毕后移除该标记
		request.removeAttribute(alreadyFilteredAttributeName);
	}
}
```

# 框架分析
SpringSecurity 采用的是责任链的设计模式，它有一条很长的过滤器链。现在对这条过滤器链的 15 个过滤器进行说明:  
(1) `WebAsyncManagerIntegrationFilter`：将Security上下文与Spring Web中用于处理异步请求映射的 WebAsyncManager 进行集成。  
(2) `SecurityContextPersistenceFilter`：在每次请求处理之前将该请求相关的安全上下文信息加载到 SecurityContextHolder 中，然后在该次请求处理完成之后，
将SecurityContextHolder 中关于这次请求的信息存储到一个“仓储”中，然后将SecurityContextHolder 中的信息清除，例如在 Session 中维护一个用户的安全信息就是这个过滤器处理的。  
(3) `HeaderWriterFilter`：用于将头信息加入响应中。  
(4) `CsrfFilter`：用于处理跨站请求伪造。  
(5) `LogoutFilter`：用于处理退出登录。 
(7) `DefaultLoginPageGeneratingFilter`：如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面  
(8) `BasicAuthenticationFilter`：检测和处理 http basic 认证  
(9) `RequestCacheAwareFilter`：用来处理请求的缓存。  
(10) `SecurityContextHolderAwareRequestFilter`：主要是包装请求对象 request。  
(11) `AnonymousAuthenticationFilter`：检测 SecurityContextHolder 中是否存在Authentication 对象，如果不存在为其提供一个匿名 Authentication。  
(12) `SessionManagementFilter`：管理 session 的过滤器    
(15) `RememberMeAuthenticationFilter`：当用户没有登录而直接访问资源时, 从 cookie 里找出用户的信息, 如果 Spring Security 能够识别出用户提供的 remember me cookie,
用户将不必填写用户名和密码, 而是直接登录进入系统，该过滤器默认不开启。  

### 核心过滤器
`UsernamePasswordAuthenticationFilter`  
用于处理基于表单的登录请求，从表单中获取用户名和密码。默认情况下处理来自 /login 的请求。从表单中获取用户名和密码时，默认使用的表单的值为 username 和 password，
这两个值可以通过设置这个过滤器的 usernameParameter 和 passwordParameter 两个参数的值进行修改。  

`ExceptionTranslationFilter`
是个异常过滤器，用来处理在认证和授权过程中抛出的 ```AccessDeniedException``` 和 ```AuthenticationException``` 等异常  

`FilterSecurityInterceptor`  
该过滤器在过滤器链中靠后，根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果访问受限会抛出相关异常，并由 ExceptionTranslationFilter 过滤器进行捕获和处理。

https://www.cnblogs.com/hello-shf/p/10800457.html
待验证问题：登录后，成功的情况下，为什么会报405错误，但是浏览器里却可以直接访问/success.html

# 相关API
### 资源访问控制相关
在配置文件中，通过以下几个方法指定访问资源需要什么样的角色或权限。主体拥有的角色或权限是在认证过程中给其加上的。  
1、hasAuthority 方法  
2、hasAnyAuthority 方法  
3、hasRole 方法  
4、hasAnyRole 方法  
注意配置文件中不需要添加”ROLE_“，因为上述的底层代码会自动添加与之进行匹配，而在认证过程中返回用户对象的时候，需要加上  
http.authorizeRequests()
    .antMatchers("/find").hasRole("admin")
return new User(userInfo.getUserName(),userInfo.getPassWord(),AuthorityUtils.commaSeparatedStringToAuthorityList("delete,ROLE_admin"))

### 权限控制相关注解  
1 、@Secured  
前提：@EnableGlobalMethodSecurity(securedEnabled=true)  
功能：判断是否具有角色，另外需要注意的是这里匹配的字符串需要添加前缀"ROLE_"。  
用法：@Secured({"ROLE_normal","ROLE_admin"})  

2、@PreAuthorize  
前提：@EnableGlobalMethodSecurity(prePostEnabled = true)  
功能：注解适合进入方法前的权限验证， @PreAuthorize 可以将登录用户的 roles/permissions 参数传到方法中。  
用法：@PreAuthorize("hasAnyAuthority('menu:system')")  

3、@PostAuthorize  
前提：@EnableGlobalMethodSecurity(prePostEnabled = true)  
功能：@PostAuthorize 注解使用并不多，在方法执行后再进行权限验证，适合验证带有返回值的权限  
用法：@PostAuthorize("hasAnyAuthority('menu:system')")  

4、@PostFilter  
功能：权限验证之后对数据进行过滤，留下用户名是 admin1 的数据  
用法：@PostFilter("filterObject.username == 'admin1'")  

5、@PreFilter  
功能：进入控制器之前对数据进行过滤  
用法：@PreFilter(value = "filterObject.id%2==0")  

# Token
### 普通令牌
基于redis存储用户信息的方式，认证服务器将用户信息存储到指定的redis库中，在资源服务获取到access_token时，进而实现权限角色的限制，会到redis中获取用户信息，适合微服务场景

### JWT令牌
① 对于Token来说，需要查库或者和服务器中的Token比对是否有效  
② JWT包含三个部分：Header头部、Payload负载和Signature签名。由三部分生成JwtToken，三部分之间用“.”号做分割。校验也是JWT内部自己实现的，并且可以将你存储时候的信息从JwtToken中取出来无须查库  
③ JWT不用查库，客户端将登录时收到的jwtToken传到后端，直接在服务端进行校验，因为用户的信息、加密信息和过期时间都在jwtToken中，而且校验的过程也是JWT自己实现的





# 如何动态更新已登录用户的信息
