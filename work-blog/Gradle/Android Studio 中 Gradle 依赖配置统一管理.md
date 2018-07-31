### Android Studio 中 Gradle 依赖统一配置
在我们的实际项目开发中，通常在一个 Project 项目中会存在多个 Module 的情况，在这些 Module 中会存在一些相同的版本依赖配置，针对进行版本升级的时候需要逐个修改，显得特别麻烦，所以将依赖的配置抽取出来是一个不错的想法。

#### 1. 项目结构
通常我们的项目在 Project 模式的下结构是：
```java
rootProject
  --module1
    --build.gradle
  --module2
    --build.gradle
  ...
  --build.gradle
```
所以针对各个 Module 的统一管理，我们可以在 Project 的根目录 build.gradle 中进行配置，或者通过新建一个 config.gradle 配置来完成。

#### 2. 在 Project 的 build.gradle 中配置
build.gradle 是针对整个 Project 级别的配置，所以在 build.gradle 中进行配置让每个 Module 去读取配置。
- 根目录 build.gradle 配置
- 新建 config.gradle 进行配置

##### 2.1 根目录 build.gradle 配置
在 Android Studio 中的 .gradle 中支持 Groovy 语言，所以我们的配置起始就是有点类似于 Java 中的存储配置变量。

**在 Gradle DSL 中通过 Project.ext 进行 Extra Properties（额外属性）**

**build.gradle**

```xml
rootProject.ext{
    android = [
            compileSdkVersion : 28,
            buildToolsVersion : "28.0.0",
            applicationId : "sw.andoter.com.gradleplugindemo",
            minSdkVersion : 15,
            targetSdkVersion : 28,
            versionCode : 1,
            versionName : "1.0"
    ]

    sdkVersion = 13
}
```

在具体的 Module 中使用：

```java
android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    defaultConfig {
        applicationId rootProject.ext.android.applicationId
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

这样就可以引用了，非常简单。但是这样如果配置量比较大，在根目录的 build.gradle 中进行配置就显得可读性非常差，所以就会使用方式二进行配置。

##### 2.2 新建 config.gradle 进行配置
通过新建配置文件进行配置，形成如下目录：

```java
rootProject
  --module1
    --build.gradle
  --module2
    --build.gradle
  ...
  --build.gradle
  --config.gradle
```

**1. 选中项目，右键新建 Gradle Script 脚本配置**

**config.gradle**

```java
rootProject.ext{
    android = [
        compileSdkVersion : 28,
        buildToolsVersion : "28.0.0",
        applicationId : "sw.andoter.com.gradleplugindemo",
        minSdkVersion : 15,
        targetSdkVersion : 28,
        versionCode : 1,
        versionName : "1.0"
    ]

    sdkVersion = 13
}
```

**在 Module 的配置中引用**
引用配置脚本文件通过 **apply from:xx**，需要注意的就是 .gradle 文件的位置，同级目录我们直接写文件名称即可，不同目录需要使用相对路径。

**Module 的 build.gradle**

```java
apply from : "../config.gradle"

android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    defaultConfig {
        applicationId rootProject.ext.android.applicationId
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

以后再对依赖包升级的时候直接修改 config.gradle 文件就 OK 了。

#### 3. 综述
纵观上面的方式，核心思想就是保存配置的数据，所以不一定非要选择在 .gradle 文件中，只要方便使用就行。比如可以放在 gradle.properties中。注意 .properties 文件中存储的是键值对 key-value 形式。

**gradle.properties**

```java
key = "I'm from gradle.properties"
```

在 Module 的 build.gradle 新建一个 Task 进行测试下:

```java
task readConfig{
    doLast{
        println key
    }
}
```

同样可以进行配置数据的读取。