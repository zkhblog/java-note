# 线程安全集合类
① 使用```CopyOnWriteArrayList```代替```ArrayList```  
② 使用```CopyOnWriteArraySet```代替```HashSet```  
③ 使用```ConcurrentHashMap```代替```HashMap```

# Lock和Condition

# ReentrantLock与ReentrantReadWriteLock
ReentrantLock实现了标准的互斥操作，也就是一次只能有一个线程持有锁，也即所谓独占锁的概念。开销较大。  
ReentrantReadWriteLock描述的是一个资源能够被多个读线程访问，或者被一个写线程访问，但是不能同时存在读写线程。读写锁的使用场景是一个共享资源被
大量读取操作，而只有少量的写操作。同一个线程可以拥有writeLock与readLock，但必须先获取 writeLock 再获取 readLock, 反过来进行获取会导致死锁

# AtomicXXXXXXX - 原子操作类

# 并发队列


# 辅助类
### CountDownLatch
CountDownLatch可以实现类似计数器的功能，计数器的初始值为指定的线程的数量，每当一个线程完成了自己的任务，计数器的值就会减1，
当计数器的值达到了0时，它表示所有的线程都完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。构造器上的计数值实际上就是闭锁需要等待的线程数量，
这个值只能被设置一次，而且CountDownLatch没有提供任何机制去重新设置这个值

```
public CountDownLatch(int count);   // 指定计数的线程

public void await();                // 让当前线程等待
public vod countDown();             // 减少需要等待的线程数量
```

### CyclicBarrier
让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被 屏障拦截的线程才会继续下面的业务
```
public CyclicBarrier(int parties, Runnable barrierAction);
    参数1:parties表示这组线程的数量;
    参数2:barrierAction 表示一组线程都到达之后需要执行的任务;
    
public int await();             // 让当前线程阻塞   
```

### Semaphore
最多允许多少个线程同时执行acquire方法和release方法之间的代码，如果方法acquire没有参数则默认是一个许可
```
public Semaphore(int permits);  // 参数permits称为许可证，即最大的线程并发数量
public void acquire();          // 表示获取许可证
public void release();          // 表示释放许可证
```

### Exchanger
Exchanger是用于线程间协作的工具类，用于线程间的数据交换，它提供一个同步点，在这个同步点两个线程可以交换彼此的数据。
这两个线程通过exchange方法交换数据， 如果第一个线程先执行exchange方法，它会一直等待第二个线程也执行exchange，当两个线程都到达同步点时，
这两个线程就可以交换数据，将本线程生产出来的数据传递给对方

