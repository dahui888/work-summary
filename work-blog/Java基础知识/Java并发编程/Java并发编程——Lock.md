#### Java并发编程——Lock和Condition
我们知道AQS类是所有并发编程锁的核心，那么在实际的使用中，我们就需要认识下Lock接口给我们定义了哪些方法。

Lock用于并发编程中针对共享资源的访问，通常，一个lock对象提供独占的方式来访问对象，即独占锁——在同一时间只有一个线程能够获取锁并访问资源。但是也有一些锁提供并发访问共享资源，比如ReadWriteLock，这类称之为共享锁。

使用synchronized同步的方法或者代码段具有隐式的锁监听器（锁）。
```java
public synchronized void doGet(){

}
```
但是lock的使用要求是块状成对结构，当不同的lock被申请的时候，他们必须依照合适的顺序在相同的作用域进行释放。
```java
Lock l = ...;
l.lock();
try {
  // access the resource protected by this lock
 } finally {
  l.unlock();
}}
```

##### 一、Lock源码分析
```java
public interface Lock {

    /**
     * Acquires the lock.
     * 该方法用于获取锁
     * 如果当前线程无法获取锁，则当前线程进入休眠状态不可用，直至当前线程获取到锁。
     * 如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。
     */
    void lock();

    /**
     * 1）如果当前线程未被中断，则获取锁。 
     * 2）如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。 
     * 3）如果当前线程已经保持此锁，则将保持计数加 1，并且该方法立即返回。
     * 4）如果锁被另一个线程保持，则出于线程调度目的，禁用当前线程，并且在发生以下两种情况之一以
	 *		前，该线程将一直处于休眠状态： 
     *		1）锁由当前线程获得；或者 
     *		2）其他某个线程中断当前线程。 
	 * 5）如果当前线程获得该锁，则将锁保持计数设置为 1。 
   	 *	如果当前线程： 
     *  		1）在进入此方法时已经设置了该线程的中断状态；
     *  		2）在等待获取锁的同时被中断。 
	 *	则抛出 InterruptedException，并且清除当前线程的已中断状态。 
	 * 6）在此实现中，因为此方法是一个显式中断点，所以要优先考虑响应中断，而不是响应锁的普通获取或
	 *	重入获取。
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 尝试获取锁，仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
     */
    boolean tryLock();

    /**
     * 
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 释放锁
     * 只有持有这个lock的线程才能释放锁
     */
    void unlock();

    /**
     * 返回一个绑定到Lock对象上的Condition实例，在获取condition对象前，当前线程
     * 必须持有对应的lock对象。
     */
    Condition newCondition();
}
```
在上面的源码中，我们可以看到Lock提供了lock()方法用于获取锁，unlock()方法用于释放锁。
比如下面的例子：
```java
public class ConcrateDemo {
	private Lock lock = new ReentrantLock();
	
	public static void main(String []args){
		MyRunnable runnable = new ConcrateDemo().new MyRunnable();
		Thread thread1 = new Thread(runnable);
		Thread thread2 = new Thread(runnable);
		thread1.start();
		thread2.start();
	}
	class MyRunnable implements Runnable{

		public void run() {
			lock.lock();
			for(int i=0;i<5;i++){
				System.out.println("currentThread:" + Thread.currentThread().getName()
						+ "==Cnt:" + i);
			}
			lock.unlock();
		}
	}
}
```
##### 二、Condition源码分析
在上面我们介绍Lock类时，有一个newCondition方法：
```java
/**
 * 返回一个绑定到Lock对象上的Condition实例，在获取condition对象前，当前线程
 * 必须持有对应的lock对象。
 */
Condition newCondition();
```
从这里可以猜想到一个Lock中应该绑定一个Condition对象。Condition是Java提供用来实现等待/通知的类。

我们知道Object对象提供了wait、waitAll、notify、notifyAll的方法用来实现线程的同步、等待和唤醒。但Condition类提供了比wait/notify更丰富的功能，Condition对象由lock对象所创建的，同时一个Lock可以创建多个Condition对象，即创建多个对象监听器，这样就可以指定唤醒具体线程，而notify是随机唤醒线程。

```java
public interface Condition {

    /**
     * 造成当前线程在接到信号或被 中断之前一直处于等待状态。
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    void await() throws InterruptedException;

    /**
     * 造成当前线程在接到信号或被 中断之前一直处于等待状态。
     */
    void awaitUninterruptibly();

    /**
     * 使当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
     * @param nanosTimeout the maximum time to wait, in nanoseconds
     * @return an estimate of the {@code nanosTimeout} value minus
     *         the time spent waiting upon return from this method.
     *         A positive value may be used as the argument to a
     *         subsequent call to this method to finish waiting out
     *         the desired time.  A value less than or equal to zero
     *         indicates that no time remains.
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    long awaitNanos(long nanosTimeout) throws InterruptedException;

    /**
     * 使当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
     * @param time the maximum time to wait
     * @param unit the time unit of the {@code time} argument
     * @return {@code false} if the waiting time detectably elapsed
     *         before return from the method, else {@code true}
     * @throws InterruptedException if the current thread is interrupted
     *         (and interruption of thread suspension is supported)
     */
    boolean await(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 唤醒一个等待线程。
     */
    void signal();

    /**
     * 唤醒所有等待线程。
     */
    void signalAll();
}
```
通过上面的源码注释能看到，Condition提供了以下方法：
- void await()：造成当前线程在接到信号或被中断之前一直处于等待状态。
- boolean await(long time, TimeUnit unit)：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
- long awaitNanos(long nanosTimeout)：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
- void awaitUninterruptibly()：造成当前线程在接到信号之前一直处于等待状态。
- boolean awaitUntil(Date deadline)：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。
- void	signal()：唤醒一个等待线程。
- void	signalAll()：唤醒所有等待线程。

在实际开发中，通过await方法进行线程的等待，signal进行唤醒。注意，Condition 实例只是一些普通的对象，它们自身可以用作 synchronized 语句中的目标，并且可以调用自己的 wait 和 notification 监视器方法。

**示例：生产消费者**
```java
/**
 * 实现生产消费者的例子
 * 要求：
 * 有两股力量：生产和消费
 * 当仓库生产满了的时候就要通知消费者进行消费，并且停止生产
 * 当仓库空的时候，消费者要通知生产者进行生产，并且停止消费
 * 其它情况，正常生产、消费。
 * 
 * 生产者与消费者模型中，要保证以下几点：
 * 1 同一时间内只能有一个生产者生产
 * 2 同一时间内只能有一个消费者消费
 * 3 共享空间空时消费者不能继续消费
 * 4 共享空间满时生产者不能继续生产 
 * @author mr_dsw
 */
public class ConcrateDemo {
	public static void main(String []args){
		Resource resource = new Resource();
		ProduceThread produceThread = new ProduceThread(resource);
		ConsumeThread consumeThread = new ConsumeThread(resource);
		//四个生产者
		new Thread(produceThread).start();
		new Thread(produceThread).start();
		new Thread(produceThread).start();
		new Thread(produceThread).start();
		//四个消费者
		new Thread(consumeThread).start();
		new Thread(consumeThread).start();
		new Thread(consumeThread).start();
		new Thread(consumeThread).start();
	}
}

class Resource{
	private final int MAX_SIZE = 10;
	private LinkedList<Object> list = new LinkedList<Object>();
	private Lock lock = new ReentrantLock();
	private Condition fullCondition = lock.newCondition();
	private Condition emptyCondition = lock.newCondition();
	
	/**
	 * 生产物品，存在多个生产者
	 */
	public void produce(){
		//如果生产满了，则就唤醒消费者
		lock.lock();
		while(list.size() == MAX_SIZE){
			System.out.println("生产满了，暂时无法生产：" + list.size());
			emptyCondition.signal();
			try {
				fullCondition.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		list.add(new Object());
		System.out.println(Thread.currentThread().getName() + "生产新产品，共有：" + list.size());
		lock.unlock();
	}
	
	/**
	 * 消费者，存在多个消费者
	 */
	public void consume(){
		lock.lock();
		while(list.size() == 0){
			System.out.println("没有物品了，需要通知生产了");
			fullCondition.signal();
			try {
				emptyCondition.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		System.out.println(Thread.currentThread().getName() + "消费产品，共有：" + list.size());
		list.remove();
		lock.unlock();
	}
}

class ProduceThread implements Runnable{
	private Resource resource;
	
	public ProduceThread(Resource resource){
		this.resource = resource;
	}
	
	public void run() {
		for(;;)
		resource.produce();
	}
}

class ConsumeThread implements Runnable{
	private Resource resource;
	
	public ConsumeThread(Resource resource){
		this.resource = resource;
	}
	
	public void run() {
		for(;;)
		resource.consume();
	}
}
```
**补充**
前面我们提到AQS是所有锁的基础，同样在AQS中的ConditionObject就是实现Condition的核心。ConditionObject的等待队列是一个FIFO队列，队列的每个节点都是等待在Condition对象上的线程的引用，在调用Condition的await()方法之后，线程释放锁，构造成相应的节点进入等待队列等待。其中节点的定义复用AQS的Node定义。

以上针对Java并发编程中的Lock和Condition进行初探，以便后续针对实现的进一步分析。