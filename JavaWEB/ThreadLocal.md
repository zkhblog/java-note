---------------------------------
https://mp.weixin.qq.com/s/bECVeuxE-WIYmvXbF2V3QA  
https://mp.weixin.qq.com/s?__biz=MzkxNTE3NjQ3MA==&mid=2247491733&idx=1&sn=2a4efe9f12a6d3009d89d703e7dadaa5&chksm=c1618decf61604fa0eb46bb65e31248db2bd555527dc558b19c5a60a2be634a24e5e1bc79e0c&scene=21#wechat_redirect
---------------------------------
# 概念
实现每一个线程都有自己专属的本地变量副本，通过get()和set()方法，获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题  
必须回收自定义的ThreadLocal变量，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的ThreadLocal变量，可能会影响后续业务逻辑和造成内存泄露问题。尽量使用try-finally块进行回收  

# 内存泄露
不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄露

### 软引用的适用场景
假如有一个应用需要读取大量的本地图片，如果每次读取图片都从硬盘读取则会严重影响性能，但是一次加载到内存中又可能会造成内存溢出  
此种场景，可以用软引用解决这个问题，设计思路是用一个hashMap保存图片路径和与相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的内存空间，从而有效
避免OOM的问题  
```java
Map<String,SoftReference<BitMap>> imageCache = new HashMap<String,SoftReference<BitMap>>();
```

# ThreadLocal的问题
1 entry对象的key为什么是弱引用指向ThreadLocal对象？  
因为自定义指向ThreadLocal对象的强引用变量在方法结束后，会被销毁，但此时ThreadLocalMap中某个entry对象的key还指向了该ThreadLocalMap对象，如果是强引用，就会导致key指向的
ThreadLocal对象不能被gc回收，造成内存泄露。但是如果是弱引用，则会大概率减少内存泄露的问题，可以使ThreadLocal对象在方法执行完毕后顺利被回收且entry的key引用指向null

2 key为null的entry的回收？
此种场景下，在thread运行结束后，ThreadLocal、ThreadLocalMap、Entry没有引用可达，在gc时会进行垃圾回收。但是在线程池场景下，线程复用就会出现内存泄露问题。  
此后调用get、set或remove方法时，就会尝试删除key为null的entry，可以释放value对象所占用的内存。**因此在不使用某个ThreadLocal对象后，需要手动调用remove方法来删除它**  

# 最佳实践
① 手动初始化  
② 建议把ThreadLocal修饰为static  
③ 用完记得手动remove

# ThreadLocal总结
1 ThreadLocal并不解决线程间数据共享的问题  
2 ThreadLocal适用于变量在线程间隔离且在方法间共享的场景  
3 ThreadLocal通过隐式的在不同线程内创建独立实例副本避免了实例线程安全的问题  
4 每个线程持有一个只属于自己的专属Map，并维护了ThreadLocal对象与具体实例的映射，该map由于只被持有它的线程访问，故不存在线程安全以及锁的问题  
5 ThreadLocalMap的Entry对ThreadLocal的引用为弱引用，避免了ThreadLocal对象无法被回收的问题  
6 会通过expungeStaleEntry、cleanSomeSlots、replaceStaleEntry这三个方法回收key为null的Entry对象的值（即具体实例）以及Entry对象本身从而防止内存泄露