# 核心概念
> Route：路由网关的基本构建块。它由ID，目的URI，断言（Predicate）集合和过滤器（filter）集合组成。如果断言聚合为真，则匹配该路由。  
> Predicate：这是一个 Java 8函数式断言。允许开发人员匹配来自HTTP请求的任何内容，例如请求头或参数。[断言工厂的配置方式](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html)  
> 过滤器：可以在发送下游请求之前或之后修改请求和响应。[过滤器工厂的配置方式](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html)
