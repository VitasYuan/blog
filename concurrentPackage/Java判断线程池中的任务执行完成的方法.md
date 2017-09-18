## 1.使用shutdown()和isTerminated()方法
shutdown()方法会将线程池关闭，调用此方法后线程池将不在接收新的任务，在调用shutdown方法后当线程池中的所有任务都已经执行完毕之后
isTerminated()方法会返回true否则返回false。以下为demo代码：

    public class ThreadPoolTest {

        private static ExecutorService executor = Executors.newFixedThreadPool(4);

        public static ConcurrentLinkedQueue<Integer> localCache = new ConcurrentLinkedQueue<Integer>();

        static {
            for (int i = 10; i<20; i++){
                localCache.add(i);
            }
            System.out.println(localCache);
        }

    }
首先定义了ThreadPoolTest类，此类中包含了localCache变量，并初始化，此变量中的数据提供给线程任务使用。下面需要定义消费者线程，代码如下：

  public class ConsumerTask implements Runnable{

          public void run() {
              System.out.println("Thread:" + Thread.currentThread().getId() + " start!threadName:" + Thread.currentThread().getName());
              while (!localCache.isEmpty()){
                  Integer i = localCache.poll();
                  if(i == null){
                      continue;
                  }
                  try {
                      Thread.sleep(1000 * i);
                  }catch (InterruptedException e){
                      e.printStackTrace();
                  }
              }
              System.out.println("Thread:" + Thread.currentThread().getId() + " finished!threadName:" + Thread.currentThread().getName());
          }

  }
此任务中将localcache中读取数据，并sleep相应的秒，最后需要定义main方法来执行线程池，代码如下：

    public static void main(String[] args){
        long startTime = System.currentTimeMillis();

        for(int i =0; i<4; i++){
            executor.submit(new ConsumerTask());
        }
        executor.shutdown();
        while (executor.isTerminated() == false){
            try {

                Thread.sleep(1000 * 10);

            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
        System.out.println("all threads have completed!cost:" + (System.currentTimeMillis() - startTime));

    }
运行后控制台输入如下：
![]()
