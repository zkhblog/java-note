```
https://blog.csdn.net/qq_21040559/article/details/122703715
```
# RocketMQ的优势
① 支持事务型消息（即消息发送和DB操作保持两方的最终一致性，RabbitMQ和Kafka不支持）  
② 支持指定次数和时间间隔的失败消息重发  
③ 支持 Consumer 端 Tag 过滤，减少不必要的网络传输（即过滤由MQ完成，而不是由消费者完成。RabbitMQ 和 Kafka 不支持）  
④ 支持重复消费（RabbitMQ 不支持，Kafka 支持）

### 概念
![img.png](images/RocketMQ各角色之间完整的交互过程.png)

##### 高可用保障
① NameServer高可用  
Broker 在启动时向所有 NameServer 注册（主要是服务器地址等） ，生产者在发送消息之前先从NameServer 获取 Broker 服务器地址列表（消费者一样），
然后根据负载均衡算法从列表中选择一台服务器进行消息发送。NameServer 与每台 Broker 服务保持长连接，并间隔 30S 检查 Broker 是否存活，
如果检测到Broker 宕机，则从路由注册表中将其移除，这样就可以实现 RocketMQ 的高可用  

② Broker高可用  
每个Broker与Name Server集群中的所有节点建立长连接，定时(每隔30s)注册Topic信息到所有Name Server。Name Server定时(每隔10s)扫描所有
存活broker的连接，如果Name Server超过2分钟没有收到心跳，则Name Server断开与Broker的连接  

③ 生产者高可用  
Producer与NameServer集群中的其中一个节点（随机选择）建立长连接，定期从NameServer取Topic路由信息，并向提供Topic服务的Master建立长连接，
且定时向Master发送心跳。Producer完全无状态，可集群部署  
Producer每隔30s（由ClientConfig的pollNameServerInterval）从Name server获取所有topic队列的最新情况，这意味着如果Broker不可用，
Producer最多30s能够感知，在此期间内发往Broker的所有消息都会失败  
Producer每隔30s（由ClientConfig中heartbeatBrokerInterval决定）向所有关联的broker发送心跳，Broker每隔10s中扫描所有存活的连接，
如果Broker在2分钟内没有收到心跳数据，则关闭与Producer的连接  

④ 消费者高可用  
Consumer与Name Server集群中的其中一个节点(随机选择)建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master、Slave建立长连接，
且定时向Master、Slave发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定  
Consumer每隔30s从Name server获取topic的最新队列情况，这意味着Broker不可用时，Consumer最多最需要30s才能感知  
Consumer每隔30s（由ClientConfig中heartbeatBrokerInterval决定）向所有关联的broker发送心跳，Broker每隔10s扫描所有存活的连接，
若某个连接2分钟内没有发送心跳数据，则关闭连接；并向该Consumer Group的所有Consumer发出通知，Group内的Consumer重新分配队列，然后继续消费  
当Consumer得到master宕机通知后，转向slave消费，slave不能保证master的消息100%都同步过来了，因此会有少量的消息丢失。但是一旦master恢复，
未同步过去的消息会被最终消费掉

# 消费者消费问题
### 消息顺序
顺序消费表示消息消费的顺序和生产者为每个消息队列发送信息时候的顺序一致，所以如果正在处理全局顺序是强制性的场景，需要确保使用的主题只有一个消息队列  
并行消费不再保证消息顺序，消费的最大并行数量受每个消费者客户端指定的线程池限制

### 消费者数量和队列数量的关系
① 如果消费者consumer机器数量和消息队列相等，则消息队列平均分配到每一个consumer上  
② 如果consumer数量大于消息队列数量，则超出消息队列数量的机器没有可以处理的消息队列  
③ 若消息队列数量不是consumer的整数倍，则部分consumer会承担跟多的消息队列的消费任务  
在扩容consumer实例数量的同时，必须同步扩容主题中的分区数量，确保consumer的实例数和分区数量是相等的  
如果consumer的实例数量超过分区数量，这样的扩容实际上是没有效果的。因为对于消费者来说，在每个分区上实际上只能支持单线程消费  

# 过滤消息

