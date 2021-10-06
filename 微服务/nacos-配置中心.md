# 核心概念
1、命名空间：用作配置隔离。（一般每个微服务一个命名空间）
①默认public。默认新增的配置都在public空间下。
②开发、测试、生产等环境可以用命名空间分割。利用命名空间做环境隔离。
做法：在bootstrap.properties配置上指定需要使用哪个命名空间的配置。
spring.cloud.nacos.config.namespace=
③也可以为每个微服务配置一个命名空间，微服务互相隔离，只加载自己命名空间下的所有配置。
2、配置集：一组相关或不相关配置项的集合。
3、配置集ID：类似于配置文件名，即Data ID
4、配置分组：默认所有的配置集都属于DEFAULT_GROUP。自己可以创建分组，比如双十一，618，双十二
spring.cloud.nacos.config.group=DEFAULT_GROUP  # 更改配置分组
5、最终方案：每个微服务创建自己的命名空间，然后使用配置分组区分环境（dev/test/prod）
6、加载多配置集
注意：存在默认分组，如果默认分组没有的配置，而配置文件里面有，则使用配置文件中的。
使用方法：
spring.cloud.nacos.config.ext-config[0].data-id=***.yml
spring.cloud.nacos.config.ext-config[0].group=***.yml
spring.cloud.nacos.config.ext-config[0].refresh=***.yml

