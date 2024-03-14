# 事务失效

①事务方法被final、static关键字修饰

②配置```@Transactional(readOnly = true)```

③事务超时时间设置过短

④使用了错误的事务传播机制，例如```Propagation.NOT_SUPPORTED```不支持事务

⑤rollbackFor属性指定的异常必须是Throwable或者其子类，并且正确指定

⑥事务注解被覆盖导致事务失效

```java
public interface MyRepository {
    @Transactional
    void save(String data);
}

public class MyRepositoryImpl implements MyRepository {
    @Override
    public void save(String data) {
        // 数据库操作
    }
}

public class MyService {

    @Autowired
    private MyRepository myRepository;

    @Transactional
    public void doSomething(String data) {
        myRepository.save(data);
    }
}

public class MyTianluoService extends MyService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void doSomething(String data) {
        super.doSomething(data);
    }
}
```

⑦事务多线程调用

在Spring事务管理器中，通过TransactionSynchronizationManager类来管理事务上下文。TransactionSynchrondizationManager内部维护了一个ThreadLocal对象，用来存储当前线程的事务上下文。在事务开始时，TransactionSynchronizationManager会将事务上下文绑定到当前线程的ThreadLocal对象中，当事务结束时，TransactionSynchronizationManager会将事务上下文从ThreadLocal对象中移除。

⑧异常类型问题

Spring默认只处理RuntimeException和Error，对于普通的Exception不会回滚，除非，用rollbackFor属性指定配置。```@Transactional(rollbackFor=Exception.class)```

对于网络超时、文件读写等checked异常，Spring无法回滚。而对于unchecked异常，事务的回滚是有效的。