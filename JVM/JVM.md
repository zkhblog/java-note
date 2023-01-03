# dump文件
### 获取dump文件
方法一：通过```jmap```工具生成，可以生成任意一个java进程的dump文件  

方法二：通过配置jvm参数生成  
选项```-XX:+HeapDumpOnOutOfMemoryError```或```-XX:+HeapDumpBeforeFullGC```
选项```-XX:HeapDumpPath```所代表的含义就是当程序出现OutOfMemory时，将会在相应目录下生成一份dump文件，如果不指定选项```-XX:HeapDumpPath```则在当前目录下生成dump文件

> 考虑到生产环境中几乎不可能在线对其进行分析，大都是采用离线分析，因此使用jMap + MAT工具是最常见的组合

方法三：使用VisualVM可以导出堆dump文件  

方法四：使用MAT既可以打开一个已有的堆快照，也可以通过MAT直接从活动Java程序中导出堆快照。该功能将借助jps列出当前正在运行的Java进程，以供选择并获取快照

### dump文件内容
MAT可以分析heap dump文件。在进行内存分析时，只要获得了反映当前设备内存映像的hprof文件，通过MAT打开就可以直观地看到当前的内存信息。  
一般说来，这些内存信息包含：  
① 所有的对象信息，包括对象实例、成员变量、存储于栈中的基本类型值和存储于堆中的其他对象的引用值  
② 所有的类信息，包括classLoader、类名称、父类、静态变量等  
③ GC Root到所有的这些对象的引用路径  
④ 线程信息，包括线程的调用栈及此线程的线程局部变量（TLS）

# Java中内存泄露的8种情况  
① 静态集合类  
静态集合类，如hashMap，LinkedList等等，如果这些容器为静态的，那么他们的生命周期与JVM程序一致，则容器中的对象在程序结束之前将不能被释放，从而造成
内存泄露。简单而言，长生命周期的对象持有短生命周期对象的引用，尽管短生命周期的对象不再使用，但是因为长生命周期对象持有它的引用而导致不能被回收。
```
public class MemoryLeak{
    static List list=new ArrayList();
    
    public void oomTest(){
        Object obj=new Object();
        list.add(obj);
    }
}
```

② 单例模式  
和静态集合类导致内存泄露的原因类似，因为单例的静态特性，它的生命周期和JVM的生命周期一样长，所以如果单例对象持有外部对象的引用，那么这个外部对象
也不会被回收，那么就会造成内存泄露  

③ 内部类持有外部类  
内部类持有外部类，如果一个外部类的实例对象的方法返回了一个内部类的实例对象。这个内部类对象长期被引用，即使那个外部类实例对象不再被使用，
但是由于内部类持有外部类的实例对象，这个外部类对象将不会被垃圾回收，将造成内存泄露  

④ 各种连接，如数据库连接，网络连接和IO连接等
在对数据库进行操作的过程中，首先需要建立与数据库的连接，当不再使用时，需要调用close()方法来释放与数据库的连接，只有连接被关闭后，垃圾回收器
才会回收对应的对象。否则，如果在访问数据库的过程中，对connection、statement或resultSet不显性地关闭，将会造成大量的对象无法被回收，从而引起内存泄露  

⑤ 变量不合理的作用域  
一个变量定义的作用范围大于其使用范围，或者没有及时把对象设置为null，都很有可能造成内存泄露  
```
public class UsingRandom{
    // 可能会造成内存泄露，msg的生命周期和对象相同，所以方法结束，msg不能回收，将其定义到方法里
    // 或者使用完msg后，将msg设置为null也行
    private String msg;
    
    public void receiveMsg(){
        // private String msg;
        readFrom();// 获取数据保存到msg中
        saveDB();// 把msg保存到数据库中
        // msg = null;
    }
}
```

⑥ 改变哈希值  
当一个对象被存储进hashSet集合后，就不能修改对象中参与计算哈希值的字段了。否则，对象修改后的哈希值与最初存储进hashSet集合中时的哈希值就不同了  
在这种情况下，即使在contains方法使用该对象的当前引用作为参数去hashSet集合中检索对象，也将返回找不到对象的结果，这将导致无法从hashSet集合中
单独删除当前对象，造成内存泄露  
这也是String被设置成不可变类型的原因，我们可以放心把String存入hashSet，或者把String当作hashMap的key  

⑦ 缓存泄露  
内存泄露的另一个常见来源是缓存。对于这个问题，可以使用WeakHashMap代表缓存，此种Map的特点是，当除了自身有对key的引用外，此key没有其他引用
那么此map会自动丢弃此值  

⑧ 监听器和回调  
如果客户端在你实现的API中注册回调，却没有显示的取消，那么就会聚集。需要确保回调立即被当作垃圾回收的最佳方法是只保存它的弱引用，例如将他们保存
成为WeakHashMap的键。  

# 常用的JVM参数选项
### 标准参数选项
直接在DOS窗口中运行java或者java -help可以看到所有的标准选项  
```
-d32            使用32位数据模型（如果可用）
-d64            使用64位数据模型（如果可用）
-server         64位机器上只支持server模式的JVM，适用于需要大内存的应用程序，默认使用并行垃圾收集器
-client         在32位windows系统上，默认使用Client类型的JVM。Client模式适用于对内存要求较小的桌面应用程序，默认使用Serial串行垃圾收集器
```

### -X参数选项
① JVM的JIT编译模式相关的选项
```
-Xint 只使用解释器，所有字节码都被解释执行，这个模式的速度是很慢的  
-Xcomp 只使用编译器，所有字节码第一次使用就被编译成本地代码，然后再执行
-Xmixed 混合模式（默认使用），刚开始的时候使用解释器慢慢解释执行，后来让JIT即时编译器根据程序运行的情况，有选择地将某些热点代码提前编译并缓存在本地，再次执行的时候效率就非常高了
```
②堆内存
```
-Xms3550m等价于-XX:InitialHeapSize，设置JVM初始堆内存大小
-Xmx3550m等价于-XX:MaxHeapSize，设置JVM最大堆内存大小
-Xss128k等价于-XX:ThreadStackSize=128k，设置每个线程堆栈大小为128k
-Xmn2g设置年轻代大小为2G，官方推荐配置为整个堆大小的3/8
```

### -XX参数选项
```Boolean类型格式```  
```-XX:+<option>```表示启用option属性  
```-XX:-<option>```表示禁用option属性  

```数值类型格式```  
```-XX:<option>=<number>```

```非数值类型格式```  
```-XX:<name>=<string>```

```
-XX:+PrintCommandLineFlags              可以让在程序运行前打印出用户手动设置或者JVM自动设置的XX选项
-XX:+PrintFlagsInitial                  表示打印出所有XX选项的默认值
-XX:+PrintFlagsFinal                    表示打印出XX选项在运行程序时生效的值
-XX:+PrintVMOptions                     打印出JVM的参数

-XX:+UseAdaptiveSizePolicy              自动选择各区大小比例，默认开启（推荐使用默认打开的-XX:+UseAdaptiveSizePolicy设置，并且不显示设置-XX:SurvivorRatio）

-XX:NewSize=1024m                       表示设置新生代初始大小为1024m
-XX:MaxNewSize=1024m                    表示设置年轻代最大值为1024m
-XX:MaxGCPauseMillis=500                表示设置GC停顿时间，为500ms
-XX:GCTimeRatio=19                      表示设置吞吐量

-XX:SurvivorRatio=8                     设置年轻代中Eden区与一个Survivor区的比值，默认为8  
-XX:NewRatio=2                          设置老年代与新生代（包含1个Eden区和2个Survivor区）的比值，默认为2  
-XX:PretenureSizeThreadshold=1024       设置让大于此阈值的对象直接分配在老年代，单位为字节（此设置只对Serial、ParNew收集器有效）
-XX:MaxTenuringThreshold=15             新生代每次MinorGC后，还存活的对象年龄+1，当对象的年龄大于设置的这个值时就进入老年代（默认值为15，一般较少改动）
-XX:+PrintTenuringDistribution          让JVM在每次MinorGC后打印出当前使用的Survivor中对象的年龄分布  

-XX:TargetSurvivorRatio                 表示MinorGC结束后Survivor区域中占用空间的期望比例
  
-XX:HeapDumpPath=/usr/local/heapdump.hprof 用来指定heap转存文件的存储路径  
-XX:+PrintFlagsFinal                    输出所有参数的名称和默认值，默认不包括Diagnostic和Experimental的参数

// 永久代相关
-XX:PermSize=256m                       设置永久代初始值为256m
-XX:MaxPermSize=256m                    设置永久代最大值为256m

// 元空间相关
-XX:MetaspaceSize   初始空间大小
-XX:MaxMetaspaceSize    最大空间，默认没有限制
-XX:+UseCompressedOops  压缩对象指针
-XX:+UseCompressedClassPointers 压缩类指针
-XX:CompressedClassSpaceSize    设置Class Metaspace的大小，默认1G

// 直接内存相关
-XX:MaxDirectMemorySize     指定DirectMemory容量，若未指定，则默认与Java堆最大值一样

// OOM相关的选项
-XX:+HeapDumpOnOutOfMemoryError     表示在内存出现OOM的时候，把Heap转存（Dump）到文件以便后续分析
-XX:+HeapDumpBeforeFullGC           表示在出现Full GC之前，生成Heap转储文件
-XX:HeapDumpPath=<path>             指定heap转存文件的存储路径
-XX:OnOutOfMemoryError              指定一个可行性程序或者脚本的路径，当发生OOM的时候，去执行这个脚本

// 对OnOutOfMemoryError的运维处理
①在run.sh启动脚本中添加JVM参数
-XX:OnOutOfMemoryError=/opt/Server/restart.sh

②restart.sh脚本
#!/bin/bash
pid=$(ps -ef|grep Server.jar|awk '{if($8=="java") {print $2}}')
kill -9 $pid
cd /opt/Server/
sh run.sh

```

> ```jinfo```支持动态修改Java虚拟机部分参数。只有被标记为manageable的可以被实时修改  
> 使用```jinfo -flag <name> = <value> <pid>```设置非Boolean类型参数  
> 使用```jinfo -flag [+|-]<name> <pid>```设置Boolean类型参数  
> ```jinfo -sysprops PID```         可以查看由System.getProperties()取得的参数  
> ```jinfo -flags PID```            查看曾经赋过值的一些参数  
> ```jinfo -flag 具体参数 PID```      查看某个java进程的具体参数的值

# 垃圾收集器
### Serial垃圾收集器
Serial收集器作为HotSpot中Client模式下的默认新生代垃圾收集器，Serial Old是运行在Client模式下默认的老年代的垃圾回收器  
-XX:+UseSerialGC 指定年轻代和老年代都使用串行收集器，等价于新生代使用Serial GC，且老年代Serial Old GC，可以获得最高的单线程收集效率

### ParNew垃圾收集器
-XX:+UseParNewGC 手动指定使用ParNew收集器执行内存回收任务，它表示年轻代使用并行收集器，不影响老年代  
-XX:ParallelGCThreads=N 限制线程数量，默认开启和CPU数据相同的线程数  

### Parallel垃圾收集器
下面两个参数分别适用于新生代和老年代，在jdk8中默认是开启的，并且开启一个时另一个也会被开启  
-XX:+UseParallelGC 手动指定年轻代使用Parallel并行收集器执行内存回收任务  
-XX:+UseParallelOldGC 手动指定老年代都是使用并行回收收集器  

-XX:ParallelGCThreads 设置年轻代并行收集器的线程数，一般地，最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能  
在默认情况下，当CPU数量小于8个，ParallelGCThreads的值等于CPU数量。当CPU数量大于8个，ParallelGCThreads的值等于[3+[5*CPU_Count]/8]  

-XX:MaxGCPauseMillis 设置垃圾收集器最大停顿时间(即STW的时间)，单位是毫秒（该参数使用需谨慎）  
①为了尽可能地把停顿时间控制在MaxGCPauseMills以内，收集器在工作时会调整Java堆大小或者其他一些参数  
②对于用户来讲，停顿时间越短体验越好，但是在服务器端，我们注重高并发，整体地吞吐量，所以服务器端适合Parallel进行控制  

-XX:GCTimeRatio 垃圾收集时间占总时间的比例（=1/(N + 1)），用于衡量吞吐量的大小  
①取值范围（0,100），默认值99，也就是垃圾回收时间不超过1%  
②与前一个-XX:MaxGCPauseMillis参与有一定矛盾性，暂停时间越长，Radio参数就容易超过设定的比例  

-XX:+UseAdaptiveSizePolicy 设置Parallel Scavenge 收集器具有自适应调节策略  
①在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点  
②在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量(GCTimeRatio)和停顿时间(MaxGCPauseMills)，让虚拟机自己完成调优工作  
③默认情况下，UseAdaptiveSizePolicy是开启状态的，导致新生代中Eden区可能占比1/6。禁用该值，并且显示设置-XX:SurvivorRatio=8才生效，即Eden区占1/8

### CMS垃圾收集器
```-XX:+UseConcMarkSweepGC``` 手动指定使用CMS收集器执行内存回收任务  
开启该参数后会自动将-XX:+UseParNewGC打开，即ParNew(Young区用)+CMS(Old区用)+Serial Old的组合  

```-XX:CMSlnitiatingOccupanyFraction``` 设置堆内存使用率的阈值，一旦达到该阈值，便于开始进行回收  
①JDK5及以前版本的默认值为68，即当老年代的空间使用率达到68%时，会执行一次CMS回收。JDK6及以上版本默认值为92%  
②如果内存增长缓慢，则可以设置一个稍大的值，大的阈值可以有效降低CMS的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个
阈值，以避免频繁触发老年代串行收集器，因此通过该选项便可以有效降低Full GC的执行次数  

-XX:+UseCMSCompactAtFullCollection 用于指定在执行完FUll GC后对内存空间进行压缩整理，以此避免内存碎片的产生，不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了  

-XX:CMSFullGCsBeforeCompaction 设置在执行多少次Full GC后对内存空间进行压缩整理  

-XX:ParallelCMSThreads 设置CMS的线程数量  
①CMS默认启动的线程数是(ParallelGCThreads+3)/4，ParallelGCThreads是年轻代并行收集器的线程数，当CPU资源比较紧张时，受到CMS收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟糕  

##### CMS其他参数
```-XX:ConcGCThreads``` 设置并发垃圾收集的线程数，默认该值是基于ParallelGCThreads计算出来的  

-XX:+UseCMSInitiatingOccupancyOnly 是否动态可调，永这个参数可以使CMS一直按CMSInitiatingOccupancyFraction设定的值启动  

-XX:+CMSScavengeBeforeMark 强制hotspot虚拟机在CMS remark阶段之前做一次minor gc，用于提高remark阶段的速度  

-XX:+CMSClassUnloadingEnable 如果有的话，启动回收Perm 区（JDK8之前）  

-XX:+CMSParallelInitialEnabled 用于开启CMS initial-mark阶段采用多线程的方式进行标记，用于提高标记速度，在java8开始已经默认开启  

-XX:+CMSParallelRemarkEnabled 用户开启CMS remark阶段采用多线程的方式进行重新标记，默认开启  

-XX:+ExplicitGCInvokesConcurrent、-XX:+ExplicitInvokesConcurrentAndUnloadsClasses这两个参数用户指定hotspot虚拟在执行System.gc()时使用CMS周期  

-XX:+CMSPreCleaningEnabled 指定CMS是否需要进行Pre cleaning这个阶段

### G1垃圾收集器
使用G1垃圾收集器，就不要再设置-Xmn、newRatio的值  

-XX:+UseG1GC 手动指定使用G1垃圾收集器执行内存回收任务  
-XX:G1HeapRegionSize 设置每个Region的大小，值是2的幂，范围是1MB到32MB之间，目标是根据最小的Java堆大小划分出约2048个区，默认是堆内存的1/2000  
-XX:MaxGCPauseMillis 设置期望达到的最大GC停顿时间指标（JVM会尽力实现，但不保证达到），默认值是200ms  
-XX:ParallelGCThread 设置STW时GC线程数的值，最多设置为8  
```-XX:ConcGCThreads``` 设置并发标记的线程数，将n设置为并行垃圾回收线程数(ParallelGCThreads)的1/4左右  
-XX:InitiatingHeapOccupancyPercent 设置触发并发GC周期的Java堆占用率阈值，超过此值，就触发GC，默认值是45  
-XX:MaxGCPauseMills 设置期望达到的最大GC停顿时间指标(JVM会尽力实现，但不保证达到)，默认值是200ms  
-XX:ParallelGCThread 设置STW时GC线程数的值，最多设置为8  
```-XX:ConcGCThreads``` 设置并发标记的线程数，将n设置为并行垃圾回收线程数(ParallelGCThreads)的1/4左右  
-XX:InitiatingHeapOccupancyPercent 设置触发并发GC周期的Java堆占用率阈值，超过此值，就触发GC，默认值是45  
-XX:G1NewSizePercent、-XX:G1MaxNewSizePercent 新生代占用整个堆内存的最小百分比(默认5%)，最大百分比(默认60%)  
-XX:G1ReservePercent = 10 保留内存区域，防止to space （Survivor中的to区）溢出  

##### Mixed GC调优参数
注意：G1收集器主要涉及到Mixed GC，Mixed GC会回收young 区和部分old 区  
G1关于Mixed GC调优常用参数  
-XX:InitiatingHeapOccupancyPercent 设置堆占用率的百分比（0到100）达到这个数值的时候触发global concurrent marking（全局并发标记），默认为45%，值为0表示间断进行全局并发标记  
-XX:G1MixedGCLiveThresholdPercent 设置Old区的region被回收时候的对象占比，默认占用率为85%，只有Old区的region中存活的对象占用达到了这个百分比，才会在Mixed GC中被回收  
-XX:G1HeapWastePercent 在global concurrent marking（全局并发标记）结束之后，可以知道所有的区有多少空间要被回收，在每次young gc之后和再次发生Mixed GC之前，会检查垃圾占比
是否达到此参数，只有达到了，下次才会发生Mixed GC  
-XX:G1MixedGCCountTarget 一次global concurrent marking（全局并发标记）之后，最多执行Mixed GC的次数，默认是8  
-XX:G1OldSetRegionThresholdPercent 设置Mixed GC收集周期中要收集的Old region数的上限，默认值是Java堆的10%

### 怎么选择垃圾回收器
①优先调整堆的大小让JVM自适应完成  
②如果内存小于100m，使用串行收集器  
③如果是单核、单机程序，并且没有停顿时间的要求，串行收集器  
④如果是多CPU、需要高吞吐量、允许停顿时间超过1s，选择并行或者JVM自己选择  
⑤如果是多CPU、追求低停顿时间，需快速响应（比如延迟不能超过1s，如互联网应用），使用并发收集器。官方推荐G1，性能高。现在互联网的项目，基本都是使用G1

# GC日志相关选项
### 常用参数
-verbose:gc 输出gc日志信息，默认输出到标准输出  
-XX:+PrintGC 等同于-verbose:gc，表示打开简化的GC日志  
-XX:+PrintGCDetails 在发生垃圾回收时打印内存回收详细的日志，并在进程退出时输出当前内存各区域分配情况  
-XX:+PrintGCTimeStamps 输出GC发生时的时间戳  
-XX:+PrintGCDateStamps 输出GC发生时的时间戳（以日期的形式，如2022-09-05T21:53:59.234+0800）  
-XX:+PrintHeapAtGC 每一次GC前和GC后，都打印堆信息  
```-Xloggc:<file>``` 把GC日志写入到一个文件中去，而不是打印到标准输出中

### 其他参数
-XX:+TraceClassLoading 监控类的加载  
-XX:+PrintGCApplicationStoppedTime 打印GC时线程的停顿时间  
-XX:+PrintGCApplicationConcurrentTime 垃圾收集之前打印出应用未中断的执行时间  
-XX:+PrintReferenceGC 记录回收了多少种不同引用类型的引用  
-XX:+PrintTenuringDistribution 让JVM在每次MinorGC后打印出当前使用的Survivor中对象的年龄分布  
-XX:+UseGCLogFileRotation 启用GC日志文件的自动转储  
-XX:NumberOfGCLogFiles=1 GC日志文件的循环数目  
-XX:GCLogFileSize=1M 控制GC日志文件的大小  

-XX:+DisableExplicitGC 禁止hotspot执行System.gc()，默认禁用  
-XX:ReservedCodeCacheSize=<n>[g|m|k]、-XX:InitialCodeCacheSize=<n>[g|m|K] 指定代码缓存的大小  
-XX:+UseCodeCacheFlushing 使用该参数让JVM放弃一些被编译的代码，避免代码缓存被占满时JVM切换到interpreted-only的情况  
-XX:+DoEscapeAnalysis 开启逃逸分析  
-XX:+UseBiasedLocking 开启偏向锁  
-XX:+UseLargePages 开启使用大页面  
```-XX:+UseTLAB``` 使用```TLAB```，默认打开  
```-XX:+PrintTLAB``` 打印```TLAB```的使用情况  
```-XX:TLABSize``` 设置```TLAB```大小  

# GC触发时机
### 触发Full GC
1 老年代空间不足  
2 方法区空间不足  
3 显示调用System.gc()  
4 Minor GC进入老年代的数据的平均大小 大于 老年代的可用内存  
5 大对象直接进入老年代，而老年代的可用空间不足






