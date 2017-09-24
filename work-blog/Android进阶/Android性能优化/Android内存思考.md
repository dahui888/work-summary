## Android内存的一些思考
>上次组内小组讨论，谈论了一些Android内存的知识，所以会后仔细研究了一下。

### 一、关于Android内存相关的基本概念
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


### 二、top指令
![top](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/top.png)

通过top指令，我们能看到应用与内存相关的VSS、RSS的占用信息，这两个量一般不用来作为内存使用的参考，top指令主要用来考察cpu的使用情况。

### 三、dumpsys meminfo指令

![dumpsys](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/dumpsys%20meminfo.png)

使用dumpsys指令可以查看应用程序的堆栈信息。主要是PSS大小信息。
- dalvik：是指dalvik所使用的内存。就是平常说的java堆，我们平常创建的对象在这里分配。
- native：是被native堆使用的内存。应该指使用C\C++在堆上分配的内存。Bitmap对象就是在这上面直接分配。
- other：是指除dalvik和native使用的内存。但是具体是指什么呢？至少包括在C\C++分配的

针对各个项有如下说明：
**Dalvik Heap**
您的应用中 Dalvik 分配占用的 RAM。Pss Total 包括所有 Zygote 分配（如上述 PSS 定义所述，通过进程之间的共享内存量来衡量）。Private Dirty 数值是仅分配到您应用的堆的实际 RAM，由您自己的分配和任何 Zygote 分配页组成，这些分配页自从 Zygote 派生应用进程以来已被修改。
注：在包含 Dalvik Other 部分的更新的平台版本上，Dalvik 堆的 Pss Total 和 Private Dirty 数值不包括 Dalvik 开销（例如即时 (JIT) 编译和垃圾回收记录），而较旧的版本会在 Dalvik 中将其一并列出。

Heap Alloc 是 Dalvik 和原生堆分配器为您的应用跟踪的内存量。此值大于 Pss Total 和 Private Dirty，因为您的进程从 Zygote 派生，且包含您的进程与所有其他进程共享的分配。

**.so mmap 和 .dex mmap**
映射的 .so（原生）和 .dex（Dalvik 或 ART）代码占用的 RAM。

**.oat mmap**
这是代码映像占用的 RAM 量，根据多个应用通常使用的预加载类计算。此映像在所有应用之间共享，不受特定应用影响。

**.art mmap**
这是堆映像占用的 RAM 量，根据多个应用通常使用的预加载类计算。

**Unknown**
系统无法将其分类到其他更具体的一个项中的任何 RAM 页。当前，此类 RAM 页主要包含原生分配，由于地址空间布局随机化 (ASLR) 而无法在收集此数据时通过工具识别。与 Dalvik 堆相同，Unknown 的 Pss Total 考虑了与 Zygote 的共享，且 Private Dirty 是仅由您的应用占有的未知 RAM。

**TOTAL**
您的进程占用的按比例分配占用内存 (PSS) 总量。等于上方所有 PSS 字段的总和。表示您的进程占用的内存量占整体内存的比重，可以直接与其他进程和可用总 RAM 比较。

**Private Dirty（USS） 和 Private Clean**
Private Dirty 和 Private Clean 是您的进程中的总分配，未与其他进程共享。它们（尤其是 Private Dirty）等于您的进程被破坏后将释放回系统中的 RAM 量。脏 RAM 是因为已被修改而必须保持在 RAM 中的 RAM 页（因为没有交换）；干净 RAM 是已从某个持久性文件（例如正在执行的代码）映射的 RAM 页，如果一段时间不用，可以移出分页。

所以一般我们分配给Android系统的堆大小是16M、24M，是只dalvik+native不能大于这个值。我们可以看到主要的内存占用集中在这个Unknown上。


### 四、补充几个常见的工具类

**Log**
我们可以通过Log.getStackTraceString(new Throwable());追踪一个方法的堆栈调用信息。

**Runtime**
可以通过该类的runtime.totalMemory()方法获取正在运行集成的内存大小。

**Debug**
Debug类为我们提供了很多获取进程内存获取的方法。getMemoryInfo等。