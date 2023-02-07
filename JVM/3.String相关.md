# String的基本特性
① 在jdk8及以前，String内部定义了final char[] value用于存储字符串数据；从jdk9开始，String不用char[]来存储了，改成了byte[]加上**编码标记**，节约了一些空间  
> 在jdk8及以前，String用字符数组存储字符，数组中每个字符占用两个字节，一共16位，但是堆空间中大多数String对象都是```Latin-1字符```，像这些字符，一个字节都能存储，如果用char[]来存储，
> 会导致数组中每个字符中有一半的空间会被浪费。因此做的改变是将字符数组替换成byte[]，这样byte[]中每个字节就能存储大部分的字符（例如```Latin-1字符```），但是像汉字这样的字符，
> 一个字节存储不下，因此改成byte[]之后，需要加上一个字符编码的标识来区分，因为汉字就是需要两个字节来存储  
> 最后的结果就是像ISO-8859-1/Latin-1等字符，用一个字节就能存储，像UTF-16还是得用两个字节来存储  
> 其他StringBuffer和StringBuilder做了同样的适配

② String代表不可变的字符序列  
③ 通过字面量的方式定义String(例如String s = "abc")，"abc"存储在字符串常量池中，即字符串常量池是不会存储相同内容的字符串的  
> 字符串常量池”：字符串常量池保证不存储相同内容的机制是因为String Pool是一个固定大小的HashTable，如果放进String Pool的String很多，就会造成严重Hash冲突，
> 从而会导致链表很长，而链表长了后直接会造成的影响就是当调用String.intern时性能会大幅下降。  
> 使用-XX:StringTableSize可设置StringTable的长度  
> 在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多时就会导致效率下降很快。  
> 在jdk7中，StringTable的长度默认值是60013，jdk8开始，要设置StringTableSize的长度的话，1009是可设置的最小值

# String的内存分配
jdk6时，字符串常量池存放在永久代中  
jdk7时，字符串常量池的位置调整到Java堆中  
jdk8时，字符串常量池任然在堆中  

调整字符串常量池位置的原因：① PermSize默认比较小；②方法区垃圾回收频率较低

# 字符串拼接操作
① 常量与常量的拼接结果在常量池，原理是编译器优化
> "a" + "b" + "c"  与 "abc"  都在常量池中，且是同一个）

② 常量池中不会存在相同内容的常量  
③ 只要其中有一个是变量，结果就在堆中。变量拼接的原理是StringBuilder  
④ 如果拼接的结果是调用intern()方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址  
⑤ 对于第三点，如果变量是用final修饰的，那么和第一点相同，即编译期就会优化了
> final String str1 = "a";  
> final String str2 = "b";  
> str1 + str2 和 "ab" 的地址值是同一个

> 对于③的执行细节：  
> ① StringBuilder sb = new StringBuilder();  
> ② sb.append("a");  
> ③ sb.append("b");  
> ④ sb.toString();  // 等价于 new String("ab")  
> 
> 案例：  
> 代码一：  
> public void method1(int highLevel) {  
>   String src = "";  
>   for (int i = 0; i< highLevel; i++) {  
>       src = src + "a";// 每次循环都会创建一个StringBuilder、String对象  
>   }
> 
> 代码二：  
> public void method2(int highLevel) {  
>   StringBuilder sb = new StringBuilder();// 只需创建一个StringBuilder  
>   for (int i = 0; i < highLevel; i++) {
>       sb.append("a");  
>   }
> }
> 
> 补充：  
> ① 在jdk5及之后使用的是StringBuilder（线程不安全，效率更高），在jdk5之前使用的是StringBuffer  
> ② 通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式  
> 对于StringBuilder的append()的方式，自始至终中只创建过一个StringBuilder的对象  
> 对于使用String字符串拼接的方式，创建过多个StringBuilder和String的对象  
> ③ 使用字符串拼接的方式，内存中由于创建了较多的StringBuilder和String对象，内存占用更大，如果进行GC，需要花费额外的时间  
> 改进的空间：④ 在实际开发中，如果基本确定前后要添加的字符串的总长度不高于某个限定值capacity的情况下，可以使用指定容量的构造器：  
> StringBuilder sb = new StringBuilder(capacity); // 底层会创建确定长度的字符数组，new char[capacity]，这样就不用担心数组扩容影响执行效率

# intern()的使用
作用：判断字符串常量池中是否存在该字符串值，如果存在，则返回字符串常量池中该对象的地址值，如果字符串常量池中不存在该值，则在常量池中放入该字符串对象，并返回这个对象的地址值  
对于程序中大量存在的字符串，尤其其中很多重复字符串时，使用intern()可以节省内存空间  

① new String("abc")会创建几个对象？一个对象是通过new关键字在堆空间中创建的，另一个对象是在字符串常量池中的对象  
② new String("a") + new String("b")  
对象一：new StringBuilder用来拼接字符串  
对象二：new String("a")存放在堆中  
对象三：常量池中的"a"  
对象四：new String("b")存放在堆中  
对象五：常量池中的"b"  
对象六：在StringBuilder的toString()方法时，会new String("ab")，此时仅仅是在堆中创建字符串对象"ab"，并不会在字符串常量池中创建常量"ab"  

①和②的区别是①在字符串常量池中存在常量对象"ab"，而②并不会字符串常量池中存在常量对象"ab"  

### jdk6和jdk7的变化
jdk6时，s.intern()会创建一个新的常量对象，放在常量池中，这样导致在堆空间存在一个字符串对象，而且在字符串常量池中存在另外一个对象，两个对象的地址不一样  
jdk7及以上，s.intern()并不会在字符串常量池中创建新的常量，而是在常量池中创建一个地址引用，将堆空间中对象的地址引用值赋值给常量池中这个引用值，这样常量池中这个引用也是指向了堆空间中
对象，因此，最后的结果就是s.intern()之后，堆空间中对象s和另外一个字面量的地址相等，因为这个字面量指向了常量池中对象引用，对象引用又指向了堆空间中的对象地址

> jdk7/8中：  
> String s1 = new String("a") + new String("b");// 执行完这行代码之后，字符串常量池中，不存在常量"ab"，仅仅在堆空间中存在对象"ab"  
> s1.intern();// 在常量池中，存放一个地址引用，指向堆空间中的对象地址  
> String s2 = "ab";  
> System.out.println(s1 == s2);// true  
>
> String s1 = new String("a") + new String("b");  
> String s2 = "ab";// 在字符串常量池中生成了对象"ab"，和上面不同的是这是常量池中已经存在对象"ab"，而不是一个地址引用  
> s1.intern();// 在常量池中查找，发现已经存在常量"ab"，因此不再是在常量池中存放一个地址引用  
> System.out.println(s1 == s2);//false

### String.intern()的总结
① jdk6中，将这个字符串对象尝试放入串池。如果串池中有，则并不会放入，返回已有的串池中的对象的地址。如果没有，会把此对象复制一份，放入串池，并返回串池中的对象地址  
② jdk7起，将这个字符串对象尝试放入串池。如果串池中有，则并不会放入，返回已有的串池中的对象的地址。如果没有，则会把对象的引用地址复制一份，放入串池，并返回串池中的引用地址

### String.intern()的效率
应用中，如果存储大量字符串，都调用intern()方法时，就会明显降低内存的大小

# StringTable的垃圾回收

# G1中的String去重操作
堆空间中，存活数据集合里面，重复的String对象有很多，因此G1实现了自动持续对重复的String对象进行去重，这样就能避免浪费内存  
> UseStringDeDuplication(bool)：开启String去重，默认是不开启的，需要手动开启  
> PrintStringDeDuplicationStatistics(bool)：打印详细的去重统计信息  
> ```StringDeduplicationAgeThreshold(uintx)```：达到这个年龄的String对象被认为是去重的候选对象
