### Android D8 编译器 和 R8 工具

Android 安装包的后缀都是 .apk， APK 是 Android Package 的缩写。在 APK 打包编译的过程中，会涉及到 `javac` 工具将 `.java` 文件编译为 `.class` 文件，然后 `.class` 文件经过脱糖由 `dex` 工具打包为 `.dex` 文件。

![dex 过程](/Users/dengshiwei/Desktop/D8R8/dex 过程.png)

- [javac](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javac.html)：用于将 `.java` 文件编译为 `.class` 文件；
- [desugar](https://developer.android.com/studio/write/java8-support)：用于将 Java 8 中的特性在 Android 平台上适配；
- [ProGuard](https://www.guardsquare.com/en/products/proguard)：用于提出无用的 Java 代码并且做一些优化；
- [DX](https://en.wikipedia.org/wiki/Dalvik_(software))：将所有的 Java 代码转换为 DEX 格式。

在 Android Studio 3.X 以后，Google 分别引入 D8 编译器和 R8 工具作为新的 DEX 编译器和混淆压缩工具。

#### 1. D8、R8 发展历程

| Android Studio 版本 | Android Gradle Plugin 版本 |           变更           |
| :-----------------: | :------------------------: | :----------------------: |
|         3.1         |           3.0.1            |         引入 D8          |
|         3.2         |           3.2.0            | 引入 R8、D8 脱糖默认开启 |
|         3.4         |           3.4.0            |       默认开启 R8        |

##### 1.1 D8 编译器

Google 在 Android Studio 3.1 版本中引入 D8 编译器作为默认的 DEX 字节码文件编译器。通过在 `gradle.properties` 中新增 `android.enableD8=true` 开启 D8 编译器。

D8 编译器特点是：

- 编译更快、时间更短；
- DEX 编译时占用内容更小；
- `.dex` 文件大小更小；
- D8 编译的 `.dex` 文件拥有相同或者是更好的运行时性能；

**根据 Google Android 团队使用 Dex 与 D8 编译器的测试对比数据：**

![dex time](/Users/dengshiwei/Desktop/D8R8/d8 time.png)

![dex size](/Users/dengshiwei/Desktop/D8R8/d8 size.png)

##### 1.2 R8 工具

Google 在 Android Studio 3.2 中引入 R8 作为 ProGuard 的替代工具，用于代码的压缩（shrinking）和混淆（obfuscation）。通过在 `gradle.properties` 中新增 `android.enableR8 = true` 开启 R8 工具。

```java
# Disables R8 for Android Library modules only.
android.enableR8.libraries = false
# Disables R8 for all modules.
android.enableR8 = false
```

##### 1.3 Android Studio 3.4 版本 D8 R8 变更

在 Android Studio 3.4 版本中，R8 把 desugaring、shrinking、obfuscating、optimizing 和 dexing 都合并到一步进行执行。在 Android Studio 3.4 以前的版本编译流程如下：

![整合前](/Users/dengshiwei/Desktop/D8R8/compile_with_d8_proguard.png)

合并之后编译流程如下：

![整合后](/Users/dengshiwei/Desktop/D8R8/compile_with_r8.png)

注意，如果我们在 build.gradle 中配置了 `useProguard = false` 则不管是否开启 R8 编译都会使用 R8 进行压缩代码。

#### 2. 脱糖

Android Studio 为使用部分 Java 8 语言功能及利用这些功能的第三方库提供内置支持。默认工具链对 javac 编译器的输出执行字节码转换（称为 desugar），从而实现新语言功能。

**脱糖**即在编译阶段将在语法层面一些底层字节码不支持的特性转换为基础的字节码结构，(比如 List 上的泛型脱糖后在字节码层面实际为 Object)； Android 工具链对 Java8 语法特性脱糖的过程可谓丰富多彩，当然他们的最终目的是一致的：使新的语法可以在所有的设备上运行。

经过上面 D8、R8 的了解，D8 已经支持脱糖，让 Java 8 提供的特性（如 lambdas）可以转换成 Java 7 特性。把脱糖步骤集成进 D8 影响了所有读或写 .class 字节码的开发工具，因为它会使用 Java 8 格式。可以在 `gradle.properties` 中禁用脱糖。

```java
android.enableIncrementalDesugaring=false.
android.enableDesugar=false
```

##### 2.1 Lambda 表达式

Java 8 中一个重大变更是引入 Lambda 表达式。

```java
public class Lambda {
    public static void main(String[] args) {
        logDebug(msg-> System.out.println(msg), "HelloWorld");
    }

    static void logDebug(Logger logger, String msg) {
        logger.log(msg);
    }

    @FunctionalInterface
    interface Logger {
        void log(String msg);
    }
}
```

通过 `javac` 指令将上面的 Lambda.java 转换为字节码。

```bash
$javac Lambda.java
```

接下来通过 `javap -v` 指令查看字节码的详细内容：

```bas
dengshiweideMacBook-Pro:Desktop dengshiwei$ javap -v Lambda.class
Classfile /Users/dengshiwei/Desktop/Lambda.class
  Last modified 2019-5-12; size 1120 bytes
  MD5 checksum 67301305720e60d4ef1ff286769768e6
  Compiled from "Lambda.java"
public class Lambda
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #9.#25         // java/lang/Object."<init>":()V
   #2 = InvokeDynamic      #0:#30         // #0:log:()LLambda$Logger;
   #3 = String             #31            // HelloWorld
   #4 = Methodref          #8.#32         // Lambda.logDebug:(LLambda$Logger;Ljava/lang/String;)V
   #5 = InterfaceMethodref #10.#33        // Lambda$Logger.log:(Ljava/lang/String;)V
   #6 = Fieldref           #34.#35        // java/lang/System.out:Ljava/io/PrintStream;
   #7 = Methodref          #36.#37        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #8 = Class              #38            // Lambda
   #9 = Class              #39            // java/lang/Object
  #10 = Class              #40            // Lambda$Logger
  #11 = Utf8               Logger
  #12 = Utf8               InnerClasses
  #13 = Utf8               <init>
  #14 = Utf8               ()V
  #15 = Utf8               Code
  #16 = Utf8               LineNumberTable
  #17 = Utf8               main
  #18 = Utf8               ([Ljava/lang/String;)V
  #19 = Utf8               logDebug
  #20 = Utf8               (LLambda$Logger;Ljava/lang/String;)V
  #21 = Utf8               lambda$main$0
  #22 = Utf8               (Ljava/lang/String;)V
  #23 = Utf8               SourceFile
  #24 = Utf8               Lambda.java
  #25 = NameAndType        #13:#14        // "<init>":()V
  #26 = Utf8               BootstrapMethods
  #27 = MethodHandle       #6:#41         // invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #28 = MethodType         #22            //  (Ljava/lang/String;)V
  #29 = MethodHandle       #6:#42         // invokestatic Lambda.lambda$main$0:(Ljava/lang/String;)V
  #30 = NameAndType        #43:#44        // log:()LLambda$Logger;
  #31 = Utf8               HelloWorld
  #32 = NameAndType        #19:#20        // logDebug:(LLambda$Logger;Ljava/lang/String;)V
  #33 = NameAndType        #43:#22        // log:(Ljava/lang/String;)V
  #34 = Class              #45            // java/lang/System
  #35 = NameAndType        #46:#47        // out:Ljava/io/PrintStream;
  #36 = Class              #48            // java/io/PrintStream
  #37 = NameAndType        #49:#22        // println:(Ljava/lang/String;)V
  #38 = Utf8               Lambda
  #39 = Utf8               java/lang/Object
  #40 = Utf8               Lambda$Logger
  #41 = Methodref          #50.#51        // java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #42 = Methodref          #8.#52         // Lambda.lambda$main$0:(Ljava/lang/String;)V
  #43 = Utf8               log
  #44 = Utf8               ()LLambda$Logger;
  #45 = Utf8               java/lang/System
  #46 = Utf8               out
  #47 = Utf8               Ljava/io/PrintStream;
  #48 = Utf8               java/io/PrintStream
  #49 = Utf8               println
  #50 = Class              #53            // java/lang/invoke/LambdaMetafactory
  #51 = NameAndType        #54:#57        // metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #52 = NameAndType        #21:#22        // lambda$main$0:(Ljava/lang/String;)V
  #53 = Utf8               java/lang/invoke/LambdaMetafactory
  #54 = Utf8               metafactory
  #55 = Class              #59            // java/lang/invoke/MethodHandles$Lookup
  #56 = Utf8               Lookup
  #57 = Utf8               (Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
  #58 = Class              #60            // java/lang/invoke/MethodHandles
  #59 = Utf8               java/lang/invoke/MethodHandles$Lookup
  #60 = Utf8               java/lang/invoke/MethodHandles
{
  public Lambda();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: invokedynamic #2,  0              // InvokeDynamic #0:log:()LLambda$Logger;
         5: ldc           #3                  // String HelloWorld
         7: invokestatic  #4                  // Method logDebug:(LLambda$Logger;Ljava/lang/String;)V
        10: return
      LineNumberTable:
        line 3: 0
        line 4: 10

  static void logDebug(Lambda$Logger, java.lang.String);
    descriptor: (LLambda$Logger;Ljava/lang/String;)V
    flags: ACC_STATIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: invokeinterface #5,  2            // InterfaceMethod Lambda$Logger.log:(Ljava/lang/String;)V
         7: return
      LineNumberTable:
        line 7: 0
        line 8: 7
}
SourceFile: "Lambda.java"
InnerClasses:
     static #11= #10 of #8; //Logger=class Lambda$Logger of class Lambda
     public static final #56= #55 of #58; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
BootstrapMethods:
  0: #27 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #28 (Ljava/lang/String;)V
      #29 invokestatic Lambda.lambda$main$0:(Ljava/lang/String;)V
      #28 (Ljava/lang/String;)V

```

在上面的详细字节码信息中，有两个重要信息需要我们注意：

**major version: 52**

主版本号是 52 对应的是 JDK 1.8。

**invokedynamic 指令**

invokedynamic 在 Java7 开始提出来，但是实际上 javac 并不支持生成 invokedynamic。在 Java 8 中，`javac` 能够生成 invokedynamic 指令, 比如 lambda 表达式。 

##### 2.2 dx 指令打包

首先我们采用 `buildToolsVersion = 22.0.1` 版本来编译上面的 class 文件。

```bash
$/Users/dengshiwei/Library/Android/sdk/build-tools/22.0.1/dx  --dex *.class
```

咦？出错了！

```bash
UNEXPECTED TOP-LEVEL EXCEPTION:
com.android.dx.cf.iface.ParseException: bad class file magic (cafebabe) or version (0034.0000)
	at com.android.dx.cf.direct.DirectClassFile.parse0(DirectClassFile.java:472)
	at com.android.dx.cf.direct.DirectClassFile.parse(DirectClassFile.java:406)
	at com.android.dx.cf.direct.DirectClassFile.parseToInterfacesIfNecessary(DirectClassFile.java:388)
	at com.android.dx.cf.direct.DirectClassFile.getMagic(DirectClassFile.java:251)
	at com.android.dx.command.dexer.Main.processClass(Main.java:704)
	at com.android.dx.command.dexer.Main.processFileBytes(Main.java:673)
	at com.android.dx.command.dexer.Main.access$300(Main.java:83)
	at com.android.dx.command.dexer.Main$1.processFileBytes(Main.java:602)
	at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:170)
	at com.android.dx.cf.direct.ClassPathOpener.process(ClassPathOpener.java:144)
	at com.android.dx.command.dexer.Main.processOne(Main.java:632)
	at com.android.dx.command.dexer.Main.processAllFiles(Main.java:510)
	at com.android.dx.command.dexer.Main.runMonoDex(Main.java:280)
	at com.android.dx.command.dexer.Main.run(Main.java:246)
	at com.android.dx.command.dexer.Main.main(Main.java:215)
	at com.android.dx.command.Main.main(Main.java:106)
...while parsing Lambda$Logger.class

```

`bad class file magic` 魔数无法识别，原因是由于 JDK 和 dx 的版本不匹配导致的。同时 `version (0034.0000)` 版本转换为十进制就是 52 即对应 JDK 1.8。这里我们更改版本为：`28.0.1`。

```bash
dengshiweideMacBook-Pro:dex dengshiwei$ /Users/dengshiwei/Library/Android/sdk/build-tools/28.0.1/dx  --dex *.class
Uncaught translation error: com.android.dx.cf.code.SimException: ERROR in Lambda.main:([Ljava/lang/String;)V: invalid opcode ba - invokedynamic requires --min-sdk-version >= 26 (currently 13)
1 error; aborting
```

这是因为 lamda 表达式在 Java 字节码层面使用了 **invokedynamic** 指令 ，而 Android 对字节码指令 **invokedynamic** 在设备sdk 版本大于 26 才支持。那么 Android 如何实现对所有设备 api 版本的 lambda 函数的支持呢？Android 是通过**脱糖**的方式来实现。

##### 2.3 D8 指令打包

同样是针对上面的文件，通过 `D8` 指令编译。

```bash
/Users/dengshiwei/Library/Android/sdk/build-tools/28.0.1/d8 *.class
```

![dex](/Users/dengshiwei/Desktop/D8R8/d8.png)

### 3. 总结

- D8 编译器作为默认的 DEX 字节码文件编译器，具有更好的性能；
- R8 作为 ProGuard 的替代工具，用于代码的压缩（shrinking）和混淆（obfuscation）；
- 脱糖用于在 Android 中支持 Java 8 部分属性。



**参考**

[深入Android对Java8支持的实现](https://juejin.im/post/5cbe60796fb9a0324d43ab97#heading-12)