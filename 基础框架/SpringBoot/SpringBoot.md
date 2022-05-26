# 自动配置
自动配好Tomcat、Spring MVC、WEB常见功能(字符编码问题、文件上传等)、默认包结构 ...  
```@SpringBootApplication```注解是合成注解
```
①@EnableAutoConfiguration  
该注解上标注有AutoConfigurationPackage注解，该注解又给容器中注册了AutoConfigurationPackages.Registrar.class组件  
AutoConfigurationPackages.Registrar.class，该组件指定了默认的包扫描路径
// 利用静态内部类给容器中导入一系列组件，实现是通过获取AutoConfigurationPackage该注解标注的类所在包名，将该包下所有组件导入进容器中，这是包扫描的原理
```

```
②@EnableAutoConfiguration  
该注解向容器中导入了AutoConfigurationImportSelector组件  
1、getAutoConfigurationEntry(annotationMetadata);                                                 // 给容器中批量导入一些组件  
2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)    // 获取到所有需要导入到容器中的配置类  
3、利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)     // 得到所有的组件  
4、classLoader.getResources(FACTORIES_RESOURCE_LOCATION)                                          // 扫描当前系统里面所有jar包下/META-INF/spring.factories位置的文件（核心是spring-boot-autoconfigure包下的配置文件）
```

```
虽然自动配置类在项目启动时会全部被加载一遍，但是在条件装配规则之下，最终会按需配置  
@ConditionalOnBean              // 当容器里有指定的bean的条件下  
@ConditionalOnMissingBean       // 当容器里不存在指定bean的条件下  
@ConditionalOnClass             // 当类路径下有指定类下  
@ConditionalOnMissingClass      // 当类路径下不存在指定类的条件下  
@ConditionalOnProperty          // 指定的属性是否有指定的值
比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，
代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。
```
### 自动配置总结
① SpringBoot先加载所有的自动配置类```xxxxxAutoConfiguration```  
② 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。从```xxxxProperties```里面拿。```xxxProperties```和配置文件进行绑定  
③ 生效的配置类就会给容器中装配很多组件  
④ 只要容器中有这些组件，相当于这些功能就有了  
⑤ 定制化配置。用户直接自己@Bean替换底层的组件，这个组件获取的配置文件中的值，用户可以修改  

# yml配置
1 单引号会将```\n```作为字符串输出，双引号会将```\n```作为换行符输出  
2 自定义的类和配置文件绑定一般没有提示  
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-configuration-processor</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# 静态资源访问原理
请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应```404``页面  

改变默认的静态资源路径  
```
spring:
  mvc:
    static-path-pattern: /res/**

  resources:
    static-locations: [classpath:/haha/]
```

静态资源访问前缀，默认无前缀  
```
spring:
  mvc:
    static-path-pattern: /res/**
```

