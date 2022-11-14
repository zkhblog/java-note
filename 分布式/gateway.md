# 核心概念
### Route  
路由网关的基本构建块。它由ID，目的URI，断言（Predicate）集合和过滤器（filter）集合组成。如果断言聚合为真，则匹配该路由。  

### Predicate
这是一个 Java8函数式断言。允许开发人员匹配来自HTTP请求的任何内容，例如请求头或参数。[断言工厂的配置方式](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html)  

① -Host：断言用户请求的域名  
② -Path：断言用户请求的路径  

### Filter
可以在发送下游请求之前或之后修改请求和响应。[过滤器工厂的配置方式](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.0.RELEASE/single/spring-cloud-gateway.html)

① -PrefixPath：可以给用户请求路径加统一前缀  
② -StripPrefix：将请求路径中过滤掉

### 运行流程
① 请求到达网关后，先经过断言Predicate判断，是否符合某个路由规则  
② 如果符合，则按路由规则路由到指定地址  
③ 请求和响应都可以通过过滤器filter进行过滤  

# 网关限流
保护微服务，防止雪崩效应

### 使用令牌桶进行请求次数限流
![img.png](images/令牌桶算法.png)

1 通过KeyResolver指定限流的key
```
// 根据IP进行限流
@Bean(name="ipKeyResolver")
public KeyResolver userKeyResolver() {
    return new KeyResolver() {
        @Override
        public Mono<String> resolve(ServerWebExchange exchange) {
            //获取远程客户端IP
            String hostName = exchange.getRequest().getRemoteAddress().getAddress().getHostAddress();
            System.out.println("hostName:"+hostName);
            return Mono.just(hostName);
        }
    };
}
```

2 指定限制流量的配置
```yaml
-name: RequestRateLimiter # 根据请求数进行限流，名字不能随便写，使用默认的factory
args: 
  key-resolver: "#{@ipKeyResolver}"
  redis-rate-limiter.replenishRate: 1
  redis-rate-limiter.burstCapacity: 1
```
