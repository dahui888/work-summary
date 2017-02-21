Gradle 是一个能通过插件形式自定义构建逻辑的优秀构建工具。
以下的一些特性让我们选择了 Gradle：

- 使用领域专用语言（DSL）来描述和控制构建逻辑
- 构建文件基于 Groovy，并允许通过 DSL 来声明元素、使用代码操作 DSL 元素这样的混合方式来自定义构建逻辑
- 内置了 Maven 和 Ivy 来进行依赖管理
- 相当灵活。允许使用最好的实现，但是不会强制实现的形式
- 插件可以提供它们的 DSL 和 API 来定义构建文件
- 优秀的 API 工具与 IDE 集成

###一、Gradle结构简单示例
Gradle 项目的构建描述定义在项目根目录下的 build.gradle 文件中。

	buildscript {
	    repositories {
	        jcenter()
	    }
	
	    dependencies {
	        classpath 'com.android.tools.build:gradle:1.3.1'
	    }
	}
	
	apply plugin: 'com.android.application'
	
	android {
	    compileSdkVersion 23
	    buildToolsVersion "23.1.0"
	}

通过上面简单的示例，build.gradle包含了 Android 构建文件的 3 个主要部分：

**buildscript { ... }** 配置了用于驱动构建的代码。上述代码声明了项目使用 jCenter 仓库，并且声明了一个 jCenter 文件的 classpath。该文件声明了项目的 Android Gradle 插件版本为 1.3.1。
>注意：这里的配置只会影响构建过程所需的类库，而不会影响项目的源代码。

项目自身需声明自身的仓库和依赖。
接着，使用了 **com.android.application 插件**。该插件用于编译 Android 应用。
最后，**android { ... }** 配置了所有 android 构建所需的参数，这也是 Android DSL 的入口点。默认情况下，只有 **compileSdkVersion** 和 **buildtoolsVersion** 这两个属性是必须的。
compileSdkVersion 属性相当于旧构建系统中 project.properites 文件中的 target 属性。这个新的属性可以跟旧的 target 属性一样指定一个 int(api level) 或者 String 类型的值。
>重要： **com.android.application** 插件不能与 java 插件同时使用，否则会导致构建错误。
注意： 你需要在相同路径下添加一个 local.properties 文件，并使用 **sdk.dir** 属性来设置 SDK 路径。或通过设置 **ANDROID_HOME** 环境变量来设置 SDK 路径。这两种方式都是一样的，根据喜好选择其中一种。
local.properties 文件示例：
	sdk.dir=/path/to/Android/Sdk

###二、项目结构
我们知道不同的工具会构建出不同结构的项目结构。