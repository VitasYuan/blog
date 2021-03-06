## 1.多线程常见的两种问题以及解决方法
首先我所理解的所有并发问题都是由于多线程同时访问共享资源所造成的。下面我们以JVM中线程共享内存中的变量为例子来分析多线程的两个问题：  
#### 1.共享资源在线程之间的可见性
当多个线程共享一个变量的时候，如果没有使用volatile关键字定义此变量时，当一个线程对此变量更新操作的时候对其他线程是不可见的。以下为相关操作的图说明：  
![](https://github.com/VitasYuan/Blog/blob/master/pictures/jvm-1-2.png)

当jvm进行运算的时候，首先将obj.count读取到各自的缓存中，分别对此变量进行赋值，然后保存到缓存中，在刷新缓存保存到jvm内存之前，这两个线程对此共享变量的操作是不可见的，这就可能会得到错误的结果。  
#### 2.线程间的竞争造成的线程安全问题
假如我们多个线程访问共享变量obj.count,并对此变量进行+1的操作，当我们不对共享变量进行同步控制的时候可能会出现以下情况：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/jvm1-1-3.png)
两个线程同时读取了jvm内存中的变量obj.count，然后对此变量+1操作，然后将计算结果写入到jvm内存，最终得到的obj.count=2，然而我们期望得到的结果是3.要解决这个问题，我们可以使用synchronized关键字对累加操作加锁，保证此操作同一个时间只有一个线程执行。如下代码所示：

    private static final Object LOCK = new Object();

      private static int count = 1;


      public static void increase(String [] args){
         synchronized (LOCK){
             count = count + 1;
         }
      }

## 2.使用volatile关键字来解决线程间变量不可见的问题
