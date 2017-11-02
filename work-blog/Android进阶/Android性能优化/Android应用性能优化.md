### Android应用性能优化

最近在研究Android性能优化的相关知识。也在网上搜索了相关的文章来学习研究，看了郭霖的[Android性能最佳实践系列](http://blog.csdn.net/guolin_blog/article/details/42238627)，郭神的这个系列主要是从内存方面去优化apk。然后陆陆续续也看到别的博客上面的介绍，多数都是介绍了内存方面的相关知识。后面发现了若水写的一篇文章，感觉很不错，介绍的很全面，[ Android应用开发性能优化完全分析](http://blog.csdn.net/yanbober/article/details/48394201)。若水的文章介绍的很详细，从UI布局方面的优化到内存分析优化，还包括常见的注意事项和技巧，总之是非常值得仔细阅读的一篇好文章。

既然这些大神们都已经总结了那么多，为什么还要写呢？直接拿来学习不就行了。首先谈谈我想规划的应能优化的专题，第一方面是从UI层面上进行优化，第二方面是从代码方面优化。通过两个方面着手去进行。这里若水的文章从这两方面都进行了详细的介绍，貌似我又没有要写的理由了。确实，关于Android性能优化的文章太多太多，但是我认为每个人都应该构建自己的只是网络图，所以我准备按照我的理解，分篇幅在唠叨一遍。

#### 一、整体介绍
在进行之前，我们先进行Eclipse上面的工具介绍，后续如果有可能在针对Android Studio上的进行补充。在Eclipse+ADT工具中，Android给我们提供了丰富的工具来帮助我们了解分析apk，这些工具都存在DDMS视图中。

![global](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/global.png)

![updateHeap](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Update%20Heap.png)：用来追踪一个进程使用的内存大小。选中要监听的进程，点击该图标。可以配合Cause GC使用。

![dump hpreof file](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Dump%20HPROF%20%20file.png)：将堆信息生成Hpreof文件，可以通过MAT工具进行分析内存。一般用于查看内存泄漏。

![cause gc](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Cause%20GC.png)：主动发起gc操作，用于观察进程的堆栈信息占用。即每次gc都会更新Heap信息。

![updateThread](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Update%20Threads.png)：跟踪进程的线程信息。选中进程后，点击该图标，会将进程的相关线程信息显示在【Threads】列表项。

![startMethodProfiling](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/StartMethodProfiling.png)：方法分析工具用于分析方法的执行时间、cpu等信息。在Android2.1之前的系统，要求机器具有sdcard并且具有读写权限保存trace文件。在2.2以后的机器上就直接在调试机器上展示了。

![Hierarchy](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Hierarchy.png)：查看Activity的布局结构信息。

![systrace](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/systrace.png)：Systrace是Android4.1中新增的性能数据采样和分析工具。它可帮助开发者收集Android关键子系统（如surfaceflinger、WindowManagerService等Framework部分关键模块、服务，View系统等）的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。

![Threads](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Threads.png)：展示进程的Threads信息。包含线程的id、name、statue等信息。

![Heap](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Heap.png)：展示进程的堆信息。


![AllocationTracker](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Allocation%20%20Tracker.png)：用来追踪每个对象Object申请的内存大小。首先切换到Allocation Tracker页，选中要追踪的进程，然后点击Start Tracking，接着在我们应用程序中所做的操作的内存变化就会被追踪记录。

#### 二、基本概念补充
1、RAM（random access memory）随机存取存储器，说白了就是内存
- 寄存器（Registers）：速度最快的存储场所，因为寄存器位于处理器内部，我们在程序中无法控制
- 栈（Stack）：存放基本类型的数据和对象的引用，但对象本身不存放在栈中，而是存放在堆中
- 堆（Heap）：堆内存用来存放由new创建的对象和数组。在堆中分配的内存，由Java虚拟机的自动垃圾回收器（GC）来管理。
- 静态域（static field）：  静态存储区域就是指在固定的位置存放应用程序运行时一直存在的数据，Java在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量
- 常量池（constant pool）：虚拟机必须为每个被装载的类型维护一个常量池。常量池就是该类型所用到常量的一个有序集和，包括直接常量（string,integer和floating point常量）和对其他类型，字段和方法的符号引用。
- 非RAM存储：硬盘等永久存储空间


**2、内存耗用名词解析**：

- VSS - Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
- RSS - Resident Set Size 实际使用物理内存（包含共享库占用的内存），表示一个进程实际在RAM中占用的内存。
- PSS - Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存），当系统中所有进程的PSS相加起来就是系统已用的全部内存。
- USS - Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存），这个值标志着一个进程所占用的时机内存大小，如果进程被杀死，则返还给系统的内存大小就是USS的大小。我们通常使用USS来观察内存泄漏。

一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS。比如进程中使用了一个被3个进程使用的9k大小的共享库，RSS的大小会统计到9K，而PSS会统计3K的大小。

这里在补充一个概念，共享库指的什么？这里需要涉及到库的分类，将库分为三种类型：静态库,共享库，动态加载库。

- 静态库实际就是一些目标文件(一般以.o结尾)的集合，静态库一般以.a结尾，只用于链接生成可执行文件阶段。具体来说，以c程序为例，一般我们编译程序源代码的时候，过程大致是这样的：以.c为后缀的源文件经过编译生成.o的目标文件，以.o为后缀的目标文件经过链接生成最终可执行的文件。我们可以在链接的时候直接链接.o的目标文件，也可以将这些.o目标文件打包集中起来，统一链接，而这个打包生成的集中了所有.o文件的文件，就是静态库。
- 共享库以.so结尾. (so == share object) 在程序链接的时候并不像静态库那样从库中拷贝使用的函数代码到生成的可执行文件中，而只是作些标记，然后在程序开始启动运行的时候，动态地加载所需库（模块）。所以，应用程序在运行的时候仍然需要共享库的支持。共享库链接出来的可执行文件比静态库链接出来的要小得多，运行多个程序时占用内存空间比也比静态库方式链接少(因为内存中只有一份共享库代码的拷贝)，但是由于有一个动态加载的过程所以速度稍慢。
- 动态加载库(dynamically loaded (DL) libraries)是指在程序运行过程中可以加载的函数库。而不是像共享库一样在程序启动的时候加载。DL对于实现插件和模块非常有用，因为他们可以让程序在允许时等待插件的加载。在Linux中，动态库的文件格式跟共享库没有区别，主要区别在于共享库是程序启动时加载，而动态加载库是运行的过程中加载。

**3、垃圾回收器缺点**：

1).占用资源时间：
Java虚拟机必须追踪运行程序中有用的对象, 而且最终释放没用的对象。这一个过程需要花费处理器的时间。

2).不可预知:
垃圾收集器线程虽然是作为低优先级的线程运行，但在系统可用内存量过低的时候，它可能会突发地执行来挽救内存资源。当然其执行与否也是不可预知的。 

3).不确定性：
不能保证一个无用的对象一定会被垃圾收集器收集，也不能保证垃圾收集器在一段Java语言代码中一定会执行。
同样也没有办法预知在一组均符合垃圾收集器收集标准的对象中，哪一个会被首先收集。 

4).不可操作 
垃圾收集器不可以被强制执行，但程序员可以通过调用System. gc方法来建议执行垃圾收集器。

**4、CPU**
CPU: 中央处理器,它集成了运算,缓冲,控制等单元,包括绘图功能.CPU将对象处理为多维图形,纹理(Bitmaps、Drawables等都是一起打包到统一的纹理).

**5、GPU**
GPU:一个类似于CPU的专门用来处理Graphics的处理器, 作用用来帮助加快格栅化操作,当然,也有相应的缓存数据(例如缓存已经光栅化过的bitmap等)机制。

**6、OpenGL ES**
OpenGL ES是手持嵌入式设备的3DAPI,跨平台的、功能完善的2D和3D图形应用程序接口API,有一套固定渲染管线流程. 附相关OpenGL渲染流程资料

**7、DisplayList**
DisplayList 在Android把XML布局文件转换成GPU能够识别并绘制的对象。这个操作是在DisplayList的帮助下完成的。DisplayList持有所有将要交给GPU绘制到屏幕上的数据信息。

**8、垂直同步VSYNC**
垂直同步VSYNC:让显卡的运算和显示器刷新率一致以稳定输出的画面质量。它告知GPU在载入新帧之前，要等待屏幕绘制完成前一帧。下面的三张图分别是GPU和硬件同步所发生的情况,Refresh Rate:屏幕一秒内刷新屏幕的次数,由硬件决定,例如60Hz.而Frame Rate:GPU一秒绘制操作的帧数,单位是30fps

#### 三、基本工具简介

##### 1、MAT的安装
打开Eclipse软件，找到Help下的Install，输入一下网址：
 http://archive.eclipse.org/mat/1.3/update-site/

安装MAT后，我们就可以直接在Eclipse中对Dump HPROF file进行直接的分析。

##### 2、hierarchyviewer视图层级工具
hierarchyviewer.bat工具是Android sdk中提供的一个分析查看Android布局视图层级的工具。工具存在目录：
android-sdk-windows\tools\hierarchyviewer.bat

##### 3、LeakCanary
LeakCanary是一款优秀的检测jave和Android内存泄漏的工具。

#### 四、性能优化博客推荐
1、[LooperJing-Android性能优化的方方面面](http://www.jianshu.com/p/b3b09fa29f65)