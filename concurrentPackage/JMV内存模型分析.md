## 1.java内存模型分析

java虚拟机运行时数据存储区域包括线程隔离和线程共享两类，整个PC的内存图如下所示：  

![](https://github.com/VitasYuan/Blog/blob/master/pictures/jvm-1-1.jpg)
下面对以上内存区域说明：  
### 1.1 register和cache
当代计算机一般有多个cpu，每个cpu有独立的寄存器用于运行时存储数据，同时每个cpu一般还会有1级或者多级高速存储的缓存，当cpu读取数据时，总是会先从缓存中读取，如果缓存中没有时才会读取主内存中的数据，先把数据加载到缓存中，然后再从缓存中读取数据；cpu写数据时，也总是会先写到缓存中，然后再某一个时间点刷新缓存，将缓存中的数据写入到主内存中。由于这种工作方式，当多线程并发执行时候，可能会出现线程间数据不可见，以及访问共享资源时会出现冲突的问题。详细介绍参考：

### 2.线程隔离内存区域-programmer counter  
每个线程都有一个程序计数器，当执行java方法的时候，次计数器的作用是用来保存次线程下一个需要执行的指令的地址。当执行navtive方法的时候，此计数器为undifined，下一条执行由本地程序计数器管理  
### 3.JVM stack  
每个线程都有一个虚拟机栈，此内存区域在线程创建的时候分配，此虚拟机栈用于调用方法，并执行方法的计算机指令，虚拟机栈可以动态分配，也可以指定初始化大小。
此块内存区域会抛出以下两种异常：  
1.StackOverflowError，当计算时候请求的内存大小大于允许的最大值。  
2.OutOfMemoryError，当动态分配内存的时候，已经没有可用内存非配给新建的线程时候会抛出此异常。  
### 4.堆
堆内存是所有线程共享的内存区域，此内存区域在虚拟机启动的时候就由虚拟机进行分配，堆内存的大小可以固定大小，也可以动态分配，实现方式由具体的jvm来决定。我们所有使用new关键字创建的对象实例都保存在此内存区域内，此内存区域也是垃圾回收的主要区域。此内存区域会抛出以下异常：  
OutOfMemoryError：当没有空间分配给新建的对象实例的时候会抛出此异常。   
### 5.方法区
方法区在逻辑上是数据堆的一部分，同样是所有线程共享的内存区域，在虚拟机启动的时候分配此内存，主要用于保存所有的类文件，包括类文件中的常量池、类方法、构造方法、类变量等数据。类加载的时候，会将这些信息保存到此内存区域。此内存区域由可能抛出以下异常：  
OutOfMemoryError：当没有可用内存空间分配给新的类信息时候，会抛出此异常。
### 6.运行时常量池
运行时常量池是用于保存类文件或者接口文件中的常量表中的数据，此数据包括编译时可知的字面值常量和运行时需要解析的方法引用和Object变量引用等。此内存区域由方法区分配，在虚拟机加载类的时候会为每个类分配一个对应的类属常量池。此内存区域会抛出以下异常：OutOfMemoryError，当没有可用内存分配给内存申请的时候会抛出此异常。具体内存分配信息参考：
