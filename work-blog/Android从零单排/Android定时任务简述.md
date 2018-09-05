### Android 定时器
在 Android 开发中倒计时、定时执行任务类的小功能点很普遍，Android 中实现可以有以下几种：
- Handler + Message 的消息组合
- Timer + TimerTask
- ScheduledExecutorService
- AlarmManager

#### 1. Handler + Message 消息组合
核心就是利用 Handler 进行循环发消息。
```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.tv_main).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                mHandler.sendMessageDelayed(Message.obtain(), 1000);
            }
        });
    }

    private static Handler mHandler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            Log.d("Time",System.currentTimeMillis() + "");
            mHandler.sendMessageDelayed(Message.obtain(), 1000);
        }
    };
}
```

#### 2. Timer + TimerTask
Timer 是一个定时工具类，可以实现任务执行一次或以一定的时间间隔重复执行。Timer 使用一个单线程来执行所有的 Tasks，所以它要求一个任务的执行周期要短，防止阻塞其它任务执行。当任务执行完的时候，Timer 的任务执行线程会 terminate，等待垃圾回收。如果一个 Timer 在创建的时候设置守护线程(daemon thread)为真，则任务执行线程执行完任务的时候就不会 terminate。Timer 任务的特点：
- 如果 Timer 的任务线程被意外关闭，会导致该 Timer 的 schedule 任务返回 IllegalStateException 异常。
- Timer 内部是线程安全的。
- Timer 定时任务不是采用真是的时间戳，而是通过 Object.wait(long) 来实现定时任务。

在 JDK1.5 中的并发包（java.util.concurrent）下 ScheduledThreadPoolExecutor 也可以实现定时任务，它的效率更高，创建 ScheduledThreadPoolExecutor 的时候执行线程个数为 1，效果就和 Timer 是相同的。

##### 2.1 基本方法
- schedule(TimerTask task, long delay)：在 delay 毫秒后执行任务一次 ，以 System.currentTimeMillis() 作为参照。
- schedule(TimerTask task, Date time)：在 time 后执行定时任务，如果 time 日期已经过去，则立即执行。
- schedule(TimerTask task, long delay, long period)：在 delay 毫秒后执行任务，并且在以**上个任务执行成功后**的 period 毫秒重复执行。
- schedule(TimerTask task, Date firstTime, long period) ：在 time 后执行定时任务，并且在以上个任务执行成功后的 period 毫秒重复执行。
- scheduleAtFixedRate(TimerTask task, long delay, long period):在 delay 毫秒后执行任务，以 period 为固定周期时间，按照一定频率来重复执行任务。

schedule 方法和 scheduleAtFixedRate 方法的区别:
- schedule 方法：“fixed-delay”；如果第一次执行时间被 delay了，随后的执行时间按照**上一次 实际执行完成的时间点**进行计算。
- scheduleAtFixedRate 方法：“fixed-rate”；如果第一次执行时间被 delay 了，随后的执行时间按照 上一次开始的 时间点 进行计算，并且为了”catch up”会多次执行任务,TimerTask 中的执行体需要考虑同步。

##### 2.2 简要源码分析
**基本结构**
```java
public class Timer {
    /**
     * The timer task queue.  This data structure is shared with the timer
     * thread.  The timer produces tasks, via its various schedule calls,
     * and the timer thread consumes, executing timer tasks as appropriate,
     * and removing them from the queue when they're obsolete.
     */
    private final TaskQueue queue = new TaskQueue();

    /**
     * The timer thread.
     */
    private final TimerThread thread = new TimerThread(queue);

}
```
Timer 类中 TaskQueue 进行 task 的存储，TimerThread 进行任务的处理。

**TaskQueue**
```java
class TaskQueue {
    /**
     * Priority queue represented as a balanced binary heap: the two children
     * of queue[n] are queue[2*n] and queue[2*n+1].  The priority queue is
     * ordered on the nextExecutionTime field: The TimerTask with the lowest
     * nextExecutionTime is in queue[1] (assuming the queue is nonempty).  For
     * each node n in the heap, and each descendant of n, d,
     * n.nextExecutionTime <= d.nextExecutionTime.
     */
    private TimerTask[] queue = new TimerTask[128];
}
```
TaskQueue 是基于一个二叉堆进行实现的。
> 二叉堆是一种特殊的堆，二叉堆是完全二叉树或者是近似完全二叉树。二叉堆满足堆特性：父节点的键值总是保持固定的序关系于任何一个子节点的键值，且每个节点的左子树和右子树都是一个二叉堆。 当父节点的键值总是大于或等于任何一个子节点的键值时为最大堆。
[二叉堆-维基百科](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%8F%89%E5%A0%86)
[二叉堆介绍-英语版](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Binary%20Heaps/heaps.html)

**schedule**
```java
public void schedule(TimerTask task, long delay, long period) {
    if (delay < 0)
        throw new IllegalArgumentException("Negative delay.");
    if (period <= 0)
        throw new IllegalArgumentException("Non-positive period.");
    sched(task, System.currentTimeMillis()+delay, -period);
}

private void sched(TimerTask task, long time, long period) {
    if (time < 0)
        throw new IllegalArgumentException("Illegal execution time.");

    // Constrain value of period sufficiently to prevent numeric
    // overflow while still being effectively infinitely large.
    if (Math.abs(period) > (Long.MAX_VALUE >> 1))
        period >>= 1;

    synchronized(queue) {
        if (!thread.newTasksMayBeScheduled)
            throw new IllegalStateException("Timer already cancelled.");

        synchronized(task.lock) {
            if (task.state != TimerTask.VIRGIN)
                throw new IllegalStateException(
                    "Task already scheduled or cancelled");
            task.nextExecutionTime = time;
            task.period = period;
            task.state = TimerTask.SCHEDULED;
        }

        queue.add(task);
        if (queue.getMin() == task)
            queue.notify();
    }
}
```
Timer 中的所有 schedule 方法本质的实现都是 sched 方法，首先判断 TimerTask 的状态是否是 VIRGIN 状态，TimerTask 的状态有四种：
- VIRGIN：任务未被 schedule，一般都是一个新的任务
- SCHEDULED：任务已经 scheduled 进行执行（未执行），如果是一个非重复执行的任务，代表任务还未被执行
- EXECUTED：已经执行过或正在执行
- CANCELLED：任务被 cancel

如果是一个新任务，则进行赋值关键字段值，然后入队列。最后由 TimerThread 进行处理。

##### 3. ScheduledThreadPoolExecutor
在 JDK1.5 以后，ScheduledThreadPoolExecutor 针对 Timer 的缺陷而设计，在多任务执行情况下，更适用。

```java
public static void main(String[] args) throws IOException{

    ScheduledExecutorService service = Executors.newScheduledThreadPool(1);
    service.scheduleAtFixedRate(new Runnable() {

        @Override
        public void run() {
            System.out.println("--HelloWorld");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            System.out.println(System.currentTimeMillis() + "--HelloWorld");

        }
    }, 1000, 1000, TimeUnit.MILLISECONDS);
}
```

#### 4. AlarmManager
AlarmManager的使用步骤
1. 获得ALarmManager实例 ALarmManager am=(ALarmManager)getSystemService(ALARM_SERVICE);
2. 定义一个PendingIntent发出广播
3. 调用ALarmManager方法，设置定时或重复提醒
4. 取消提醒：

该函数会将所有跟这个PendingIntent相同的Alarm全部取消，怎么判断两者是否相同，android使用的是intent.filterEquals()，具体就是判断两个PendingIntent的action、data、type、class和category是否完全相同。

```java
Intent intent = new Intent(AlarmTest.this, AlarmActivity.class);
intent.setAction("111111");
// 创建PendingIntent对象
PendingIntent pi = PendingIntent.getActivity(AlarmTest.this, 0, intent, 0);
Calendar c = Calendar.getInstance();
// 根据用户选择时间来设置Calendar对象
System.out.println("hourOfDay = " + hourOfDay);
System.out.println("minute = " + minute);
c.set(Calendar.HOUR, hourOfDay);
c.set(Calendar.MINUTE, minute);
// 设置AlarmManager将在Calendar对应的时间启动指定组件
aManager.set(AlarmManager.RTC_WAKEUP,c.getTimeInMillis(), pi);
```