# 事务回滚规则
是指spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。
默认配置下，spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。
可以明确的配置在抛出那些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过setRollbackOnly()方法来指示一个事务必须回滚，
在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

# 本地事务失效问题
同一个对象内事务方法互调默认失效，原因是绕过了代理对象，因为事务是使用代理对象进行控制的  
解决办法：可以使用代理对象来调用事务方法  
① 引入```spring-boot-starter-aop```，它引入了aspectj  
② 使用```@EnableAspectJAutoProxy(exposeProxy=true)```注解开启动态代理功能，以后所有的动态代理都是aspectj创建的（不引入aspectj，默认都是通过jdk创建动态代理）  
③
````
OrderServiceImpl orderService = (OrderServiceImpl) AopContent.currentProxy();  
orderService.b();
````