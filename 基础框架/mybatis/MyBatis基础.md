# 插件机制
1 MyBatis通过插件(Interceptor)可以做到拦截四大对象相关方法的执行，根据需要来完成相关数据的动态改变  
2 四大对象在各自创建时，都会执行interceptorChain.pluginAll()，会经过每个插件对象的plugin()方法，目的是为当前的四大对象创建代理

一、主要概念  
构建SqlSessionFactory  
①可以通过SqlSessionFactoryBuilder获得  
②从XML文件构建SqlSessionFactory的实例  
③直接从Java代码创建配置  

二、配置  
2.1 configuration（配置）

2.2 properties（属性）  
2.2.1 如果一个属性在不只一个地方进行了配置，MyBatis将按照下面顺序进行加载  
①首先读取在properties元素体内指定的属性  
②然后根据properties元素中的resource属性读取类路径下属性文件，或根据url属性指定的路径读取属性文件，并覆盖之前读取过的同名属性  
③最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性  

2.2.2 启用默认值特性  
```
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- 启用默认值特性 -->
</properties>
```

2.3 settings（设置）
```
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

2.4 typeAliases（类型别名）
2.4.1 MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。
2.4.2 实现自定义类型处理器：实现org.apache.ibatis.type.TypeHandler 接口， 或继承类 org.apache.ibatis.type.BaseTypeHandler  
2.4.3 改变类型处理器处理的Java类型  
一是在类型处理器的配置元素（typeHandler元素）上增加一个javaType属性（比如：javaType="String"）  
二是在类型处理器的类上增加一个@MappedTypes注解指定与其关联的Java类型列表。如果在javaType属性中也同时指定，则注解上的配置将被忽略  
2.4.4 改变关联的JDBC类型  
在类型处理器的配置元素上增加一个jdbcType属性（比如：jdbcType="VARCHAR"）
在类型处理器的类上增加一个@MappedJdbcTypes注解指定与其关联的JDBC类型列表。如果两者同时指定，则注解上的配置将被忽略  


2.5 typeHandlers（类型处理器）
定义：

2.6 objectFactory（对象工厂）
MyBatis在创建结果对象的新实例时，都会使用对象工厂实例来完成。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候
通过参数构造方法来实例化。如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。

2.7 plugins（插件）
其作用就是在这些被拦截的方法执行前后加上某些逻辑，也可以在在执行这些被拦截的方法时执行自己的逻辑而不再执行被拦截的方法。

实现方法：
①创建拦截器，并指定要拦截的方法签名  
②继承配置类后覆盖其中的某个方法，再将其传递到SqlSessionFactoryBuilder.build(config)方法即可

2.8 environments（环境配置）
每个数据库对应一个 SqlSessionFactory 实例

2.8.1 transactionManager（事务管理器）
在MyBatis中有两种类型的事务管理器（type="JDBC|MANAGED"）
2.8.1.1 JDBC
这个配置直接使用了JDBC的提交和回滚设施，依赖从数据源获得的连接来管理事务作用域  
2.8.1.2 MANAGED

> 在事务管理器实例化后，所有在XML中配置的属性将会被传递给setProperties()方法。除了事务管理器外，还需要创建一个Transaction接口的实现类。
> 通过这两个接口，可以完全自定义MyBatis对事务的处理

> 如果你正在使用Spring+MyBatis，则有必要配置事务管理器，因为Spring模块会使用自带的管理器来覆盖前面的配置

2.8.2 dataSource（数据源）

2.9 databaseIdProvider（数据库厂商标识）

2.10 mappers（映射器）
指定映射文件的位置，