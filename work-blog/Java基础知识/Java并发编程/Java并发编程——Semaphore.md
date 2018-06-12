#### 1. 简介
前面我们针对CountDownLatch和CyclicBarrier进行了学习，CountDownLatch用于帮助我们实现“倒计时”的功能，当count=0的时候则触发await的线程进行执行。而CyclicBarrier则适用于到达某一状态后接着执行下一步操作的情况。本次介绍的Semaphore是一个计信号数量。

Semaphore通过acquire()方法申请信号量，如果当前无信号量可用，则线程处于阻塞状态，如果有可用信号量，线程正常进行。release方法用于释放一个“锁定占用”的信号量。

**Semaphore 通常用于限制访问某些资源（物理或逻辑的）的线程数目。**

下面看一个jdk中带的例子，该例子中展示了使用Semaphone控制数据的访问。
```java
class Pool {
    private static final int MAX_AVAILABLE = 100;
    private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);

    public Object getItem() throws InterruptedException {
     available.acquire();
     return getNextAvailableItem();
    }

    public void putItem(Object x) {
     if (markAsUnused(x))
       available.release();
    }

    // Not a particularly efficient data structure; just for demo

    protected Object[] items = ... whatever kinds of items being managed
    protected boolean[] used = new boolean[MAX_AVAILABLE];

    protected synchronized Object getNextAvailableItem() {
     for (int i = 0; i < MAX_AVAILABLE; ++i) {
       if (!used[i]) {
          used[i] = true;          
          return items[i];
       }
     }
     return null; // not reached
    }

    protected synchronized boolean markAsUnused(Object item) {
     for (int i = 0; i < MAX_AVAILABLE; ++i) {
       if (item == items[i]) {
          if (used[i]) {
            used[i] = false;
            return true;
          } else
            return false;
       }
     }
     return false;
    }
}
```

#### 2、Api分析
Semaphore提供了两个关键的方法acquire()和release()方法。分别用于申请信号量和释放信号量。

如下面示例代码所示：
```java
public class FirstDemo {

	static Semaphore semaphore = new Semaphore(2);
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		for(int i=0;i<4;i++){
			new Thread(){
				@Override
				public void run() {
					System.out.println(Thread.currentThread().getName() + " ready");
					try {
						semaphore.acquire();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(Thread.currentThread().getName() + " go");
					semaphore.release();
				}
			}.start();
		}
	}
}
```
执行结果：
```java
Thread-0 ready
Thread-0 go
Thread-2 ready
Thread-2 go
Thread-1 ready
Thread-3 ready
Thread-1 go
Thread-3 go
```
从执行结果来看，最多只能连续执行go两次，间接的印证最多同时只有两个信号量在工作。补充一个特殊情况，当信号量Semaphore = 1 时，它可以当作互斥锁使用。其中0、1就相当于它的状态，当=1时表示其他线程可以获取，当=0时，排他，即其他线程必须要等待。

#### 3. 源码解析
在源码中，Semaphore同样包含一个继承AQS的子类Sync。
```java
public class Semaphore implements java.io.Serializable {
    private static final long serialVersionUID = -3222578661600680210L;
    /** All mechanics via AbstractQueuedSynchronizer subclass */
    private final Sync sync;

    /**
     * Synchronization implementation for semaphore.  Uses AQS state
     * to represent permits. Subclassed into fair and nonfair
     * versions.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;

        Sync(int permits) {
            setState(permits);
        }

        final int getPermits() {
            return getState();
        }

        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }

        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }

        final void reducePermits(int reductions) {
            for (;;) {
                int current = getState();
                int next = current - reductions;
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                if (compareAndSetState(current, next))
                    return;
            }
        }

        final int drainPermits() {
            for (;;) {
                int current = getState();
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }
    ....
}
```
通过使用AQS类的state值来存储信号量，原理类似于CountDownLatch，都是通过AQS的state值进行计数。

创建一个Semaphore对象可以通过构造方法：
```java
/**指定信号量个数*/
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

/**
 * Creates a {@code Semaphore} with the given number of
 * permits and the given fairness setting.
 *
 * @param permits the initial number of permits available.
 *        This value may be negative, in which case releases
 *        must occur before any acquires will be granted.
 * @param fair {@code true} if this semaphore will guarantee
 *        first-in first-out granting of permits under contention,
 *        else {@code false}
 */
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```
通过上面的构造函数，可以看到默认情况下Semaphore使用的非公平同步状态，同样也可以通过构造函数进行指定状态。

**acquire()方法**
```java
public void acquire() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;

    NonfairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}
```
在acquire方法中，调用sync的acquireSharedInterruptibly(1)方法，如果线程被iterrupted则抛出 InterruptedException异常。通过tryAcquireShared(arg) 获取状态值，最关键的代码是：
```java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```
获取当前的state值，如果可用state值-1后小于0，即当前信号量被占满，则返回一个负数，然后执行doAcquireSharedInterruptibly()方法添加到等待队列。

#### 4. 总结
无论ReentrantLock或CyclicBarrier内部都是基于AQS进行实现，实现的原理有所差异而已。