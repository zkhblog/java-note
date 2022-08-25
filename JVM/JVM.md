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
② 
```
-Xms<size>等价于-XX:InitialHeapSize，设置初始Java堆大小
-Xmx<size>等价于-XX:MaxHeapSize，设置最大Java堆大小
-Xss<size>等价于-XX:ThreadStackSize，设置Java线程堆栈大小
```

### -XX参数选项
##### Boolean类型格式
```-XX:+<option>```表示启用option属性  
```-XX:-<option>```表示禁用option属性  
```
①打印设置的XX选项及值
-XX:+PrintCommandLineFlags 可以让在程序运行前打印出用户手动设置或者JVM自动设置的XX选项
-XX:+PrintFlagsInitial 表示打印出所有XX选项的默认值
-XX:+PrintFlagsFinal 表示打印出XX选项在运行程序时生效的值
-XX:+PrintVMOptions 打印出JVM的参数

②
-XX:+UseAdaptiveSizePolicy 自动选择各区大小比例，默认开启（推荐使用默认打开的-XX:+UseAdaptiveSizePolicy设置，并且不显示设置-XX:SurvivorRatio）

```

##### 数值类型格式
```-XX:<option>=<number>```
```
-XX:NewSize=1024m 表示设置新生代初始大小为1024m
-XX:MaxGCPauseMillis=500 表示设置GC停顿时间，为500ms
-XX:GCTimeRatio=19 表示设置吞吐量

-XX:SurvivorRatio=8 设置年轻代中Eden区与一个Survivor区的比值，默认为8  
-XX:NewRatio=2 设置老年代与新生代（包含1个Eden区和2个Survivor区）的比值，默认为2  
-XX:PretenureSizeThreadshold=1024 设置让大于此阈值的对象直接分配在老年代，单位为字节（此设置只对Serial、ParNew收集器有效）
-XX:MaxTenuringThreshold=15 新生代每次MinorGC后，还存活的对象年龄+1，当对象的年龄大于设置的这个值时就进入老年代（默认值为15，一般较少改动）
-XX:+PrintTenuringDistribution 让JVM在每次MinorGC后打印出当前使用的Survivor中对象的年龄分布  

```
##### 非数值类型格式
```-XX:<name>=<string>```  
例如：  
```-XX:HeapDumpPath=/usr/local/heapdump.hprof```用来指定heap转存文件的存储路径  
```-XX:+PrintFlagsFinal``` 输出所有参数的名称和默认值，默认不包括Diagnostic和Experimental的参数，

> ```jinfo```支持动态修改Java虚拟机部分参数。只有被标记为manageable的可以被实时修改  
> 使用```jinfo -flag <name> = <value> <pid>```设置非Boolean类型参数  
> 使用```jinfo -flag [+|-]<name> <pid>```设置Boolean类型参数

![img_1.png](temp/img_1.png)
![img_2.png](temp/img_2.png)
![img_3.png](temp/img_3.png)
默认情况下，UseAdaptiveSizePolicy是开启状态的，导致新生代中Eden区可能占比1/6。禁用该值，并且显示设置-XX:SurvivorRatio=8才生效，即Eden区占1/8。
![img_4.png](temp/img_4.png)
![img_6.png](temp/img_6.png)
![img_7.png](temp/img_7.png)
![img_8.png](temp/img_8.png)

垃圾收集器相关选项
![img_9.png](temp/img_9.png)
![img_10.png](temp/img_10.png)
![img_11.png](temp/img_11.png)
![img_12.png](temp/img_12.png)
![img_13.png](temp/img_13.png)
![img_14.png](temp/img_14.png)
![img_15.png](temp/img_15.png)
![img_16.png](temp/img_16.png)
![img_17.png](temp/img_17.png)
![img_18.png](temp/img_18.png)
使用G1垃圾收集器，就不要再设置-Xmn、newRatio的值

![img_19.png](temp/img_19.png)
![img_20.png](temp/img_20.png)
![img_21.png](temp/img_21.png)
![img_22.png](temp/img_22.png)
![img_23.png](temp/img_23.png)
![img_24.png](temp/img_24.png)
![img_25.png](temp/img_25.png)

哪些情况下会触发Full GC？
①老年代空间不足  
②方法区空间不足  
③显示调用System.gc()  
④Minor GC进入老年代的数据的平均大小 大于 老年代的可用内存  
⑤大对象直接进入老年代，而老年代的可用空间不足  

![img_26.png](temp/img_26.png)
![img_27.png](temp/img_27.png)
![img_28.png](temp/img_28.png)
![img_29.png](temp/img_29.png)
![img_30.png](temp/img_30.png)
![img_31.png](temp/img_31.png)






