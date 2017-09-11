## 1.功能介绍
Spring框架提供了线程池和定时任务执行的抽象接口：TaskExecutor和TaskScheduler来支持异步执行任务和定时执行任务功能。同时使用框架自己定义的抽象接口来屏蔽掉底层JDK版本间以及Java EE中的线程池和定时任务处理的差异。  
另外Spring还支持集成JDK内部的定时器Timer和Quartz Scheduler框架。
## 2.线程池的抽象：TaskExecutor
TaskExecutor涉及到的相关类图如下：  
![](https://github.com/VitasYuan/Blog/blob/master/pictures/spring-1-1.png)
TaskExecutor接口源代码如下所示：  

    public interface TaskExecutor extends Executor {

    	/**
    	 * Execute the given {@code task}.
    	 * <p>The call might return immediately if the implementation uses
    	 * an asynchronous execution strategy, or might block in the case
    	 * of synchronous execution.
    	 * @param task the {@code Runnable} to execute (never {@code null})
    	 * @throws TaskRejectedException if the given task was not accepted
    	 */
    	@Override
    	void execute(Runnable task);

    }
此接口和Executor几乎完全一样，只定义了一个接收Runnable参数的方法，据Spring官方介绍此接口最初是为了在其他组建中使用线程时，将JKD抽离出来而设计的。在Spring的一些其他组件中比如ApplicationEventMulticaster，Quartz都是使用TaskExecutor来作为线程池的抽象的。  
## 3.Spring提供的TaskExecutor的实现类  

    org.springframework.core.task.SimpleAsyncTaskExecutor
此实现支持任务的异步执行，但是此实现没有线程的复用，每次执行一个提交的任务时候都会新建一个线程，任务执行完成后会将线程关闭，最大并发数默认是没有限制的，但是可以通过调用下面的方法来设置最大并发数。一般使用线程池来代替此实现，特别是执行一些生命周期很短的任务的时候。


    public void setConcurrencyLimit(int concurrencyLimit) {
    		this.concurrencyThrottle.setConcurrencyLimit(concurrencyLimit);
    	}
Spring还提供了同步任务执行的实现类：

      org.springframework.core.task.SyncTaskExecutor

此类中只有一个方法，代码如下：  

    @Override
    	public void execute(Runnable task) {
    		Assert.notNull(task, "Runnable must not be null");
    		task.run();
    	}
此方法中直接调用传入的Runable对象的run方法，因此在执行此方法的时候不会另外开启新的线程，只是普通的方法调用，同步执行提交的Runable对象。
Spring有两个线程池的实现类，分别为：SimpleThreadPoolTaskExecutor和ThreadPoolTaskExecutor，其中当我们有Quarts和非Quarts共享同一个线程池的需求的时候使用SimpleThreadPoolTaskExecutor，除了这种情况，我们一般是使用
ThreadPoolTaskExecutor，此实现可以通过属性注入来配置线程池的相关配置。 ThreadPoolTaskExecutor中属性注入的源码如下：此配置可以在运行期修改，代码中修改过程使用了同步控制。

    /**
	 * Set the ThreadPoolExecutor's core pool size.
	 * Default is 1.
	 * <p><b>This setting can be modified at runtime, for example through JMX.</b>
	 */
	public void setCorePoolSize(int corePoolSize) {
		synchronized (this.poolSizeMonitor) {
			this.corePoolSize = corePoolSize;
			if (this.threadPoolExecutor != null) {
				this.threadPoolExecutor.setCorePoolSize(corePoolSize);
			}
		}
	}
## 4.TaskExecutor使用Demo
首先定义一个任务如下所示：  

    public class DataSimulation implements Runnable{

      private HourAverageValueDao hourAverageValueDao;

      @Override
      public void run() {

          Random random = new Random();
          AverageValue averageValue = new AverageValue();
          averageValue.setAverageVaule(random.nextInt(100));
          averageValue.setCreatedTime(new Date());
          hourAverageValueDao.insert(averageValue);

      }
    }
此任务中产生一个随机数，并封装成一个类对象，并将此数据插入到数据库中。
然后需要定一个类，使用TaskExecutor，代码如下：

    public class DataFacotory {

        private TaskExecutor executor;

        public TaskExecutor getExecutor() {
            return executor;
        }

        public void setExecutor(TaskExecutor executor) {
            this.executor = executor;
        }

        public void dataFactory(){

            for (int i =0; i < 10; i++){
                executor.execute(new DataSimulation());
            }
        }
    }
此类中定义了TaskExecutor的属性，并定一个方法，此方法中提交10个任务到TaskExecutor，下面只需配置Spring文件，注入TaskExecutor就可以实现线程池的使用。配置文件如下所示：

    <bean id = "taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
          <property name="corePoolSize" value = "5"></property>
          <property name = "maxPoolSize" value="10"></property>
          <property name="queueCapacity" value="25"></property>
      </bean>
完成配置后即可使用此线程池。Spring提供的线程池可以通过配置文件配置线程池的配置，相比JDk自带的线程池是一个很大的优势。
## 5.为什么使用线程池
1.通过使用线程池来实现线程的复用，减少线程创建和销毁的开销   
2.将执行线程的任务交给线程池来操作，一定意义上实现了解耦  
3.使用线程池可以控制任务的最大并发数目，这个在防止内存溢出以及并发优化方面有很重要的作用。

## 6.定时任务抽象类：TaskScheduler

TaskScheduler接口源代码如下：  

    public interface TaskScheduler {
      //通过触发器来决定task是否执行
    ScheduledFuture schedule(Runnable task, Trigger trigger);
    //在starttime的时候执行一次
    ScheduledFuture schedule(Runnable task, Date startTime);
    从starttime开始每个period时间段执行一次task
    ScheduledFuture scheduleAtFixedRate(Runnable task, Date startTime, long period);
    每隔period执行一次
    ScheduledFuture scheduleAtFixedRate(Runnable task, long period);
    从startTime开始每隔delay长时间执行一次
    ScheduledFuture scheduleWithFixedDelay(Runnable task, Date startTime, long delay);
    每隔delay时间执行一次
    ScheduledFuture scheduleWithFixedDelay(Runnable task, long delay);

}

指定开始时间的接口，如果时间已经是过去的某一时间点，则此任务会马上执行一次。以上几种执行方式传入Trigger的方式是用的最多的，Trigger接口中只定义了一个方法：  

    Date nextExecutionTime(TriggerContext triggerContext);
其中参数类型TriggerContext的定义如下：

    public interface TriggerContext {

  	/**
  	 * Return the last <i>scheduled</i> execution time of the task,
  	 * or {@code null} if not scheduled before.
  	 */
  	Date lastScheduledExecutionTime();

  	/**
  	 * Return the last <i>actual</i> execution time of the task,
  	 * or {@code null} if not scheduled before.
  	 */
  	Date lastActualExecutionTime();

  	/**
  	 * Return the last completion time of the task,
  	 * or {@code null} if not scheduled before.
  	 */
  	Date lastCompletionTime();

    }

提供了获取上一次任务执行信息的接口。我们通过实现Trigger接口可以实现自定义触发器来执行执行task。当然Spring也提供了两个默认的实现类：PeriodicTrigger和CronTrigger。

## 7.TaskScheduler定时任务Demo
首先在Spring配置文件中启用注解配置如下：

    <task:annotation-driven scheduler="myScheduler"/> //指定scheduler属性是可选项，不添加也可以正常使用
    <task:scheduler id="myScheduler" pool-size="10"/>

然后创建service，并在service中使用@Scheduled注解创建定时任务，代码如下：

    @Component
    public class SchedulerPoolTest {

      @Scheduled(cron = "0 * * * * ?")
      public void task1(){
          System.out.println("test");
          Thread thread =  Thread.currentThread();
          System.out.println("ThreadName:" + thread.getName() + ",id:" + thread.getId() + ",group:" + thread.getThreadGroup());

      }

      @Scheduled(fixedDelay = 5000)
      public void task2(){
          System.out.println("test");
          Thread thread =  Thread.currentThread();
          System.out.println("ThreadName:" + thread.getName() + ",id:" + thread.getId() + ",group:" + thread.getThreadGroup());

      }

    }
只是添加以上内容可能还不能正常执行task，还需要注意以下两点：

1.必须将SchedulerPoolTest类包含在spring所扫描的包里面  
  配置如下：      

    <context:component-scan base-package="com.zjut.task" />

2.需要在web.xml中添加spring配置文件的监听器，代码如下：

    <context-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>classpath*:spring-task.xml</param-value>
     </context-param>

     <listener>
       <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
添加以上内容后，启动服务器，将会定时执行任务。

## 8.Cron表达式
Cron表达式由6个字符串组成，每个字符串分别代表：  

    {秒} {分} {时} {日} {月} {周}
其中每个字符串所允许的取值范围为：

    字段名                 允许的值                        允许的特殊字符  
    秒                    0-59                            , - * /  
    分                    0-59                            , - * /  
    小时                  0-23                            , - * /  
    日                    1-31                            , - * ? / L W C  
    月                    1-12 or JAN-DEC                 , - * /  
    周几                  1-7 or SUN-SAT                   , - * ? / L C #

常用的Cron表达式：  

    "0 10,44 14 ? 3 WED" 每年三月的星期三的下午2:10和2:44触发
    "0 0/5 14,18 * * ?" 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
    "15-30/5 * * * * ?" 每分钟的15秒到30秒之间开始触发，每隔5秒触发一次
    "0 15 10 ? * 5#3" 每个月第三周的星期四的10点15分0秒触发任务

注：问号是用于避免日和周的设定由冲突而用的，当其中一个设置了具体的值，另外一个必须使用？。另外推荐一个Cron表达式生成的链接：<a >http://www.cronmaker.com/</a>
## 9.@Async注解
Async注解提供了异步调用方法的功能，当调用由此注解的方法的时候方法调用者会马上返回而不会等待调用的方法执行完成，被调用的方法会从线程池中分配一个线程来执行此方法。
## 10.Spring定时任务中并发执行的问题
同一个任务，当上一个任务没有执行完成的时候，新的任务不会执行。
不同任务的情况下：TODO...
