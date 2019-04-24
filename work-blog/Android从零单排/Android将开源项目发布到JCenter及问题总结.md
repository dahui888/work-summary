### Android 将开源项目发布到JCenter及问题总结

在Android开发中，我们都会通过compile来集成一些开源的项目，那么我们如何发布我们的开源项目让别人集成呢？下面我也来看网上的一些文章学习总结下。

#### 集成方式
网上出现了很多集成方式，越来越简单。具体可以搜索下。我这里只总结我使用的方式，以便下次使用。

首先我们去[bintray](https://bintray.com/dsw)网站注册一个账号。然后接下来就是使用bintray插件集成。

##### 首先打开我们的Project的gradle。配置如下：

```java
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
新增路径：
```java
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'
```

##### 打开我们要发布的module的gradle文件。
```java
apply plugin: 'com.android.library'
//配置插件方法1
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'
//提交到仓库中的版本号
version = "1.0.0"
android {
    compileSdkVersion 19
    buildToolsVersion "19.1.0"

    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:19.1.0'
}

//方法1
def siteUrl = 'https://github.com/dengshiwei/CalendarComponent'      // 项目的主页   这个是说明，可随便填
def gitUrl = 'https://github.com/dengshiwei/CalendarComponent'      // Git仓库的url  这个是说明，可随便填
group = "com.com.dsw.calendar"    // 这里是groupId ,必须填写  一般填你唯一的包名

install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                // Add your description here
                name 'CalendarComponent'     //项目描述
                url siteUrl
                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'dsw'        //填写开发者的一些基本信息
                        name 'mr_dsw'    //填写开发者的一些基本信息
                        email 'dengshiwei_it@163.com'   //填写开发者的一些基本信息
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    options.addStringOption('Xdoclint:none', '-quiet')
    options.addStringOption('encoding', 'UTF-8')
    options.addStringOption('charSet', 'UTF-8')
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
artifacts {
    archives javadocJar
    archives sourcesJar
}

Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")    //读取 local.properties 文件里面的 bintray.user
    key = properties.getProperty("bintray.apikey")   //读取 local.properties 文件里面的 bintray.apikey
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = "calendarcomponent"    //发布到JCenter上的项目名字，必须填写
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}

//这段代码一会给你们解释哈
javadoc {
    options{
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version true
        links "http://docs.oracle.com/javase/7/docs/api"
    }
}
```
对比项目，缺什么补什么。

##### 执行指令
**1、执行gradlew install**
执行install task。很关键的一步，我也是卡在这里出现好几个错误，不过好歹都修复了。出现：BUILD SUCCESSFUL就成功了。

**2、gradlew bintrayUpload**
执行这个就是上传我们的库到bintray。

在 local.properties 中配置自己的用户名和 apikey，可以在个人信息的 View Profile 中查看。

**3、最后到网站上找到我们的项目，点击Add to jCenter**

最后我们就完成了。

终于搞好了。
