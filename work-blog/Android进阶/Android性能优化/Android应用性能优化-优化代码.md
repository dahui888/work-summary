### Android应用性能优化——优化UI体验

在上面的一篇介绍中，我么知道可以通过优化布局结构、使用include、merge标签来优化布局结构。在这篇笔记中，我们将介绍下代码层面的分析优化。在第一篇概括中，我们介绍了DDMS中给我们提供的常用工具。在这里我们将在介绍代码层面的优化工具。

#### 一、Allocation Tracker内存分配追踪
Allocation Tracker帮助我们进行操作的内存分配追踪，通过这个工具可以查看到哪个对象、哪个类的哪行代码以及哪个线程调用的这个内存分配操作。

![eclipseTracker]()

在Eclipse模式下的DDMS提供的AllocationTracker只能根据申请的内存对象进行查看，选中对应的记录可以在下面的对话框中追踪到对应的代码片段。

![](Allocation)

接下来我们看看Android Studio中提供的内存追踪工具。在Android Studio中集成了Monitors视图，该视图中提供了很多实用检测工具，有Memory、CPU、Network、GPU的参数直观检测。而AllocationTracker工具就在Memory视图中。

![StudioTool]()

然后我们点击AllocationTracker按钮，开始检测追踪。

![StudioStart]()

同时在上面展示出检测结果图：

![Android Studio]()

在上面的结果中，我们可以看到AndroidStudio给我们提供了更加人性化的工具，首先我们可以对结果进行分类：根据Method名称或者Allocator（内存分配器）进行分类。同时也可以通过形状图来直观观察。

**Group by Method**

![method分类]()

Method分类通过方法名进行分类。我们可以看到不同方法占用内存的大小不一，我们主要是分析占用比例比较大的内存。我们展开dispatchTransformedTouchEvent方法，可以看到自定义View的onTouchEvent所占的比例。

**Group by Allocator**

![byallocator]()

通过Allocator进行分类，我们可以更直接快速定位到我们的代码中，比如上面我们可以看到我们ScrollLinearLayout的内存使用情况。同时我们邮件选中类，可以通过Jump To Source跳转到源代码。

**饼状图**

![circle]()

饼状图是以圆心为起点，最外层是其内存实际分配的对象，每一个同心圆可能被分割成多个部分，代表了其不同的子孙，每一个同心圆代表他的一个后代，每个分割的部分代表了某一带人有多人，你双击某个同心圆中某个分割的部分，会变成以你点击的那一代为圆心再向外展开。如果想回到原始状态，双击圆心就可以了。

**柱状图**

![layout]()

柱状图以左边为起始点，从左到右的顺序是某个的堆栈信息顺序，纵坐标上的宽度是以其Count/Size的大小决定的。柱状图的内容其实和轮胎图没什么特别的地方。



#### 二、TraceView追踪运行时间
通过上面的方法，可以帮助我们查看各个部分的占用资源大小，那么占用内存大一般时间久比较长，占用的cpu资源就比较高。这里我们通过Start Method Profiling工具进行追踪。使用步骤：点击选中Start Method Profiling工具，然后操作我们需要监控的操作即可。

如下图，我们点击Start Method Profiling采集的追踪数据。

![eclipsemethodprofiling]()

在上图中，显示了每个方法的执行时间。

| 属性吗 | 含义 |
|--------|--------|
|   name  |   线程中调运的方法名 |
|   Incl CPU Time  |   当前方法（包含内部调运的子方法）执行占用的CPU时间 |
|   Excl CPU Time  |   当前方法（不包含内部调运的子方法）执行占用的CPU时间 |
|   Incl Real Time  |   当前方法（包含内部调运的子方法）执行的真实时间，ms单位 |
|   Excl Real Time  |   当前方法（不包含内部调运的子方法）执行的真实时间，ms单位 |
|   Calls+Recur Calls/Total  |   当前方法被调运的次数及递归调运占总调运次数百分比 |
|   CPU Time/Call |   当前方法调运CPU时间与调运次数比，即当前方法平均执行CPU耗时时间 |
|   Real Time/Call  |   当前方法调运真实时间与调运次数比，即当前方法平均执行真实耗时时间 |

这里影响性能的有以下两种因素：
- 方法调运一次需要耗费很长时间导致卡顿；
- 方法调运一次耗时不长，但被频繁调运导致累计时长卡顿。

Android Studio中，工具名称是“Start Method tracing”.

![methodtracing]()

点击之后就可以监听我们的操作了。

![studiotrading]()


#### 三、trace.txt分析ANR异常
**ANR(Application Not Responding)定义**

在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应用程序无响应（ANR：Application Not Responding）对话框。用户可以选择“等待”而让程序继续运行，也可以选择“强制关闭”。所以一个流畅的合理的应用程序中不能出现anr，而让用户每次都要处理这个对话框。因此，在程序里对响应性能的设计很重要，这样系统不会显示ANR给用户。
 
**出现ANR的原因**

默认情况下，在android中Activity的最长执行时间是5秒，BroadcastReceiver的最长执行时间则是10秒。超出就会提示应用程序无响应（ANR：Application Not Responding）对话框。概括起来有以下三种类型：

1. KeyDispatchTimeout(5 seconds) --主要类型

	按键或触摸事件在特定时间内无响应

2. BroadcastTimeout(10 seconds)

	BroadcastReceiver在特定时间内无法处理完成

3. ServiceTimeout(20 seconds) --小概率类型

	Service在特定的时间内无法处理完成

出现Application Not Responding的提示后，系统会将日志LOG写到到data\anr\traces.txt文件

![tracetxt]()

上图中，我们可以看到出现ANR的位置就在

	at com.iflytek.speech.libisssr.endAudioData(Native Method)


