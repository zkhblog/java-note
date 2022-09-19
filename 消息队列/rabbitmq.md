# 基础特性
## 消息应答
1、消费者在接收到消息并处理后，告诉rabbitmq它已经处理了，rabbitmq可以把该消息删除了。  
2、消息应答的方法 用于肯定确认： channel.basicAck(deliveryTag,multiple)。
multiple的true代表批量应答channel上未应答的消息，false代表只应答指定tag的消息。  
用于否定确认：  
①channel.basicNack(deliveryTag,multiple,requeue)。第三个参数如果设置为true，则消息重新回到queue，broker会重新发送该消息给消费端。  
②channel.basicReject()，与channel.basicNack()相比少一个参数，不处理该消息了直接拒绝，可以将其丢弃了。  
3、消息重新入队：如果消费者由于某些原因失去连接（其通道关闭，连接已关闭或TCP连接丢失），导致消息未发送ACK确认，Rabbit MQ了解到消息未完全处理，
并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡， 也可以确保不会丢失任何消息。

## 持久化
1 队列实现持久化 在声明队列的时候把durable参数设置为持久化  
声明的时候指定队列持久化  
2 消息实现持久化 让消息实现持久化需要修改消息生产者代码，添加Message.PERSISTENT_TEXT_PLAIN这个属性  
在MessageProperties.PERSISTENT_TEXT_PLAIN添加属性    
3 交换机持久化

## RabbitMQ不公平分发
channel.basicQos(1);

## RabbitMQ预取值
因为消息的发送是异步的，并且消费者的手动确认本质上也是异步的，所以channel上肯定不止只有一个消息。因此希望开发人员能限制未确认的消息缓冲区的大小，通过使用basic.ops方法设置“预取值”来完成。  
该值定义通道上允许的未确认消息的最大数量。 一旦数量达到配置的数量，RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认，

## TTL
既可以给消息设置TTL属性，又可以给队列设置TTL属性。如果同时配置，较小的值将会被使用。  
1、如果设置了队列的TTL属性，那么一旦消息过期，就会被丢弃，如果配置了死信队列，就会被丢到死信队列中，使用参数：x-message-ttl，单位：ms  
```
@Bean
public Queue ttlQueue() {  
    Map<String,Object> map = new HashMap<>();
    map.put("x-message-ttl",5000); 
  
    return new Queue("ttl_queue", true,false,false,args);
}
```
2、如果设置了消息的TTL属性，那么一旦消息过期，也不一定会被丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间
```
// 消息加工类，可以针对消息进行特殊处理
// 单位：ms
MessagePostProcessor processor = new MessagePostProcessor() {
    @Override
    public Message postProcessMessage(Message message) throws AmqpException {
        // 设置消息的编码格式
        message.getMessageProperties().setContentEncoding("UTF-8");
        
        // 设置消息5秒后过期
        message.getMessageProperties().setExpiration("5000");
 
        // 设置消息优先级 （优先级分为消息优先级和队列优先级）
        // ——队列优先级高的会先被处理，消息优先级高的会先被消费
        message.getMessageProperties().setPriority(5);
 
        // 设置消息持久化
        message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
        return message;
    }
}
```

另外，还需要注意的一点是，如果不设置 TTL，表示消息永远不会过期，如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者， 否则该消息将会被丢弃。

# 高级队列
## 死信队列
正常消息变成死信的条件：  
1、消息ttl过期 AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();  
2、队列达到最大长度 params.put("x-max-length",6);（设置正常队列长度的限制）  
3、消息被拒 channel.basicNack(); channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false)，并且设置 requeue=false ，代表拒绝重新入队，否则如果配置了死信交换机将发送到死信队列中

## 延迟队列
用来存放在指定时间被处理的元素的队列。队列内部是有序的。  
设置队列的TTL属性，可以保证消息被延迟消费，但是如果要实现消息粒度上的TTL，即通过给消息指定TTL，实现任意时间延迟消费，
这里面存在的问题就是消息可能并不会按时“死亡”，因为rabbitmq只会检查第一个消息是否过期，如果过期则丢弃或被丢到死信队列，如果第一个消息的延迟时长很长，而第二个消息的延迟时长很短，其实第二个消息并不会优先得到执行。
需要安装延迟队列插件rabbitmq_delayed_message_exchange（可以让消息在设置的TTL时间及时死亡），并解压到插件目录，使用如下命令重启mq
```
/usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins/rabbitmq-plugins enable 插件名
```

## 备份交换机
备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当我们为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，
将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，
这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。
![img.png](images/备份交换机理论图.png)
```
 //声明确认 Exchange 交换机的备份交换机
 @Bean("confirmExchange")
 public DirectExchange confirmExchange(){
    ExchangeBuilder exchangeBuilder =
    ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME)
    .durable(true)
    //设置该交换机的备份交换机
    .withArgument("alternate-exchange", BACKUP_EXCHANGE_NAME);
    return (DirectExchange)exchangeBuilder.build();
 }
```
> 生产者回退消息与备份交换机同时开启的时候，不可路由的消息会被发送到备份交换机

## 优先级队列
①队列设置优先级  
params.put("x-max-priority",10);  
②消息设置优先级  
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(5).build();

## 惰性队列
概念：尽可能将消息存入磁盘，在消费者消费时才会被加载到内存中，即支持更多的消息存储。  
队列具有两种模式：default和lazy。  
params.put("x-queue-mode","lazy");  

# 重要知识点
## 消息消费的两种模式
①推模式  
MQ主动将消息推送给消费者，这种方式需要消费者设置一个缓冲区去缓存消息，适合持续获得消息的场景  
```
 // 当监听的队列有消息时，就会触发该方法
 @RabbitListener(queues = RabbitConfig.JAVABOY_QUEUE_NAME)
 public void handle(String msg) {
    System.out.println("msg = " + msg);
 }
```

②拉模式  
消费者主动从MQ拉取消息。消费一条消息  
```
Object o = rabbitTemplate.receiveAndConvert(RabbitConfig.JAVABOY_QUEUE_NAME);
System.out.println("o = " + new String(((byte[]) o),"UTF-8"));
```

## 幂等性
消费者已经成功消费消息，在给MQ返回ACK时网络中断，故MQ未收到确认消息，该消息会重新发给其他消费者，或在网络重连后再次发送给该消费者，但实际上该消费者
已成功消费该条消息，造成了消息的重复消息。  
解决办法：利用redis执行setNx命令，从而实现不重复消费

## 延迟队列的选择
延迟队列的选择有Java的DelayQueue、Redis的zSet、Quartz、Kafka的时间轮等。

## 消费端限流
1、确保ACK机制为手动确认； 2、perfetch=1；（表示消费端每次从MQ拉取一条消息来消费，直到手动确认消费完毕后，才会继续拉取下一条消息）

## 消息追踪
1、FireHose 机制是将生产者投递给rabbitmq的消息，rabbitmq投递给消费者的消息a按照指定的格式发到默认的exchange上。
这个默认的交换机的名称是amqp.rabbitmq.trace，它是一个topic类型的交换机。 开启FireHose的命令：rabbitmqctl trace_on 关闭FireHose的命令：rabbitmqctl
trace_off  
2、rabbitmq_tracing 启用插件：rabbitmq-plugins enable rabbitmq_tracing

# 消息的可靠投递
要确保消息成功发送，只需做好三件事即可：  
1、确认消息到达Exchange  
2、确认消息到达Queue  
3、开启定时任务，定时投递那些发送失败的消息（自己实现）

## 手动应答（ACK）
手动确认：acknowledge="manual"

## 开启事务机制（事务模式效率低下）
<details>
<summary>点此展开</summary>
首先需要提供一个事务管理器  

```
@Bean
RabbitTransactionManager transactionManager(ConnectionFactory connectionFactory) {
    return new RabbitTransactionManager(connectionFactory);
}
```

在消息生产者上面做两件事：添加事务注解并设置通信信道为事务模式

```
@Service
public class MsgService {
    @Autowired
    RabbitTemplate rabbitTemplate;

    @Transactional      // 发送消息的方法上添加@Transactional注解标记事务
    public void send() {
        rabbitTemplate.setChannelTransacted(true);      // 开启事务模式
        rabbitTemplate.convertAndSend(RabbitConfig.JAVABOY_EXCHANGE_NAME,RabbitConfig.JAVABOY_QUEUE_NAME,"hello rabbitmq!".getBytes());
        int i = 1 / 0;
    }
}
```

生产者发送消息的步骤：  
1、客户端发出请求，将信道设置为事务模式。  
2、服务端给出回复，同意将信道设置为事务模式。  
3、客户端发送消息。 （除了第三步是本来就有的，其他几个步骤都是平白无故多出来的）  
4、客户端提交事务。  
5、服务端给出响应，确认事务提交。

</details>

## 发布确认模式
概述：  
生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都将会被指派一个唯一的ID。 一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确 到达目的队列了。  
如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。  

种类：  
①单个确认发布（同步）  
发布一个消息之后只有它被确认，后续的消息才能继续发布，waitForConfirmsOrDie(long)这个方法只有在消息被确认的时候才被返回，如果在指定时间内没有确认将抛出异常  
所以发布速度特别慢，并且发布的消息没有确认会阻塞后续消息的发布  
②批量确认发布（同步）  
缺点就是当出现问题时，不知道是哪个消息出现问题导致的，所以必须整批保存在内存中，以便后续重新发送  
③异步确认发布  
利用回调函数来实现消息的可靠传递  

具体实现：  
①配置开启消息发送方确认机制  
```
// 发布消息成功到交换机后会触发回调方法
spring.rabbitmq.publisher-confirm-type=correlated
// 此配置表示一旦路由错误，交换机会回退消息给生产者
spring.rabbitmq.publisher-returns=true（配置消息到达队列的回调）
```

②开启监听  
```
@Configuration
public class RabbitConfig implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {
    public static final String JAVABOY_EXCHANGE_NAME = "javaboy_exchange_name";
    public static final String JAVABOY_QUEUE_NAME = "javaboy_queue_name";
    private static final Logger logger = LoggerFactory.getLogger(RabbitConfig.class);
    @Autowired
    RabbitTemplate rabbitTemplate;
    @Bean
    Queue queue() {
        return new Queue(JAVABOY_QUEUE_NAME);
    }
    @Bean
    DirectExchange directExchange() {
        return new DirectExchange(JAVABOY_EXCHANGE_NAME);
    }
    @Bean
    Binding binding() {
        return BindingBuilder.bind(queue())
                .to(directExchange())
                .with(JAVABOY_QUEUE_NAME);
    }

    @PostConstruct      // 因为两个接口的实现类是rabbitTemplate对象的内部属性，所以要使用特殊的注入方式，将目标对象注入给rabbitTemplate对象
    public void initRabbitTemplate() {
        // 设置服务器端确认回调（成功与否都会回调）
        rabbitTemplate.setConfirmCallback(this);
        
        // 设置不可路由到目的队列的回调
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setReturnsCallback(this);
    }

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack) {
            logger.info("{}:消息成功到达交换器",correlationData.getId());
        }else{
            logger.error("{}:消息发送失败", correlationData.getId());
        }
    }

    @Override
    public void returnedMessage(ReturnedMessage returned) {
        logger.error("{}:消息未成功路由到队列",returned.getMessage().getMessageProperties().getMessageId());
    }
}
```

> 如何处理异步未确认消息：把未确认消息放到一个基于内存的能被发布线程访问的队列，比如说用concurrentLinkedQueue这个队列在confirm callbacks与发布线程之间进行消息的传递  

# 失败重试
## 自带重试机制
如果发送方连不上MQ，Spring Boot中有相应的重试机制，这个重试机制和MQ没有关系，利用的是Spring的retry机制完成
```
# 开启重试机制
spring.rabbitmq.template.retry.enabled=true
# 重试起始间隔时间
spring.rabbitmq.template.retry.initial-interval=1000ms
# 最大重试次数
spring.rabbitmq.template.retry.max-attempts=10
# 最大重试间隔时间
spring.rabbitmq.template.retry.max-interval=10000ms
# 间隔时间乘数。（这里配置间隔时间乘数为 2，则第一次间隔时间 1 秒，第二次重试间隔时间 2 秒，第三次 4 秒，以此类推）
spring.rabbitmq.template.retry.multiplier=2
```

## 业务重试
主要针对消息没有到达交换机的情况，如果消息没有到达交换机，触发生产者的确认回调机制  
① 建表
![img.png](images/发送到MQ的消息记录表.png)
status：表示消息的状态，有三个取值，0，1，2 分别表示消息发送中、消息发送成功以及消息发送失败。  
tryTime：表示消息的第一次重试时间（消息发出去之后，在 tryTime 这个时间点还未显示发送成功，此时就可以开始重试了）。  
count：表示消息重试次数。  
② 在消息发送的时候，我们就往该表中保存一条消息发送记录，并设置状态 status 为 0，tryTime 为 1 分钟之后。  
③ 在 confirm 回调方法中，如果收到消息发送成功的回调，就将该条消息的 status 设置为1（在消息发送时为消息设置 msgId，在消息发送成功回调时，通过 msgId 来唯一锁定该条消息）。
④ 另外开启一个定时任务，定时任务每隔 10s 就去数据库中捞一次消息，专门去捞那些 status 为 0 并且已经过了 tryTime 时间记录，把这些消息拎出来后，首先判断其重试次数是否已超过 3 次，如果超过 3 次，则修改该条消息的 status 为 2，表示这条消息发送失败，并且不再重试。对于重试次数没有超过 3 次的记录，则重新去发送消息，并且为其 count 的值+1。

# 自动补偿机制（消费者处理消息）

在RabbitMQ里，如果消费者在处理消息时，业务逻辑出现异常，默认会执行补偿机制（也就是消息重试机制）。如果业务逻辑出现异常，是不会消费消息的。  
原理：  
①@RabbitListener 底层使用了AOP进行拦截，如果程序没有抛异常，自动提交事务。  
②如果AOP使用异常通知拦截获取异常信息的话，自动实现补偿机制，该消息会缓存到RabbitMQ服务端进行缓存，一直重试到不抛异常为准。  
配置重试的次数与时间：  
![img.png](images/自动重试机制配置参数.png)
说明：消费者获取到消息后，有一些情况需要重试机制（如调用第三方接口，但接口暂时无法访问...），有一些情况不需要重试机制（如抛出空指针异常...），因为 异常的代码是一直都会有异常的。

# 十三 消息可靠性保障（高级篇）

1、消息补偿机制  
![img.png](images/消息补偿流程图.png)  
简单说明：  
消息补偿机制需要建立在业务数据库和MQ数据库的基础之上 , 当我们发送消息时 , 需要同时将消息数据保存在数据库中, 两者的状态必须记录。 然后通过业务数据库和MQ数据库的对比检查消费是否成功，不成功，进行消息补偿措施，重新发送消息处理
①消息补偿机制核心 : 发现未成功消费的消息, 并且重新发送消息  
②消息回调检查服务 : 发送正常消息同时发送一个延迟消息, 当监听到延迟消息的时候, 检查MDB中是否有消费记录 , 如果没有代表存在消息丢失, 重新发送消息  
③消息定时检查服务 : 设置定时任务, 定时比对业务DB和MDB ,中的数据是否一致, 如果不一致一定存在消息丢失, 重新发送

# 十四 消息幂等性保障
## 14.1 消息去重解决方案
简单处理：通过唯一id去重，再次消费时去数据库中查一次，若已经存在，则不消费  
复杂处理：并发场景下，可能在没成功消费之前，又有一条消息到来，那么就会导致重复的消费逻辑被执行，从而引发重复消费的问题（如主键冲突抛出异常、库存被重复扣减而没释放等）  
并发场景下的解决方案：  
①开启事务，把select改成select ... for update  
②更新状态采取乐观锁  
③分布式场景下，采取消息表+本地事务的实现方式：（赖于关系型数据库的事务，且必须要把事务包裹于整个消息消费的环节）  
针对基于数据库事务的消费逻辑，可以将插入消息表（处理好主键冲突的问题）和更新订单表（原消费逻辑）的动作放到同一个事务中一起提交  
这种场景下，可以通过【消息表的插入、消费逻辑的执行、消费位点的更新】这三部分确保  
④更通用的解决方案（不依赖于事务而实现消息的去重）：对消息表增加消费状态（消费中、消费完成、消费失败）  
![img.png](images/基于消息幂等表的非事务方案.png)
说明：只有消费完成的消息才会被幂等处理掉，而对于已有消费中的消息，后面重复的消息会触发延迟消费，之所以触发延迟消费是为了控制并发场景下，第二条消息在第一条消息没完成的过程中，  
去控制消息不丢（如果直接幂等，同一个消息id的话会丢失消息），因为上一条消息如果没有消费完成的时候，第二条消息你已经告诉broker成功了，那么第一条消息这时候失败broker也不会重新投递了）  





幂等性指一次和多次请求某一个资源，对于资源本身应该具有同样的结果。也就是说，其任意多次执行对资源本身所产生的影响均与一次执行的影响相同。  
![img.png](images/消息幂等性机制1.png)
乐观锁解决方案  
![img.png](images/消息幂等性保障2.png)



https://mp.weixin.qq.com/s/5szA0KBpFn9G3DeS9C0U3w












# 常见问题
### 消息堆积
① 排查消费者运行效率、性能问题  
② 消费者多线程处理  
③ 增加更多的消费者

### 消息丢失
① 生产者环节丢失消息  
生产者在消息投递环节开启消息确认机制，发送失败时候重试  
② 队列中丢失消息  
投递消息的时候，交换机、队列、消息都设置为持久化方式```durable=true```  
③ 消费者中丢失消息  
通道设置成每次只处理一条消息，关闭消费者自动应答，消费完成手动应答  

### 有序消费
消费者开启了多线程  
消息队列分配消息后，创建多个内存队列，每个线程消费者去消费内存队列里面的消息

### 消息重复消费
消费者消费消息的时候，MQ没有收到消息的ACK应答  
① 消费者消费后，记录通过缓存记录消息的消费标识，消息id如redis的setNx  
② 如果消费成功且ACK成功，则删除记录的消息标记  
③ 如果ACK失败，消息下次被消费的时候，先去查询消息的消费标识，已经消费则直接ACK，未消费则继续消费

### 消息幂等性
消息无论怎么执行都是同样的结果

### 非幂等性案例
扣款、扣库存、添加数据









