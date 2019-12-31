# ThreadPoolExecutor

## 案例：

借鉴网上一个案例，来对比说明，自己管理多线程，与exectors线程池管理的优缺点。

自己管理线程：

```java
	// 提前准备好十本书
		final BlockingQueue<Book> books = new LinkedBlockingDeque<Book>(10);
		for (int i = 0; i < 10; i++) {
			try {
				books.put(new Book());
			} catch (Exception e) {
				// ignore
			}
		}
		System.out.println("start work...");
		// 创建两个书籍抄写员线程
		Thread[] scribes = new Thread[2];
		for (int scribeIndex = 0; scribeIndex < 2; scribeIndex++) {
			scribes[scribeIndex] = new Thread(new Runnable() {
				@Override
				public void run() {
					for (; ; ) {
						if (Thread.currentThread().isInterrupted()) {
							System.out.println("time arrives, stop writing...");
						}
						try {
							Book currentBook = books.poll(5, TimeUnit.SECONDS);
							currentBook.copy();
						} catch (Exception e) {
							System.out.println("time arrives, stop writing...");
							return;
						}
					}
				}
			});
			scribes[scribeIndex].setDaemon(false); // 设置为非守护线程
			scribes[scribeIndex].start();
		}

		// 工作已经安排下去了，安心等待就好了
		try {
			Thread.sleep(10000l);
		} catch (Exception e) {
			// ignore
		}

		// 时间到了，提醒两个抄写员停止抄写
		for (int scribeIndex = 0; scribeIndex < 2; scribeIndex++) {
			scribes[scribeIndex].interrupt();
		}

		System.out.println("end work...");
```

ThreadPoolExecutor管理：

```java
		ExecutorService executorService = Executors.newFixedThreadPool(2);
		for (int i = 0; i < 10; i ++) {
			executorService.submit(new Runnable() {
				public void run() {
					new Book().copy();
				}
			});
		}

		// 工作已经安排下去了，安心等待就好了
		try {
			Thread.sleep(10000l);
		} catch (Exception e) {
			// ignore
		}

		executorService.shutdownNow();
		System.out.println("end work...");
```



## 线程池作用：

- 降低资源消耗：通过重用已经创建的线程来降低线程创建和销毁的消耗

- 提高响应速度：任务到达时不需要等待线程创建就可以立即执行。

- 提高线程的可管理性：线程池可以统一管理、分配、调优和监控

  

## ThreadPoolExecutor基本

### 生命周期

​	**RUNNING**：允许接收新任务并且处理队列中的任务

​	**SHUTDOWN**：不再接收新的任务，仅消化完队列中的任务

​	**STOP**：不仅不再接收新的任务，连队列中的任务都不再消化处理了，并且尝试中断正在执行任务的线程

​	**TIDYING**：所有任务执行完了，工作线程数`workCount`也被设为0，线程的状态也被设为**TIDYING**，并开始调用钩子函数terminated()

​	**TERMINATED**：钩子函数`terminated()`执行完毕后的状态，即：终止状态。

### 生命状态的转化：

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/53B7D56067CC491C833A157C0F033CF1/8334)

### 状态字：

ThreadPoolExecutor把线程池状态和线程池容量打包成一个int型变量;

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/7C68EB87E99D4F36954BA11190B5F3E9/8336)



```java
 	//线程个数掩码位数
 	private static final int COUNT_BITS = Integer.SIZE - 3;
	//线程最大个数
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
	//线程池状态
    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
```

### 线程池参数：

- corePoolSize：线程池维护线程的最少数量

- maximumPoolSize：线程池维护线程的最大数量

- keepAliveTime： 线程池维护线程所允许的空闲时间

- unit： 线程池维护线程所允许的空闲时间的单位

- threadFactory:线程工厂，主要用来创建线程，比如指定线程的名字

- workQueue： 线程池所使用的缓冲队列

- handler： 线程池对拒绝任务的处理策略


### 线程池阻塞队列：

ArrayBlockingQueue:

是一个有边界的阻塞队列，它的内部实现是一个数组。它的容量在初始化时就确定不变。
 LinkedBlockingQueue：

阻塞队列大小的配置是可选的，其内部实现是一个链表。
 PriorityBlockingQueue：

是一个没有边界的队列，所有插入到PriorityBlockingQueue的对象必须实现java.lang.Comparable接口，队列优先级的排序就是按照我们对这个接口的实现来定义的。
 SynchronousQueue：

队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。






## 线程池执行流程：

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/C8643135F16740678F8D6A328BD889AD/8347)



1. 调用ThreadPoolExecutor的execute提交线程，首先检查CorePool，如果CorePool内的线程小于CorePoolSize，新创建线程执行任务。

2. 如果当前CorePool内的线程大于等于CorePoolSize，那么将线程加入到BlockingQueue。

3. 如果不能加入BlockingQueue，在小于MaxPoolSize的情况下创建线程执行任务。

4. 如果线程数大于等于MaxPoolSize，那么执行拒绝策略。

   

   **源码如下：**

```java
 public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
     	//如果运行的线程小于corePoolSize
        if (workerCountOf(c) < corePoolSize) {
            //通过给定的命令来启动一个新线程；
            //addWorker的调用会自动地检查runState和workerCount，从而通过返回false来防止在不应该添加线程的情况下添加错误警报。
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
     	//当前核心线程池中全部线程都在运行workerCountOf(c) >= corePoolSize，所以此时将线程放到任务队列中
     	//线程池是否处于运行状态，且是否任务插入任务队列成功
        if (isRunning(c) && workQueue.offer(command)) {
            
            int recheck = ctl.get();
            ////线程池是否处于运行状态，如果不是则使刚刚的任务出队
            if (! isRunning(recheck) && remove(command))
                reject(command);
            //如果当前线程池为空，则添加一个线程
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
     //插入队列不成功，且当前线程数数量小于最大线程池数量，此时则创建新线程执行任务，创建失败执行拒绝策略*
        else if (!addWorker(command, false))
            reject(command);
    }
```



## 线程池创建

ThreadPoolExecutor构造函数：

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```

根据入参的不同，来构造不同类型的线程池；

### newFixedThreadPool

创建一个核心线程个数和最大线程个数都为nThreads的线程池，且阻塞队列长度为Integer.MAX_VALUE，keepAliveTime = 0 说明只要线程个数比核心线程个数多且闲时回收。

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/5B7BABDB49734B558F377D0DCEBD32ED/8340)

### newSingleThreadExecutor

创建一个核心线程个数和最大线程个数都为1的线程池，并且阻塞队列长度为Integer.MAX_VALUE。keepAliveTime()= 0 说明只要线程个数比核心线程数多且当前空闲就回收。

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/693E9F45F53A4DFD924069CF1DA7B8D2/8343)

### newCacheThreadpool:

创建一个按需创建线程的线程池，初始线程个数为0，最多为Integer.MAX-VALUE，并且为阻塞队列。keepAliveTime()表示60s内空闲就回收。这种类型特殊之处在于，加入同步队列的任务会被马上执行，同步队列中最多只有一个任务。

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/81766398550C4725B479D05B86C619C3/8345)

### newScheduledThreadPool：

该方法创建了一个固定长度的线程池，可以以延迟或者定时的方式执行任务；有个**ScheduledThreadPoolExecutor**子类继承了**ThreadPoolExecutor** ，后续单独解析



### 总结：

线程池不使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样 的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

>  说明： Executors 返回的线程池对象的弊端如下：

1. FixedThreadPool 和 SingleThreadPool : 允许的请求队列长度为 Integer.MAX_VALUE ，可能会堆积大量的请求，从而导致 OOM 。

2. CachedThreadPool 和 ScheduledThreadPool : 允许的创建线程数量为 Integer.MAX_VALUE ，可能会创建大量的线程，从而导致 OOM 。

   

## execute&submit区别：

### execute:

提交的方式只能提交一个Runnable的对象，且该方法的返回值是void，也即是提交后如果线程运行后，和主线程就脱离了关系了，当然可以设置一些变量来获取到线程的运行结果。并且当线程的执行过程中抛出了异常通常来说主线程也无法获取到异常的信息的，只有通过ThreadFactory主动设置线程的异常处理类才能感知到提交的线程中的异常信息。

### submit：

​		ThreadPoolExecutor继承了AbstractExecutorService，AbstractExecutorService实现了ExecutorService里面的submit方法。

**submit的三种传参方法：**

这种提交的方式会返回一个Future对象，这个Future对象代表这线程的执行结果，

当主线程调用Future的get方法的时候会获取到从线程中返回的结果数据。

如果在线程的执行过程中发生了异常，get会获取到异常的信息。


```java
   public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }
```

