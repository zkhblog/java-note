```
https://mp.weixin.qq.com/s?__biz=MzIyODE5NjUwNQ==&mid=2653347271&idx=1&sn=8c34202a99f11cfd0cc86f4a837d7345&chksm=f387d431c4f05d2749f909f2200a24da0fe25a84e5d8ff647e98de3dac15cc18e5432738a867&scene=21#wechat_redirect
```
# 事务回滚规则
是指spring事务管理器回滚一个事务的推荐方法是在当前事务的上下文内抛出异常。spring事务管理器会捕捉任何未处理的异常，然后依据规则决定是否回滚抛出异常的事务。
默认配置下，spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。
可以明确的配置在抛出那些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过setRollbackOnly()方法来指示一个事务必须回滚，
在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

# 事务原理