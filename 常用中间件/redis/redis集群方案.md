# redis主从复制
主节点不用做任何修改，直接启动服务即可。从节点需要修改```redis.conf```配置文件，加入配置```slaveof <主节点ip地址> <主节点端口号>```  
master节点负责写操作，slave节点负责读操作。因此可以通过这种方式配置多个从节点进行读操作，主节点进行写操作，实现读写分离  

> 主从复制模式下对于主节点没有实现高可用

# 哨兵机制
## 基础概念
redis具备主从复制的功能，但是当主服务器发生故障时，需要人工干预，修改每个实例的配置文件，然后重启，这样不仅费时费力，还有可能导致出错概率
比较高，最终导致服务不可用。那么哨兵机制解决了这个问题，帮我们自动实现故障转移，不需要人工干预，高效、精准的实现故障转移  
① 集群监控：负责监控redis master和slave进程是否正常工作  

> 哨兵有哪些监控  
> ① 哨兵节点每隔10秒向主节点和从节点发送info指令，获取最新的拓扑结构，更新自身保存的节点信息  
> ② 每隔2秒哨兵节点向redis指定频道上发送哨兵节点对主节点的判断和哨兵节点自身的信息，其他哨兵节点也会订阅这个频道，
> 来了解其他节点的信息及对主节点的判断  
> ③ 每隔1秒哨兵向redis节点和其他哨兵节点发送ping信息，进行心跳检测

② 消息通知：如果某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员  
③ 故障转移：如果master node挂掉了，会自动转移到salve node上  
④ 配置中心：如果故障转移发生了，通知client客户端新的master地址

```
sentinel命令说明：  
① sentinel monitor mymaster 127.0.0.1 6379 1  
mymaster主节点名，可以任意起名，但必须和后面的配置保持一致  
127.0.0.1 主节点连接地址  
1 将主服务器判断为失效需要投票，这里设置至少需要1个sentinel同意  

② sentinel down-after-milliseconds mymaster 10000  
设置sentinel认为服务器已经短线所需的毫秒数  

③ sentinel failover-timeout mymaster 60000  
设置failover（故障转移）的过期时间。当failover开始后，在此时间内仍然没有触发任何failover操作，当前sentinel会认为此次failover失败  

④ sentinel parallel-syncs mymaster 1  
故障转移之后，进行新的主从复制，配置项指定了最多有多少个slave对新的master进行同步，那可以理解为1是串行复制，大于1是并行复制
```

## 主观下线和客观下线
主观下线：当sentinel系统中其中一个server认为redis中某个实例宕机或不可用，则就标记为主观下线  
客观下线：如果被标记主观下线的redis实例是主节点，则还需要获得其他sentinel节点的判断，如果超过法定的数量的投票认为该redis server不可用，
则标记该redis 主节点为客观下线

## 主节点客观下线后，怎么进行故障转移
主要是两步，第一步是选举哨兵leader，第二步是由哨兵leader负责故障转移
### 选举哨兵leader
① 当master下线后，每个哨兵都可以选择自己作为leader，将其他发送给其他哨兵  
② 其他哨兵接收到请求后，可以选择同意或者不同意（根据判定基础决定）  
③ 如果最终某个哨兵节点获得了超过半数的投票，则该哨兵节点就成为了leader，负责故障转移

### 故障转移
##### 选举新的master
① 过滤与哨兵断开连接时间比较长的节点  
② 优先选择replica-priority低的  
③ 选择偏移量比较大的（表明复制的数据越多）  
④ 运行id越小

##### 执行故障转移
① sentinel leader向新主节点发送```slave no one```命令，让它成为独立节点  
② sentinel leader向其他从节点发送```slave ip port```，让它从新的主节点同步

## client是和哨兵通信还是和redis主从通信
① client连接集群，首先会连接sentinel，然后订阅相关的频道获取主从切换、切换进度、新master地址等信息  
② 拿到redis连接地址后，则会与redis master建立连接  
③ 当sentinel执行了故障转移，选举了新的redis master之后，也会在client订阅的频道中发送最新的master redis地址  
④ client拿到最新的地址后，还是同样的建立新的redis连接

# redis-cluster
