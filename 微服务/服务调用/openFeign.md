# 远程调用的流程
1 @RequestBody将对象转换为json  
2 找到指定服务，给指定接口发送请求  
3 将json放到请求体里，发送请求  
4 对方服务收到请求后，解析请求体里面的json数据  
5 将请求体中的json数据转换为对方服务的参数类型（注意：只需要两边的字段名称和参数类型一致即可）

# openFeign远程调用的过程
调用feignClient的接口方法时，openFeign会通过动态代理的方式生成代理方法。真正调用时不是调用接口方法，而是调用代理方法，
在代理方法的调用过程中，会先把服务名称和url拼接成一个http地址，然后对拼接好的地址发起http请求。在调用的时候，
会使用ribbon去解析地址中的服务名称，然后在注册中心中发现服务的ip地址

# 远程调用的流程
1 @RequestBody将对象转换为json  
2 找到指定服务，给指定接口发送请求  
3 将json放到请求体里，发送请求  
4 对方服务收到请求后，解析请求体里面的json数据  
5 将请求体中的json数据转换为对方服务的参数类型（注意：只需要两边的字段名称和参数类型一致即可）

在远程调用时，使用的是restTemplate.postForObject(Url url,Object request,Class<T> responseType)
而在服务提供方，方法的参数上不能缺少@RequestBody

前面在使用Ribbon+RestTemplate时，利用RestTemplate对Http请求的封装处理，形成了一套模板化的调用方法。
但是实际情况是往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。
在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它（以前是Dao接口上面标注Mapper注解，现在是一个微服务接口上面
标注一个Feign注解即可），即可完成对服务提供方的接口绑定，简化了使用Ribbon时，自动封装服务调用客户端的工作量。

Feign接口和@FeignClient(value="服务名")注解实现远程调用，在主启动类上要开启Feign的远程调用@EnableFeignClients，
Feign自带负载均衡配置项，它内部集成的有Ribbon

OpenFeign超时控制：
openFeign默认等待1秒钟，超过后报错
解决办法：设置Feign客户端超时时间（openFeign默认支持Ribbon），因此配置方式如下：
ribbon:
#指的是建立连接所用的时间，适用于网络正常的情况下，两端连接所用的时间
ReadTimeOut： 5000
#指的是建立连接后从服务器读取到可用资源的时间
ConnectTimeout: 5000