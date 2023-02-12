# 概述
同步场景下，使用```seata```的AT模式或者TCC模式解决一致性问题  

1 AT模式适合的场景  
① 基于支持本地ACID事务的关系型数据库  
② Java应用，通过JDBC访问数据库  

2 TCC模式支持的场景
需要自定义实现```prepare```、```commit```、```rollback```的逻辑，适合非关系型数据库  

### 核心概念包括以下3个角色
TM：事务的发起者。用来告诉 TC，全局事务的开始，提交，回滚  
RM：具体的事务资源，每一个 RM 都会作为一个分支事务注册在 TC  
TC：事务的协调者```seata-server```，用于接收我们的事务的注册，提交和回滚

### ```seata```的使用
##### 下载解压```seata```的server端  
① 修改 conf/registry.conf 配置，设置 registry 和 config 节点中的type为```nacos```  
② 修改 conf/```nacos-config.txt```配置，将 ```store.mode``` 改为db，并修改数据库相关配置  
③ 初始化```seata```的```nacos```配置，成功后在```nacos```的配置列表中能看到```seata```的相关配置

##### 应用配置
① 需在业务相关的数据库中添加 undo_log 表，用于保存需要回滚的数据  
② 直接把 ```seata-server``` 中的registry.conf复制到每个服务中去即可，不需要修改  
③ 每个服务各自修改配置文件  
④ 配置数据源代理  
```Seata```是通过代理数据源实现分布式事务，所以需要配置```io.seata.rm.datasource.DataSourceProxy```的Bean，
且是@Primary默认的数据源，否则事务不会回滚，无法实现分布式事务
```java
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
因为使用了mybatis的starter所以需要排除DataSourceAutoConfiguration，不然会产生循环依赖  
```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
```
⑤ 事务发起者添加全局事务注解````@GlobalTransactional````  



