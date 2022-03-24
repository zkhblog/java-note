# 权限管理中的相关概念
① 用户认证：用户能否访问该系统；  
② 用户授权：指的是验证某个用户是否有权限执行某个操作。

# 主要API说明
extends WebSecurityConfigurerAdapter    配置资源访问是否受限  
implement UserDetailsService            实现该接口可以自定义控制认证逻辑  
PasswordEncoder
> String encode(CharSequence rawPassword);表示把参数按照特定的解析规则进行解析  
> boolean matches(CharSequence rawPassword, String encodedPassword);表示验证从存储中获取的编码密码与编码后提交的原始密码是否匹配。
> 如果密码匹配，则返回 true；如果不匹配，则返回 false。第一个参数表示需要被解析的密码。第二个参数表示存储的密码。  
> default boolean upgradeEncoding(String encodedPassword) {表示如果解析的密码能够再次进行解析且达到更安全的结果则返回 true，否则返回
> false。默认返回 false。  
> 
> BCryptPasswordEncoder是官方推荐的密码解析器


# 框架分析
SpringSecurity 采用的是责任链的设计模式，它有一条很长的过滤器链。现在对这条过滤器链的 15 个过滤器进行说明:  
(1) `WebAsyncManagerIntegrationFilter`：将Security上下文与Spring Web中用于处理异步请求映射的 WebAsyncManager 进行集成。  
(2) `SecurityContextPersistenceFilter`：在每次请求处理之前将该请求相关的安全上下文信息加载到 SecurityContextHolder 中，然后在该次请求处理完成之后，
将SecurityContextHolder 中关于这次请求的信息存储到一个“仓储”中，然后将SecurityContextHolder 中的信息清除，例如在 Session 中维护一个用户的安全信息就是这个过滤器处理的。  
(3) `HeaderWriterFilter`：用于将头信息加入响应中。  
(4) `CsrfFilter`：用于处理跨站请求伪造。  
(5) `LogoutFilter`：用于处理退出登录。 
(7) `DefaultLoginPageGeneratingFilter`：如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面。
(8) `BasicAuthenticationFilter`：检测和处理 http basic 认证。  
(9) `RequestCacheAwareFilter`：用来处理请求的缓存。  
(10) `SecurityContextHolderAwareRequestFilter`：主要是包装请求对象 request。  
(11) `AnonymousAuthenticationFilter`：检测 SecurityContextHolder 中是否存在Authentication 对象，如果不存在为其提供一个匿名 Authentication。  
(12) `SessionManagementFilter`：管理 session 的过滤器    
(15) `RememberMeAuthenticationFilter`：当用户没有登录而直接访问资源时, 从 cookie 里找出用户的信息, 如果 Spring Security 能够识别出用户提供的 remember me cookie,
用户将不必填写用户名和密码, 而是直接登录进入系统，该过滤器默认不开启。  

# 核心过滤器
`UsernamePasswordAuthenticationFilter`  
用于处理基于表单的登录请求，从表单中获取用户名和密码。默认情况下处理来自 /login 的请求。从表单中获取用户名和密码时，默认使用的表单的值为 username 和 password，
这两个值可以通过设置这个过滤器的 usernameParameter 和 passwordParameter 两个参数的值进行修改。  

`FilterSecurityInterceptor`  


`ExceptionTranslationFilter`
处理 AccessDeniedException 和 AuthenticationException 异常。

`FilterSecurityInterceptor`  
该过滤器是过滤器链的最后一个过滤器，根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果访问受限会抛出相关异常，并由 ExceptionTranslationFilter 过滤器进行捕获和处理。




https://www.cnblogs.com/hello-shf/p/10800457.html
待验证问题：登录后，成功的情况下，为什么会报405错误，但是浏览器里却可以直接访问/success.html