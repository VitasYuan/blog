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
![](https://github.com/VitasYuan/Blog/blob/master/pictures/concurrent-1-3.png)

### 此方法的缺点：  
使用此方法判断线程池是否全部完成，需要调用shutdown方法，即需要关闭线程池，当在定时任务中使用线程池执行任务的时候往往是不想每次都将线程池关闭的，因此在这种情况下这种方法不适合使用。

## 2.使用Future返回值来判断线程池中任务的完成情况
使用Future来判断线程是否执行完毕，不需要关闭线程池，main方法代码如下：  

    public static void main(String[] args){
            long startTime = System.currentTimeMillis();
            List<Future> futureList = new ArrayList<Future>();

            for(int i =0; i<4; i++){
                futureList.add(executor.submit(new ConsumerTask()));
            }

            while (!futureList.isEmpty()){
                for(int i = futureList.size() - 1; i >= 0; i--){
                    if(futureList.get(i).isDone()){
                        futureList.remove(i);
                    }
                }

                try {
                    Thread.sleep(10 * 1000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }

            System.out.println("all threads have completed!cost:" + (System.currentTimeMillis() - startTime));

        }
执行此方法后输出如下：
![](https://github.com/VitasYuan/Blog/blob/master/pictures/concurrent1-4.png)
说明：
ExecutorService中的submit接口说明如下：

     /**
      * Submits a Runnable task for execution and returns a Future
      * representing that task. The Future's {@code get} method will
      * return {@code null} upon <em>successful</em> completion.
      *
      * @param task the task to submit
      * @return a Future representing pending completion of the task
      * @throws RejectedExecutionException if the task cannot be
      *         scheduled for execution
      * @throws NullPointerException if the task is null
      */
     Future<?> submit(Runnable task);
此接口返回Future代表所执行提交的任务的线程，而Future中有如下几个接口：

    //取消线程
    boolean cancel(boolean mayInterruptIfRunning);
    //判断线程是否被取消
    boolean isCancelled();
    //判断线程是否执行完毕
    boolean isDone();
    //获取线程返回值
    V get() throws InterruptedException, ExecutionException;

其中我们用到的是isDone方法来判断线程是否执行完。
