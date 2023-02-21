# 主从复制
主节点不用做任何修改，直接启动服务即可。从节点需要修改```redis.conf```配置文件，加入配置```slaveof <主节点ip地址> <主节点端口号>```  
master节点负责写操作，slave节点负责读操作。因此可以通过这种方式配置多个从节点进行读操作，主节点进行写操作，实现读写分离  

主从复制模式下对于主节点没有实现高可用

# 哨兵
```yaml
# 外部可以访问
bind 0.0.0.0
sentinel monitor mymaster 127.0.0.1 6379 1
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

# 注意：如果有 sentinel monitor mymaster 127.0.0.1 6379 2 配置，则注释掉
```
> 参数说明：  
> ① ```sentinel monitor mymaster 127.0.0.1 6379 1```  
> ```mymaster```主节点名，可以任意起名，但必须和后面的配置保持一致  
> ```127.0.0.1``` 主节点连接地址  
> ```1``` 将主服务器判断为失效需要投票，这里设置至少需要1个sentinel同意  
> 
> ② ```sentinel down-after-milliseconds mymaster 10000```  
> 设置sentinel认为服务器已经短线所需的毫秒数  
> 
> ③ ```sentinel failover-timeout mymaster 60000```  
> 设置failover（故障转移）的过期时间。当failover开始后，在此时间内仍然没有触发任何failover操作，当前sentinel会认为此次failover失败  
> 
> ④ ```sentinel parallel-syncs mymaster 1```  
> 故障转移之后，进行新的主从复制，配置项指定了最多有多少个slave对新的master进行同步，那可以理解为1是串行复制，大于1是并行复制

# redis-cluster
