## Android 权限简述

Android 是一个权限分隔的操作系统，其中每个应用都有其独特的系统标识（Linux 用户 ID 和组 ID）。系统各部分也分隔为不同的标识，Linux 据此将不同的应用以及应用与系统分隔开来。

Android 安全架构的中心设计点是：在默认情况下任何应用都没有权限执行对其他应用、操作系统或用户有不利影响的任何操作。这包括读取或写入用户的私有数据（例如联系人或电子邮件）、读取或写入其他应用程序的文件、执行网络访问、使设备保持唤醒状态等。

由于每个 Android 应用都是在进程沙盒中运行，因此应用必须显式共享资源和数据。它们的方法是声明需要哪些权限来获取基本沙盒未提供的额外功能。应用以静态方式声明它们需要的权限，然后 Android 系统提示用户同意。

### 1. Android 权限基本使用

#### 1.1 系统权限

Android 应用默认情况下未关联权限，意味着它无法执行对用户体验或设备上任何数据产生不利影响的任何操作。通常使用权限必须在 Manifest.xml 文件中使用 `<uses-permission>` 注册权限。

比如读取联系人权限：

```xml
 <uses-permission android:name="android.permission.READ_CONTACTS"/>
```



#### 1.2 自定义权限

Android 系统中提供了上百个权限，涵盖为 CAMERA、LOCATION、PHONE、SENSORS、CONTACTS 等等分组中。但是在具体开发中，会出现应用中的组件需要对第三方应用暴露，所以通过使用 `android:permission` 属性设置访问权限提高安全性。Android 系统允许四大组件跨应用调用，暴露的组件如果设置了 `android:permission` 属性，则调用方（客户端）必须通过 `uses-permission` 注册权限才能访问。

##### 1.2.1 自定义权限

自定义权限在 `Manifest.xml` 中使用 `<permission>` 标签进行定义。

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.sensorsdata.permissiondemo">
    <permission
        android:name="com.sensorsdata.analytics.sdk.permission"
        android:protectionLevel="normal"
        android:label="神策 SDK 权限"
        android:description="神策 SDK 使用权限"
        android:permissionGroup="sensors_permission"/>

    <permission-group android:name="sensors_permission"/>
</manifest>
```

Permission 属性：

- android:name：权限名称，必填项

- android:protectionLevel：权限等级。

  |  ProtectionLevel  | 描述                                                         |
  | :---------------: | ------------------------------------------------------------ |
  |      normal       | 普通权限，如果应用声明了此权限，也不会提示安装应用的用户授权。 |
  |     dangerous     | 危险权限，拥有此权限可能会访问用户私人数据或者控制设备，给用户带来负面影响，这种类型的权限需要用户授权同意。 |
  |     signature     | 该权限级别，只有当发请求的应用和接收此请求的应用使用同一签名文件，并且声明了该权限才会授权，并且是默认授权，不会提示用户授权 |
  | signatureOrSystem | 该权限级别是系统授权的系统应用或者相同签名的应用，一般避免使用该级别，因为 signature 已经能满足大部分需求。 |

- android:label：权限简单描述

- android:description：权限详细描述

- android:permissionGroup：权限分组

##### 1.2.2 自定义权限建议

根据 [Android 开发文档表述](https://developer.android.com/guide/topics/security/permissions?hl=zh-cn#normal-dangerous)，针对自定义权限的建议如下：

- 如果要设计一套向彼此显示功能的应用，请尽可能将应用设计为每个权限只定义一次。如果所有应用并非使用同一证书签署，则必须这样做。即使所有应用使用同一证书签署，最佳做法也是每个权限只定义一次。
- 如果功能仅适用于使用与提供应用相同的签名所签署的应用，您可能可以使用签名检查避免定义自定义权限。当一个应用向另一个应用发出请求时，第二个应用可在遵从该请求之前验证这两个应用是否使用同一证书签署。
- 如果您要开发一套只在您自己的设备上运行的应用，则应开发并安装管理该套件中所有应用权限的软件包。此软件包本身无需提供任何服务。它只是声明所有权限，然后套件中的其他应用通过 `<uses-permission>` 元素请求这些权限。

##### 1.2.3 自定义权限使用

我们在 `Manifest.xml` 文件中定义自定义权限，同时使用权限来限制访问系统或应用的全部组件，达到暴露的组件访问更加安全的效果。

**1. Activity 权限**

在 `<activity>` 标签中通过 `<android:permission>` 声明权限，限制谁可以启动 `Activity`。在 `Context.startActivity()`和 `Activity.startActivityForResult()`时会检查权限；如果调用方没有所需的权限，则调用会抛出 `SecurityException`。

**2. Service 权限**

在 `<service>` 标签中通过 `<android:permission>` 声明权限，限制第三方应用谁可以绑定或启动服务。在 `Context.startService()、Context.stopService()`和 `Context.bindService()`时会检查权限；如果调用方没有所需的权限，则调用会抛出 `SecurityException`。

**3. BroadcastReceiver 权限**

在 <receiver> 标签中通过 `<android:permission>` 声明权限，限制第三方应用谁可以发送或接收广播数据。在 `Context.sendBroadcast()` 返回后检查权限，因为系统会尝试将提交的广播传递到指定的接收方。因此，权限失效不会导致向调用方抛回异常；只是不会传递该`intent`。同样，可以向 `Context.registerReceiver()`提供权限来控制谁可以广播到以编程方式注册的接收方。另一方面，可以在调用 `Context.sendBroadcast()`时提供权限来限制允许哪些 `BroadcastReceiver` 对象接收广播。

**4. ContentProvider 权限**

在 `<provider>` 标签中通过 `<android:permission>` 声明权限，限制第三方应用谁可以访问`ContentProvider`中的数据。（内容提供程序有重要的附加安全工具可用，称为 `URI` 权限）与其他组件不同，您可以设置两个单独的权限属性：`android:readPermission`限制谁可以读取提供程序，`android:writePermission`限制谁可以写入提供程序。

请注意，如果提供程序有读取和写入权限保护，仅拥有写入权限并不表示您可以读取提供程序。第一次检索提供程序时将会检查权限（如果没有任何权限，将会抛出 `SecurityException`），对提供程序执行操作时也会检查权限。使用 `ContentResolver.query()` 需要拥有读取权限；使用 `ContentResolver.insert()、ContentResolver.update()、ContentResolver.delete()`需要写入权限。在所有这些情况下，没有所需的权限将导致调用抛出 `SecurityException`。



### 2. Android 6.0 权限变更

#### 2.1 运行时权限

在 Android 6.0 中引入了一种新的权限管理模式，即用户可以在运行时管理应用权限。这种模式让用户可以更好的了解和控制权限，同时为开发者精简了安装和自动更新过程，用户可为所安装的应用分别授予和撤销权限。

对于以 Android 6.0（API 级别 23）或更高版本为目标平台的应用(targetSdkVersion >= 23)，请务必在运行时检查和请求权限。调用新增的 `checkSelfPermission()` 方法确应用是否已被授予权限。要请求权限调用新增的 `requestPermissions()`方法。即使您的应用并不以 Android 6.0（API 级别 23）为目标平台，您也应该在新权限模式下测试您的应用。

- 如果设备运行的是 Android 6.0（API 级别 23）或更高版本，并且应用的 `targetSdkVersion` 是 23 或更高版本，则应用在运行时向用户请求权限。用户可随时调用权限，因此应用在每次运行时均需检查自身是否具备所需的权限。
- 如果设备运行的是 Android 5.1（API 级别 22）或更低版本，并且应用的 `targetSdkVersion` 是 22 或更低版本，则系统会在用户安装应用时要求用户授予权限。如果将新权限添加到更新的应用版本，系统会在用户更新应用时要求授予该权限。用户一旦安装应用，他们撤销权限的唯一方式是卸载应用。

通常在 `Manifest.xml` 中列出的`正常权限`（即 normal 级别，不会对用户的隐私或设备造成很大风险），系统会自动授予这些权限。如果在 `Manifest.xml` 中

#### 2.2 权限分类

系统权限分为几个保护级别。两个最重要保护级别是`正常权限`和`危险权限`：

- `正常权限`涵盖应用需要访问其沙盒外部数据或资源，但对用户隐私或其他应用操作风险很小的区域。例如，设置时区的权限就是正常权限。如果应用声明其需要正常权限，系统会自动向应用授予该权限。如需当前正常权限的完整列表，请参阅[正常权限](https://developer.android.com/guide/topics/security/normal-permissions.html?hl=zh-cn)。
- `危险权限`涵盖应用需要涉及用户隐私信息的数据或资源，或者可能对用户存储的数据或其他应用的操作产生影响的区域。例如，能够读取用户的联系人属于危险权限。如果应用声明其需要危险权限，则用户必须明确向应用授予该权限。

>特殊权限：有许多权限其行为方式与正常权限及危险权限都不同。`SYSTEM_ALERT_WINDOW` 和 `WRITE_SETTINGS` 特别敏感，因此大多数应用不应该使用它们。如果某应用需要其中一种权限，必须在清单中声明该权限，并且发送请求用户授权的 intent。系统将向用户显示详细管理屏幕，以响应该 intent。

##### 2.2.1 权限组

所有危险的 Android 系统权限都属于权限组。如果设备运行的是 Android 6.0（API 级别 23），并且应用的 `targetSdkVersion`是 23 或更高版本，则当用户请求危险权限时系统会发生以下行为：

- 如果应用请求其清单中列出的危险权限，而应用目前在权限组中没有任何权限，则系统会向用户显示一个对话框，描述应用要访问的权限组。对话框不描述该组内的具体权限。例如，如果应用请求 `READ_CONTACTS` 权限，系统对话框只说明该应用需要访问设备的联系信息。如果用户批准，系统将向应用授予其请求的权限。
- 如果应用请求其清单中列出的危险权限，而应用在同一权限组中已有另一项危险权限，则系统会立即授予该权限，而无需与用户进行任何交互。例如，如果某应用已经请求并且被授予了 `READ_CONTACTS` 权限，然后它又请求 `WRITE_CONTACTS`，系统将立即授予该权限。

任何权限都可属于一个权限组，包括正常权限和应用定义的权限。但权限组仅当权限危险时才影响用户体验。可以忽略正常权限的权限组。

如果设备运行的是 Android 5.1（API 级别 22）或更低版本，并且应用的 `targetSdkVersion` 是 22 或更低版本，则系统会在安装时要求用户授予权限。再次强调，系统只告诉用户应用需要的权限组，而不告知具体权限。

| 权限组       | 权限                                                         |
| ------------ | ------------------------------------------------------------ |
| `CALENDAR`   | `READ_CALENDAR`      <br>`WRITE_CALENDAR`                    |
| `CAMERA`     | `CAMERA`                                                     |
| `CONTACTS`   | `READ_CONTACTS`<br>`WRITE_CONTACTS`<br>`GET_ACCOUNTS`        |
| `LOCATION`   | `ACCESS_FINE_LOCATION`<br>`ACCESS_COARSE_LOCATION`           |
| `MICROPHONE` | `RECORD_AUDIO`                                               |
| `PHONE`      | `READ_PHONE_STATE`<br> `CALL_PHONE`<br>`READ_CALL_LOG`<br>`WRITE_CALL_LOG`<br>`ADD_VOICEMAIL`<br>`USE_SIP`<br>`PROCESS_OUTGOING_CALLS` |
| `SENSORS`    | `BODY_SENSORS`                                               |
| `SMS`        | `SEND_SMS`<br>`RECEIVE_SMS`<br>`READ_SMS`<br>`RECEIVE_WAP_PUSH`<br>`RECEIVE_MMS` |
| `STORAGE`    | `READ_EXTERNAL_STORAGE`<br>`WRITE_EXTERNAL_STORAGE`          |

#### 2.3 动态申请权限

动态申请权限的核心方法：

- int checkSelfPermission(Context context, String permission) 用来检测应用是否已经具有权限

- void requestPermissions(Context context, String[] permissions, int requestCode) 进行请求单个或多个权限

- void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) 请求权限结果回调

相机权限属于危险权限，下面针对动态申请权限示例：

**1. 首先在 `Manifest.xml` 中注册权限**

```xml
<uses-permission android:name="android.permission.CAMERA"/>
```

**2. 动态申请权限**

```java
public class MainActivity extends AppCompatActivity {
    private static final int REQUEST_CAMERA = 1;
    private static final int REQUEST_PERMISSION_CAMERA_CODE = 100;
    private String mFilePath;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mFilePath = Environment.getExternalStorageDirectory().getPath() + "/" + "temp.png";// 获取SD卡路径
        findViewById(R.id.btn_camera).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    if (checkCameraPermission()) {
                        startCamera();
                    } else {
                        ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.CAMERA}, REQUEST_PERMISSION_CAMERA_CODE);
                    }
                } else {
                    startCamera();
                }
            }
        });
    }

    private boolean checkCameraPermission() {
        return ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED;
    }

    private void startCamera() {
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);// 启动系统相机  
        Uri photoUri = Uri.fromFile(new File(mFilePath)); // 传递路径  
        intent.putExtra(MediaStore.EXTRA_OUTPUT, photoUri);// 更改系统默认存储路径  
        startActivityForResult(intent, REQUEST_CAMERA);
    }

    private boolean isAllPermissionsGranted(@NonNull int[] grantResults) {
        for (int grant : grantResults) {
            if (grant != PackageManager.PERMISSION_GRANTED) {
                return false;
            }
        }
        return true;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        if (REQUEST_PERMISSION_CAMERA_CODE == requestCode) {
            if (isAllPermissionsGranted(grantResults)) {
                Toast.makeText(this, "授予权限", Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(this, "拒绝权限", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
```

当应用安装在 Android 6.0 及以上版本中，就需要动态申请权限。

> 如果不在 Manifest.xml 中注册权限，则不会弹出授权对话框。



### 3. 题外话

更多关于 Android 权限的内容可参照 Android 官方文档。[Android 系统权限](https://developer.android.com/guide/topics/security/permissions?hl=zh-cn#normal-dangerous)

关于动态申请权限优良的开源库：[RxPermissions](https://github.com/tbruyelle/RxPermissions)