# 概述
同步场景下，使用```seata```的AT模式或者TCC模式解决一致性问题  
AT模式适合的场景：基于支持本地ACID事务的关系型数据库；Java应用，通过JDBC访问数据库  
TCC模式支持的场景：需要自定义实现```prepare```、```commit```、```rollback```的逻辑，适合非关系型数据库  
通过对本地关系数据库的分支事务的协调来驱动完成全局事务，是工作在应用层的中间件。```Seata``` 把一个分布式事务理解成一个包含了若干分支事务的全局事务。
全局事务的职责是协调其下管辖的分支事务达成一致，要么一起成功提交，要么一起失败回滚。此外，通常分支事务本身就是一个关系数据库的本地事务

![img.png](images/AT模式示意图.png)
① TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID  
② XID 在微服务调用链路的上下文中传播  
③ RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖  
④ TM 向 TC 发起针对 XID 的全局提交或回滚决议  
⑤ TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求

### 配置数据源代理
```Seata```是通过代理数据源实现分布式事务，所以需要配置```io.seata.rm.datasource.DataSourceProxy```的Bean，
且是@Primary默认的数据源，否则事务不会回滚，无法实现分布式事务
```java
// 因为使用了mybatis的starter所以需要排除DataSourceAutoConfiguration，不然会产生循环依赖  
// @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class DataSourceProxyConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DruidDataSource druidDataSource() {
        return new DruidDataSource();
    }

    @Primary
    @Bean
    public DataSourceProxy dataSourceProxy(DruidDataSource druidDataSource) {
        return new DataSourceProxy(druidDataSource);
    }
}
```

### AT模式的核心要点
AT模式分成两个阶段，主要逻辑全部在第一个阶段，第二阶段主要做回滚或日志清理工作  

① 一阶段中将业务数据和回滚日志记录在同一个本地事务中提交，然后释放本地锁和连接资源  
在第一阶段中，```seata```会拦截业务SQL，首先解析SQL语义，找到要操作的业务数据，在数据被操作前，保存下来记录undo log，然后执行业务SQL更新数据，更新之后再次保存数据redo log，
这些操作都在本地数据库事务内完成，这样保证了一阶段的原子性  

② 二阶段提交时异步化，非常快速的完成；回滚时通过一阶段的回滚日志进行反向补偿  
相比一阶段，二阶段比较简单，负责整体的回滚和提交，如果之前的一阶段中有本地事务没有通过，那么就执行全局回滚，否则执行全局提交，回滚用到的就是一阶段记录的undo log，通过回滚记录生成
反向更新SQL并执行，以完成分治的回滚。当然事务完成后会释放所有资源和删除所有日志

### ```seata```的实现说明
两阶段实现：第一阶段是将业务数据和回滚日志记录在同一个本地事务中提交，然后释放掉锁；和连接资源；第二阶段由协调者判断如果成功，进行异步提交，
如果失败则通过一阶段的回滚日志进行反向补偿。

# 总结
总的来说在```Seata```的中AT模式基本可以满足百分之80的分布式事务的业务需求，AT模式实现的是最终一致性，所以可能存在中间状态，而XA模式实现的强一致性，所以效率较低一点，
而Saga可以用来处理不同开发语言之间的分布式事务，所以关于分布式事务的四大模型，基本可以满足所有的业务场景，其中XA和AT没有业务侵入性，而Saga和TCC具有一定的业务侵入。




