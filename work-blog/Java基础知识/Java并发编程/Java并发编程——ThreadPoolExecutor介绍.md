### Java 线程池的理解与使用
无论是 Java 开发或 Android 开发对线程池都不陌生。在 Android 开发中线程池常用作异步网络请求，通过 Executors 工具类提供的静态方法去创建线程池。

一个线程的生命周期可以简单概括为如下三个阶段：
- T1：线程创建时间
- T2：线程执行时间
- T3：线程销毁时间

针对 T1 + T3 > T2 的任务请求，如果大量这样的请求，就涉及到我们频繁创建线程、销毁线程，造成资源的浪费。

线程池的出现就是为解决并发请求数量多，但每个线程执行时间段的问题。并发请求时候，如果频繁创建、销毁线程会大大浪费系统资源，降低系统效率。

线程池的应用范围：

1. 需要大量的线程来完成任务，且完成任务的时间比较短。 WEB服务器完成网页请求这样的任务，使用线程池技术是非常合适的。因为单个任务小，而任务数量巨大，你可以想象一个热门网站的点击次数。 但对于长时间的任务，比如一个Telnet连接请求，线程池的优点就不明显了。因为Telnet会话时间比线程的创建时间大多了。

2. 对性能要求苛刻的应用，比如要求服务器迅速相应客户请求。

3. 接受突发性的大量请求，但不至于使服务器因此产生大量线程的应用。突发性大量客户请求，在没有线程池情况下，将产生大量线程，虽然理论上大部分操作系统线程数目最大值不是问题，短时间内产生大量线程可能使内存到达极限，并出现"OutOfMemory"的错误。

Java 并发相关的类存放在 java.util.concurrent 包下面，根据[开篇](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E2%80%94%E2%80%94%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%BC%80%E7%AF%87.md)时候针对线程池绘制的 UML 图，可以知道线程池的核心实现 ThreadPoolExecutor 类。

几个月前，阿里发布一套关于 Android 开发的规范，其中有一条建议大意是“使用线程池的时候避免使用 Executors 类创建，使用 ThreadPoolExecutor 进行创建”。

所以我们将从以下几个方面来探索 ThreadPoolExecutor 的源码：
1. 线程池状态
2. 线程池的创建
3. 任务执行
4. 存储与容量调整
5. 拒绝策略
6. 线程池的关闭
7. 配置线程池

#### 1. 线程池的状态
在 ThreadPoolExecutor 的源码中，通过一个原子变量来存储状态。
```java
/**原子变量，用来存储个数和状态*/
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
/**位数，Integer.SIZE = 32,所以该值=29*/
private static final int COUNT_BITS = Integer.SIZE - 3;
/**容量大小1<<29 -1*/
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

// 运行状态存储在高位
private static final int RUNNING    = -1 << COUNT_BITS;
private static final int SHUTDOWN   =  0 << COUNT_BITS;
private static final int STOP       =  1 << COUNT_BITS;
private static final int TIDYING    =  2 << COUNT_BITS;
private static final int TERMINATED =  3 << COUNT_BITS;

// 封装成trl以及获取运行状态和个数的方法
private static int runStateOf(int c)     { return c & ~CAPACITY; }  //计算state的状态
private static int workerCountOf(int c)  { return c & CAPACITY; }   //计算worker的个数
private static int ctlOf(int rs, int wc) { return rs | wc; }
```
ctl 用于控制鲜橙汁的状态和个数，由两个核心的“字段”组成：
- workerCount，有效的线程个数
- runState，标记线程的状态

在 ctl 的设计中，为了让这两个字段包装在一起，同时由于 Integer 的长度是4个字节32位，所以限制 workCount 的大小是 (2^29)-1 （大概5亿），而不是使用 (2^31)-1 （大约20亿），这样就形成了高3位是状态，底28位是个数。

线程池的生命周期由5个状态值组成：
- RUNNING：线程池可以接收任务，并且执行队列中的任务
- SHUTDOWN：线程池不接收新任务，但是会执行队列中存储的任务
- STOP：线程池不接收新任务，并且不会执行队列中的任务，同时打断正在执行的任务
- TIDYING：所有任务都执行结束会切换到这个状态，同时 workCount 为0。
- TERMINATED：terminated() 执行完成

常见的线程状态切换状态：
- RUNNING 状态：线程池创建后，完成初始化时进入 RUNNING 状态
- (RUNNING or SHUTDOWN) -> STOP：调用 shutdownNow() 方法进入 STOP 状态
- SHUTDOWN -> TIDYING：当队列和线程池是空的时候进入 TIDYING 状态
- STOP -> TIDYING：当 pool 是空的时候
- YING -> TERMINATED：当 terminated() 方法执行完成后进入 TERMINATED 状态。

![线程池状态切换](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/state.png)

#### 2. 线程池的创建
ThreadPoolExecutor 提供4个构造方法，最核心的是：
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```
参数说明：
- corePoolSize：核心线程数，数值不能小于0，可以为0。
- maximumPoolSize：最大线程数，必须大于0并且核心线程数不能大于最大线程数。
这里会涉及到线程池关于线程创建的策略：
    1. 如果线程池的线程数小于核心线程数，则新来的任务会创建新线程处理，尽管有空闲的核心线程。
    2. 如果线程池中的线程数大于核心线程且小于最大线程数，如果 workQueue 未满，则先将任务入队列，如果 workQueue 满了，则进行创建新线程。如果已经到达最大数，没有空闲线程能处理任务，则会执行拒绝策略，发出异常。
    3. 如果核心线程数等于最大线程数，如果线程数已到最大值且 workQueue 未满，则将请求入队列，等待有空闲线程进行执行。
    4. 如果线程池线程数大于最大线程数且队列满了，则会触发拒绝策略的执行。
- keepAliveTime：存活最大时长，当线程处于 idle 状态等待新任务的最长时间。
    - timeUnit：时长单位
    - TimeUnit.DAYS：天
    - TimeUnit.HOURS：小时
    - TimeUnit.MINUTES：分钟
    - TimeUnit.SECONDS：秒
    - TimeUnit.MILLISECONDS：毫秒，千分之一秒
    - TimeUnit.MICROSECONDS：微妙，百万分之一秒
    - TimeUnit.NANOSECONDS：纳秒，十亿分之一秒
- workQueue：提交任务的存储队列，它是一个接口，有以下实现类：
    1. SynchronousQueue：无缓存，一进一出模式
    2. ArrayBlockingQueue：基于数组的有界阻塞队列，必须指定大小，无法自动扩容，特点先进先出。
    3. LinkedBlockingQueue：一种基于链表形成的队列，可以不指定大小（默认 Integer 最大值），特点先进先出。
    4. PriorityBlockingQueue：无界阻塞队列。
- ThreadFactory：使用ThreadFactory创建各种线程池中的线程。
- rejectedExeceptionHandler：线程池的拒绝策略。
    1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
    1. ThreadPoolExecutor.DiscardPolicy：丢弃任务，不抛出异常。
    1. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
    1. ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

#### 3. 任务执行
线程池中通过 execute(Runnable command) 提交线程任务。由于 ThreadPoolExecutor 实现 ExecutorService 接口，所以还有 submit() 接口用于执行线程任务。

**execute 源码**
```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    else if (!addWorker(command, false))
        reject(command);
}
```
这里的流程可以简单分为3部分：
1. 如果线程池内的线程小于核心线程，则调用 addWorker 方法尝试添加新线程，addWorker 方法会自动检查 runState 和 workerCount ，当线程池不在运行状态或队列为空的时候都会返回false，或者当 core=true 的时候，判断线程数大于核心线程数也返回 false，或不是核心线程的时候，线程数大于最大值也返回 false。
2. 如果任务能够被入队，我们还是要进行 double-check 来确保线程池中是否有线程销毁或线程池是否被关闭。
所以重新检查状态，如有必要则进行队列回滚。
3. 如果无法入队列，队列满了，就会试图去创建一个新线程。如果失败说明休闲城池被关闭，拒绝执行此任务。

**addWorker**
```java
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 如果线程池是关闭状态，并且队列为空，则返回false
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            //如果线程个数大于容器最大值，则返回false
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // 重新判定状态
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    //结合上面推论，如果线程池是 Running 状态，并且线程个数小于核心线程数或最大值，则创建新线程
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
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
简单来说，在执行execute()方法时如果状态一直是RUNNING时，的执行过程如下：

1. 如果workerCount < corePoolSize，则创建并启动一个线程来执行新提交的任务；
2. 如果workerCount >= corePoolSize，且线程池内的阻塞队列未满，则将任务添加到该阻塞队列中；
3. 如果workerCount >= corePoolSize && workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务；
4. 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

![execute](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/execute.png)

#### 4. 存储与容量调整
在 ThreadPoolExecutor 中通过 BlockingQueue 进行线程任务的存储，常见使用的有三种：
1. ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；
2. LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；
3. synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。

ThreadPoolExecutor 提供 setCorePoolSize() 和 setMaximumPoolSize() 两个方法来进行容量的调整：
- setCorePoolSize：设置核心池大小
- setMaximumPoolSize：设置线程池最大能创建的线程数目大小

#### 5. 拒绝策略
当线程池阻塞队列满了，并且线程个数达到了最大值，如果还有任务提交，则会才去拒绝策略拒绝新任务，有以下四种策略：
1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。（默认策略）
1. ThreadPoolExecutor.DiscardPolicy：丢弃任务，不抛出异常。
1. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
1. ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

#### 6. 线程池的关闭
ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：
- shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

#### 7. 配置线程池
一般需要根据任务的类型来配置线程池大小：
- 如果是CPU密集型任务，就需要尽量压榨CPU，参考值可以设为 NCPU+1
- 如果是IO密集型任务，参考值可以设置为2*NCPU

　　当然，这只是一个参考值，具体的设置还需要根据实际情况进行调整，比如可以先将线程池大小设置为参考值，再观察任务运行情况和系统负载、资源利用率来进行适当调整。

关于线程池优秀的文章已经很多了，看源码的目的是为了加深线程池配置时策略的了解，把握住究竟是新建线程还是入队列。

参考文章：
[1、Java并发编程：线程池的使用](https://www.cnblogs.com/dolphin0520/p/3932921.html)

[2、深入理解Java线程池：ThreadPoolExecutor](https://www.jianshu.com/p/d2729853c4da)