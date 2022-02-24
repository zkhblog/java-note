1、产生堆外内存溢出OutOfDirectMemoryError  
1）SpringBoot2.0以后默认使用lettuce作为操作redis的客户端。它使用netty进行网络通信  
2）lettuce的BUG导致netty堆外内存溢出 -Xmx300m netty如果没有指定堆外内存，默认使用堆外-Xmx300m  
3）可以通过-Dio.netty.maxDirectMemory去调大堆外内存  
4）解决方案：  
升级lettuce客户端  切换使用jedis  