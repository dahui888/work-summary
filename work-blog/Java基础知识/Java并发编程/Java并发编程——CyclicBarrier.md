#### 1 简介
工作中我们肯定遇到过这样的场景：“开启多个线程分别执行不同的任务，等到所有线程的任务都执行完毕，然后在进行下一步的操作”。通常遇到这样的需求，我们通过ReentrantLock结合Condition或者通过Object的wait、notify来实现。

针对上述场景，在JUC包中已经提供了满足此类需求的**CyclicBarrier**类来实现。CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它的作用是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞。

**示例代码**
```java
public class MyTest {
	static CyclicBarrier cyclicBarrier;
	public static void main(String[]args){
		cyclicBarrier = new CyclicBarrier(3,new Runnable() {
			
			@Override
			public void run() {
				System.out.println("全部就绪，开始登车");
			}
		});
		for(int i=0;i<3;i++){
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					System.out.println(Thread.currentThread().getName() + "--到达车门");
					try {
						cyclicBarrier.await();
					} catch (InterruptedException e) {
						e.printStackTrace();
					} catch (BrokenBarrierException e) {
						e.printStackTrace();
					}
					System.out.println(Thread.currentThread().getName() + "--已登车");
				}
			}).start();
		}
	}
}
```
执行结果:
```xml
Thread-1--到达车门
Thread-2--到达车门
Thread-0--到达车门
全部就绪，开始登车
Thread-0--已登车
Thread-2--已登车
Thread-1--已登车
```
从输出的日志结果可以得出执行顺序，当三个线程都进行await时候，即都到达屏障，然后屏障开启，各个线程接着往下执行。

**场景一：将屏障的数3修改为2**
```java
Thread-0--到达车门
Thread-2--到达车门
全部就绪，开始登车
Thread-1--到达车门
Thread-2--已登车
Thread-0--已登车
```
当两个线程到达屏障时，屏障打开。但是为什么有个线程没上车呢？

**场景二“将屏障数设置为5**
```java
Thread-0--到达车门
Thread-2--到达车门
Thread-1--到达车门
```
始终无法打开屏障，导致线程都在等待。

#### 2 CyclicBarrier源码解析
CyclicBarrier的源码不多，结合上面的场景理解，他更像是一个特定场景下的工具类。
```java
public class CyclicBarrier {
    /**
     * 静态内部类，当前屏障是否被破坏
     */
    private static class Generation {
        boolean broken = false;
    }

    /** 实现的Lock */
    private final ReentrantLock lock = new ReentrantLock();
    /** Condition用来实现wait */
    private final Condition trip = lock.newCondition();
    /** 等待的屏障数 */
    private final int parties;
    /* 到达屏障要执行的Runnable */
    private final Runnable barrierCommand;
    /** The current generation */
    private Generation generation = new Generation();

    /**
     * Number of parties still waiting. Counts down from parties to 0
     * on each generation.  It is reset to parties on each new
     * generation or when broken.
     */
    private int count;

    public CyclicBarrier(int parties, Runnable barrierAction) {
        if (parties <= 0) throw new IllegalArgumentException();
        this.parties = parties;
        this.count = parties;
        this.barrierCommand = barrierAction;
    }

    public CyclicBarrier(int parties) {
        this(parties, null);
    }
}
```
从CyclicBarrier的成员来看，它本质上是基于ReentrantLock独占锁实现，通过Lock和Condition的结合，在加上计数器来实现。它的核心方法是await()。

```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen;
    }
}

/**
 * Main barrier code, covering the various policies.
 */
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    /**获取CyclicBaerrier的内部锁*/
    final ReentrantLock lock = this.lock;
    /**获取锁*/
    lock.lock();
    try {
        /**存储当前的Generation*/
        final Generation g = generation;
        /**判断当前的屏障是否被破坏，如果破坏则抛出BrokenBarrierException异常*/
        if (g.broken)
            throw new BrokenBarrierException();
        /**判断当前线程是否被interrupted，如果被打断，则breakBarrier破坏屏障*/
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }
       /**记录当前屏障等待个数*/ 
       int index = --count;
       if (index == 0) {  // 最后一个预留到达屏障的线程
           boolean ranAction = false;
           try {
               final Runnable command = barrierCommand;
               /**执行barrierCommand指令*/
               if (command != null)
                   command.run();
               ranAction = true;
               /**执行下一个Generation*/
               nextGeneration();
               return 0;
           } finally {
               /**如果barrierCommand执行失败，进行屏障破坏处理*/
               if (!ranAction)
                   breakBarrier();
           }
       }

        // 如果当前线程不是最后一个到达的线程
        for (;;) {
            try {
                if (!timed)///调用Condition的await()方法阻塞
                    trip.await();
                else if (nanos > 0L)///调用Condition的awaitNanos()方法阻塞
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                /**如果当前线程被中断，则判断是否有其他线程已经使屏障破坏。若没有则进行屏障破坏处理，并抛出异常；否则再次中断当前线程*/
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();

            if (g != generation)
                return index;

            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}

private void nextGeneration() {
    // signal completion of last generation
    trip.signalAll();
    // set up next generation
    count = parties;
    generation = new Generation();
}
```
可以看到，核心的思想就是先判断当前执行的线程是否到达了最后一个屏障，如果到达最后一个屏障：“判断barrierCommand是否为空，不为空执行barrierCommand任务，接着执行nextGeneration方法。在nextGeneration方法中通过Condition的signalAll唤醒其它阻塞的线程开始继续执行。”

#### 3 总结
通过上面的源码分析，我们也可以得知为什么屏障打开有个人没有上车。假定有n个线程，当执行到n-1个时，这n-1个都通过Condition的wait方法进行了等待。当执行到最后一个线程n时，则通过Condition的signalAll唤醒其它阻塞的线程继续执行，同时最后一个线程并没有执行wait方法，所以也顺利执行。但是>n的线程则会执行了wait方法，最后没有线程唤醒，所以无法上车。