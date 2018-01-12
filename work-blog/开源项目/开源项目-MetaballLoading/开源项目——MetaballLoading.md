### 精美加载动画——MetaballLoading
在手机应用的开发过程中，永远都少不了Loading动画，一个精巧的Loading动画，能舒缓客户的等待情绪，所以如何设计一个Loading动画也显得很重要。今天给大家推荐一个Loading动画——MetaballLoading。

**效果图**
![metabalLoading](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE-MetaballLoading/metaball.gif)

这个Loading动画主要是用到了贝塞尔曲线的知识。通过调整贝塞尔曲线。如下图：
![metaball2](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE-MetaballLoading/metaball2.gif)

当然在实现的过程中会涉及到很多几何变换，这里需要我们自习阅读代码体会用处。
![code]()


原代码地址：
https://github.com/dodola/MetaballLoading/blob/master/app/src/main/java/com/dodola/animview/MetaballView.java