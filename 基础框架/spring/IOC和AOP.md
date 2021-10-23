# 一 IOC
## 1.1 bean的生命周期

## 1.2 循环依赖
一级缓存：成品对象
二级缓存：半成品对象
三级缓存：lambda表达式

### 1.2.1 三级缓存的泛型研究
ObjectFactory函数式接口的getObject()方法 ：当作一个参数传递到方法中，一般情况下传的是lambda表达式，
可以通过getObject()方法来执行lambda表达式

### 1.2.2 创建流程中的核心步骤
getBean()->doGetBean()->createBean()->doCreateBean()->createBeanInstance()->populateBean()

### 1.2.3 缓存个数
如果只有一个缓存，能不能解决循环依赖：不能区分成品对象和半成品对象。不能解决。  
如果有两个缓存，能不能解决循环依赖：能解决。但是循环依赖过程中包含了代理对象的创建，那么就必须要使用三级缓存了。

三级缓存能解决的原因

# 二 AOP
## 2.1 