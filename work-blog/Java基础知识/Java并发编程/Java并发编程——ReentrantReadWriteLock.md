[TOC]
### Java并发编程——ReentrantReadWriteLock
>在AQS的介绍中，锁分为独占锁和共享锁，在上节中我们介绍了独占锁ReentrantLock，本次将针对另一个独占锁ReentrantReadWriteLock进行学习。

#### 1. ReadWriteLock
ReadWriteLock属于独占锁，它内部包含一对相互关联的Lock锁，分别用于读写操作。**只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。**

read-write锁相比互斥锁提供了更大的并发量级别。虽然一次只有一个线程（writer 线程）可以修改共享数据，但在许多情况下，任何数量的线程可以同时读取共享数据（reader 线程），读-写锁利用了这一点。从理论上讲，与互斥锁相比，使用读-写锁所允许的并发性增强将带来更大的性能提高。在实践中，只有在多处理器上并且只在访问模式适用于共享数据时，才能完全实现并发性增

相比于互斥锁，使用read-write锁提供了更强的并发能力。但是提升性能取决于**读写操作期间读取数据相对于修改数据的频率，以及数据的争用——即在同一时间试图对该数据执行读取或写入操作的线程数。**例如，某个最初用数据填充并且之后不经常对其进行修改的 collection，因为经常对其进行搜索（比如搜索某种目录），所以这样的 collection 是使用读-写锁的理想候选者。但是，如果数据更新变得频繁，数据在大部分时间都被独占锁，这时，就算存在并发性增强，也是微不足道的。更进一步地说，如果读取操作所用时间太短，则读-写锁实现（它本身就比互斥锁复杂）的开销将成为主要的执行成本，在许多读-写锁实现仍然通过一小段代码将所有线程序列化时更是如此。最终，只有通过分析和测量，才能确定应用程序是否适合使用读-写锁。

所以根据上面推断，读写锁适用于读取较多、写比较少的场景。
```java
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading.
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing.
     */
    Lock writeLock();
}
```
ReadWriteLock提供了两个方法分别用于获取Read-Lock和Write-Lock。

#### 2. ReentrantReadWriteLock特点
ReentrantReadWriteLock与ReentrantLock实现非常类似。它具有以下属性。

**1、锁获取顺序**
ReentrantLock不会将读取者优先或写入者优先强加给锁访问的排序。但是，它确实支持可选的公平 策略。

**2、默认模式，非公平模式**
当非公平策略（默认）构造时，未指定进入读写锁的顺序，受到 reentrancy 约束的限制。连续竞争的非公平锁可能无限期地推迟一个或多个 reader 或 writer 线程，但吞吐量通常要高于公平锁。

**3、公平模式**
当使用公平策略是，线程使用一种近似同步到达的顺序争夺资源。当线程释放当前持有锁时，等待时间最长的write线程获取到写入锁，如果有一组等待时间大于所有正在等待的 writer 线程 的 reader 线程，将为该组分配写入锁。

当一个线程尝试获取公平策略的read-lock时，如果write-lock被持有或者有等待的write线程，则当前线程会阻塞。直到当前最旧的等待 writer 线程已获得并释放了写入锁之后，该线程才会获得读取锁。当然，如果等待 writer 放弃其等待，而保留一个或更多 reader 线程为队列中带有写入锁自由的时间最长的 waiter，则将为那些 reader 分配读取锁。

**4、锁降级**
重入还允许从写入锁降级为读取锁，其实现方式是：先获取写入锁，然后获取读取锁，最后释放写入锁。但是，从读取锁升级到写入锁是不可能的。
```java
 class CachedData {
   Object data;
   volatile boolean cacheValid;
   ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();

   void processCachedData() {
     rwl.readLock().lock();
     if (!cacheValid) {
        // Must release read lock before acquiring write lock
        rwl.readLock().unlock();
        rwl.writeLock().lock();
        // Recheck state because another thread might have acquired
        //   write lock and changed state before we did.
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // Downgrade by acquiring read lock before releasing write lock
        rwl.readLock().lock();
        rwl.writeLock().unlock(); // Unlock write, still hold read
     }

     use(data);
     rwl.readLock().unlock();
   }
 }
```

**5、锁获取的中断**
读取锁和写入锁都支持锁获取期间的中断。

**6、Condition 支持**
写入锁提供了一个 Condition 实现，对于写入锁来说，该实现的行为与 ReentrantLock.newCondition() 提供的 Condition 实现对 ReentrantLock 所做的行为相同。当然，此 Condition 只能用于写入锁。

读取锁不支持 Condition，readLock().newCondition() 会抛出 UnsupportedOperationException。

在使用某些种类的 Collection 时，可以使用 ReentrantReadWriteLock 来提高并发性。通常，在预期 collection 很大，读取者线程访问它的次数多于写入者线程，并且 entail 操作的开销高于同步开销时，这很值得一试。例如，以下是一个使用 TreeMap 的类，预期它很大，并且能被同时访问。
```java
class RWDictionary {
    private final Map<String, Data> m = new TreeMap<String, Data>();
    private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private final Lock r = rwl.readLock();
    private final Lock w = rwl.writeLock();

    public Data get(String key) {
        r.lock();
        try { return m.get(key); }
        finally { r.unlock(); }
    }
    public String[] allKeys() {
        r.lock();
        try { return m.keySet().toArray(); }
        finally { r.unlock(); }
    }
    public Data put(String key, Data value) {
        w.lock();
        try { return m.put(key, value); }
        finally { w.unlock(); }
    }
    public void clear() {
        w.lock();
        try { m.clear(); }
        finally { w.unlock(); }
    }
 }
```

**实现注意事项**
此锁最多支持 65535 个递归写入锁和 65535 个读取锁。试图超出这些限制将导致锁方法抛出 Error。

#### 3. ReentrantReadWriteLock源码分析
##### 3.1 基本框架结构组成
```java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {
    private static final long serialVersionUID = -6992448646407690164L;
    /** Inner class providing readlock */
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** Inner class providing writelock */
    private final ReentrantReadWriteLock.WriteLock writerLock;
    /** Performs all synchronization mechanics */
    final Sync sync;

    /**
     * Creates a new {@code ReentrantReadWriteLock} with
     * default (nonfair) ordering properties.
     */
    public ReentrantReadWriteLock() {
        this(false);
    }

    /**
     * Creates a new {@code ReentrantReadWriteLock} with
     * the given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
	.....
}
```
ReentrantReadWriteLock内部由一个ReadLock对象和一个WriteLock对象组成，分别是由内部类实现。同时包含实现同步功能的Sync对象。在构造方法总，支持无参构造函数和有参构造函数，默认是采用非公平策略。所以根据构造函数中**FairSync、NonfairSync、ReadLock和WriteLock这四个类的实现。**

因为使用的Visual Studio绘制的UML图，没发现内部类嵌套的绘制关系图标，所以使用组合替代了。
![ReentrantReadWriteLock](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/ReentrantReadWriteLock%E7%B1%BB%E5%9B%BE.png)

1.  ReentrantReadWriteLock实现了ReadWriteLock接口。ReadWriteLock是一个读写锁的接口，提供了"获取读锁的readLock()函数" 和 "获取写锁的writeLock()函数"。
2.  ReentrantReadWriteLock中包含：sync对象，读锁readerLock和写锁writerLock。读锁ReadLock和写锁WriteLock都实现了Lock接口。读锁ReadLock和写锁WriteLock中也都分别包含了"Sync对象"，它们的Sync对象和ReentrantReadWriteLock的Sync对象 是一样的，就是通过sync，读锁和写锁实现了对同一个对象的访问。
3.  和"ReentrantLock"一样，sync是Sync类型；而且，Sync也是一个继承于AQS的抽象类。Sync也包括"公平锁"FairSync和"非公平锁"NonfairSync。sync对象是"FairSync"和"NonfairSync"中的一个，默认是"NonfairSync"。

##### 3.2 Sync类的源码
通过上面的UML类图关系可以看出，最终ReadLock和WriteLock执行的本质方法都是在Sync类中。
```java
/**
 * ReentrantReadWriteLock内部的实现机制
 * @author Iflytek_dsw
 *
 */
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 6317671515068378041L;

    /**
     * 这些量用来计算Read和Write锁的数量，ReentrantReadWriterLock使用一个32位的int类型来表示锁被占用的线程数
     * （ReentrantLock中的state）采取的办法是：
     * 高16位用来表示读锁（共享锁）占有的线程数量，用低16位表示写锁（独占锁）被占用的数量
     */
    static final int SHARED_SHIFT   = 16;
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

    /**
     * 通过进行右移16位计算出共享锁（读取锁）的占用个数
     */
    static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
    /**
     * 通过进行&运算计算出低16位的写锁（独占锁）个数
     */
    static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

    /**
     * 定义一个容器来计算保存每个线程read锁的个数
     */
    static final class HoldCounter {
        int count = 0;
        // Use id, not reference, to avoid garbage retention
        final long tid = Thread.currentThread().getId();
    }

    /**
     * ThreadLocal的使用来针对每个线程进行存储HoldCounter 
     */
    static final class ThreadLocalHoldCounter
        extends ThreadLocal<HoldCounter> {
        public HoldCounter initialValue() {
            return new HoldCounter();
        }
    }

    /**
     * 一个read线程的HoldCounter，当为0的时候进行删除。
     */
    private transient ThreadLocalHoldCounter readHolds;

    /**
     *  cachedHoldCounter 缓存的是最后一个获取线程的HolderCount信息，
     *  该变量主要是在如果当前线程多次获取读锁时，减少从readHolds中获取HoldCounter的次数
     */
    private transient HoldCounter cachedHoldCounter;

    /**
     * firstReader is the first thread to have acquired the read lock.
     * firstReaderHoldCount is firstReader's hold count.
     */
    private transient Thread firstReader = null;
    private transient int firstReaderHoldCount;

    Sync() {
        readHolds = new ThreadLocalHoldCounter();
        setState(getState()); // ensures visibility of readHolds
    }

    /**
     * Returns true if the current thread, when trying to acquire
     * the read lock, and otherwise eligible to do so, should block
     * because of policy for overtaking other waiting threads.
     */
    abstract boolean readerShouldBlock();

    /**
     * Returns true if the current thread, when trying to acquire
     * the write lock, and otherwise eligible to do so, should block
     * because of policy for overtaking other waiting threads.
     */
    abstract boolean writerShouldBlock();

    protected final boolean tryRelease(int releases) {
    	/**当前线程不支持锁的时候，抛IllegalMonitor异常*/
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        int nextc = getState() - releases;
        /**当前锁线程的个数是否为0个*/
        boolean free = exclusiveCount(nextc) == 0;
        if (free)
            setExclusiveOwnerThread(null);
        setState(nextc);
        return free;
    }

    protected final boolean tryAcquire(int acquires) {
		//省略
    }

    protected final boolean tryReleaseShared(int unused) {
		//省略
    }

    private IllegalMonitorStateException unmatchedUnlockException() {
        return new IllegalMonitorStateException(
            "attempt to unlock read lock, not locked by current thread");
    }

    protected final int tryAcquireShared(int unused) {
		//省略
    }

    /**
     * Full version of acquire for reads, that handles CAS misses
     * and reentrant reads not dealt with in tryAcquireShared.
     */
    final int fullTryAcquireShared(Thread current) {
		//省略
    }

    /**
     * Performs tryLock for write, enabling barging in both modes.
     * This is identical in effect to tryAcquire except for lack
     * of calls to writerShouldBlock.
     */
    final boolean tryWriteLock() {
		//省略
    }

    /**
     * Performs tryLock for read, enabling barging in both modes.
     * This is identical in effect to tryAcquireShared except for
     * lack of calls to readerShouldBlock.
     */
    final boolean tryReadLock() 
		//省略
    }

    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    // Methods relayed to outer class

    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    final Thread getOwner() {
        // Must read state before owner to ensure memory consistency
        return ((exclusiveCount(getState()) == 0) ?
                null :
                getExclusiveOwnerThread());
    }

    final int getReadLockCount() {
        return sharedCount(getState());
    }

    final boolean isWriteLocked() {
        return exclusiveCount(getState()) != 0;
    }

    final int getWriteHoldCount() {
        return isHeldExclusively() ? exclusiveCount(getState()) : 0;
    }

    final int getReadHoldCount() {
		//省略
    }

    /**
     * Reconstitute this lock instance from a stream
     * @param s the stream
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        readHolds = new ThreadLocalHoldCounter();
        setState(0); // reset to unlocked state
    }

    final int getCount() { return getState(); }
}
```
上面的代码中省略多个关键方法的代码实现，代码比较多。后面在具体的流程中会单独挑出来分析。这里我们只要知道在Sync类中定义了如下关键方法：
- readerShouldBlock()
- writerShouldBlock()
- tryAcquire(int acquires)
- tryReleaseShared(int unused)
- tryAcquireShared(int unused)
- tryWriteLock()
- tryReadLock()

##### 3.3 FairSync公平锁
ReentrantReadWriteLock同样支持公平锁和非公平锁。FairSync的实现如下：
```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -2274990926593161451L;
    final boolean writerShouldBlock() {
        return hasQueuedPredecessors();
    }
    final boolean readerShouldBlock() {
        return hasQueuedPredecessors();
    }
}
```
在Sync类中预留了writerShouldBlock()和readerShouldBlock()两个抽象方法供子类重写。在FairSync中返回的是hasQueuedPredecessors()方法的返回值。
```java
/**
 * 当前线程前面是否有处理的线程，如果有返回true，反之返回false。
 * 队列为空同样返回false
 */
public final boolean hasQueuedPredecessors() {
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```
##### 3.4 NonfairSync非公平锁
```java
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -8159625535654395037L;
    final boolean writerShouldBlock() {
        return false; // writers can always barge
    }
    final boolean readerShouldBlock() {
        /* As a heuristic to avoid indefinite writer starvation,
         * block if the thread that momentarily appears to be head
         * of queue, if one exists, is a waiting writer.  This is
         * only a probabilistic effect since a new reader will not
         * block if there is a waiting writer behind other enabled
         * readers that have not yet drained from the queue.
         */
        return apparentlyFirstQueuedIsExclusive();
    }
}
```
在非公平锁中，write返回的是false。而在读锁中则返回apparentlyFirstQueuedIsExclusive()方法。
```java
final boolean apparentlyFirstQueuedIsExclusive() {
    Node h, s;
    return (h = head) != null &&
        (s = h.next)  != null &&
        !s.isShared()         &&
        s.thread != null;
}
```
该方法如果头节点不为空，并头节点的下一个节点不为空，并且不是共享模式【独占模式，写锁】、并且线程不为空，则返回true。

这个方法判断队列的head.next是否正在等待独占锁（写锁，因为在ReentrantReadWriteLock中读写锁共用一个队列）。当然这个方法执行的过程中队列的形态可能发生变化。这个方法的意思是：读锁不应该让写锁始终等待，因为在同一时刻读写锁只能有一种锁在“工作”。

##### 3.5 ReadLock源码解读
ReadLock是读取锁的获取。
```java
public static class ReadLock implements Lock, java.io.Serializable {
	private static final long serialVersionUID = -5992448646407690164L;
	private final Sync sync;

	/**
	 * Constructor for use by subclasses
	 *
	 * @param lock the outer lock object
	 * @throws NullPointerException if the lock is null
	 */
	protected ReadLock(ReentrantReadWriteLock lock) {
		sync = lock.sync;
	}

	/**
	 * Acquires the read lock.
	 * 申请read锁
	 * 如果write锁没有被别的线程持有，则立即返回read锁。如果write锁被占用，则当前线程会被阻塞，直至获取到read锁。
	 */
	public void lock() {
		sync.acquireShared(1);
	}


	public void lockInterruptibly() throws InterruptedException {
		sync.acquireSharedInterruptibly(1);
	}

	/**
	 * Acquires the read lock only if the write lock is not held by
	 * another thread at the time of invocation.
	 *
	 * <p>If the write lock is held by another thread then
	 * this method will return immediately with the value
	 * {@code false}.
	 *
	 * @return {@code true} if the read lock was acquired
	 */
	public boolean tryLock() {
		return sync.tryReadLock();
	}

	public boolean tryLock(long timeout, TimeUnit unit)
			throws InterruptedException {
		return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
	}

	/**
	 * Attempts to release this lock.
	 *
	 * <p> If the number of readers is now zero then the lock
	 * is made available for write lock attempts.
	 */
	public  void unlock() {
		sync.releaseShared(1);
	}

	/**
	 * Throws {@code UnsupportedOperationException} because
	 * {@code ReadLocks} do not support conditions.
	 *
	 * @throws UnsupportedOperationException always
	 */
	public Condition newCondition() {
		throw new UnsupportedOperationException();
	}

	/**
	 * Returns a string identifying this lock, as well as its lock state.
	 * The state, in brackets, includes the String {@code "Read locks ="}
	 * followed by the number of held read locks.
	 *
	 * @return a string identifying this lock, as well as its lock state
	 */
	public String toString() {
		int r = sync.getReadLockCount();
		return super.toString() +
			"[Read locks = " + r + "]";
	}
}
```
ReadLock继承Lock类，并实现了对应的申请所、释放锁方法。
###### 3.5.1 ReadLock申请锁lock
```java
public void lock() {
	sync.acquireShared(1);
}
```
read锁是共享锁，这里调用sync对象的acquireShared()方法来申请锁。
```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```
这里tryAcquireShared方法执行就到了Sync类中。
```java
protected final int tryAcquireShared(int unused) {
	/*
	 * Walkthrough:
	 * 1. 如果write锁被其它线程占用，申请fail
	 * 2. Otherwise, this thread is eligible for
	 *    lock wrt state, so ask if it should block
	 *    because of queue policy. If not, try
	 *    to grant by CASing state and updating count.
	 *    Note that step does not check for reentrant
	 *    acquires, which is postponed to full version
	 *    to avoid having to check hold count in
	 *    the more typical non-reentrant case.
	 * 3. If step 2 fails either because thread
	 *    apparently not eligible or CAS fails or count
	 *    saturated, chain to version with full retry loop.
	 */
	/**获取当前的线程*/
	Thread current = Thread.currentThread();
	/**获取当前锁的状态值*/
	int c = getState();
	/**如果写锁（独占锁）占有个数不为0，并且持有线程不等于当前线程，返回-1*/
	if (exclusiveCount(c) != 0 &&
		getExclusiveOwnerThread() != current)
		return -1;
	/**获取读取锁（共享锁）的个数*/
	int r = sharedCount(c);
	/** 如果当前read锁不被阻塞，并且个数小于最大MAX_COUNT,同时CAS操作成功*/
	if (!readerShouldBlock() &&
		r < MAX_COUNT &&
		compareAndSetState(c, c + SHARED_UNIT)) {
		if (r == 0) {//如果当前共享锁个数为0，没有被持有
			/**当前线程赋值给firstReader，并且firstReadHoldCount赋值为1*/
			firstReader = current;
			firstReaderHoldCount = 1;
		} else if (firstReader == current) {
			/**如果当前线程已经持有所，则firstReaderHoldCount自加1*/
			firstReaderHoldCount++;
		} else {/**其它情况则是新来的一个线程*/
			/**新建一个HoldCounter对象存储cachedHoldCounter*/
			HoldCounter rh = cachedHoldCounter;
			/**如果cachedHoldCounter为空，或tid不等于当前线程*/
			if (rh == null || rh.tid != current.getId())
				cachedHoldCounter = rh = readHolds.get();
			else if (rh.count == 0)
				readHolds.set(rh);
			rh.count++;
		}
		return 1;
	}
	/**如果锁被占用等不满足上述情况，则通过fullTryAcquireShared进行申请*/
	return fullTryAcquireShared(current);
}
```
在tryAcquireShared方法中， 如果当前read锁不被阻塞，并且个数小于最大MAX_COUNT,同时CAS操作成功则申请锁成功。反之通过fullTryAcquireShared方法进行申请。
```java
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    /**采用自旋的方式，直至申请到锁*/
    for (;;) {
        int c = getState();
        /**如果Write锁被占用，并且持有线程不是当前线程则返回-1*/
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) {//Write锁被持有
            // 如果最近没有申请过read锁
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {//没有申请过
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != current.getId()) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        /**Read锁已经到达最大值*/
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != current.getId())
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```
采用自旋的方式，其它逻辑同tryAcquireShared类似。
- 如果Write锁被占有，并且持有写锁的线程不是当前线程直接返回-1
- 如果Write锁当前线程持有，并且在CLH队列中下一个节点是Write锁，则返回-1

最后在所有申请失败返回-1的时候，则通过doAcquireShared()方法进行申请。
```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
通过上面的分析ReadLock的lock执行流程如下：
![lock](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/lock_read.png)


###### 3.5.2 tryLock()尝试申请锁
前面在ReentrantLock的笔记中，同样有tryLock方法。其实它们都差不多。
```java
public  boolean tryLock() {
    return sync.tryReadLock();
}
```
tryLock的内部调用的Sync类的tryReadLock()方法。
```java
final boolean tryReadLock() {
	Thread current = Thread.currentThread();
	for (;;) {
		int c = getState();
		/**Write锁被占用并且占用线程不是当前线程，返回false*/
		if (exclusiveCount(c) != 0 &&
			getExclusiveOwnerThread() != current)
			return false;
		/**返回read锁的个数*/
		int r = sharedCount(c);
		/**如果是最大值，则抛出异常*/
		if (r == MAX_COUNT)
			throw new Error("Maximum lock count exceeded");
		if (compareAndSetState(c, c + SHARED_UNIT)) {
			if (r == 0) {//Read的个数为0，则将当前线程赋值给firstReader，并且firstReaderHoldCount赋值为1
				firstReader = current;
				firstReaderHoldCount = 1;
			} else if (firstReader == current) {//如果firstReader==当前线程，则firstReaderHoldCount自加1
				firstReaderHoldCount++;
			} else {
				HoldCounter rh = cachedHoldCounter;
				if (rh == null || rh.tid != current.getId())
					cachedHoldCounter = rh = readHolds.get();
				else if (rh.count == 0)
					readHolds.set(rh);
				rh.count++;
			}
			return true;
		}
	}
}
```

###### 3.5.3 锁的释放
```java
public  void unlock() {
    sync.releaseShared(1);
}
```
unlock释放锁最终还是在Sync类的releaseShared方法中进行释放。releaseShared方法是在AQS类中。
```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
在AQS中，我们知道releaseShared方法是AQS预留给子类进行实现的，所以最终的实现还是在Sync类中。
```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != current.getId())
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

#### 4 总结
上面我们针对ReadLock的锁的申请、释放进行了分析，WriteLock由于是个独占锁，大致跟ReentrantLock流程差不多，不做过多分析。总体说来，还是需要多读几遍源码才能理解透彻。
最后给一个使用的示例：
```java
public class MyTest {

	static StudentList studentList = new StudentList();
	public static void main(String[]args){
		for(int i=0;i<3;i++){
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					studentList.addItem();
				}
			}).start();
		}
		
		for(int i=0;i<3;i++){
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					studentList.printList();
				}
			}).start();
		}
	}
}

class StudentList{
	private List<Integer> listNumber = new ArrayList<Integer>();
	
	private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
	private ReadLock readLock;
	private WriteLock writeLock;
	public StudentList(){
		readLock = reentrantReadWriteLock.readLock();
		writeLock = reentrantReadWriteLock.writeLock();
	}
	
	public void printList(){
		readLock.lock();
		for(int i=0;i<listNumber.size();i++){
			System.out.println(Thread.currentThread().getName() + "--printList--current:" + i);
		}
		readLock.unlock();
	}
	
	public void addItem(){
		writeLock.lock();
		for(int i=listNumber.size();i<5;i++){
			System.out.println(Thread.currentThread().getName() + "--addItem--current:" + i);
			listNumber.add(i);
		}
		writeLock.unlock();
	}
}
```
执行结果：
```java
Thread-0--addItem--current:0
Thread-0--addItem--current:1
Thread-0--addItem--current:2
Thread-0--addItem--current:3
Thread-0--addItem--current:4
Thread-3--printList--current:0
Thread-3--printList--current:1
Thread-3--printList--current:2
Thread-3--printList--current:3
Thread-3--printList--current:4
Thread-4--printList--current:0
Thread-5--printList--current:0
Thread-4--printList--current:1
Thread-5--printList--current:1
Thread-4--printList--current:2
Thread-5--printList--current:2
Thread-4--printList--current:3
Thread-5--printList--current:3
Thread-4--printList--current:4
Thread-5--printList--current:4
```
通过运行结果我们可以证明一个结论：**Read锁是共享锁，每个线程都能获取到。Write锁是独占锁，只能同时被一个线程持有。**

当我们把Write锁释放代码注释：
```java
public void addItem(){
    writeLock.lock();
    for(int i=listNumber.size();i<5;i++){
        System.out.println(Thread.currentThread().getName() + "--addItem--current:" + i);
        listNumber.add(i);
    }
    //writeLock.unlock();
}

Thread-0--addItem--current:0
Thread-0--addItem--current:1
Thread-0--addItem--current:2
Thread-0--addItem--current:3
Thread-0--addItem--current:4
```
通过运行结果可以有结论：**当Write锁被持有的时候，Read锁是无法被其它线程申请的，会处于阻塞状态。直至Write锁释放。同时也可以验证到当同一个线程持有Write锁时是可以申请到Read锁。**

推荐阅读：
JDK1.8源码分析之ReentrantReadWriteLock（七）：https://www.cnblogs.com/leesf456/p/5419132.html