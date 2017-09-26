## 1.ArrayList多线程安全问题分析
在分析ArrayList线程安全问题之前，我们线对此类的源码进行分析，找出肯能出现线程安全问题的地方，然后代码进行验证和分析。
### 1.1 数据结构
ArrayList内部是使用数组保存元素的，数据定义如下：  

    transient Object[] elementData; // non-private to simplify nested class access
在ArrayList中此数组即是共享资源，当多线程对此数据进行操作的时候如果不进行同步控制，即有可能会出现线程安全问题。
### 1.2 add方法可能出现的问题分析
首先我们看一下add的源码如下：

    public boolean add(E e) {
        ensureCapacityInternal(size + 1);
        elementData[size++] = e;
        return true;
    }
此方法中有两个操作，一个是数组容量检查，另外就是将元素放入数据中。我们先看第二个简单的开始分析，当多个线程执行顺序如下所示的时候，会出现最终数据元素个数小于期望值。
![](https://github.com/VitasYuan/Blog/blob/master/pictures/concurrent-1-6.png)
按照此顺序执行完之后，我们可以看到，elementData[n]的只被设置了两次，第二个线程设置的值将前一个覆盖，最后size=n+1。下面使用代码进行验证此问题。
### 1.2.1 代码验证
首先先看下以下代码，开启1000个线程，同时调用ArrayList的add方法，每个线程向ArrayList中添加100个数字，如果程序正常执行的情况下应该是输出：

    list size is :10000  

代码如下：

    private static List<Integer> list = new ArrayList<Integer>();

        private static ExecutorService executorService = Executors.newFixedThreadPool(1000);

        private static class IncreaseTask extends Thread{
            @Override
            public void run() {
                System.out.println("ThreadId:" + Thread.currentThread().getId() + " start!");
                for(int i =0; i < 100; i++){
                    list.add(i);
                }
                System.out.println("ThreadId:" + Thread.currentThread().getId() + " finished!");
            }
        }

        public static void main(String[] args){
            for(int i=0; i < 1000; i++){
                executorService.submit(new IncreaseTask());
            }
            executorService.shutdown();
            while (!executorService.isTerminated()){
                try {
                    Thread.sleep(1000*10);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
            System.out.println("All task finished!");
            System.out.println("list size is :" + list.size());
        }
当执行此main方法后，输出如下：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/concurrent1-5.png)
从以上执行结果来看，最后输出的结果会小于我们的期望值。即当多线程调用add方法的时候会出现元素覆盖的问题。
### 1.2.2 数组容量检测的并发问题
在add方法源码中，我们看到在每次添加元素之前都会有一次数组容量的检测，add中调用此方法的源码如下：

    ensureCapacityInternal(size + 1);

容量检测的相关源码如下：

    private void ensureCapacityInternal(int minCapacity) {
           if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
               minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
           }

           ensureExplicitCapacity(minCapacity);
       }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
容量检测的流程图如下所示：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/concurrent-1-7.png)
我们以两个线程执行add操作来分析扩充容量可能会出现的并发问题：
当我们新建一个ArrayList时候，此时内部数组容器的容量为默认容量10，当我们用两个线程同时添加第10个元素的时候，如果出现以下执行顺序，可能会抛出java.lang.ArrayIndexOutOfBoundsException异常。
![](https://github.com/VitasYuan/Blog/blob/master/pictures/concurrent-1-8.png)
第二个线程往数组中添加数据的时候由于数组容量为10，而此操作往index为10的位置设置元素值，因此会抛出数组越界异常。
