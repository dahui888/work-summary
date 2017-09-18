### Groovy基础学习总结
作为一名Android开发工程师，能够接触到Groovy这门类Java语音必然是因为Gradle的编译。Gradle是Android Studio默认的构建工具，而Gradle又是基于Groovy语言进行搭建构建。


#### 一、Groovy是什么？
Groovy是一种类Java开发语音，基于Java并扩展Java。所以Groovy的语法跟Java很相似，同时Groovy又吸收了Python、Ruby等脚本语音的特点，所以Groovy更像是一种类Java的脚本语音，同时Java程序员上手Groovy难度不大。

由于Groovy是基于Java扩展而来，所以Groovy是一种运行在JVM上的动态语言。实际上，由于 Groovy Code  在真正执行的时候已经变成了 Java字节码，所以 JVM 根本不知道自己运行的是 Groovy代码。

#### 二、Groovy环境搭建
Groovy的搭建工具：
1. Groovy的SDK下载
2. JDK的安装
3. IntelliJ IDEA编译器

我的开发环境是使用IntelliJ IDEA编译器进行搭建，然后在Setting——>中勾选Groovy。

#### 三、Groovy工程创建
在IntelliJ IDEA工具中创建一个Groovy工程的步骤如下：
File->New Project->Groovy

![Groovy]()

我们创建一个HelloWorld程序。

![HelloWorld]()

通过上面，可以看到Groovy也是可以运行Java程序的，然后我们编译，查看out文件夹中编译生成的文件。可以看到是一个编译后的HelloWorld.class文件。同时也印证了Groovy是基于Java扩展而来运行在JVM上的动态语言。

#### 四、Groovy程序结构和基本数据类型
##### 一、Groovy程序结构
Groovy基于Java语言又融合了Python、Ruby等脚本特点，所以它的程序结构特点也融合了其它语言的特点。

1. Groovy语句结尾可以不用分号。这里基本是和Python保持一致。
2. Groovy支持动态类型，<b>即定义变量的时候可以不用指定变量的类型，同时定义函数参数以及函数的返回值类型时都可以不进行指定。</b>
3. Groovy中可以通过def关键字来进行定义变量。这里是否使用def并无强制要求，但是为了Code的可读性，建议最好统一添加上。
4. 函数返回值，Groovy中函数返回值可以通过return xx来设置返回值。当然如果不使用return语句，则默认最后一行代码的执行结果当做返回值。
5. Groovy中函数可以不带括号，但是很容易引起和属性的混淆，所以建议还是使用括号，针对系统带的常见函数可以不使用括号。

##### 二、Groovy中的数据类型
Groovy的数据类型可以分为以下三类：
- Java基本数据类型，int ，boolean 等， 这些 Java  中
的基本数据 类型在 ，在 Groovy  代码中其实对应的是它们的包装数据类型。如 比如 int  对应为
Integer ，boolean  对应为 Boolean
- Groovy容器类，其实跟Java的集合类似，主要有：List、Map、Range。
    1. List：链表，其底层对应 Java 中的 List 接口，一般用 ArrayList 作为真正的实
    现类。
    2.  Map：键-值表，其底层对应 Java 中的 LinkedHashMap。
    3.  Range：范围，它其实是 List 的一种拓展。
- 闭包。这个数据很常用，在Groovy中很重要，Gradle中大量使用，有点类似Java中代码块的味道。

这里，我们着重学习下闭包这种类型，我们学习了解Groovy的出发点就是编译配置Gradle，而闭包这种类型在Gradle中经常使用，我们对这种类型又比较陌生。

**1、什么是闭包？**
闭包，英文叫 Closure，是 Groovy 中非常重要的一个数据类型或者说一种概念了。Closure定义的格式：

	def xxx = {paramters -> code} //或者
	def xxx = {无参数，纯 code} 这种 case 不需要->符号

闭包示例：
```python
def noParamClosure = {
    println("No Params Closure")
}
def tempClosure = {grade,age,name->
    println(name + ":" + age + ":" + grade)
}
```

**如果闭包没自定义参数的话，则隐含有一个参数，这个参数名字叫 it ，和 this  的作用类
似。it **

示例：我们看一个闭包的简单使用。
```java
def noParamClosure = {
    println("No Params Closure")
}
def tempClosure = {grade,age,name->
    println(name + ":" + age + ":" + grade)
}

def itParam = {
    println("This is params:$it")
}

noParamClosure.call();
tempClosure.call(60,21,"Groovy")
itParam("GroovyIt")
```
**2、Groovy使用注意事项**
1. Groovy 中，当函数的最后一个参数是闭包的话，可以省略圆括号。
