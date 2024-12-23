 #  BeanFactory和ApplicationContext

HierarchicalBeanFactory 定义了父子工厂，用于实现父子容器。

ListableBeanFactory的重要实现是DefaultListableBeanFactory，保存了ioc容器中的核心信息，即BeanDefinition。

AutowireCapableBeanFactory提供了自动装配功能。

ApplicationContext扩展了BeanFactory，提供了更多高级的功能，比如事件传播、AOP编程支持等，而BeanFactory主要提供了基本的依赖注入功能，使用BeanFactory通常需要手动管理Bean的生命周期，例如通过调用getBean()方法来获取Bean的实例，而ApplicationContext则会自动加载Bean定义文件，如XML配置文件或者注解定义的Bean，在启动容器时就会自动实例化单例Bean。

# BeanDefinition的加载过程

### 加载资源：dom解析

```java

Resource接口
ResourceLoader接口

伪代码如下：
ResourceLoader {
  Resource getResource(String location);
}

策略模式的理解
```

### 注册组件

### DefaultListableBeanFactory

# IOC容器的刷新过程



# @Autowired、@Value、@Inject

# 增强器

①```SmartInstantiationAwareBeanPostProcessor ```，该增强器是Bean在进行代理增强期间进行使用

②

# FactoryBean独有的初始化方式

①FactoryBean在Spring容器中一开始保存的是工厂本身

②在第一次获取目标组件时，也就是FactoryBean能产生的对象时

③Spring.getBean会在底层所有组件挨个遍历哪个组件的类型是目标组件

④找到FactoryBean，发现它是工厂，它能产生目标组件

⑤调用工厂方法（getObject）创建目标对象

⑥普通的单实例Bean保存在singletonObject这里面

⑦FactoryBean产生的Bean，缓存在factoryBeanCache中，下一次直接从这里拿，所有FactoryBean默认还是单实例

# 循环引用问题的解决

```java
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean)); //三级缓存中的Bean也会被后置处理来增强，
		}
```

# AOP

