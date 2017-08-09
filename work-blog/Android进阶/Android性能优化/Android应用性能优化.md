### Android应用性能优化

最近在研究Android性能优化的相关知识。也在网上搜索了相关的文章来学习研究，看了郭霖的[Android性能最佳实践系列](http://blog.csdn.net/guolin_blog/article/details/42238627)，郭神的这个系列主要是从内存方面去优化apk。然后陆陆续续也看到别的博客上面的介绍，多数都是介绍了内存方面的相关知识。后面发现了若水写的一篇文章，感觉很不错，介绍的很全面，[ Android应用开发性能优化完全分析](http://blog.csdn.net/yanbober/article/details/48394201)。若水的文章介绍的很详细，从UI布局方面的优化到内存分析优化，还包括常见的注意事项和技巧，总之是非常值得仔细阅读的一篇好文章。

既然这些大神们都已经总结了那么多，为什么还要写呢？直接拿来学习不就行了。首先谈谈我想规划的应能优化的专题，第一方面是从UI层面上进行优化，第二方面是从代码方面优化。通过两个方面着手去进行。这里若水的文章从这两方面都进行了详细的介绍，貌似我又没有要写的理由了。确实，关于Android性能优化的文章太多太多，但是我认为每个人都应该构建自己的只是网络图，所以我准备按照我的理解，分篇幅在唠叨一遍。

#### 一、整体介绍
在进行之前，我们先进行Eclipse上面的工具介绍，后续如果有可能在针对Android Studio上的进行补充。在Eclipse+ADT工具中，Android给我们提供了丰富的工具来帮助我们了解分析apk，这些工具都存在DDMS视图中。

![global](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/global.png)

![updateHeap](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Update%20Heap.png)：用来追踪一个进程使用的内存大小。选中要监听的进程，点击该图标。可以配合Cause GC使用。

![dump hpreof file](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Dump%20HPROF%20%20file.png)：将堆信息生成Hpreof文件，可以通过MAT工具进行分析内存。一般用于查看内存泄漏。

![cause gc](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Cause%20GC.png)：主动发起gc操作，用于观察进程的堆栈信息占用。

![updateThread](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Update%20Threads.png)：跟踪进程的线程信息。选中进程后，点击该图标，会将进程的相关线程信息显示在【Threads】列表项。

![startMethodProfiling](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/StartMethodProfiling.png)：方法分析工具用于分析方法的执行时间、cpu等信息。在Android2.1之前的系统，要求机器具有sdcard并且具有读写权限保存trace文件。在2.2以后的机器上就直接在调试机器上展示了。

![Hierarchy](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Hierarchy.png)：查看UI的布局结构信息。

![systrace](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/systrace.png)：Systrace是Android4.1中新增的性能数据采样和分析工具。它可帮助开发者收集Android关键子系统（如surfaceflinger、WindowManagerService等Framework部分关键模块、服务，View系统等）的运行信息，从而帮助开发者更直观的分析系统瓶颈，改进性能。

![Threads](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Threads.png)：展示进程的Threads信息。

![Heap](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Heap.png)：展示进程的堆信息。


![AllocationTracker](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Allocation%20%20Tracker.png)：用来追踪每个对象Object申请的内存大小。首先切换到Allocation Tracker页，选中要追踪的进程，然后点击Start Tracking，接着在我们应用程序中所做的操作的内存变化就会被追踪记录。

#### 二、基本概念补充



#### 三、基本工具简介

##### 1、MAT的安装
打开Eclipse软件，找到Help下的Install，输入一下网址：
 http://archive.eclipse.org/mat/1.3/update-site/


##### 2、