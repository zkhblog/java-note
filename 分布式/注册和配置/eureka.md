# 自我保护机制
某时刻某一个微服务不可用，eureka不会立即清理，依旧会对该微服务的信息进行保存  

默认情况下，当eureka server在一定时间内没有收到实例的心跳，便会把该实例从注册表中删除（默认是90秒），但是，如果短时间内丢失大量的实例心跳，便会触发eureka server的自我保护机制，比如在开发测试时，需要频繁地重启微服务实例，但是我们很少会把eureka server一起重启（因为在开发过程中不会修改eureka注册中心），当一分钟内收到的心跳数大量减少时，会触发该保护机制。可以在eureka管理界面看到Renews threshold和Renews(last min)，当后者（最后一分钟收到的心跳数）小于前者（心跳阈值）的时候，触发保护机制，会出现红色的警告：EMERGENCY!EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT.RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEGING EXPIRED JUST TO BE SAFE.从警告中可以看到，eureka认为虽然收不到实例的心跳，但它认为实例还是健康的，eureka会保护这些实例，不会把它们从注册表中删掉。
该保护机制的目的是避免网络连接故障，在发生网络故障时，微服务和注册中心之间无法正常通信，但服务本身是健康的，不应该注销该服务，如果eureka因网络故障而把微服务误删了，那即使网络恢复了，该微服务也不会重新注册到eureka server了，因为只有在微服务启动的时候才会发起注册请求，后面只会发送心跳和服务列表请求，这样的话，该实例虽然是运行着，但永远不会被其它服务所感知。所以，eureka server在短时间内丢失过多的客户端心跳时，会进入自我保护模式，该模式下，eureka会保护注册表中的信息，不在注销任何微服务，当网络故障恢复后，eureka会自动退出保护模式。自我保护模式可以让集群更加健壮。
但是我们在开发测试阶段，需要频繁地重启发布，如果触发了保护机制，则旧的服务实例没有被删除，这时请求有可能跑到旧的实例中，而该实例已经关闭了，这就导致请求错误，影响开发测试。所以，在开发测试阶段，我们可以把自我保护模式关闭，只需在eureka server配置文件中加上如下配置即可：eureka.server.enable-self-preservation=false【不推荐关闭自我保护机制】

```
可参考如下文章：https://blog.csdn.net/wudiyong22/article/details/80827594
```


# 注册中心
Eureka包含两个组件：EurekaServer和EurekaClient
EurekaServer提供服务注册服务：各个微服务节点通过配置后，会在EurekaServer中进行注册，
这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。
EurekaClient通过注册中心进行访问
这是一个Java客户端，用于简化EurekaServer的交互，客户端同时也具备一个内置的、使用轮询负载算法的负载均衡器。
在应用启动后，将会向EurekaServer发送心跳（默认周期30秒）。如果EurekaServer在多个心跳周期内没有接收到某个节点的心跳，
EurekaServer将会从服务注册列表中把这个服务节点移除（默认90秒）

Eureka的自我保护机制：某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存。暂时保留没有心跳的
微服务的信息，期待着它的恢复。
从设计上来说，是因为Eureka满足CAP理论的AP分支，也就是不保证强一致性，（那么有这样的机制就可以实现容错的目的---猜测）。
出厂默认，自我保护机制是开启的，使用eureka.server.enable-self-preservation=false可以禁用自我保护模式

Eureka集群原理说明：
1、先启动Eureka注册中心
2、启动服务提供者payment支付服务
3、支付服务启动后会把自身信息（比如服务地址以别名方式注册进Eureka）
4、消费者order服务在需要调用接口时，使用服务别名去注册中心获取实际的RPC远程调用地址
5、消费者获得调用地址后，底层实际是利用HttpClient技术实现远程调用
6、消费者获得服务地址后会缓存在本地jvm内存中，默认每间隔30秒更新一次服务调用地址

只要服务注册中心不是Eureka，就不使用@EurekaServer和@EurekaClient，
代替的是@EnableDiscoveryClient，该注解用于向使用consul和zookeeper作为注册中心时注册服务

三种注册中心的区别主要体现在设计上，即CAP理论的取向，Eureka是AP的实现，Zookeeper和Consul是CP的实现
因此，Eureka有搭建集群的必要性，微服务在Zookeeper上创建的是临时节点，微服务下线，临时节点**立即**被删除，
真实的网络设计上，一般首选AP，要满足网站的高可用特性，为了保证可用性，系统B可以返回旧值，保证系统的可用性。

