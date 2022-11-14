# Fegin远程调用丢失请求头

在使用Fegin进行远程调用时，会创建一个新的Request，这个对象没有请求头信息，所以后面的服务拿不到cookie信息

```java
@Bean
public RequestInterceptor requestInterceptor(){
    return new RequestInterceptor(){
        @Override
        public void apply(RequestTemplate requestTemplate){
            // RequestContextHolder里面保存了请求信息
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            // 同步请求头数据
            requestTemplate.header("Cookie",request.getHeader("Cookie"));
        }
    }
}
```

# 线程间共享数据(ThreadLocal)

// 获取线程内的数据

```java
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
```

// 在别的异步线程执行如下代码，即将上面得到的数据放入当前线程

```java
 RequestContextHolder.setRequestAttributes();
```

# nginx搭建域名访问环境

①本地域名映射host文件
②远程DNS域名解析器进行域名映射
③可以用nginx直接反向代理到服务，也可以先到网关，再到服务
④nginx配置文件说明
⑤请求头中的Host值会被拿去匹配server块中的server_name值，匹配到了，就将按照这个规则进行处理
⑥在http块的upstream标签中配置上游服务组，
然后在server块的location标签中配置指定的域名代理到哪个上游服务
⑦nginx在代理给网关的时候，会丢失请求的Host信息
proxy_set_header Host $host;
⑧网关的路由配置
8.1 路径映射：可以对路径进行重写
8.2 域名映射：一般放在路径映射后面，避免覆盖路径映射的请求
⑨动静分离
9.1 将项目的静态资源放在nginx里面
9.2 规则：/static/**下所有请求都由nginx直接返回
做法：加一个server块，将/static/路径的请求通过root标签转给本地的static文件夹，然后把找到的资源直接返回给客户端，
剩下的请求转给网关进行处理

二 线上应用内存崩溃宕机
内存溢出(OutOfMemory)后，应用不会再提供服务，用户请求不会被处理
可以调大堆内存-Xmx=4G -Xms=4G
调大新生代-Xmn=4G

三 分布式缓存
【SpringBoot整合Redis的BUG】
并发下，使用redis作为缓存，可能会产生堆外内存溢出：OutOfDirectMemoryError
SpringBoot2.0以后默认使用lettuce作为操作redis的客户端，它使用netty进行网络通信，lettuce的BUG导致netty堆外内存溢出；
netty如果没有指定堆外内存，默认使用-Xmx指定的值，而内存没有及时释放
解决方案：不以通过-Dio.netty.maxDirectMemory只去调大堆外内存，这样做不够。最好的解决办法是切换使用jedis。
切换方法是在spring-boot-starter-data-redis中排除lettuce-core，额外引入jedis依赖包

```xml
<dependency>
    <groupId></groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>reids.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```



lettuce和jedis都是操作redis的底层客户端，Spring将其再次封装成redisTemplate  

【使用redis做缓存的实践】
1）读数据存在的问题  
①缓存穿透：
查询一个一定不存在的数据，并且又没有将查询结果null写入缓存，这导致每次查询都要到存储层，缓存失去了意义
解决办法：缓存空数据（spring.cache.redis.cache-null-values=true），并加入短暂的过期时间
②缓存击穿：
对于一些设置了过期时间的key（又称之为热点数据），在大量请求同时进来前正好失效，那么所有对这个key的数据查询都落到数据库，
这种现象就是缓存击穿
解决办法：加锁（默认是无加锁的，sync=true）
具体处理方法是先从缓存中获取，获取不到再加锁，加锁后再从缓存中获取，还是获取不到就会查库
此处注意两点：一是加锁后要再查一遍缓存，因为不能确定后面的线程是在缓存没有的情况下再去查库的
二是释放锁和设置缓存是一个原子操作，否则会导致时序问题，可能会查两遍数据库
③缓存雪崩：
是指在设置缓存时key采用了相同的过期时间，导致缓存在某一时刻同时失效，因此请求会全部转发到数据库，数据库瞬时压力过大而崩溃
在过期时间基础上加随机值，不让缓存集体失效
2）写数据存在的问题：【缓存一致性问题】  
双写模式（写完数据库后立即写缓存）和失效模式（写完数据库后删缓存）均存在缓存一致性问题  
**最终解决方案：**  
①加过期时间足以解决大部分业务对于缓存的要求
一般确保最终一致性即可，只要控制好设置数据库最新数据的时间和用户通过缓存获取到正确数据之间能容忍的时间即可，
比如在1分钟后缓存过期，此后缓存中的数据和数据库是一致的
②通过加锁保证并发读写
读写数据的时候，加分布式的读写锁，写锁导致读锁过程阻塞，而读锁之间不互斥（适合读多写少）
③也可以使用canal订阅binlog的方式


【分布式锁】  
本地锁，只能锁住当前进程（synchronized、juc包下的lock锁）
需要分布式锁保证全部服务同时只有一个线程被处理
演进过程
1、设置过期时间才能保证即使因程序异常导致锁没有被删除，最终也会因到期而失效
2、设置值的操作和设置过期时间的操作要保证原子性（set key value [EX s] [PX ms] [NX|XX]）
3、要保证删除锁的时候删除的是自己的锁，设置值的时候，设置UUID，这样每个线程设置的值都不一样
4、确定锁的值和当前线程存的值是一样的，再去删锁，这个过程耗时，此时删除的不一定是自己的锁对应的那个值，可能当前线程
对应的锁的值已经过期，而此时锁是别的线程保存的

第一步：  
使用redis实现分布式锁，首先需要保证加锁【①+②】和删除锁【③+④】的原子性  
①set key value NX（不存在才设置值）  
②set key value EX 100 NX（设置过期时间）  
③判断分布式锁的值与当前线程保存的值是否相等，相等才发删除该锁的命令  
④delete key  
第二步：  
保证③和④的原子性操作需要使用Lua脚本：  
String script = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end"
第三步：  
锁的自动续期  

# Spring　Cache



# redisson
lock.lock()  
阻塞式等待：默认加的锁都是30S时间  
1）、锁的自动续期，如果业务超长，运行期间自动给锁续上新的30S。不用担心业务时间长，锁自动过期被删掉  
2）、加锁的业务只要运行完成，就不会给当前锁续期，即使不手动解锁，锁默认在30S以后也会自动删除  
3）、只要占锁成功，就会启动一个定时任务，该定时任务的作用是重新给锁设置过期时间，新的过期时间就是看门狗的默认时间，每隔10秒，
都会将锁的过期时间重新设置成30S

lock.lock(10,TimeUnit.SECONDS)  
**10秒自动解锁，自动解锁时间一定要大于业务的执行时间**  【因为没有自动续期功能，不这样设置会导致锁的误删问题】  
注意：此方法在锁时间到了以后，不会自动续期  
1）如果我们传递了锁的超时时间，就发送给redis执行脚本，进行占锁，默认超时时间就是我们指定的时间  
2）如果我们未指定锁的超时时间，就使用30*1000【LockWatchDogTimeOut看门狗的默认时间】  

【锁的粒度】  
锁的粒度和锁的名字相关，锁的名字相同则是同一把锁，否则是不同的锁。因此像商品信息，都要带上各自id，确保不同的商品
使用的是不同的锁，而分类信息则用一把锁即可。

【异步编排】
①创建线程池对象（new ThreadPoolExecutor）  
②使用completableFuture提供的API实现异步编排

```java
// 自定义线程池
@Bean
public ThreadPoolExecutor threadPoolExecutor(ThreadPoolConfigProperties pool){
    return new ...
}
```



# 短信服务
重定向要携带数据，利用session原理，可以将数据放在session中  
spring mvc中与model同样用法的对象RedirectAttributes：模拟重定向携带数据

# 分布式Session问题
①相同服务，若请求被负载均衡到第二个服务，但是第二个服务没有保存session数据，导致了session不同步问题
②不同服务，session不能在不同域名之间进行访问，会导致session不能共享的问题

解决步骤：

第一步：

```xml
spring-session-data-redis
```

第二步：

spring.session.store-type=redis

第三步：

```java
@Bean
public CookieSerializer cookieSerializer(){
	DefaultCookieSerializer defaultCookieSerializer = new DefaultCookieSerializer();
    defaultCookieSerializer.setDomainName("gulimall.com");//扩大cookie的作用域范围
    defaultCookieSerializer.setCookieName("GULIMALL");
    
    return defaultCookieSerializer;
}

@Bean
public RedisSerializer<Object> springSessionDefaultRedisSerializer(){
    return new GenericJackson2JsonReidsSerializer();
}
```

第四步：

```java
@EnableRedisHttpSession
```



解决问题：
①整合SpringSession将session保存到redis中
②将后端响应的cookie的作用域扩大到父域（修改CookieSerializer这个Bean的设置信息）
③使用JSON的序列化方式来序列化对象数据到redis中（往容器中加入RedisSerializer的Bean）

【SpringSession的核心原理】（装饰者模式）
①对原始请求（request）进行包装成SessionRepositoryRequestWrapper，也包装了原始响应对象
②将包装后的对象应用到后面的整个执行链，也传到了controller中
③在controller中，给session放数据，是通过(HttpServletRequest)request.getSession()方法获取到原始的session对象
④重写了getSession方法，所以原始的获取session对象的逻辑被改变了，现在是从SessionRepository中获取到的
⑤SessionRepository接口的实现类之前就被注入到容器中了，这个Bean的作用就是redis操作session数据进行增删改查的工具类
整个流程得以执行是在SessionRepositoryFilter过滤器中实现的

# 多系统的单点登录

# JWT

# RabbitMQ
①引入starter后，RabbitAutoConfiguration就会自动生效，会给容器中自动配置多个组件
②

@EnableRabbit

@RabbitListener：可以标注在类+方法上（监听哪些队列）

@RabbitHandler：标注在方法上（重载区分不同的消息）



```java
/**
* 使用JSON序列化机制，进行消息转换
*/
@Bean
public MessageConverter messageConverter() {
    return new Jackson2JsonMessageConverter();
}
```



# 确保消息可靠抵达

## ①publisher confirmCallBack	确认模式（未成功发送到broker）

```xml
#开启发送端确认
spring.rabbitmq.publisher-confirms = true
```

定制rabbitTemplate

```java
@PostConstruct
public void initRabbitTemplate() {
    // 设置确认回调
    rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback(){
        /**
        *	correlationData：当前消息的唯一关联数据（即消息的唯一id）
        *	ack：消息是否成功收到
        *	cause：失败的原因
        */
        @Override
        public void confirm(CorrelationData correlationData,boolean ack,String cause) {
            
        }
    })
}
```



## ②publisher returnCallBack	退回模式（未投递到队列）

```xml
spring.rabbitmq.publisher-returns=true
#只要抵达队列，以异步方式优先回调returnConfirm
spring.rabbitmq.template.mandatory=true
```

```java
    // 设置消息抵达队列的确认回调
    rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback(){
        /**
        *	只要消息没有投递给指定的队列，就触发这个失败回调
        *	message：投递失败的消息详细信息
        *	replyCode：回复的状态码
        *	replyText：回复的文本内容
        *	exchange：
        *	routingKey：
        */
        @Override
        public void returnedMessage(Message message,int replyCode,String replyText,String exchange,String routingKey) {
            
        }
    })
```

## ③consumer	ack机制

```java
# 手动ACK消息
spring.rabbitmq.listener.simple.acknowledge-mode = manual
```

```java
long deliveryTage = message.getMessageProperties().getDeliveryTag();
try{
    // false表示非批量模式签收
    channel.basicAck(deliveryTag,false);
    
    // multiple表示可以批量拒收，requeue=true表示重新入队，否则直接丢弃
    channel.basicNack(long deliveryTag,boolean multiple,boolean requeue);
    // 这个API不可以（除此之外，其他与上面相同）
    channel.basicReject(long deliveryTag,boolean requeue);
}catch(Exception e) {
    // 网络中断等，channel关闭
}
```



### 做好消息确认机制，且将每一个发送的消息都在数据库做好记录，定期将失败的消息再次发送一遍

## ④消息重复

建表，发送的消息每一个都有业务的唯一标识，处理过就不用处理了

## ⑤消息积压

将消息取出来，记录数据库，离线慢慢处理

# 接口幂等性问题



# 用户登录的设计



# 分布式事务

本地事务失效问题：

描述：同一个对象内事务方法互调默认失效，是因为绕过了代理对象，而事务是使用代理对象来控制的

解决办法：

①引入spring-boot-starter-aop

②@EnableAspectJAutoProxy(expose=true)

开启aspectj动态代理功能，以后所有的动态代理都是aspectj创建的（即使没有接口也可以创建动态代理）,expose表示对外暴露代理对象

③以后本类互调用代理对象

```java
OrderServiceImpl orderService = (OrderServiceImpl)AopContext.currentProxy();
```

```java
@Transactional
本地事务，在分布式系统中，只能控制住自己的回滚，控制不了其他服务的回滚
```

## seata使用：

①导入依赖：spring-cloud-starter-alibaba-seata

②解压并启动seata-server

③所有要用到分布式事务的微服务要使用seata的DataSourceProxy代理自己的数据源

④修改registry.conf的配置：vgroup_mapping.{application.name}-fescar-service-group="default"

⑤@GlobalTransactional

# 定时任务

@EnableScheduling开启定时任务

@Scheduled开启一个定时任务

特点：①spring中没有第七位的年

​			②定时任务不应该阻塞，默认不应该阻塞

解决办法：①让业务异步执行

```java
CompletableFuture.runAsync(()->{
    xxx},executor)
```

②让定时任务异步执行

```java
@EnableAsync和@Async
```









