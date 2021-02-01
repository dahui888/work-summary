### 【译】作为一个经验丰富的 Android 工程师，你能深入地回答这些问题吗？



作者 [Saumye Srivastava](https://saumyesrivastava.medium.com/how-in-depth-can-you-answer-these-as-an-android-engineer-f61a7c2d736c) 在 `Flipkart` 做过面试官，面试过优步、微软和谷歌等公司以及一些初创公司之后，他为 `Android` 的面试者准备了一份清单，希望他们能给出答案。

> *“The ability to ask the right questions is more than half the battle of finding the answer”*
>
> *- Thomas Watson*



# **Questions**

1. `Layout Inflater` 是如何将 `XML` 解析成 `View` 的？属性是如何读取以及生效的？以及一个 `View` 在完全展示到屏幕前需要经过哪些步骤？
2. 说一说你对 `findViewById` 的了解？给定 `2` 个 `View` 视图引用，您将如何查找它们的祖先视图？
3. 详细解释下当你点击屏幕时触发了哪些 `listeners`，包括它们执行的先后顺序？以及子布局和父布局之间的事件处理机制。
4. `Thread`、`Handler`、`Looper`、`MessageQueue` 是什么？
5. `Android` 上有哪些不同的并发方法？您能解释一下 `ExecutorService` 与 `CachedThreadPool`、`FixedThreadPool`、`AsyncTasks` 与 `HandlerThreads` 之间的区别吗？
6. `Android` 上的垃圾回收是如何工作的？解释计数、标记和扫描三种垃圾回收算法的含义。
7. 思考并列出应用程序组件和服务之间通信的所有方式？
8. 什么时候使用 `bindService`？`ServiceConnection` 是什么？绑定服务时的内部工作原理。
9. 使用 `Messenger` 和 `AIDL` 有什么区别？使用 `AIDL` 进行进程间通信有什么好处？
10. `In-memory vs Bundle vs SharedPrefs vs Database`，你是如何使用的？
11. 列出所有你能想到的减少 `apk` 大小的方法。
12. `ANR`、掉帧和冻结帧是如何引起的，它们有什么不同？
13. `Proguard/R8` 的 `3` 个步骤是什么，如何启用和禁用它们？
14. 什么是 `consumer proguard`，它与 `proguard` 有何不同？
15. 对于一个经过 Proguard/R8 混淆 apk ，如何能查看到不混淆的堆栈调用日志？
16. `SharedPrefs` 的 `commit` 和 `apply` 方法有什么区别？`SharedPrefs` 与 `Databases` 哪个性能更好？
17. 如何实现一个可被观察的 `SharedPrefs` 或 `Databases` ，比如监听键/表/查询？
18. `Android` 上进程间的数据共享和控制共享有何不同？以及如何实现？
19. `Serializable` 和 `Parcelable` 有什么区别？`Serializable`  的缺点是什么？
20. 在 `apk` 中保护 `api` 密钥有哪些方式？
21. 什么是 `middle attack`？如何保护您的 `api` 不受它和其他攻击的影响？
22. 详细解释 `Android` 打包过程？什么是 `R.java、resources.arsc、manifest merging & signing`？
23. `app bundle` 和 `apk` 有何不同？什么是 `build flavours`？它们与 `product flavours` 有什么不同？
24. `Activity & Activity`, `Fragment & Fragment` 和 `Activity & Fragment` 的通信方式有哪些？
25. `Activity` 的启动模式之间有什么区别？
26. `viewholder` 模式是什么？在 `ListView` 中你是如何实现的？解释下 `RecyclerView` 中实现监听的最佳方式。
27. `RecyclerView` 的性能优化方式有哪些？
28. `RecyclerView` 中如何实现一个 `Sticky header`？
29. 数据集中的更改如何反映在 `RecyclerView` 中？什么是 `DiffUtil`？
30. `Java` 中的反射是什么？为什么建议尽量少用？
31. `Java` 中的泛型是什么？在什么情况下它们有用？您如何限制它们？
32. 什么是 `Android` 中的 `Content Provider`？您如何访问它提供的内容？
33. 在分隔文件的数据集上实现 `Content Provider` 有可能吗？
34. `Android` 上的数据库是如何存储数据的？什么是 `ORM & Data Access Object`？优化数据库读、写和更新的方法有哪些？
35. `SQLite` 的线程安全性如何？表级锁还是数据库级锁？
36. 什么是依赖注入模式（`Dependency Injection Pattern`）？它与服务注册模式（`Service Registry Pattern`）相比如何？
37. `Dagger` 和 `Koin` 是什么？它们内部是如何工作的？
38. 在处理 `webview` 时，安全性有哪些问题。
39.  `unit` 测试和 `instrumentation` 测试的区别是什么? `Mockito vs RoboElectric`？
40. `Android` 下的 `CI/CD` 是什么？在代码合并、测试、发布和推出之前，您经历过哪些流程审核？

