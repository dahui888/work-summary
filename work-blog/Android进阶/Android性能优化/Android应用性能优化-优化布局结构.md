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

#### 三、使用GPU分析过度绘制
在UI性能分析中，还可以通过GPU来分过度绘制的相关内容。在开发者模式下：
设置 -> 开发者选项 -> 调试GPU过度绘制 -> 显示GPU过度绘制

![gpu_sle](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/gpu.png)

选中后，屏幕上有各种颜色，此时你可以切换到需要检测的程序，对于各个色块，对比一张Overdraw的参考图：

![overdraw](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/overdraw.png)

颜色的表示度意义：
- 红色：4x过度绘制，最严重
- 浅红色：3x过度绘制，比较严重
- 绿色：2x过度绘制，一般
- 蓝色：1x过度绘制
- 无色：WebView渲染区域。

过度绘制时指在同一个像素上进行了多次绘制，绘制的越多越浪费资源。最理想的状态时1x过度绘制，但是在实际中可能不是那么现实，但是要作为一个指标，尽量来保证GPU绘制次数比较少。通过优化布局结构、减少设置多余的背景等方式来减少过度绘制。

我们来看看几个过度绘制的例子：

![gpu_overdraw1](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/gpu_overdraw1.png)
![gpu_overdraw](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/gpu_overdraw.png)

来看一个布局合理的例子：

![gpu_overdraw2](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Android%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/img/gpu_overdraw2.png)


#### 四、使用Lint工具检测
Android给我们提供非常便捷的工具-Lint。用于检测我们普通的常见问题。Lint工具会自动给我们检测出问题。在Eclipse中使用方式如下：
选中项目邮件——>Android Tools——>Run Lint：Check for Common Errors。
如下面我选中的一个项目。
![LintError]()

通过上面的例子，我们可以看到Lint给我们提示出：无用的控件、无用的字符串、不合适的单位。

#### 五、总结
**1、Hierarchy Viewer的使用范围**
出于安全考虑，Hierarchy Viewer只能连接Android开发版手机或是模拟器。好在已经有牛人给我们提供了工具[ViewServer](https://github.com/romainguy/ViewServer)。
**2、降低过度绘制，减少不必要的backgroud背景设置。**
如果一个ParentView设置了背景图，子View也设置了背景图，很容易早晨背景的过度绘制。避免无用的背景绘制。
**3、清除无用的组件，优化布局结构。**
**4、尽量少使用layout_weight属性，它会导致measure进行二次测量，浪费时间。**
**5、清楚无用的View Node，当一个布局没有子节点也没有背景background，可以删除掉。**
**6、降低布局结构层次，尽量使用扁平化布局层次，比如RelativeLayout、FrameLayout。Android中默认的布局层次是10。**
**7、降低过度绘制，通过canvas的clipXX方法来降低过度绘制区域。**
**8、使用< inclue/>、< merge/>标签复用控件。**