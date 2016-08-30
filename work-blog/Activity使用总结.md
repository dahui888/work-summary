### Activity使用总结

>作为Android入门级别的组件，Activity在Android的开发中承载了太多的东西，我们的项目开发中少不了与Activity打交道。所以需要我们熟练掌握Activity的使用。

总体介绍围绕下面的几个议题进行讨论，以前也[总结过Activity的特点](http://blog.csdn.net/mr_dsw/article/details/48198387)，那时候刚学习Android总结的也过于死板，这次准备集中把Android知识进行梳理一下，所以比较有总结性。

1. Activity的使用。
1. Activity的模式。
1. Activity的通讯
1. Activity的管理。
1. Activity常见问题：
    - 从一个应用打开另一个应用。
    - 完全退出应用。
    - 动画切换

###一、Activity常见使用
在项目的开发中，Activity承载了我们的页面展示，它通过setContentView(int Id)方法绑定显示的布局，然后进行显示。Activity的基本使用。

     protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }	