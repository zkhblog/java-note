# 三大组件

Channel、Buffer、Selector

### ByteBuffer

①向buffer写入数据，例如调用channel.read(buffer)

②调用flip()切换至读模式

③从buffer读取数据，例如调用buffer.get()

④调用clear()或compact()切换至写模式

⑤重复1-4步骤

### ByteBuffer大小分配

每个channel都需要记录可能被切分的消息，因为ByteBuffer不能被多个channel共同使用，因此需要为每个channel维护一个独立的ByteBuffer。

ByteBuffer不能太大，比如一个ByteBuffer 1M的话，要支持百万连接就要1TB内存，因此需要设计大小可变的ByteBuffer。

①一种思路是首先分配一个较小的buffer，例如4K，如果发现容量不够，再分配8k的buffer，将4kbuffer内容拷贝至8k buffer，优点是消息连续容易处理，缺点是数据拷贝耗费性能。

②另一种思路是用多个数组组成buffer，一个数组不够，把多出来的内容写入新的数组，与前面的区别是消息存储不连续解析复杂，优点是避免了靠北引起性能损耗。



