# 实际使用案例
### 客户端向服务端写数据流程
① 写请求直接发送给Leader节点  
![img.png](../images/写请求直接发送给Leader节点.png)

② 写请求发送给follower节点  
![img.png](../images/写请求发送给follower节点.png)

### 监听服务器动态上下线
![img.png](../images/监听服务器动态上下线.png)

# 选举机制
### 第一次启动的选举机制
![img.png](../images/第一次启动的选举机制.png)

### 非第一次启动的选举机制
![img.png](../images/非第一次选举.png)

# 分布式锁

# Paxos算法、ZAB算法

# 源码解读