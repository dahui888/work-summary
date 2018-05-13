#### Java并发编程——AQS源码解析

- 什么是AQS
- AQS有什么用
- AQS实现方式

##### 一、AQS是什么？
AQS是一个基于先进先出（FIFO）等待队列的实现阻塞锁和同步器的框架。AQS通过一个volatile int state变量来保存锁的状态。子类必须通过：
- getState()：获取当前的同步状态
- setState(int newState)：设置当前同步状态
- compareAndSetState(int expect,int update)：使用CAS设置当前状态，该方法能够保证状态设置的原子性。

三个方法修改获取锁的状态值state。
>CAS (compare and swap) 比较并交换，就是将内存值与预期值进行比较，如果相等才将新值替换到内存中，并返回true表示操作成功；如果不相等，则直接返回false表示操作失败。CAS操作大多都是靠CPU原语来实现。CAS操作经常被用来实现无锁数据结构，在java.util.concurrent包中就有很多这样的数据结构：ConcurrentLinkedQueue、ConcurrentLinedDeque、ConcurrentHashMap、ConcurrentSkipListMap、ConcurrentSkipListSet。

**1、AQS支持的锁的类别**
AQS支持独占锁和共享锁两种。
- 独占锁：锁在一个时间点只能被一个线程占有。根据锁的获取机制，又分为“公平锁”和“非公平锁”。等待队列中按照FIFO的原则获取锁，等待时间越长的线程越先获取到锁，这就是公平的获取锁，即公平锁。而非公平锁，线程获取的锁的时候，无视等待队列直接获取锁。ReentrantLock和ReentrantReadWriteLock.Writelock是独占锁。
- 共享锁：同一个时候能够被多个线程获取的锁，能被共享的锁。JUC包中ReentrantReadWriteLock.ReadLock，CyclicBarrier，CountDownLatch和Semaphore都是共享锁。

**2、基于AQS实现锁**
AQS中没有实现任何的同步接口，所以一般子类通过继承AQS以内部类的形式实现锁机制。一般通过继承AQS类实现同步器，通过getState、setState、compareAndSetState来监测状态，并重写以下方法：
- tryAcquire()：独占方式。尝试获取资源，成功则返回true，失败则返回false。
- tryRelease()：独占方式。尝试释放资源，成功则返回true，失败则返回false。
- tryAcquireShared()：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
- tryReleaseShared()：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。
- isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。

**3、AQS同步器的存储结构**

##### 二、AQS源码解析
**1、存储节点Node**
我们一直在说AQS是基于FIFO队列的存储结构，它是以内部类Node节点的形式进行存储。这个等待队列是CLH同步队列。
```java
static final class Node {
    /** 共享节点模式下的节点 */
    static final Node SHARED = new Node();
    /** 独占模式下的节点 */
    static final Node EXCLUSIVE = null;

    /** 取消状态 */
    static final int CANCELLED =  1;
    /** 后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行 */
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;
    /**
     * 下一次共享式同步状态获取将会无条件地传播下去
     */
    static final int PROPAGATE = -3;

    /**
     * Status field, taking on only the values:
     *   SIGNAL:     The successor of this node is (or will soon be)
     *               blocked (via park), so the current node must
     *               unpark its successor when it releases or
     *               cancels. To avoid races, acquire methods must
     *               first indicate they need a signal,
     *               then retry the atomic acquire, and then,
     *               on failure, block.
     *   CANCELLED:  This node is cancelled due to timeout or interrupt.
     *               Nodes never leave this state. In particular,
     *               a thread with cancelled node never again blocks.
     *   CONDITION:  This node is currently on a condition queue.
     *               It will not be used as a sync queue node
     *               until transferred, at which time the status
     *               will be set to 0. (Use of this value here has
     *               nothing to do with the other uses of the
     *               field, but simplifies mechanics.)
     *   PROPAGATE:  A releaseShared should be propagated to other
     *               nodes. This is set (for head node only) in
     *               doReleaseShared to ensure propagation
     *               continues, even if other operations have
     *               since intervened.
     *   0:          None of the above
     *
     * The values are arranged numerically to simplify use.
     * Non-negative values mean that a node doesn't need to
     * signal. So, most code doesn't need to check for particular
     * values, just for sign.
     *
     * The field is initialized to 0 for normal sync nodes, and
     * CONDITION for condition nodes.  It is modified using CAS
     * (or when possible, unconditional volatile writes).
     */
    volatile int waitStatus;

    /**
     * 前驱节点
     */
    volatile Node prev;

    /**
     * 后驱节点
     */
    volatile Node next;

    /**
     * 获取同步状态的线程
     */
    volatile Thread thread;

    /**
     * Link to next node waiting on condition, or the special
     * value SHARED.  Because condition queues are accessed only
     * when holding in exclusive mode, we just need a simple
     * linked queue to hold nodes while they are waiting on
     * conditions. They are then transferred to the queue to
     * re-acquire. And because conditions can only be exclusive,
     * we save a field by using special value to indicate shared
     * mode.
     */
    Node nextWaiter;

    /**
     * Returns true if node is waiting in shared mode.
     */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**
     * Returns previous node, or throws NullPointerException if null.
     * Use when predecessor cannot be null.  The null check could
     * be elided, but is present to help the VM.
     *
     * @return the predecessor of this node
     */
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```
在Node内部类中，声明了pre、next节点用于队列的连接，同时保存了waitStatus状态。

**2、AQS源码解析**
在面向对象的世界中，想要了解一个类有什么特点，就要看它的属性。通过查看源码，我们看到AQS类包含：
```java
/**
 * Head of the wait queue, lazily initialized.  Except for
 * initialization, it is modified only via method setHead.  Note:
 * If head exists, its waitStatus is guaranteed not to be
 * CANCELLED.
 */
private transient volatile Node head;

/**
 * Tail of the wait queue, lazily initialized.  Modified only via
 * method enq to add new wait node.
 */
private transient volatile Node tail;

/**
 * The synchronization state.
 */
private volatile int state;
```
- 前驱头节点-head
- 后驱尾节点-tail
- 同步器状态-state

AQS基于FIFO队列，接下来就依照acquire-release、acquireShared-releaseShared的次序来分析入队和出队。

**1、acquire(int)**
此方法是独占模式下线程获取共享资源的顶层入口。如果获取到资源成功，线程直接返回，否则进入等待队列，直到获取到资源为止，且整个过程忽略中断的影响。这也正是lock()的语义，当然不仅仅只限于lock()。获取到资源后，线程就可以去执行其临界区代码了。下面是acquire源码：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
在acquire方法中调用了tryAcquire()、acquireQueued()、addWaiter()三个方法。

首先通过**tryAcquire方法尝试申请独占锁**。如果获取成功则返回true，否则返回false。
```java
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}
```
呦西，源码中直接throw一个异常。结合我们前面自定义锁的知识，AQS只是一个框架，具体资源获取和释放方式交由自定义同步器实现。AQS这里只定义了一个接口，具体资源的获取交由自定义同步器去实现了（通过state的get/set/CAS）！！！至于能不能重入，能不能加塞，那就看具体的自定义同步器怎么去设计了！！！当然，自定义同步器在进行资源访问时要考虑线程安全的影响。

这里之所以没有定义成abstract，是因为独占模式下只用实现tryAcquire-tryRelease，而共享模式下只用实现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。说到底，Doug Lea还是站在咱们开发者的角度，尽量减少不必要的工作量。

接着就是**addWaiter()方法用于将当前线程添加到等待队列的队尾，并返回当前线程所在的节点**。
```java
private Node addWaiter(Node mode) {
	//以给定的Node节点模式构建当前线程的Node节点，在acquire方法中传入的是EXCLUSIVE独占式节点
    Node node = new Node(Thread.currentThread(), mode);
    // 将尾节点进行保存
    Node pred = tail;
    if (pred != null) {//如果尾节点不为null
		//将尾节点设置尾新节点的prev节点
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {//通过CAS保证，确保节点能够被线程安全的添加
			//将当前界定指向前驱的next节点
            pred.next = node;
            return node;
        }
    }
	//如果尾节点为null，则通过enq进行入队
    enq(node);
    return node;
}

//同步器通过死循环的方式来保证节点的正确添加，在“死循环” 中通过CAS将节点设置成为尾节点之后，
//当前线程才能从该方法中返回，否则当前线程不断的尝试设置。
private Node enq(final Node node) {
	//CAS"自旋"，直到成功加入队尾
    for (;;) {
		//尾节点临时存储
        Node t = tail;
        if (t == null) { // Must initialize
			//如果tail为null，则将
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
在addWaiter(Node node)方法中，将当前线程节点添加到等待队列中。

![enq](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/enq.png)

**acquireQueued在队列中的线程获取锁**
```java
/**
* Acquires in exclusive uninterruptible mode for thread already in
* queue. Used by condition wait methods as well as acquire.
*
* @param node the node
* @param arg the acquire argument
* @return {@code true} if interrupted while waiting
* 
* acquireQueued方法当前线程在死循环中获取同步状态，而只有前驱节点是头节点才能尝试获取同步状态（锁）（ p == head && tryAcquire(arg)）
*     原因是:1.头结点是成功获取同步状态（锁）的节点，而头节点的线程释放了同步状态以后，将会唤醒其后继节点，后继节点的线程被唤醒后要检查自己的前驱节点是否为头结点。
*           2.维护同步队列的FIFO原则，节点进入同步队列之后，就进入了一个自旋的过程，每个节点（或者说是每个线程）都在自省的观察。
* 
*/
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        //死循环检查（自旋检查）当前节点的前驱节点是否为头结点，才能获取锁
        for (;;) {
            // 获取节点的前驱节点
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {//节点中的线程循环的检查，自己的前驱节点是否为头节点
                //将当前节点设置为头结点，移除之前的头节点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 否则检查前一个节点的状态，看当前获取锁失败的线程是否要挂起
            if (shouldParkAfterFailedAcquire(p, node) &&
                //如果需要挂起，借助JUC包下面的LockSupport类的静态方法park挂起当前线程，直到被唤醒
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        //如果有异常
        if (failed)
            //取消请求，将当前节点从队列中移除
            cancelAcquire(node);
    }
}
```
通过addWaiter方法添加到等待队列中后，在通过acquireQueued方法进行锁的获取

函数流程：
- tryAcquire()尝试直接去获取资源，如果成功则直接返回；
- addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
- acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
- 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

**独占式锁获取流程**
调用同步器的acquire(int arg)方法可以获取同步状态，该方法对中断不敏感，即线程获取同步状态失败后进入同步队列，后续对线程进行中断操作时，线程不会从同步队列中移除。获取流程：
1. 当前线程通过tryAcquire()方法尝试获取锁，成功则直接返回，失败则进入队列排队等待，通过CAS获取同步状态。
2. 如果尝试获取锁失败的话，构造同步节点（独占式的Node.EXCLUSIVE），通过addWaiter(Node node,int args)方法,将节点加入到同步队列的队列尾部。
3. 最后调用acquireQueued(final Node node, int args)方法，使该节点以死循环的方式获取同步状态，如果获取不到，则阻塞节点中的线程。acquireQueued方法当前线程在死循环中获取同步状态，而只有前驱节点是头节点的时候才能尝试获取锁（同步状态）（ p == head && tryAcquire(arg)）。

![lock](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/Java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/screenshot/lock.png)

**2、release(int)：独占锁的释放**
在AQS中通过release方法进行锁的释放。
```java
public final boolean release(int arg) {
   //调用tryRelease方法释放
    if (tryRelease(arg)) {//如果释放成功
        Node h = head;
        //如果头节点不为null，并且头结点的waitStatus值不为0，即有状态
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
   //没有释放成功返回false
    return false;
}

// tryRelease() 尝试释放当前线程的同步状态（锁）
protected final boolean tryRelease(int releases) {
        //c为释放后的同步状态
      int c = getState() - releases;
      //判断当前释放锁的线程是否为获取到锁（同步状态）的线程，不是抛出异常（非法监视器状态异常）
      if (Thread.currentThread() != getExclusiveOwnerThread())
          throw new IllegalMonitorStateException();
      boolean free = false;
      //如果锁（同步状态）已经被当前线程彻底释放，则设置锁的持有者为null，同步状态（锁）变的可获取
      if (c == 0) {
          free = true;
          setExclusiveOwnerThread(null);
      }
      setState(c);
      return free;
  }

private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    //从队列尾部开始往前去找最前面的一个waitStatus小于0的节点。
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //唤醒后继节点对应的线程
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
release()是独占模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果彻底释放了（即state=0）,它会唤醒等待队列里的其他线程来获取资源。

**3、acquireShared(int)**
此方法是共享模式下线程获取共享资源的顶层入口。它会获取指定量的资源，获取成功则直接返回，获取失败则进入等待队列，直到获取到资源为止，整个过程忽略中断。下面是acquireShared()的源码：
```java
public final void acquireShared(int arg) {
     if (tryAcquireShared(arg) < 0)
         doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);//加入队列尾部
    boolean failed = true;//是否成功标志
    try {
        boolean interrupted = false;//等待过程中是否被中断过的标志
        for (;;) {
            final Node p = node.predecessor();//前驱
            if (p == head) {//如果到head的下一个，因为head是拿到资源的线程，此时node被唤醒，很可能是head用完资源来唤醒自己的
                int r = tryAcquireShared(arg);//尝试获取资源
                if (r >= 0) {//成功
                    setHeadAndPropagate(node, r);//将head指向自己，还有剩余资源可以再唤醒之后的线程
                    p.next = null; // help GC
                    if (interrupted)//如果等待过程中被打断过，此时将中断补上。
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            
            //判断状态，寻找安全点，进入waiting状态，等着被unpark()或interrupt()
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
这里tryAcquireShared()依然需要自定义同步器去实现。但是AQS已经把其返回值的语义定义好了：负值代表获取失败；0代表获取成功，但没有剩余资源；正数表示获取成功，还有剩余资源，其他线程还可以去获取。所以这里acquireShared()的流程就是：

1. tryAcquireShared()尝试获取资源，成功则直接返回；
2. 失败则通过doAcquireShared()进入等待队列park()，直到被unpark()/interrupt()并成功获取到资源才返回。整个等待过程也是忽略中断的。

doAcquireShared(int)此方法用于将当前线程加入等待队列尾部休息，直到其他线程释放资源唤醒自己，自己成功拿到相应量的资源后才返回。

**4、releaseShared()**
releaseShared()是共享模式下线程释放共享资源的顶层入口。它会释放指定量的资源，如果成功释放且允许唤醒等待线程，它会唤醒等待队列里的其他线程来获取资源。
```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {//尝试释放资源
        doReleaseShared();//唤醒后继结点
        return true;
    }
    return false;
}
```
此方法的流程也比较简单，一句话：释放掉资源后，唤醒后继。跟独占模式下的release()相似，但有一点稍微需要注意：独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。例如，资源总量是13，A（5）和B（7）分别获取到资源并发运行，C（4）来时只剩1个资源就需要等待。A在运行过程中释放掉2个资源量，然后tryReleaseShared(2)返回true唤醒C，C一看只有3个仍不够继续等待；随后B又释放2个，tryReleaseShared(2)返回true唤醒C，C一看有5个够自己用了，然后C就可以跟A和B一起运行。而ReentrantReadWriteLock读锁的tryReleaseShared()只有在完全释放掉资源（state=0）才返回true，所以自定义同步器可以根据需要决定tryReleaseShared()的返回值。

**总结**
本节中针对AQS进行了简单的学习，总结中参照了以下文章，推荐大家去阅读下。
- JUC回顾之-AQS同步器的实现原理：https://www.cnblogs.com/200911/p/60313
- 50.html
- JDK源码之AQS源码剖析：http://www.cnblogs.com/showing/p/6858410.html
- Java并发之AQS详解：https://www.cnblogs.com/waterystone/p/4920797.html