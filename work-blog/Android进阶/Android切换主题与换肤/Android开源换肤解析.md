## Android开源换肤解析
在上一篇[Android小结](http://blog.csdn.net/mr_dsw/article/details/52988298)博客中我们简要的介绍了常见的换肤手法，换肤可以贯穿在整个Android开发中，所以必然可以做成一个框架集的形式。下面我们就来看看一些Android换肤开源组件。

#### 一、Colorful基于Theme的换肤
[Colorful](https://github.com/bboyfeiyu/Colorful。)是一个基于Theme实现的开源控件，作者是mr_simple。Colorful无需重启Activity、无需自定义View，方便的实现日间、夜间模式。在上篇文章中，我们介绍了基于Theme的切换都需要重新OnCreate Activity。但是Colorful却不需要重启Activity，下面就让我们一睹风采。

