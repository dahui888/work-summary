#### Android开源工具库

##### RxTools
Android开发人员不得不收集的工具类集合 | 支付宝支付 | 微信支付（统一下单） | 微信分享 | Zip4j压缩（支持分卷压缩与加密） | 一键集成UCrop选择圆形头像 | 一键集成二维码和条形码的扫描与生成 | 常用Dialog | WebView的封装可播放视频 | 仿斗鱼滑动验证码 | Toast封装 | 震动 | GPS | Location定位 | 图片缩放 | Exif 图片添加地理位置信息（经纬度） | 蛛网等级 | 颜色选择器 | 编译运行一下说不定会找到惊喜

**主要内容有：**
**一、自定义控件介绍**
- RxAutoImageView             : ImageView实现自动左右移动效果
- RxBarCode                   : 条形码控件
- RxCaptcha                   : 验证码控件
- RxCardStackView             : 银行卡组叠加效果
- RxCobwebView                : 蛛网等级控件
- RxHeartLayout               : 直播爱心点赞控件
- RxNetSpeedView              : 显示当前网速控件
- RxPopupView                 : 自定义PopupWindow控件
- RxProgressBar               : 自定义进度条
- RxQRCode                    : 二维码控件
- RxRoundProgress             : 实现弧形进度条
- RxRulerWheelView            : 刻度横向滚动控件
- RxRunTextView               : TextView实现跑马灯效果
- RxScaleImageView            : 图片缩放控件
- RxSeatAirplane              : 飞机票选座控件
- RxSeatMovie                 : 电影院选座控件
- RxShineButton               : 点赞按钮
- RxShoppingView              : 商品数量加减控件
- RxSwipeCaptcha              : 滑块验证码控件(仿斗鱼验证码)
- RxTextAutoZoom              : 文字根据布局大小自动缩放效果
- RxTextViewVertical          : 单行文字上下滚动
- RxTextViewVerticalMore      : 多行文字上下滚动
- RxTitle                     : 自定义标题控件
- RxToast                     : Toast的封装

**二、Dialog的封装（RxDialog）**
- skipTools                   : 隐藏头部导航栏状态栏
- setFullScreen               : 文字根据布局大小自动缩放效果
- setFullScreenWidth          : 设置宽度match_parent
- setFullScreenHeight         : 设置高度为match_parent
- setOnWhole                  : 设置成全局Dialog

**三、Activity的封装**
- ActivityBase                : 封装了FragmentActivity与Context成员参数
- ActivityBaseLocation        : 封装了定位操作的Activity
- ActivityCodeTool            : 封装了生成二维码与条形码的Activity
- ActivityScanerCode          : 封装了扫描二维码与条形码的Activity
- ActivityWebView             : 封装了可播放视频、获取网页标题并可根据内容自动适应大小WebView的Activity

##### 功能模块介绍
**常用功能 -> RxTool.java**
- hideKeyboard                : 点击隐藏软键盘
- countDown                   : 倒计时(获取验证码倒计时)
- showToast                   : 封装了Toast的方法
- fixListViewHeight           : 手动计算出listView的高度，但是不再具有滚动效果
- createQRImage               : 生成二维码
- drawLinecode                : 生成条形码
- Md5                         : 生成MD5加密32位字符串
- delayToDo                   : 延时操作
- isFastClick                 : 是否快速点击
- setEdTwoDecimal             : EditText 首位小数点自动加零，最多两位小数
- setEditNumberPrefix         : EditText 前缀自动补零

**Activity相关 ->RxActivityTool**
- addActivity                 : 添加Activity 到栈
- currentActivity             : 获取当前的Activity（堆栈中最后一个压入的)
- finishActivity              : 结束当前Activity（堆栈中最后一个压入的）
- finishAllActivity           : 结束所有的Activity
- AppExit                     : 退出当前APP
- getActivityStack            : 获取Activity栈
- 
- 单个Activity操作
- isExistActivity             : 判断是否存在指定Activity
- launchActivity              : 打开指定的Activity
- skipActivity                : 跳转到指定Activity
- skipActivityAndFinish       : 跳转到指定Activity并关闭当前Activity
- skipActivityAndFinishAll    : 跳转后Finish之前所有的Activity
- skipActivityForResult       : activityForResult封装
- getLauncherActivity         : 获取launcher activity

**动画相关 ->RxAnimationTool**
- animationColorGradient      : 颜色渐变动画
- cardFilpAnimation           : 卡片翻转动画
- zoomIn                      : 缩小动画
- zoomOut                     : 放大动画

**应用相关 ->RxAppTool**
- InstallAPK                  : 安装APK
- installApp                  : 安装App（支持7.0）
- installAppSilent            : 静默安装App
- uninstallApp                : 卸载App
- uninstallAppSilent          : 静默卸载App
- isAppRoot                   : 判断App是否有root权限
- launchApp                   : 打开App
- getAppPackageName           : 获取App包名
- getAppDetailsSettings       : 获取App具体设置
- getAppName                  : 获取App名称
- getAppIcon                  : 获取App图标
- getAppPath                  : 获取App路径
- getAppVersionName           : 获取App版本号
- getAppVersionCode           : 获取App版本码
- isSystemApp                 : 判断App是否是系统应用
- isAppDebug                  : 判断App是否是Debug版本
- getAppSignature             : 获取App签名
- getAppSignatureSHA1         : 获取应用签名的的SHA1值
- isInstallApp                : 判断App是否安装
- getAppInfo                  : 获取当前App信息
- getBean                     : 得到AppInfo的Bean
- getAllAppsInfo              : 获取所有已安装App信息
- isAppBackground             : 判断当前App处于前台还是后台

**状态栏相关 -> RxBarTool.java**
- setTransparentStatusBar     : 设置透明状态栏(api大于19方可使用)
- hideStatusBar               : 隐藏状态栏
- noTitle                     : 隐藏Title
- FLAG_FULLSCREEN             : 设置全屏
- getStatusBarHeight          : 获取状态栏高度
- isStatusBarExists           : 判断状态栏是否存在
- getActionBarHeight          : 获取ActionBar高度
- showNotificationBar         : 显示通知栏
- hideNotificationBar         : 隐藏通知栏
- invokePanels                : 反射唤醒通知栏

**剪贴板相关 -> RxClipboardTool.java**
- copyText                    : 复制文本到剪贴板
- getText                     : 获取剪贴板的文本
- copyUri                     : 复制uri到剪贴板
- getUri                      : 获取剪贴板的uri
- copyIntent                  : 复制意图到剪贴板
- getIntent                   : 获取剪贴板的意图

还有很多模块，非常全，这里我就不一一列举了。文章末尾有地址。

看下一些效果图：
**RxPhotoTool操作UCrop裁剪图片**
![RxPhoto](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94Android%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/RxPhotoTool%E6%93%8D%E4%BD%9CUCrop%E8%A3%81%E5%89%AA%E5%9B%BE%E7%89%87.jpg)

**二维码与条形码的扫描与生成**
![二维码与条形码的扫描与生成](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94Android%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/%E4%BA%8C%E7%BB%B4%E7%A0%81%E4%B8%8E%E6%9D%A1%E5%BD%A2%E7%A0%81%E7%9A%84%E6%89%AB%E6%8F%8F%E4%B8%8E%E7%94%9F%E6%88%90.jpg)

![第二章](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94Android%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/%E4%BA%8C%E7%BB%B4%E7%A0%81%E4%B8%8E%E6%9D%A1%E5%BD%A2%E7%A0%81%E7%9A%84%E6%89%AB%E6%8F%8F%E4%B8%8E%E7%94%9F%E6%88%901.jpg)

**常用的Dialog展示**

![dialog1](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94Android%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/%E5%B8%B8%E7%94%A8%E7%9A%84Dialog%E5%B1%95%E7%A4%BA.png)

![dialog2](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94Android%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/%E5%B8%B8%E7%94%A8%E7%9A%84Dialog%E5%B1%95%E7%A4%BA2.png)

![dialog3](https://github.com/dengshiwei/work-summary/blob/master/work-blog/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE/%E5%BC%80%E6%BA%90%E9%A1%B9%E7%9B%AE%E2%80%94%E2%80%94Android%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%BA%93/%E5%B8%B8%E7%94%A8%E7%9A%84Dialog%E5%B1%95%E7%A4%BA3.png)