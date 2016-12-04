## Android开源换肤解析
在上一篇[Android小结](http://blog.csdn.net/mr_dsw/article/details/52988298)博客中我们简要的介绍了常见的换肤手法，换肤可以贯穿在整个Android开发中，所以必然可以做成一个框架集的形式。下面我们就来看看一些Android换肤开源组件。

#### 一、Colorful基于Theme的换肤
[Colorful](https://github.com/bboyfeiyu/Colorful。)是一个基于Theme实现的开源控件，作者是mr_simple。Colorful无需重启Activity、无需自定义View，方便的实现日间、夜间模式。在上篇文章中，我们介绍了基于Theme的切换都需要重新OnCreate Activity。但是Colorful却不需要重启Activity，下面就让我们一睹风采。

#### 1、Colorful的使用：
基于Theme的换肤，就需要我们定义好我们的属性和Theme属性，这些都是必备的。使用Colorful：

    Colorful mColorful;
    // 构建Colorful对象来绑定View与属性的对象关系
    mColorful = new Colorful.Builder(this)
            .backgroundDrawable(R.id.root_view, R.attr.root_view_bg)
            // 设置view的背景图片
            .backgroundColor(R.id.change_btn, R.attr.btn_bg)
            // 设置背景色
            .textColor(R.id.textview, R.attr.text_color)
            .setter(listViewSetter) // 手动设置setter
            .create(); // 设置文本颜色
    mColorful.setTheme(R.style.NightTheme)
    
这就是简单的使用。

##### 2、源码研究：
我们查看下Colorful这个控制类：

	public final class Colorful {
        /**
         * Colorful Builder
         */
        Builder mBuilder;

        /**
         * private constructor
         * 
         * @param builder
         */
        private Colorful(Builder builder) {
            mBuilder = builder;
        }

        /**
         * 设置新的主题
         * 
         * @param newTheme
         */
        public void setTheme(int newTheme) {
            mBuilder.setTheme(newTheme);
        }

        /**
         * 
         * 构建Colorful的Builder对象
         * 
         * @author mrsimple
         * 
         */
        public static class Builder {
            /**
             * 存储了视图和属性资源id的关系表
             */
            Set<ViewSetter> mElements = new HashSet<ViewSetter>();
            /**
             * 目标Activity
             */
            Activity mActivity;

            /**
             * @param activity
             */
            public Builder(Activity activity) {
                mActivity = activity;
            }

            /**
             * @param fragment
             */
            public Builder(Fragment fragment) {
                mActivity = fragment.getActivity();
            }

            private View findViewById(int viewId) {
                return mActivity.findViewById(viewId);
            }

            /**
             * 将View id与存储该view背景色的属性进行绑定
             * @param viewId
             *            控件id
             * @param colorId
             *            颜色属性id
             * @return
             */
            public Builder backgroundColor(int viewId, int colorId) {
                mElements.add(new ViewBackgroundColorSetter(findViewById(viewId),
                        colorId));
                return this;
            }

            /**
             * 将View id与存储该view背景Drawable的属性进行绑定
             * @param viewId
             *            控件id
             * @param colorId
             *            Drawable属性id
             * @return
             */
            public Builder backgroundDrawable(int viewId, int drawableId) {
                mElements.add(new ViewBackgroundDrawableSetter(
                        findViewById(viewId), drawableId));
                return this;
            }

            /**
             * 将TextView id与存储该TextView文本颜色的属性进行绑定
             * @param viewId
             *            TextView或者TextView子类控件的id
             * @param colorId
             *            颜色属性id
             * @return
             */
            public Builder textColor(int viewId, int colorId) {
                TextView textView = (TextView) findViewById(viewId);
                mElements.add(new TextColorSetter(textView, colorId));
                return this;
            }

            /**
             * 用户手动构造并且添加Setter
             * @param setter
             *            用户自定义的Setter
             * @return
             */
            public Builder setter(ViewSetter setter) {
                mElements.add(setter);
                return this;
            }

            /**
             * 设置新的主题
             * @param newTheme
             */
            protected void setTheme(int newTheme) {
                mActivity.setTheme(newTheme);
                makeChange(newTheme);
            }

            /**
             * 修改各个视图绑定的属性
             */
            private void makeChange(int themeId) {
                Theme curTheme = mActivity.getTheme();
                for (ViewSetter setter : mElements) {
                    setter.setValue(curTheme, themeId);
                }
            }

            /**
             * 创建Colorful对象
             * @return
             */
            public Colorful create() {
                return new Colorful(this);
            }
        }
    }
    
（1）、从Color的结构定义中可以看到，Colorful的构造函数是一个私有的构造函数，即作者不希望我们直接创建一个Colorful的对象，这里作者引入一个staitc Builder类，用于构造控件和属性的对应关系。

（2）、Builder类源码结构解析：
在Builder源码类中，我们可以看到里面包含一个

	/**
     * 存储了视图和属性资源id的关系表
     */
    Set<ViewSetter> mElements = new HashSet<ViewSetter>();

这就是用于存储我们的控件和属性值，然后通过

1. public Builder backgroundColor(int viewId, int colorId)：将View id与存储该view背景色的属性进行绑定
1. public Builder backgroundDrawable(int viewId, int drawableId)：将View id与存储该view背景Drawable的属性进行绑定
1. public Builder textColor(int viewId, int colorId)：将TextView id与存储该TextView文本颜色的属性进行绑定

通过这三个方法绑定控件的id和控件绑定的自定义属性id，然后在方法的内部，构建ViewSetter对象存储到Set集合。

当我们建立好这种控件和属性的对应关系，调用setTheme(id)进行Theme的主题设置，然后该方法调用

    /**
     * 修改各个视图绑定的属性
     */
    private void makeChange(int themeId) {
        Theme curTheme = mActivity.getTheme();
        for (ViewSetter setter : mElements) {
            setter.setValue(curTheme, themeId);
        }
    } 
    
我们可以看到该方法里面是遍历Set集合，然后调用ViewSetter的setValue设置值进行修改。总体的思路设计就是这样。

（3）、上面我们看到了ViewSetter类。该类封装了我们的控件id和属性id。

	/**
     * ViewSetter，用于通过{@see #mAttrResId}
     * 设置View的某个属性值,例如背景Drawable、背景色、文本颜色等。如需修改其他属性,可以自行扩展ViewSetter.
     * 
     * @author mrsimple
     * 
     */
    public abstract class ViewSetter {

        /**
         * 目标View
         */
        protected View mView;
        /**
         * 目标view id,有时在初始化时还未构建该视图,比如ListView的Item View中的某个控件
         */
        protected int mViewId;
        /**
         * 目标View要的特定属性id
         */
        protected int mAttrResId;

        public ViewSetter(View targetView, int resId) {
            mView = targetView;
            mAttrResId = resId;
        }

        public ViewSetter(int viewId, int resId) {
            mViewId = viewId;
            mAttrResId = resId;
        }

        /**
         * 
         * @param newTheme
         * @param themeId
         */
        public abstract void setValue(Theme newTheme, int themeId);

        /**
         * 获取视图的Id
         * 
         * @return
         */
        protected int getViewId() {
            return mView != null ? mView.getId() : -1;
        }

        protected boolean isViewNotFound() {
            return mView == null;
        }

        /**
         * 
         * @param newTheme
         * @param resId
         * @return
         */
        protected int getColor(Theme newTheme) {
            TypedValue typedValue = new TypedValue();
            newTheme.resolveAttribute(mAttrResId, typedValue, true);
            return typedValue.data;
        }
    }
    
这里主要就是看getColor方法，通过resolveAttibute方法获取Theme里面的属性值，然后设置到具体的控件上。这样就直接设置上了，实现了换肤，并且不需要重启Activity。比如最直接的例子，TextColorSetter：

	public class TextColorSetter extends ViewSetter {

        public TextColorSetter(TextView textView, int resId) {
            super(textView, resId);
        }

        public TextColorSetter(int viewId, int resId) {
            super(viewId, resId);
        }

        @Override
        public void setValue(Theme newTheme, int themeId) {
            if (mView == null) {
                return;
            }
            ((TextView) mView).setTextColor(getColor(newTheme));
        }
    }
    
这里的setValue方法就是直接设置值。

##### 3、总评
总体来说，Colorful基于Theme依然没有避免掉Theme的弊病，原理是将Thme中的属性值获取出来，设置到对应的控件上。而且现在只有setColor方法，即只支持获取Color的属性值设置，其他的图片之类的还没有丰富。总体功能还不是很完善。

#### 二、Android-Skin-Loader

Android-Skin-Loader是一个比较优秀的换肤框架，地址[android-sgik-loader](https://github.com/fengjundev/Android-Skin-Loader)，该工程的核心

Android-Skin-Loader
├── android-skin-loader-lib      // 皮肤加载库
├── android-skin-loader-sample   // 皮肤库应用实例
├── android-skin-loader-skin     // 皮肤包生成demo
└── skin-package                 // 皮肤包输出目录

通过这个框架可以对控件进行随意换肤。

在这个框架中，使用了自定义的Factory对象来完成换肤的操作，这点熟悉LayoutInflater的都知道，这个用于初始化我们的View，具体的我们可以参照[Android 探究 LayoutInflater setFactory](http://blog.csdn.net/lmj623565791/article/details/51503977)文章学习下。
换肤需要解决的核心问题有两个：
(1)、外部资源的加载
(2)、定位到需要换肤的View
第一个资源加载的问题可以通过构造AssetManager，反射调用其addAssetPath就可以完成。

第二个问题，就可以利用在onCreateView中，根据view的属性来定位，例如你可以让需要换肤的view添加一个自定义的属性skin_enabled=true（最开始有打印属性），并且利用一些手段拿到构造到的view，就能在View构造阶段定位的需要换肤的View。

在Android-Skin-Loader中，你需要集成封装的BaseActivity或BaseFragment来进行实现，我们看一下BaseActivity的源码。

	/**
     * Base Activity for development
     * 
     * <p>NOTICE:<br> 
     * You should extends from this if you what to do skin change
     * 
     * @author fengjun
     */
    public class BaseActivity extends Activity implements ISkinUpdate, IDynamicNewView{

        /**
         * Whether response to skin changing after create
         */
        private boolean isResponseOnSkinChanging = true;

        private SkinInflaterFactory mSkinInflaterFactory;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            mSkinInflaterFactory = new SkinInflaterFactory();
            getLayoutInflater().setFactory(mSkinInflaterFactory);
        }

        @Override
        protected void onResume() {
            super.onResume();
            SkinManager.getInstance().attach(this);
        }

        @Override
        protected void onDestroy() {
            super.onDestroy();
            SkinManager.getInstance().detach(this);
            mSkinInflaterFactory.clean();
        }

        /**
         * dynamic add a skin view 
         * 
         * @param view
         * @param attrName
         * @param attrValueResId
         */
        protected void dynamicAddSkinEnableView(View view, String attrName, int attrValueResId){	
            mSkinInflaterFactory.dynamicAddSkinEnableView(this, view, attrName, attrValueResId);
        }

        protected void dynamicAddSkinEnableView(View view, List<DynamicAttr> pDAttrs){	
            mSkinInflaterFactory.dynamicAddSkinEnableView(this, view, pDAttrs);
        }

        final protected void enableResponseOnSkinChanging(boolean enable){
            isResponseOnSkinChanging = enable;
        }

        @Override
        public void onThemeUpdate() {
            if(!isResponseOnSkinChanging){
                return;
            }
            mSkinInflaterFactory.applySkin();
        }

        @Override
        public void dynamicAddView(View view, List<DynamicAttr> pDAttrs) {
            mSkinInflaterFactory.dynamicAddSkinEnableView(this, view, pDAttrs);
        }
    }
    
通过源码，我们可以知道，我们需要换肤的界面需要集成BaseActivity才行。在BaseActivity中封装了自定义的SkinInflaterFactory对象，同时实现了ISkinUpdate、IDynamicNewView两个接口，用于回调换肤的过程以及添加换肤的View。在BaseActivity中封装了dynamicAddSkinEnableView、enableResponseOnSkinChanging两个方法，用于添加添加换肤的View以及设置是否可用换肤。方法实现就是通过SkinInflaterFactory中封装的方法完成需要换肤View的封装，然后实现换肤。

在Android-Skin-Loader中的核心类就是SkinInflaterFactory类，该类定义了实现换肤的方法。下面就看看该类的源码实现：

	/**
     * Supply {@link SkinInflaterFactory} to be called when inflating from a LayoutInflater.
     * 
     * <p>Use this to collect the {skin:enable="true|false"} views availabled in our XML layout files.
     * 
     * @author fengjun
     */
    public class SkinInflaterFactory implements Factory {

        private static final boolean DEBUG = true;

        /**
         * Store the view item that need skin changing in the activity
         */
        private List<SkinItem> mSkinItems = new ArrayList<SkinItem>();

        @Override
        public View onCreateView(String name, Context context, AttributeSet attrs) {
            // if this is NOT enable to be skined , simplly skip it 
            boolean isSkinEnable = attrs.getAttributeBooleanValue(SkinConfig.NAMESPACE, SkinConfig.ATTR_SKIN_ENABLE, false);
            if (!isSkinEnable){
                    return null;
            }

            View view = createView(context, name, attrs);

            if (view == null){
                return null;
            }

            parseSkinAttr(context, attrs, view);

            return view;
        }

        /**
         * Invoke low-level function for instantiating a view by name. This attempts to
         * instantiate a view class of the given <var>name</var> found in this
         * LayoutInflater's ClassLoader.
         * 
         * @param context 
         * @param name The full name of the class to be instantiated.
         * @param attrs The XML attributes supplied for this instance.
         * 
         * @return View The newly instantiated view, or null.
         */
        private View createView(Context context, String name, AttributeSet attrs) {
            View view = null;
            try {
                if (-1 == name.indexOf('.')){
                    if ("View".equals(name)) {
                        view = LayoutInflater.from(context).createView(name, "android.view.", attrs);
                    } 
                    if (view == null) {
                        view = LayoutInflater.from(context).createView(name, "android.widget.", attrs);
                    } 
                    if (view == null) {
                        view = LayoutInflater.from(context).createView(name, "android.webkit.", attrs);
                    } 
                }else {
                    view = LayoutInflater.from(context).createView(name, null, attrs);
                }

                L.i("about to create " + name);

            } catch (Exception e) { 
                L.e("error while create 【" + name + "】 : " + e.getMessage());
                view = null;
            }
            return view;
        }

        /**
         * Collect skin able tag such as background , textColor and so on
         * 
         * @param context
         * @param attrs
         * @param view
         */
        private void parseSkinAttr(Context context, AttributeSet attrs, View view) {
            List<SkinAttr> viewAttrs = new ArrayList<SkinAttr>();

            for (int i = 0; i < attrs.getAttributeCount(); i++){
                String attrName = attrs.getAttributeName(i);
                String attrValue = attrs.getAttributeValue(i);

                if(!AttrFactory.isSupportedAttr(attrName)){
                    continue;
                }

                if(attrValue.startsWith("@")){
                    try {
                        int id = Integer.parseInt(attrValue.substring(1));
                        String entryName = context.getResources().getResourceEntryName(id);
                        String typeName = context.getResources().getResourceTypeName(id);
                        SkinAttr mSkinAttr = AttrFactory.get(attrName, id, entryName, typeName);
                        if (mSkinAttr != null) {
                            viewAttrs.add(mSkinAttr);
                        }
                    } catch (NumberFormatException e) {
                        e.printStackTrace();
                    } catch (NotFoundException e) {
                        e.printStackTrace();
                    }
                }
            }

            if(!ListUtils.isEmpty(viewAttrs)){
                SkinItem skinItem = new SkinItem();
                skinItem.view = view;
                skinItem.attrs = viewAttrs;

                mSkinItems.add(skinItem);

                if(SkinManager.getInstance().isExternalSkin()){
                    skinItem.apply();
                }
            }
        }

        public void applySkin(){
            if(ListUtils.isEmpty(mSkinItems)){
                return;
            }

            for(SkinItem si : mSkinItems){
                if(si.view == null){
                    continue;
                }
                si.apply();
            }
        }

        public void dynamicAddSkinEnableView(Context context, View view, List<DynamicAttr> pDAttrs){	
            List<SkinAttr> viewAttrs = new ArrayList<SkinAttr>();
            SkinItem skinItem = new SkinItem();
            skinItem.view = view;

            for(DynamicAttr dAttr : pDAttrs){
                int id = dAttr.refResId;
                String entryName = context.getResources().getResourceEntryName(id);
                String typeName = context.getResources().getResourceTypeName(id);
                SkinAttr mSkinAttr = AttrFactory.get(dAttr.attrName, id, entryName, typeName);
                viewAttrs.add(mSkinAttr);
            }

            skinItem.attrs = viewAttrs;
            addSkinView(skinItem);
        }

        public void dynamicAddSkinEnableView(Context context, View view, String attrName, int attrValueResId){	
            int id = attrValueResId;
            String entryName = context.getResources().getResourceEntryName(id);
            String typeName = context.getResources().getResourceTypeName(id);
            SkinAttr mSkinAttr = AttrFactory.get(attrName, id, entryName, typeName);
            SkinItem skinItem = new SkinItem();
            skinItem.view = view;
            List<SkinAttr> viewAttrs = new ArrayList<SkinAttr>();
            viewAttrs.add(mSkinAttr);
            skinItem.attrs = viewAttrs;
            addSkinView(skinItem);
        }

        public void addSkinView(SkinItem item){
            mSkinItems.add(item);
        }

        public void clean(){
            if(ListUtils.isEmpty(mSkinItems)){
                return;
            }

            for(SkinItem si : mSkinItems){
                if(si.view == null){
                    continue;
                }
                si.clean();
            }
        }
    }
    
通过自定义的Factory实现自定义的View的解析，通过自定义属性skin:enable="true|false"来进行设置控件是否能够进行换肤。我们可以看到onCreateView的实现。

	boolean isSkinEnable = attrs.getAttributeBooleanValue(SkinConfig.NAMESPACE, SkinConfig.ATTR_SKIN_ENABLE, false);
    
在这里通过AttributeSet来获取控件的自定义属性(skin:enable)的值。如果isSkinEnable为false，即不是换肤标识的View，直接return null。如果是标识位换肤的控件，则通过createView（）方法创建View。然后根据name的类型进行判断创建View的种类。最后通过parseSkinAttr()方法遍历View的属性进行搜集。在这里补充下一个属性：private List< SkinItem> mSkinItems，这个成员变量用于记录存储的View。这里使用了SkinItem对象，该对象用于封装View以及对应的属性。

	public class SkinItem {
	
        public View view;

        public List<SkinAttr> attrs;

        public SkinItem(){
            attrs = new ArrayList<SkinAttr>();
        }

        public void apply(){
            if(ListUtils.isEmpty(attrs)){
                return;
            }
            for(SkinAttr at : attrs){
                at.apply(view);
            }
        }

        public void clean(){
            if(ListUtils.isEmpty(attrs)){
                return;
            }
            for(SkinAttr at : attrs){
                at = null;
            }
        }

        @Override
        public String toString() {
            return "SkinItem [view=" + view.getClass().getSimpleName() + ", attrs=" + attrs + "]";
        }
    }
    
这里SkinItem封装了View以及对应的attrs属性，同时定义了apply()、clean()两个方法，用于启动和清除。然后通过parseSkinAttr()方法遍历View的属性，通过SkinItem进行存储。在SkinInflaterFactory类中，通过dynamicAddSkinEnableView、addSkinView来添加View视图。然后通过调用applySkin()方法进行换肤。

	public void applySkin(){
		if(ListUtils.isEmpty(mSkinItems)){
			return;
		}
		
		for(SkinItem si : mSkinItems){
			if(si.view == null){
				continue;
			}
			si.apply();
		}
	}
    
在applySkin()中通过遍历SkinItem来完成换肤，skinItem.apply方法的本质又是什么呢？

	public void apply(){
		if(ListUtils.isEmpty(attrs)){
			return;
		}
		for(SkinAttr at : attrs){
			at.apply(view);
		}
	}
    
在这个方法中又出现了一个SkinAttr类，然后本质就是针对属性进行设置。在这里，有BackgroundAttr、DividerAttr等几个子类，进行实现。

总体来说，Android-Skin-Loader就是通过自定义Factory来实现换肤的操作。

通过上面的例子，我们了解了整体的换肤实现方式，主要就是采用资源的实现方式。以及针对View进行换肤操作的设置。