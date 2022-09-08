# 注册中心

# 配置中心
## 配置中心管理
①命名空间：用于配置隔离。默认使用public命名空间下的配置。  
在bootstrap.properties上指定使用哪个命名空间的配置```spring.cloud.nacos.config.namespace=7fd7e137-21c4-4723-a042-d527149e63e0```  

②配置分组：默认所有的配置集都属于DEFAULT_GROUP。  
指定配置分组```spring.cloud.nacos.config.ext-config[0].group=provider```

③加载多配置文件  
在bootstrap.properties中添加：  
```
spring.cloud.nacos.config.ext-config[0].data-id=redis.properties  
# 开启动态刷新配置，否则配置文件修改，工程无法感知  
spring.cloud.nacos.config.ext-config[0].refresh=true
```

## 最佳实践
1 命名空间区分业务功能，分组区分环境  
2 配置中心配置数据集DataID和配置内容  
3 开启动态刷新配置```@RefreshScope```  
4 获取配置项的值```@Value```  
5 优先使用配置中心的配置  
6 使用命名空间nameSpace来创建各服务的配置  
7 使用分组group来区分不同的环境  
8 使用多配置集extension-configs区分不同类型的配置