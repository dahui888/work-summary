#### 1. 简介
##### 1.1 概述
前面已经介绍SyclicBarrier、CountDownLatch、Semaphore三个并发编程中的工具类，还剩下最后一个Exchanger。Exchanger（交换者）是一个用于线程间数据交换协作的工具类。它提供一个同步点，在这个同步点多个线程间两两之间线程可以交换彼此的数据。这两个线程通过exchange方法交换数据， 如果第一个线程先执行exchange方法，它会一直等待第二个线程也执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。

##### 1.2 Api接口
查阅Exchanger的源码，可以发现给我们提供的公共方法只有三个。
1. Exchanger()：无参构造方法
2. exchange(V)：exchange方法用于交互数据V
3. exchange(V,long,TimeUnit)：延迟一定时间交换数据

Exchanger源码简洁，但是它的设计思想还是比较复杂的。CyclicBarrier、CountDownLatch通过借助AbstractQueuedSynchronized的state字段进行“临界点”的标识，Exchanger是如何实现"临界点"的判断呢？

##### 1.3 实例演示
创建连个线程，进行内部数据交换。
```java
public class TestJava {
	private static final Exchanger<String> exgr = new Exchanger<String>();

	public static void main(String[] args) {
		new Thread((new Runnable() {
			@Override
			public void run() {
				try {
					String A = "数据A";
					String B = exgr.exchange(A);
					System.out.println(Thread.currentThread().getName() + "：A录入的是："
							+ A + ",B录入是：" + B);
				} catch (InterruptedException e) {
				}
			}
		})).start();

		new Thread((new Runnable() {
			@Override
			public void run() {
				try {
					String B = "数据B";// B录入银行流水数据
					String A = exgr.exchange(B);
					System.out.println(Thread.currentThread().getName() +  "：A录入的是："
							+ A + ",B录入是：" + B);
				} catch (InterruptedException e) {
				}
			}
		})).start();
	}
}
```
运行结果：
```java
Thread-0：A录入的是：数据A,B录入是：数据B
Thread-1：A录入的是：数据A,B录入是：数据B
```
在两个线程中分别调用exchange方法进行数据的交换，当第二个线程调用exchange方法的时候，数据进行交换。

#### 2. 源码解析
Exchanger是一个用于成对线程之间交换数据的同步器。主要用于遗传算法和管道通讯等两两交换比对的场景。

内部结构：
```java
private static final class Node extends AtomicReference<Object> {  
    /** 创建这个节点的线程提供的用于交换的数据。 */  
    public final Object item;  
    /** 等待唤醒的线程 */  
    public volatile Thread waiter;  
    /** 
     * Creates node with given item and empty hole. 
     * @param item the item 
     */  
    public Node(Object item) {  
        this.item = item;  
    }  
}  
  
/** 
 * 一个Slot就是一对线程交换数据的地方。 
 * 这里对Slot做了缓存行填充，能够避免伪共享问题。 
 * 虽然填充导致浪费了一些空间，但Slot是按需创建，一般没什么问题。 
 */  
private static final class Slot extends AtomicReference<Object> {  
    // Improve likelihood of isolation on <= 64 byte cache lines  
    long q0, q1, q2, q3, q4, q5, q6, q7, q8, q9, qa, qb, qc, qd, qe;  
}  
  
/** 
 * Slot数组，在需要时才进行初始化。 
 * 用volatile修饰，因为这样可以安全的使用双重锁检测方式构建。 
 */  
private volatile Slot[] arena = new Slot[CAPACITY];  
/** 
 * arena(Slot数组)的容量。设置这个值用来避免竞争。 
 */  
private static final int CAPACITY = 32;  
/** 
 * 正在使用的slot下标的最大值。当一个线程经历了多次CAS竞争后， 
 * 这个值会递增；当一个线程自旋等待超时后，这个值会递减。 
 */  
private final AtomicInteger max = new AtomicInteger();  
```
一个无参的构造方法，用于创建一个Exchanger实例。内部结构很清晰，首先内部包含一个Slot数组，默认容量是32，用来避免以一些竞争，有点类似于ConcurrentHashMap的策略；其次，交换数据的场所就是Slot，它本身进行了cache line填充，避免了伪共享问题；最后，每个要进行数据交换的线程在内部会用一个Node来表示。

    伪共享说明：假设一个类的两个相互独立的属性a和b在内存地址上是连续的(比如FIFO队列的头尾指针)，那么它们通常会被加载到相同的cpu cache line里面。并发情况下，如果一个线程修改了a，会导致整个cache line失效(包括b)，这时另一个线程来读b，就需要从内存里再次加载了，这种多线程频繁修改ab的情况下，虽然a和b看似独立，但它们会互相干扰，非常影响性能。

**关键技术点1：CacheLine填充**
在上面的代码中，Slot其实就是一个AtomicReference，其里面的q0, q1,..qd那些变量，都是多余的，不用的。那为什么要添加这些多余的变量呢？
是为了让不同的Slot不要落在cpu的同一个CacheLine里面。因为cpu从内存读取数据的时候，不是一个字节一个字节的读，而是按块读取，这里的块也就是“CacheLine”，一般一个CacheLine大小是64Byte。
保证一个Slot的大小 >= 64Byte，这样更改一个Slot，就不会导致另外一个Slot的cpu cache失效，从而提高性能。


通过前面示例，我们知道Exchanger类最核心的是exchange方法。
```java
/**
 * 等待另一个线程调用exchange方法到达“临界点”，除非当前线程interrupted。
 * 如果另一个线程已经在等待（已调用exchange方法），则这个线程被唤醒并且接收
 * 传送过来的数据。同时当前线程立即返回进行交换数据。
 * 如果没有其它线程调用exchange方法，则当前线程不可用，除非以下两种情况出现：
 * 		1.有其它线程调用exchange方法。
 *		2.当前线程被interrupted
 * @param x 交换数据
 * @return 另外一个线程交换的数据
 * @throws InterruptedException if the current thread was
 *         interrupted while waiting
 */
public V exchange(V x) throws InterruptedException {
    if (!Thread.interrupted()) {
        Object v = doExchange((x == null) ? NULL_ITEM : x, false, 0);
        if (v == NULL_ITEM)
            return null;
        if (v != CANCEL)
            return (V)v;
        Thread.interrupted(); // Clear interrupt status on IE throw
    }
    throw new InterruptedException();
}
```
如果当前线程没有被interrupted，则调用doExchange方法进行数据交换。
```java
private Object doExchange(Object item, boolean timed, long nanos) {
    Node me = new Node(item);                 // 将当前数据进行存储，创建Node
    int index = hashIndex();                  // 当前Slot的下标index位置
    int fails = 0;                            // CAS操作失败的次数

    for (;;) {
        Object y;                             
        // 当前Slot值
        Slot slot = arena[index];
        /**懒汉式初始化，当前slot为null的时候，则在当前index创建新Slot*/
        if (slot == null)
            createSlot(index);                // 继续循环
        else if ((y = slot.get()) != null &&  // 如果当前slot值不为null，且值y没改变，则将当前slot片段设置为null。
                 slot.compareAndSet(y, null)) {
            Node you = (Node)y;               // 如果当前node为null，则将新值进行CAS操作赋值。
            if (you.compareAndSet(null, item)) {
                LockSupport.unpark(you.waiter);
                return you.item;
            }                                 // Else cancelled; continue
        }
        else if (y == null &&                 // 如果当前slot存储的值为null，并且通过CAS操作赋值me成功。
                 slot.compareAndSet(null, me)) {
            // 如果当前index==0，则加入等待队列，等待与别人交换，即下标为0的位置始终是等待别人来交换的位置
            if (index == 0)                  
                return timed ?
                    awaitNanos(me, slot, nanos) :
                    await(me, slot);
            Object v = spinWait(me, slot);    // 如果当前index！=0
            /**是不是CANCEL*/
            if (v != CANCEL)
                return v;
            me = new Node(item);              // Throw away cancelled node
            int m = max.get();
            if (m > (index >>>= 1))           // 当前位置不为0，则index/2不停缩小，直至找到交换值。
                max.compareAndSet(m, m - 1);  // Maybe shrink table
        }
        else if (++fails > 1) {               // Allow 2 fails on 1st slot
            int m = max.get();
            if (fails > 3 && m < FULL && max.compareAndSet(m, m + 1))
                index = m + 1;                // Grow on 3rd failed slot
            else if (--index < 0)
                index = m;                    // Circularly traverse
        }
    }
}
```
所以，exchange的思路是：
1. 根据每个线程的thread id, hash计算出自己所在的slot index； 
2. 如果运气好，这个slot被人占着（slot里面有node)，并且有人正在等待交换，那就和它进行交换； 
3. slot为空的(slot里面没有node)，自己占着，等人交换。没人交换，向前挪个位置，把当前slot里面内容取消，index减半，再看有没有交换； 
4. 挪到0这个位置，还没有人交互，那就阻塞，一直等着。别的线程，也会一直挪动，直到0这个位置。

所以0这个位置，是一个交易的“终结点”位置！别的位置上找不到人交易，最后都会到0这个位置。

至此，整个逻辑梳理完毕。

**参考阅读**
http://blog.csdn.net/chunlongyu/article/details/52504895
http://brokendreams.iteye.com/blog/2253956