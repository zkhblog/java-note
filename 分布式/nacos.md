# 配置中心管理
①命名空间：用作配置隔离。默认public。默认新增的配置都在public空间下。  
在bootstrap.properties配置上指定需要使用哪个命名空间的配置。
```
spring.cloud.nacos.config.namespace=7fd7e137-21c4-4723-a042-d527149e63e0
```
②配置分组：默认属于DEFAULT_GROUP。  
```
# 指定分组
spring.cloud.nacos.config.ext-config[0].group=provider
```
③加载多配置文件  
在bootstrap.properties中添加：  
```
spring.cloud.nacos.config.ext-config[0].data-id=redis.properties  
# 开启动态刷新配置，否则配置文件修改，工程无法感知  
spring.cloud.nacos.config.ext-config[0].refresh=true
```
最佳实践：命名空间区分业务功能，分组区分环境