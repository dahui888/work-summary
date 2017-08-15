### Android应用性能优化——优化UI体验

UI可谓是一个应用的门面，精美的UI设计可以使我们的应用更加炫酷，更能抓住用户的眼球。所以在软件交互设计过程中，设计师都会绞尽脑汁去设计一个动画、UI布局。而为了实现这些效果，我们会通过布局嵌套、自定义View、动画等方式来满足设计师的需要。由此就要求我们对UI的布局设计等进行优化，提高UI效率。

#### 一、Android的绘制渲染机制
在我们实际开发App的过程中，我们都只到如果一个页面设计过于复杂，嵌套的控件太多会造成页面滑动起来比较卡顿的情况，那么Android的绘制视图UI的流程是怎么样的呢？为什么会造成卡顿呢？

**绘制渲染流程**
UI对象—->CPU处理为多维图形,纹理 —–通过OpeGL ES接口调用GPU—-> GPU对图进行光栅化(Frame Rate ) —->硬件时钟(Refresh Rate)—-VSYNC垂直同步—->投射到屏幕。

在Android的绘制过程中，Android系统每隔16ms重新发起一次VSYNC消息，触发UI绘制。即我们在这个UI的渲染周期是16ms，如下图示例：

![VSYNC](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/VSYNC.png)

如果我们的一个CPU和GPU的处理周期大于16ms会出现什么情况呢？即当系统发起一次VSYNC消息时，CPU和GPU还在进行相关的计算工作，这个时候，我们的UI展示就会出现卡顿现象。

![OverVSYNC](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/OverVSYNC.png)

通过上面我们知道了造成UI卡顿的原因是由于在CPU和GPU渲染过程超过16ms导致的，渲染的过程是由CPU和GPU共同协作完成的，影响CPU性能的主要因素是layouts、invalidations、Overdraw。

![cpu-gpu](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/CPU-GPU.png)

那么我们知道原因了，就可以利用Android提供的工具来帮助我们进行UI的性能分析优化：
- Hierarchy Viewer去检测UI布局结构，去除不必要的嵌套，分析UI的layout的时间。
- Lint：通过Lint工具检测无用的资源、布局结构。
- Show GPU Overdraw去检测Overdraw，最终可以通过移除不必要的背景以及使用canvas.clipRect、QuickReject解决大多数问题。

#### 二、优化布局结构-Hierarchy Viewer
Hierarchy Viewer是Android系统给我们提供的UI布局结构分析神器。工具所在目录在：android-sdk-windows\tools\hierarchyviewer.bat。或者可以使用monitor.bat工具打开，目录在android-sdk-windows\tools\monitor.bat。
打开后如下展示：

![hierarchyviewer](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/hierarchyviewer.png)

通过HierarchyViewer可以展示一个Activity中的布局结构，展示了所有的View树。
- Save as PNG用来把左侧树保存为一张图片
- Capture Layers用来保存psd的PhotoShop分层素材
- Load View Hierarchy用来手动刷新变化
- Display View选中的View在单独视图中展示
- Request Layout用来发起layout
- Invalidate Layout刷新布局
- Profile Node：获取选中node的layout的时间。

这里我们点击Profile Node计算出时间，单独拿出来一个Node进行分析：
![content](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/Content.png)

在上面单个Node中，我们可以看到View的measure、layout、draw的时间。上图最底那三个彩色原点代表了当前View的性能指标，从左到右依次代表测量、布局、绘制的渲染时间，红色和黄色的点代表速度渲染较慢的View（当然了，有些时候较慢不代表有问题，譬如ViewGroup子节点越多、结构越复杂，性能就越差）。


对于Android的UI来说，invalidate和requestLayout是最重要的过程，Hierarchyviewer提供了帮助我们Debug特定的UI执行invalidate和requestLayout过程的途径，方法很简单，只要选择希望执行这两种操作的View点击按钮就可以。当然，我们需要在例如onMeasure()这样的方法中打上断点。这个功能对于UI组件是自定义的非常有用，可以帮助单独观察相关界面显示逻辑是否正确。