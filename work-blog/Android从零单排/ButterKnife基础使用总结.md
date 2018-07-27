## ButterKnife基础使用总结

### 一、前言
#### 1、ButterKnife简介
butterknife是出自Android大神JakeWharton之手的一个开源库，它通过注解的方式来替代android中view的相关操作。专注于Android系统的View,可以减少大量的findViewById以及setOnClickListener代码，可视化一键生成。这个开源库可以让我们从大量的findViewById()和setonclicktListener()解放出来，其对性能的影响微乎其微(查看过Butter Knife的源码，其自定义注解的实现都是限定为RetentionPolicy.CLASS，也就是到编译出.class文件为止有效，在运行时不额外消耗性能，其是通过java注解自动生成java代码的形式来完成工作)。

项目地址：[https://github.com/JakeWharton/butterknife](https://github.com/JakeWharton/butterknife)

#### 2、ButterKnife优势

1. 通过BindXXX等方式来替代View的常见操作，简化代码，提升开发效率。
1. 通过注解的方式实现，但是运行时不会影响App的效率。
1. 代码清晰，可读性强。


----------

### 二、如何引用ButterKnife？

在Module对应的gradle配置中，添加如下：

	buildscript{
     repositories{
         mavenCentral()
     }

    dependencies{
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
	    }
	}
	
	apply plugin: 'com.neenbedankt.android-apt'
	
	dependencies {
	    compile fileTree(dir: 'libs', include: ['*.jar'])
	    testCompile 'junit:junit:4.12'
	    compile 'com.android.support:appcompat-v7:24.2.1'
	    compile 'com.jakewharton:butterknife:8.4.0'
	    apt 'com.jakewharton:butterknife-compiler:8.4.0'
	}

----------

由于在 Android Studio3.1 版本中已经将 apt 的注解编译更换成 annotationProcessor，所以最新的配置只需要在 module 的 build.gradle 中添加：
```java
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation 'com.jakewharton:butterknife:8.8.1'
    annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.1'
}
```

### 三、ButterKnife常见使用

1. 控件id 注解： @BindView(int id)

		public class MainActivity extends AppCompatActivity {
		    @BindView(R.id.recyclerView)
		    public RecyclerView recyclerView;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        ButterKnife.bind(this);
		    }
		}

1. 多个控件id 注解： @BindViews(id1，id2)

		public class BindViewActivity extends AppCompatActivity {

		    @BindViews({R.id.title_name, R.id.describtion})
		    public List<TextView> textViews;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_bind_view);
		        ButterKnife.bind(this);
		    }
		}

1. 字符串id 注解：@BindString(id)，绑定String字符串

		public class BindViewActivity extends AppCompatActivity {

		    @BindViews({R.id.title_name, R.id.describtion})
		    public List<TextView> textViews;
		
		    @BindView(R.id.btn_number)
		    Button btn_number;
		
		    @BindString(R.string.title_name)
		    String title;
		
		    @BindString(R.string.describtion_name)
		    String describtion;
		
		    @BindString(R.string.button_text)
		    String button;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_bind_view);
		        ButterKnife.bind(this);
		
		        textViews.get(0).setText(title);
		        textViews.get(1).setText(describtion);
		        btn_number.setText(button);
		    }
		}

1. 字符串数组id注解：@BindArray(id)

		public class BindArrayString extends AppCompatActivity {
		    @BindArray(R.array.city)
		    String[] citys;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        ButterKnife.bind(this);
		    }
		}

1. Bitmap图像id 注解： @BindBitmap(id)

		public class BindArrayString extends AppCompatActivity {
		    @BindBitmap(R.mipmap.ic_launcher)
		    public Bitmap launcher_bitmap;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        ButterKnife.bind(this);
		    }
		}

1. 颜色值id  注解：@BindColor(id)

		public class BindArrayString extends AppCompatActivity {
		    @BindColor(R.color.colorAccent)
		    int colorAccent;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        ButterKnife.bind(this);
		    }
		}

1. 布尔类型值id  注解：@BindBool(id)

		public class BindArrayString extends AppCompatActivity {
		    @BindBool(R.bool.check)
		    boolean isCheck;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
		        ButterKnife.bind(this);
		    }
		}


----------
### 三、ButterKnife快捷插件
#### Zelezny : Butterknife插件的使用
>这是一款在Android Studio中使用的插件，用于快速生成ButterKnife的使用。可参照下图示例：

![first](http://img.blog.csdn.net/20161031141739545)

![sec](http://img.blog.csdn.net/20161031141833705)

![third](http://img.blog.csdn.net/20161031140736060)


两个晚上终于把总结写好了，春节就这么点成就吧！
