# CompletableFuture

优点：  
1 异步任务结束时，会自动回调某个对象的方法  
2 主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行  
3 异步任务出错时，会自动回调某个对象的方法

```java
// 获得结果
get()
get(long timeout,TimeUnit unit)
join()
getNow(T valueIfAbsent)

// 主动触发计算
complete(T valueIfAbsent)

// 对计算结果进行处理
thenApply()：当前步出错，不走下一步
handle()：当前步错，可以带着错误走下一步

// 对计算结果进行消费
thenAccept：接收任务的处理结果，并消费处理，无返回结果

// 对计算速度进行选用
applyA.applyToEither(applyB,...)

// 对计算结果进行合并
thenCombine
```

总结：
①没有传入自定义线程池，都用默认线程池ForkJoinPool
②传入自定义线程池
如果执行第一个任务时，传入了一个自定义线程池，调用thenRun方法执行第二个任务时，则第二个任务和第一个任务共用同一个线程池
如果执行第一个任务时，传入了一个自定义线程池，调用thenRunAsync执行第二个任务时，则第一个任务使用的是自定义线程池，第二个任务使用的是ForkJoinPool
③第一个任务处理太快，按照系统优化切换原则，会直接使用main线程处理