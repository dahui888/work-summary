#### 1. 简介
在上篇中我们介绍了SyclicBarrier类的使用，通过SyclicBarrier我们可以完成一些分批执行汇总的任务，而此次介绍的CountDownLatch则是实现类似“倒计时”的功能。

#### 2. Api分析
CountDownLatch源码很简洁，它提供两个典型的方法来实现“倒计时功能”。CountDownLatch在构造方法中指定门闩锁latch的个数，当latch的个数为0的时候则唤醒所有等待状态下的线程。
- countDown()：通过调用countDown方法实现门闩锁-1的效果。
- await()：让当前线程进入等待。
- getCount()：获取门闩锁的个数。

下面我们通过一个示例代码来看一下：
```java
public class MyTest {
	static CountDownLatch countDownLatch;
	public static void main(String[]args){
		countDownLatch = new CountDownLatch(3);
		for(int i=0;i<3;i++){
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					System.out.println(Thread.currentThread().getName() + "--到达车门");
					countDownLatch.countDown();
					System.out.println(Thread.currentThread().getName() + "--已登车");
				}
			}).start();
		}
		
		try {
			countDownLatch.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName() + "--登车完毕");
	}
}
```
在示例代码中，通过构造函数创建一个指定latch个数为3的CountDownLatch的方法，同时创建了三个线程，在每个线程的内部分别调用一次countDown()方法，最后在主线程中调用await方法进行等待阻塞，最后完成输出。
执行结果：
```java
Thread-2--到达车门
Thread-2--已登车
Thread-0--到达车门
Thread-0--已登车
Thread-1--到达车门
main--登车完毕
Thread-1--已登车
```
从上面可以得知CountDownLatch以下特点：
1. 线程中执行countDown方法后能继续执行后续代码，线程不会阻塞等待；
2. latch个数为0的时候会立即唤醒await等待的线程；

#### 3. CountDownLatch源码解析
结合上面对CountDownLatch功能的描述，假如让我们实现一个这样的功能，我们首先想到的思路应该是在工具类中维持一个Count计数变量，然后维持对该变量的判断。那么我们来看看CountDownLatch是怎么实现的。
```java
public class CountDownLatch {
    /**
     * Synchronization control For CountDownLatch.
     * Uses AQS state to represent count.
     */
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    /**
     * 让当前线程进行等待，直到锁个数为0或者当前线程interrupt
     *	如果当前锁latch个数为0，则立即返回
     */
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    /**
     * 减少latch锁，当latch锁个数为0的时候释放所有等待线程。
     */
    public void countDown() {
        sync.releaseShared(1);
    }

    public long getCount() {
        return sync.getCount();
    }

    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```
首先从构造方法来看，CountDownLatch提供了有参构造函数：
```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```
这里的Sync类是CountDownLatch内部集成AQS实现的一个内部类。
```java
Sync(int count) {
    setState(count);
}
```
在Sync的构造函数中，调用的是AQS的setState方法修改state值，将state值修改为count值。

通过上面的介绍，对于CountDownLatch类来说，使用最多的就是countDown和await方法。

**countDown()方法**
```java
public void countDown() {
    sync.releaseShared(1);
}

/**AQS类中定义的该方法*/
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

```
在countDown方法内部是调用Sync的releaseShared方法，尝试释放公共锁状态。
```java
protected boolean tryReleaseShared(int releases) {
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c-1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```
采用自旋的方式，获取当前state值，如果state值为0就返回false，即释放失败。反之则state-1，通过CAS操作成功，返回nextc == 0；当最后通过咨询的方式返回true的时候（state ==0），则执行releaseShared方法中的doReleaseShared()方法，在doReleaseShared()方法内部通过unparkSuccessor()方法唤醒阻塞的线程开始执行。

**await()方法**
```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

/***AQS中的方法*/
public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}

protected int tryAcquireShared(int acquires) {
    return (getState() == 0) ? 1 : -1;
}
```
这里调用acquireSharedInterruptibly()方法申请锁，如果锁被占有state！=0，则通过doAcquireSharedInterruptibly()进行锁申请。

#### 4. CountDownLatch的使用场景
- 确保某个计算在其需要的所有资源都被初始化之后才继续执行。
- 确保某个服务在其依赖的所有其他服务都已启动后才启动。
- 等待知道某个操作的所有者都就绪在继续执行。