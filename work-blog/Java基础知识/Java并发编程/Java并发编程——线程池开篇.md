>前面我们针对Lock锁、编发编程中的工具类进行了学习。通过这些知识可以完成基本的并发编程程序设计。后面就开始学习JUC中的Executor框架。

由于线程的生命周期中包括创建、就绪、运行、阻塞、销毁阶段，当我们待处理的任务数目较小时，我们可以自己创建几个线程来处理相应的任务，但当有大量的任务时，由于创建、销毁线程需要很大的开销，运用线程池这些问题就大大的缓解了。

在整个Executor框架中，分以下四部分进行介绍：

[一、Executor框架概述]()

[二、ThreadPoolExecutor线程池介绍]()

[三、Executors工具类的基础使用与源码分析]()

[四、Feature、Callable和FeatureTask的使用与源码分析]()

**Executor框架类图**
![Executor](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/Executor%E7%B1%BB%E5%9B%BE%E6%A1%86%E6%9E%B6.png)

#### 1. Executor概述
Executor接口作为Executor框架的核心，只定义了一个execute(Runnable)方法。
```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the <tt>Executor</tt> implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution.
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```
execute方法用于执行给定的Runnable，具体的执行策略依赖于具体的Executor实例。

**Executor类设计的思想是解耦task的提交和执行逻辑。**
```java
Executor executor = anExecutor;
executor.execute(new RunnableTask1());
executor.execute(new RunnableTask2());
```
注意Executor中执行task并不一定是在异步线程中执行。比如下面的例子就是在提交任务的线程中执行：
```java
class DirectExecutor implements Executor {
    public void execute(Runnable r) {
        r.run();
    }
}
```
通常情况下，在execute方法中执行任务的线程会跟提交任务的线程区分开，放在异步中进行执行。比如下面的小例子：
```java
class NewThreadTaskExecutor implements Executor{

	@Override
	public void execute(Runnable command) {
		new Thread(command).start();
	}
}
```
一般在实际使用中，Executor的实现会添加很多设计方案来控制task的执行逻辑。比如可以将task通过Queue或Stack进行管理来控制。
```java
class SerialTaskExecutor implements Executor{
	private final BlockingQueue<Runnable> taskQueue = new LinkedBlockingDeque<>();
	private final Executor executor = null;
	private Runnable active;
	
	public SerialTaskExecutor(Runnable active) {
		super();
		this.active = active;
	}

	@Override
	public void execute(Runnable command) {
		if(command == null){
			throw new NullPointerException();
		}
		/**将任务添加到队列中*/
		taskQueue.offer(command);
		scheduleNext();
	}

	private void scheduleNext() {
		while((active = taskQueue.poll()) != null){
			executor.execute(active);
		}
	}
}
```
Executor的实现通过实现execute()方法达到针对task执行逻辑的定制。

#### 2. ExecutorService
ExecutorService是一个继承于Executor的接口，它丰富了Executor提供的接口，提供了关闭的shutdown()方法以及创建返回Future的submit()方法。

**2.1 shutdown关闭**
如果ExecutorService已关闭，提交任务会抛出异常，ExecutorService提供了两个关闭方法：
- shutdown()：关闭ExecutorService，在关闭前会允许已提交的任务继续执行完成，完成后进行关闭。
- shutdownNow()：立即关闭ExecutorService，并尝试终止正在执行的task任务。

在使用ExecutorService的时候，如果ExecutorService已不在使用则必须进行关闭释放所占用的资源。

**2.2 创造构建返回Feature**
```java
/**
 * Submits a Runnable task for execution and returns a Future
 * representing that task. The Future's <tt>get</tt> method will
 * return <tt>null</tt> upon <em>successful</em> completion.
 *
 * @param task the task to submit
 * @return a Future representing pending completion of the task
 * @throws RejectedExecutionException if the task cannot be
 *         scheduled for execution
 * @throws NullPointerException if the task is null
 */
Future<?> submit(Runnable task);
```

#### 3. AbstractExecutorService
AbstractExecutorService是ExecutorService的具体实现类，实现了：
- submit方法
- invokeAny方法
- invokeAll方法

在AbstractExecutorService中通过newTaskFor()方法构建一个RunnableFuture对象，然后传递给execute方法进行执行。
```java
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}
```

#### 4. ThreadPoolExecutor和ScheduledThreadPoolExecutor
线程池类。线程池类主要是为了解决两类问题：
- 针对大量的异步请求，通过重用线程池中的线程，来减少每个线程创建和销毁的性能开销。
- 对线程进行一些维护和管理，比如定时开始，周期执行，并发数控制。

#### 5. Executors
Executors工具类给我们提供了方便的方法直接创建线程池。
- ExecutorService newFixedThreadPool(int nThreads)：创建一个指定大小的线程池
- ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory)：创建一个指定大小的线程池
- ExecutorService newSingleThreadExecutor()：创建一个单线程的线程池
- ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory)
- ExecutorService newCachedThreadPool()：创建一个单线程的线程池
- ExecutorService newCachedThreadPool(ThreadFactory threadFactory)：根据用户的任务数创建相应的线程来处理，该线程池不会对线程数目加以限制，完全依赖于JVM能创建线程的数量，可能引起内存不足。
- ScheduledExecutorService newSingleThreadScheduledExecutor()
- ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
- ScheduledExecutorService newScheduledThreadPool(int corePoolSize)

#### 6. ThreadFactory
ThreadFactory用于创建线程的工具类，主要是减少new Thread()创建线程的这种动作。
```java
class SimpleThreadFactory implements ThreadFactory{

	@Override
	public Thread newThread(Runnable r) {
		return new Thread(r);
	}
}
```

#### 7. Future
在Java并发编程中，通过Future来达标异步执行的结果返回，同时可以来检测异步的执行结果是否结束。执行结果只有在异步线程执行完毕后通过get方法进行获取，如果异步没有执行完毕，则会处于阻塞等待的状态。同时提供cancel方法进行任务的取消。
```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newSingleThreadExecutor();
    Future<String> future = executorService.submit(new Callable<String>() {

        @Override
        public String call() throws Exception {
            return "异步线程执行结果";
        }
    });
    System.out.print(Thread.currentThread().getName() + ":" + future.get().toString());
}
```

以上就是线程池Executor框架中涉及的关键接口和类，总体功能介绍就这么多。

