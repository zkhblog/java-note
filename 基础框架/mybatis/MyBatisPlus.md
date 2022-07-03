# 主键策略  
①数据库ID自增（id-type: AUTO）  
②自定义主键策略（）

# 元数据拦截器


# 自定义全局方法
```java
ISqlInjector
```
### 应用一：逻辑删除


# 元数据处理器
```java
MetaObjectHandler
```

### 应用一：公共字段填充
①注解填充字段@TableFile(fill=FieldFill.INSERT)  
②自定义公共字段填充处理器  
③MP全局注入

# MP自带插件
1 分页插件  

2 执行性分析插件  
**不建议在生产环境使用**  
利用了explain分析，分析的结果中extra字段若不为```using where```，则是全表更新，需要被禁止的操作  

3 性能分析插件  
@Deprecated   **不建议在生产环境使用**  

4 乐观锁插件  
利用版本号机制实现