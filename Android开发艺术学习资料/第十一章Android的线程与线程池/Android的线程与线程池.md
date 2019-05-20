概述
	Android中有许多特有的扮演线程的角色：AsyncTask、HandlerThread、IntentService

线程池概述
	线程的创建和销毁都需要相应的开销。当系统中存在大量的线程时，系统会通过时间片轮转的方式调度每一个线程，因此线程不可能做到绝对的并行，除非线程数量小于等于CPU的核心数，一般来说这是不可能的。频繁的创建和销毁线程，并不是高效的做法。而是采用线程池

	一个线程池会缓存一定数量的线程，通过线程池就可以避免因为频繁创建和销毁线程所带来的系统开销。
	
	Android的线程池来源于JAVA，主要通过Executor来派生自特定类型的线程池，不同的线程池具有不同的特性

主线程和子线程
	主线程：进程所拥有的线程，JAVA默认情况一个进程只有一个线程，这个线程就是主线程。主线程主要处理界面交互相关的逻辑，由于用户随时会与界面发生交互，因此主线程在任何时候都必须有较高的响应速度，主线程不能执行耗时操作。子线程可以执行耗时操作，除了主线程之外的都是子线程
	
	Android沿用Java的线程模型，主线程又称为UI线程。在主线程执行耗时操作会出现ANR异常

# Android中的线程状态 #
    
AsyncTask 不同的API在多任务的并发执行上表现不太相同
	封装了线程池和Handler，为了方便开发者在子线程中更新UI。
	轻量级的异步任务类，在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI。
	不建议执行特别耗时的操作，特别耗时的操作，建议使用线程池
	
	是一个泛型类，提供Params，Progress，Result这三个泛型参数，代表的是各个参数的类型，如果不需要参数类型，可以使用void代替。
	四个核心方法：
		1、onPreExecute：主线程执行，异步任务开始之前调用，做一些准备工作
		2、doInbackground：线程池中运行，执行异步任务，形参为params数组，代表异步任务的输入参数。此方法中可以使用publishProgress会调用onProgressUpdate方法更新进度。此方法的返回结果传递给onPostExecute方法
		3、onProgressUpdate：主线程更新进度
		4、onPostExecute：异步任务执行之后，调用此方法，形参为执行的结果值，即doInBackground的返回值

	除了这四个方法，还有onCancelled方法，异步任务被取消时执行，异步任务被取消，执行onCancelled，onPostExecute则不会被调用
	
	源码分析：
		主要类  WorkerRunnable implements Callable  ， FutureTask implements RunnableFuture extends Runnable, Future   
			   FutureTask在构造器中初始化的时候会将WorkerRunnable的实例传入，持有WorkerRunnable的实例，WorkerRunnable持有所有的params  FutureTask是一个并发类，充当Runnable的角色

			   SerialExecutor 一个串行的线程池，一个进程的所有的AsyncTask全部在这个串行的线程池中排队执行。
			   FutureTask会交给SerialExecutor的execute方法处理，SerialExecutor会先把FutureTask对象插入到任务队列mTasks中，如果这个时候没有正在运行的AsyncTask任务，就会调用scheduleNext方法来执行下一个AsyncTask任务，。同时当一个AsyncTask任务执行完之后，AsyncTask会继续执行其它任务直到所有的任务都被执行为止。
		
		SerialExecutor 中的队列  final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
								     mTasks.offer(new Runnable() { //offer 在队尾插入元素
						                public void run() {
						                    try {
						                        r.run();
						                    } finally {
						                        scheduleNext();
						                    }
						                }
						            });
						            if (mActive == null) {
						                scheduleNext();
						            }
							  protected synchronized void scheduleNext() {
							   if ((mActive = mTasks.poll()) != null) {//poll() 检索删除此队列的头部，如果没有就返回null
							     
									THREAD_POOL_EXECUTOR.execute(mActive);

							            }
							        }

	AsyncTask 中有两个线程池(SerialExecutor和THREAD_POOL_EXECUTOR(ThreadPoolExecutor))和一个Handler(InternalHandler)，SerialExecutor用于任务的排队，THREAD_POOL_EXECUTOR用于真正的执行任务，InternalHandler用于将执行线程从线程池切换到主线程


HandlerThread
	底层直接使用Thread，带有Looper的线程
IntentService
	封装了HandlerThread和Handler，类似于后台线程，但是由于是一个服务，优先级较高，不容易被杀死

# Android中的线程池 #
	线程池的优点
		1、重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销
		2、有效控制线程池的最大并发数，避免大量的线程之间因相互抢占资源而导致阻塞现象
		3、能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能

	Android中的线程池的概念来自于Java中的Executor，Executor是一个接口，真正的线程池的实现为ThreadPoolExecutor。

ThreadPoolExecutor
	构造器介绍，构造器不同的参数会直接影响线程池的功能特性，常用构造方法介绍
	public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) 
	
	
	corePoolSize 线程池的核心线程数，默认情况下，核心线程会在线程池中一直存活，即使处于闲置状态。如果将ThreadPoolExecutor的				 allowCoreThreadTimeOut属性设置为true，则处于闲置的核心线程在等待新任务来到时会有超时策略，时间间隔由					 keepAliveTime所指定，等待时间超过keeoAliveTime所指定的时长后，核心线程就会被终止
	
	maximumPoolSize 线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务将会被阻塞

	keepAliveTime 非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收，当ThreadPoolExecutor的								  allowCoreThreadTimeOut属性设置为true时，也作用于核心线程

	unit  指定keeoAliveTime的时间单位，

	workQueue 线程池中的任务队列，通过线程池的executor方法提交的Runnable对象会存储在这个参数中

	threadFactory 线程工厂，为线程池提供创建新线程的功能。

	ThreadPoolExecutor执行规则
		1、如果线程池的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务
		2、如果线程池的线程数量已经达到或者超过核心线程的数量，那么任务会被插入到任务队列中等待执行
		3、如果已经无法将任务插入到任务队列中，大概是因为任务队列已经满了，这时候如果线程数量未达到线程池规定的最大值，会立刻启动一个非核心线程来执行任务
		4、如果当前线程数量已经达到线程池规定的最大值，那么就拒绝执行此任务，ThreadPoolExecutor会调用RehectedExecutionHandler的rejectedExecution方法来通知调用者

	 private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();//获取CPU核心数
	private static final BlockingQueue<Runnable> sPoolWorkQueue = new LinkedBlockingQueue<Runnable>(128);//任务队列容量为128

	if (threadPool != null && !threadPool.isShutdown()//关闭
					&& !threadPool.isTerminated()) {//Terminated 终止
				threadPool.remove(runnable);//取消线程任务
			}

## 线程池的分类 ##
	Android中四类具有不同特性的线程池，他们均是直接或者间接地配置ThreadPoolExecutor来实现自己的功能特性

FixedThreadPool   fixed固定的，不变的
	通过Executors的newFixedThreadPool方法来创建，是一种线程数量固定的线程池，线程处于空闲状态时，并不会被回收，除非线程池关闭，所有线程处于活动状态时，新任务就会处于等待状态，知道有线程空闲出来。只有核心线程并且没有超时机制，意味着能够更加快速地响应外界的请求。队列也是没有大小限制的

CacheThreadPool
	通过Executors的newCacheThreadPool方法来创建，是一种线程数量固定的线程池。只有非核心线程，最大线程数是Integer.MAX_VALUE.内部队列相当于一个空集合，会导致任何任务都会被立即执行，SynchronousQueue是一个无法存储元素的队列。这类线程池适合执行大量的耗时较少的任务，当整个线程池都处于闲置状态时，线程会因为超时而停止，这时候该线程池实际是没有任何线程的，几乎不占用任何系统的资源。

ScheduledThreadPool
	通过Executors的newScheluedThreadPool方法来创建，核心线程数量可设置，非核心线程数量没有限制，非核心线程闲置时会被立即回收，适合做定时和具有周期的重复任务

SingleThreadPool
	通过Executors的newSingleThreadExecutor方法来创建，只有一个核心线程，一个非核心线程，确保所有的任务都在同一个线程中书序执行。不需要处理线程同步的问题


	


