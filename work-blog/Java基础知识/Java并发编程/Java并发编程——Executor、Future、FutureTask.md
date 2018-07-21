### Execotors、 Future、Callable和FutureTask的使用与源码分析
#### 1. Executors 工具类介绍
在上篇文章中，我们针对线程池 TreadPoolExecutor 类的基本用法进行了总结。在实际工作中，配置一个适合需求的线程池还是一件复杂的工作，所以在 JDK 中提供 Executors 类用于创建常见的线程池：
1. ExecutorService newFixedThreadPool(int nThreads)：创建一个固定大小为 nThreads 的线程池，多余的任务会放入队列中处理
2. ExecutorService newSingleThreadExecutor()：创建一个单线程的线程库
3. ExecutorService newCachedThreadPool()：创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程
4. ScheduledExecutorService newScheduledThreadPool(int corePoolSize)：建一个定长线程池，支持定时及周期性任务执行

**newFixedThreadPool 方法**
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
可以看到，源码中使用 ThreadPoolExecutor 类创建核心线程等于最大线程数的线程池，所以 newFixedThreadPool 的特点是，重复利用固定的线程数来执行任务，如果当前线程都在工作中，则将任务如队列。可能这里大家有疑问了，为什么没有拒绝策略了，这里的拒绝策略是默认的 AbortPolicy，队列的大小是默认的 Integer.SIZE,大约是20亿，所以一般情况下不会出现队列满的情况。

**newSingleThreadExecutor 方法**
```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
newSingleThreadExecutor 方法的核心是通过 ThreadPoolExecutor 创建一个固定大小为1的线程池，也就是说 getActiveCount、getPoolSize、getMaximumPoolSize 值都是1.这里如果当前活跃的线程由于异常死掉了，线程池会重新创建一个线程代替原来的线程，同一时刻，只能有一个线程。

**newCachedThreadPool 方法**
```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```
由于使用 SynchronousQueue 作为阻塞队列，所以它的特点是：当有空闲线程存活的时候，复用空闲线程，否则去创建新线程。

**newScheduledThreadPool 方法**
```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
    
public ScheduledThreadPoolExecutor(int corePoolSize) {
super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
      new DelayedWorkQueue());
}
```
ScheduledExecutorService 类多用于定时任务的场景。

**总结**
Executor 方法的本质就是封装 ThreadPoolExecutor 类，创造比较常用额线程池类。

#### 2. Future、FutureTask 和 Callable
在上篇关于 ThreadPoolExecutor 的文章中提到 executre 方法执行 Runnable 任务，同时也提及到 submit 方法。
```java
   public <T> Future<T> submit(Callable<T> task) 
```
那么Future、FutureTask 和 Callable 有什么用途呢？

##### 2.1 Future
Future 用处就是对于具体的 Runnable 或者 Callable 任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。
```java
public interface Future<V> {

    /**
     * 尝试取消任务
     * 任务已经执行完成、已经被取消或者任务不能取消的状态时会返回false
     * 如果返回ture，并且task没有开始，则应该取消不执行。
     * 如果任务已经开始了，就由mayInterruptIfRunning参数决定是否打断
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * 如果在任务执行完成之前调用返回ture
     */
    boolean isCancelled();

    /**
     * 任务完成返回true
     */
    boolean isDone();

    /**
     * 等待执行完成获取结果
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * 等待最多 timeout 时间来获取结果，指定时间内没完成获取到结果，返回null
     * @param timeout the maximum time to wait
     * @param unit the time unit of the timeout argument
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     * @throws TimeoutException if the wait timed out
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
根据源码来看，Future 的功能定义就是：
1. 判断任务是否完成；
2. 能够中断任务；
3. 能够获取任务执行结果。

##### 2.2 FutureTask
直接看源码定义：
```java
public class FutureTask<V> implements RunnableFuture<V>{
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
}

public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```
可以看出 FutureTask 实现 RunnableFuture 接口，而 RunnableFuture 接口是继承 Runnable 和 Future 接口。所以 FutureTask 兼具 Runnable 和 Future 的特性，通过它的构造函数可以看到，Future 可以封装 Callable 对象然后作为提交任务。

**这里有个关键点是 submit 方法的参数类型不同会引起返回值的 Future 不同。**

事实上，FutureTask是Future接口的一个唯一实现类。

##### 2.3 Callable
Callable 接口位于 java.util.concurrent 包下，在它里面也只声明了一个方法,Callable 接口的定义跟 Runnable类似：
```java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```
可以看到 Callable 是一个泛型接口，返回值类型就是定义的泛型类型，所以一般我们配合 FutureTask 和 ExecutorService 做任务提交以及获取。
```java
<T> Future<T> submit(Callable<T> task);
<T> Future<T> submit(Runnable task, T result);
Future<?> submit(Runnable task);
```
一般情况下我们使用第一个 submit 方法和第三个 submit 方法，第二个 submit 方法很少使用。大家可能会有疑惑，Runnable 对象没有返回值的，怎么能获取到返回值呢？这里其实是使用的 FutureTask，FutureTask 实现了 Runnable接口。

##### 2.4 submit 流程分析
先来看关联类的 UML 图：
![ThreadPool](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/ThreadPoolExecutor.png)

**submit 方法**
通过上面的 UML 图可知，submit 方法是在 ExecutorService 接口中定义，由 AbstractExecutorService 类进行实现。
```java
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
    return new FutureTask<T>(runnable, value);
}

protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}

/**
 * @throws RejectedExecutionException {@inheritDoc}
 * @throws NullPointerException       {@inheritDoc}
 */
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}

/**
 * @throws RejectedExecutionException {@inheritDoc}
 * @throws NullPointerException       {@inheritDoc}
 */
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}

/**
 * @throws RejectedExecutionException {@inheritDoc}
 * @throws NullPointerException       {@inheritDoc}
 */
public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```
可以看到在 submit 方法中，最终都是转换成 RunnableFuture 对象，而这个 RunnableFuture 对象本质是指向 FutureTask 类型。在最终执行的时候又都采用了 execute 方法进行执行。

前面我们提到 submit 方法中传入的任务类型会影响返回值，究竟是哪里的问题呢？
首先看传入 task 类型是 Callable 类型，调用 newTaskFor 方法生成 FutureTask 对象。
```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```
所以在后续的执行中就调用的是 callbale 对象的 call 方法执行，并将结果存储在运行的 FutureTask 中进行返回，正常获取。

如果传入的类型是 Runnable，同样调用 newTaskFor 方法生成 FutureTask 对象。
```java
 public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}

public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```
可以看到，经过一系列的转折，最终是转换成一个 RunnableAdapter，这个 RunnableAdapter 的值就是传入的 result，没传入就是 null。

针对 Runnable 类型参数概括一下，这段可能比较绕，所以多结合源码理解下过程：
1. submit 方法中传入 Runnable 类型，一般为了获取结果，会将 Callable 对象构建成 FutureTask 类型在传入，（此处记作FutureA）；
2. 调用 newTaskFor 方法生成 FutureTask 对象(记作FutureB)，这个对象就是我们 submit  方法返回的 Future 对象；
3. 在 FutureTask 的构造方法中调用 Executors.callable(runnable, result) 方法构建一个 Callable 对象存储在 FutureTask（即FutureB） 的成员变量 callable 中。其中 result 默认为 null，由于传入的是 Runnable 类型，所以在构建的时候是通过新建一个 Callable 的子类 RunnableAdapter 进行封装。
4. 当 task 任务经过入队成功开始执行的时候，就是执行的 callable 的 call 方法。由于当前的 Callable 对象是 RunnableAdapter 类型，所以最终是调用传入的 Runnable（FutureTask类型）的 run 方法，并且返回值是 result。
5. 经过这样的一波三折，最终回到构建原始的 FutureTask 的 Callable 中调用 call 方法，计算结果就被存储在传入作为参数的 FutureTask中，而返回值的 Future 结果就是 result。

所以在 FutureTask + Callable 结合使用时，如果通过 submit 返回值来获取计算结果就会出现为 null 的情况。

在 ThreadPoolExecutor 的介绍中，我们针对 execute 进行了大致的流程介绍，并没有涉及到实际的执行流程，所以在这里我们针对 submit 方法的执行捋一遍流程。

还是先从** addWorker **看起：
```java
private boolean addWorker(Runnable firstTask, boolean core) {
    /**
      * 此处省略 n 行代码，它们的主要作用是判断线程池的状态是否是运行状态，以及线程数是否超标。
      * 如果线程池是运行状态，并且线程没超标，则往下执行创建线程。
      */  

    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 将 firstTask 封装成 Worker对象。
        w = new Worker(firstTask);
        // 获取封装的任务线程
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // 重新检查线程池的状态
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // 线程池是运行状态 或 是关闭状态但是没任务，则添加 work 到集合
                    workers.add(w);
                    // 获取添加后的集合大小，修改 poolSize 的值
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            // 成功添加，开启线程执行
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
addWorkder 的逻辑大致可以分为以下步骤：
1. 它们的主要作用是判断线程池的状态是否是运行状态，以及线程数是否超标。如果线程池是运行状态，并且线程没超标，则往下执行创建线程。
2. 创建 Worker 对象。
3. 添加 Worker 对象到集合。
4. 获取 Worker 线程并执行。

我们想要获得最终的执行转换，如何转到我们定义的接口，就需要扒下 Worker 的外衣来看看。
```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable
{
    private static final long serialVersionUID = 6138294804551838833L;

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask;
    /** Per-thread task counter */
    volatile long completedTasks;

    /**
     * Creates with given first task and thread from ThreadFactory.
     * @param firstTask the first task (null if none)
     */
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }

    /** Delegates main run loop to outer runWorker  */
    public void run() {
        runWorker(this);
    }

    // 省略 n 行代码
}
```
首先需要明确 Worker 类是 ThreadPoolExecutor 的内部类。Worker 类是集成 AbstractQueuedSynchronizer 的子类，AbstractQueuedSynchronizer 应该都熟悉了（前面我们在并发编程锁的系列介绍过），同时实现了 Runnable 接口。它的内部包含一个 Runnable 和 Thread 对象，而这个 Thread 对象是通过创建。
```java
this.thread = getThreadFactory().newThread(this);
```
将自身作为一个参数进行创建。getThreadFactory() 方法 ThreadPoolExecutor 提供的获取 ThreadFactory 方法，最终的实现是在Executor 的内部类 DefaultThreadFactory 中进行实现。
```java
static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        // 将我们的任务 Runnable 对象作为参数常见 Thread
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        // 判断是否是守护线程，如果是守护线程，设为为 false，让它不是守护线程                      
        if (t.isDaemon())
            t.setDaemon(false);
        // 设置线程的优先级为普通
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```
这样 Worker 以自身为参数创建一个线程，当线程启动的时候就会执行 worker 的run 方法。最终执行到 runWorker(this)。
```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    // 获取 Worker 封装的 task 任务
    Runnable task = w.firstTask;
    // 销毁 workder 对象的 Runnable 引用，并释放锁
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            // 在任务执行前，先获取锁，确保线程执行的时候线程池不被打断
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 在 task 执行前，调用 beforeExecute 方法抛出异常，task 不会执行。该方法是空。
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```
这里的关键方法 task.run();由于 task 是 FutureTask 类型，所以程序运行到 FutureTask 的 run 方法中。
```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                // 调用自定义的 callable 对象的 call 方法，获取计算结果
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                // 设置计算结果返回值
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}

protected void set(V v) {
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        finishCompletion();
    }
}
```
最终，所有的执行和结果存储都回归到 FutureTask 中。

![addWorker流程](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/%E5%85%B3%E8%81%94%E7%B1%BB.png)

至此，整个的流程逻辑分析完毕。


#### 3. 演示实例
下面通过一些简单的实例模拟下如何使用，主要有：
- Future + Callable
- FutureTask + Callable

##### 3.1 Future + Callable
```java
public class HelloWorld {
    
    Future<String> feature;
	public static void main(String[] args) {
		ExecutorService executorService = Executors.newSingleThreadExecutor();
		Callables callable = new Callables();
		Future<Integer> future = executorService.submit(callable);
		executorService.shutdown();
		try {
			System.out.println("主线程获取结果：" + future.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}
	
	static class Callables implements Callable<Integer>{

		@Override
		public Integer call() throws Exception {
			Thread.sleep(1000);
			int sum =0;
			for(int i=0;i<100;i++) {
				sum += i;
			}
			System.out.println("子线程计算结果：" + sum);
			return sum;
		}
	}
}
```
执行结果：
```java
子线程计算结果：4950
主线程获取结果：4950
```
##### 3.2 FutureTask + Callable
```java
public class HelloWorld {
    
    Future<String> feature;
	public static void main(String[] args) {
		ExecutorService executorService = Executors.newSingleThreadExecutor();
		Callables callable = new Callables();
		FutureTask<Integer> futureTask = new FutureTask<>(callable);
		Future<?> future = executorService.submit(futureTask);
		executorService.shutdown();
		try {
			System.out.println("主线程获取结果：" + future.get() + "==" + futureTask.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}
	
	static class Callables implements Callable<Integer>{

		@Override
		public Integer call() throws Exception {
			Thread.sleep(1000);
			int sum =0;
			for(int i=0;i<100;i++) {
				sum += i;
			}
			System.out.println("子线程计算结果：" + sum);
			return sum;
		}
	}
}
```
执行结果：
```xml
子线程计算结果：4950
主线程获取结果：null==4950
```
可以看到，我们通过 submit 返回的 Future 获取的结果是 null。

这篇文章之后，并发编程的文章也算告一段落，当然还有很多没有涉及到，后面有时间在继续吧！新工作中使用很多注解、AST的知识，所以后面的时间基本都去研究这类的知识了。