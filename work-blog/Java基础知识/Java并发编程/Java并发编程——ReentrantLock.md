### Java并发编程——ReentrantLock
前面我们对AQS、Lock和Condition进行了学习，我们知道Java并发编程中的锁机制都是基于AQS框架而来，那么今天我们就来学习ReentrantLock锁。

- 锁的本质是什么？加锁的本质是什么

#### 1. ReentrantLock是什么？
ReentrantLock是个可重入的互斥锁 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁定相同的一些基本行为和语义，但功能更强大。ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。可以使用 isHeldByCurrentThread() 和 getHoldCount() 方法来检查此情况是否发生。

>可重入性：从名字上理解，ReenTrantLock的字面意思就是再进入的锁，其实synchronized关键字所使用的锁也是可重入的，两者关于这个的区别不大。两者都是同一个线程没进入一次，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

ReentrantLock的组织结构：
![reentrantLock]()

从图中可以看出：
(01) ReentrantLock实现了Lock接口。
(02) ReentrantLock与sync是组合关系。ReentrantLock中，包含了Sync对象；而且，Sync是AQS的子类；更重要的是，Sync有两个子类FairSync(公平锁)和NonFairSync(非公平锁)。ReentrantLock是一个独占锁，至于它到底是公平锁还是非公平锁，就取决于sync对象是"FairSync的实例"还是"NonFairSync的实例"。

#### 2. ReentrantLock源码解析
既然ReentrantLock作为锁，还是看它的基本构成：
```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;
    ....此处省略N多代码
```
很简单，内部就包含一个Sync类对象。Sync类又未何方神圣？

###### 2.1 Sync类源码

```java
/**
 * Base of synchronization control for this lock. Subclassed
 * into fair and nonfair versions below. Uses AQS state to
 * represent the number of holds on the lock.
 */
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;

    /**
     * Performs {@link Lock#lock}. The main reason for subclassing
     * is to allow fast path for nonfair version.
     */
    abstract void lock();

    /**
     * Performs non-fair tryLock.  tryAcquire is
     * implemented in subclasses, but both need nonfair
     * try for trylock method.
     */
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    // Methods relayed from outer class

    final Thread getOwner() {
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }

    final int getHoldCount() {
        return isHeldExclusively() ? getState() : 0;
    }

    final boolean isLocked() {
        return getState() != 0;
    }

    /**
     * Reconstitutes this lock instance from a stream.
     * @param s the stream
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        setState(0); // reset to unlocked state
    }
}
```
从类的注释中可以知道，Sync继承AQS类，并且是ReentrantLock类同步的基础，它有fair和nonfair两个子类。通过AQS类的state状态number值来记录是否持有锁。基本组成：
- abstract void lock()
- boolean nonfairTryAcquire(int acquires)
-  boolean tryRelease
-  boolean isHeldExclusively()
-  Thread getOwner()
-  getHoldCount()

在Sync类中声明了抽象方法lock供子类来实现。Sync类中实现了锁的获取、释放、是否持有等方法。

**2.1.1 NonfairSync非公平锁**
```java
/**
 * Sync object for non-fair locks
 */
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```
在非公平锁中继承实现了lock方法。首先通过compareAndSetState()方法获取state状态，如果成功，则设置setExclusiveOwnerThread独占锁的线程。
```java
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```
可以看到CAS方法本质是调用Unsafe类的compareAndSwapInt方法，compareAndSwapInt的实现：
```java
/**
 * Atomically update Java variable to <tt>x</tt> if it is currently
 * holding <tt>expected</tt>.
 * @return <tt>true</tt> if successful
 */
public final native boolean compareAndSwapInt(Object o, long offset,
                                              int expected,
                                              int x);
```
可以看到, 不是用Java实现的, 而是通过JNI调用操作系统的原生程序。那么看一下它的JNI实现：
```java
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```
可以看到实际上调用Atomic类的cmpxchg方法。 Atomic的cmpxchg
这个类的实现是跟操作系统有关, 跟CPU架构也有关, 如果是windows下x86的架构
实现在hotspot\src\os_cpu\windows_x86\vm\目录的atomic_windows_x86.inline.hpp文件里 。
```java
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  // alternative for InterlockedCompareExchange
  int mp = os::is_MP();
  __asm {
    mov edx, dest
    mov ecx, exchange_value
    mov eax, compare_value
    LOCK_IF_MP(mp)
    cmpxchg dword ptr [edx], ecx
  }
}
```
在这里可以看到是用嵌入的汇编实现的, 关键CPU指令是 cmpxchg
到这里没法再往下找代码了. 也就是说CAS的原子性实际上是CPU实现的. 其实在这一点上还是有排他锁的. 只是比起用synchronized, 这里的排他时间要短的多. 所以在多线程情况下性能会比较好。

第二步，如果通过CAS操作失败，则主动通过acquire方法获取锁。
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

**2.1.2 公平锁FairSync**
```java
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```
在公平锁的lock方法实现中，同样可以看到直接通过acquire(1)方法获取锁的状态，最终通过CAS方法保证锁的状态。

ReentrantLock支持公平所和非公平所，同时内部定义了Sync类实例，只需要根据构造方法中传入的要求创建对应的实例即可。

##### 2.2 ReentrantLock方法解析
**2.2.1 lock()**
```java
public void lock() {
    sync.lock();
}
```
如果锁没有被其它线程持有，立即获取到锁，同时将锁的持有count+1.如果当前线程已经过去到锁，则将count+1并立即返回。如果锁被其它线程持有，则当前线程在获取锁之前不可用。

**2.2.2 lockInterruptibly()**
```java
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```
获取锁，有以下场景：
- 如果锁没有被其它线程获取，则立即获取锁并返回，并且在锁计数器上+1
- 如果当前线程已经持有锁，则立即返回并在锁计数器上+1
- 如果锁被另外一个线程占用，则当前线程不可用直到发生以下两个事情：①当前线程主动获取到锁；②其它线程调用当前线程的interrupts方法。

**2.2.3 tryLock()**
```java
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```
尝试获取锁，如果锁没有被其它线程持有，则立即获取到并返回true并且锁计数器+1，反之返回false。尽管如果lock被设置为公平锁，但是调用tryLock就会立即申请到锁（如果锁可用）。不管别的线程是否在等待状态。

**2.2.4 unlock()**
```java
public void unlock() {
    sync.release(1);
}
```
尝试释放锁，如果当前线程持有锁，则锁计数器减少-1。如果当前计数器为0，就释放锁。如果当前线程没有持有锁，则抛出IllegalMonitorStateException异常。

**2.2.5 getHoldCount()**
```java
public int getHoldCount() {
    return sync.getHoldCount();
}
```
获取锁在“持有”的线程上的锁计数器个数。

**2.2.6 isHeldByCurrentThread（）**
```java
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
```
当前线程是否持有该锁。

#### 2.3 ReentrantLock与Synchronized	的区别
1. ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。
2. ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。
3. ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。
4. ReenTrantLock与synchronized相比，ReentrantLock提供了更多，更加全面的功能，具备更强的扩展性。例如：时间锁等候，可中断锁等候，锁投票。

**参考阅读**
1. ReenTrantLock可重入锁（和synchronized的区别）总结：http://www.cnblogs.com/baizhanshi/p/7211802.html
2. Java多线程系列--“JUC锁”02之 互斥锁ReentrantLock：http://www.cnblogs.com/skywang12345/p/3496101.html
