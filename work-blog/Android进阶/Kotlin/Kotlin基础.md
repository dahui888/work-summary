### Kotlin基础
![Kotlin](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Kotlin/Kotlin.png)

#### 一、概述
Kotlin是一种在Java虚拟机上运行的静态类型变成语言，被称之为Android世界的Swift，由JetBrains设计开发并开源。

Kotlin可以编译成Java字节码，也可以编译成JavaScript，方便在没有JVM的设备上运行。在Google I/O 2017中，Google宣布Kotlin成为Android官方开发语言。

Kotlin程序文件以.kt结尾，如：hello.kt。Kotlin的优点：
- 简洁：大大减少样板代码的数量
- 安全：避免空指针异常等整个类的错误
- 互操作性：充分利用JVM、Android和浏览器的现有库
- 工具友好：可用任何Java IDE或者使用命令行构建

#### 二、开发环境搭建

** IntelliJ IDEA环境搭建**
有过使用Android Studio的经验的都知道，可以在设置里面的插件中搜索Kotlin插件，然后安装使用。具体步骤：Settings——>Plugins——>Install JetBrains plugins——>搜索Kotlin。

![KotlinSettings](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Kotlin/KotlinSetting.png)

![KotlinInstall](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Kotlin/KotlinInstall.png)

**Kotlin Eclipse 环境搭建**
Eclipse 通过 Marketplace 安装 Kotlin 插件，打开 Eclipse，选择 Help -> Eclipse Marketplace… 菜单，搜索 Kotlin 插件：

![Eclipse](https://github.com/dengshiwei/work-summary/blob/master/work-blog/Android%E8%BF%9B%E9%98%B6/Kotlin/Eclipse.png)

**Kotlin Android 环境搭建**
我们知道Android Studio也是基于IntelliJ IDELA开发组建而来，所以我们也是同样搜索Kotlin插件集成。

#### 三、Kotlin基础语法
Kotlin文件以.kt结尾。

##### 1、包声明
包名一般在Kotlin文件的开头进行声明。
```java
package com.dsw.main

fun main(args: Array<String>) {
    println("Hello World!")
}
```
Kotlin源文件不需要匹配相互匹配的目录和包名，源文件可以放在任何文件目录。如果没有指定包名，默认为default包。

在创建一个Java文件时，系统会默认给我们导入很多包。同样，在Kotlin文件中也会默认导入：
- kotlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.*
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*

##### 2、函数的定义
在Java中，我们通过：[修饰符] 返回值类型 方法名(参数)格式来定义一个函数。在Python中通过def来定义函数，那么在Kotlin中以关键字fun来定义函数，参数格式为：参数：类型，比如a:Int。
```java
fun sum(a : Int, b : Int) : Int{
    return a + b
}
```
上面的函数定义格式基本跟Java类似，既然Kotlin以简洁为长，那么我们看看还可以直接这么写。
```java
fun sum(a : Int, b : Int) = a + b
```
直接将**返回值表达式跟函数体卸载一起，同时省去返回值类型，返回值类型自动推断。**public 方法则必须明确写出返回类型
```java
public fun sum(a: Int, b: Int): Int = a + b
```
**无返回值类型的函数**
```java
fun printSum(a: Int, b: Int): Unit { 
    print(a + b)
}


// 如果是返回 Unit类型，则可以省略(对于public方法也是这样)：
public fun printSum(a: Int, b: Int) { 
    print(a + b)
}
```
**可变长参数函数**
函数的可变长参数用**vararg**关键字进行声明。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
   print(sum(1,2,3,100))
}

fun sum(vararg v : Int) : Int{
    var sum = 0
   for (vi in v){
       sum += vi
   }
    return sum
}
```
**lambda函数**
lambda表达式使用示例：
```java
// 测试
fun main(args: Array<String>) {
    val sumLambda: (Int, Int) -> Int = {x,y -> x+y}
    println(sumLambda(1,2))  // 输出 3
}
```
##### 3、变量和常量的定义
在Kotlin中通过var关键字定义可变的变量，val关键字定义不常量。
```java
var <标识符> : <类型> = <初始化值>
val <标识符> : <类型> = <初始化值>
```
**常量与变量都可以没有初始化值,但是在引用前必须初始化。编译器支持自动类型判断,即声明时可以不指定类型,由编译器判断。**
```java
val a: Int = 1
val b = 1       // 系统自动推断变量类型为Int
val c: Int      // 如果不在声明时初始化则必须提供变量类型
c = 1           // 明确赋值


var x = 5        // 系统自动推断变量类型为Int
x += 1           // 变量可修改
```
##### 4、字符串模板
在Java中我们通过format方法使用可以有字符串占位，同样在Kotlin中通过字符串模板实现：
- $ 表示一个变量名或者变量值
- $varName 表示变量值
- ${varName.fun()} 表示变量的方法返回值

```java
fun main(args: Array<String>) {
    var a = 1;
    var b = "a is $a"
    print(b)//输出a is 1
}
```

##### 5、NULL检查机制
在Java语言开发中，我们经常会遇到NULLPointException的错误，而Kotlin语言就可以很好的解决这个问题。在Kotlin语言中，将系统中的类型分为可空类型和非空类型。String为不可空类型，String?为可空类型，如果将不可空类型赋值为null将会编译不通过。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
    var b : String? = "abc"
    print(b.length)
}
```
在上面的代码中，我们声明b是可空的，所以这样编译就不会通过，尽管它实际并没有为空。编译会提示：
```java
Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?
```
这句话的意思是我们可以使用**?.或!!.**来引用可为空对象。
```java
fun main(args: Array<String>) {
    var b : String? = "abc"
    println(b?.length)
    println(b!!.length)
}
```

##### 6、类型检测及自动类型转换
Java中我们通过instance of来检测一个对象是否是某种类型，在Kotlin中通过is来进行检测。
```java
fun getStringLength(obj: Any): Int? {
    //obj在&&右边自动动转换成"String"类型
    if (obj is String && obj.length > 0)
        return obj.length
    return null
}
```
##### 7、区间
区间表达式由具有操作符形式 .. 的 rangeTo 函数辅以 in 和 !in 形成。step关键字用来指定步长，until关键字用来排除结束元素。

区间是为任何可比较类型定义的，但对于整型原生类型，它有一个优化的实现。以下是使用区间的一些示例:
```java
for (i in 1..4) print(i) // 输出“1234”

for (i in 4..1) print(i) // 什么都不输出

if (i in 1..10) { // 等同于 1 <= i && i <= 10
    println(i)
}

// 使用 step 指定步长
for (i in 1..4 step 2) print(i) // 输出“13”

for (i in 4 downTo 1 step 2) print(i) // 输出“42”


// 使用 until 函数排除结束元素
for (i in 1 until 10) {   // i in [1, 10) 排除了 10
     println(i)
}
```
#### 四、Kotlin基本数据类型
Kotlin 的基本数值类型包括 Byte、Short、Int、Long、Float、Double 等。不同于Java的是，字符不属于数值类型，是一个独立的数据类型。

Kotlin中没有基础数据类型，只有封装的数字类型，每定义一个变量，Kotlin就自动封装一个对象。**在Kotlin中三个===表示比较对象的地址，两个==表示比较数值大小。**

##### 1、类型转换
这里同Java一样，高级数据类型可以隐式转换为低级数据类型，低级数据类型无法隐式	转换到高级数据类型。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
   val b: Byte = 1 // OK, 字面值是静态检测的
   val i: Int = b // 错误
}
```
每种数据类型都有下面的方法，可以转化为其它的数据类型：
- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double
- toChar(): Char

##### 2、位操作符
在Kotlin中提供了同Java一样的位操作符。
- shl(bits)：左移位（同Java<<）
- shr(bits)：右移位（同Java>>）
- ushr(bits)：无符号右移位（同Java>>>）
- and(bits)：与（同Java的&）
- or(bits)：或（同Java的|）
- xor(bits)：异或（同Java的^）
- inv()：取反（同Java的~）

```java
package com.andoter.dsw
fun main(args: Array<String>) {
   var index = 4
   println("index.shl:" + index.shl(1))
   println("index.shr:" + index.shr(1))
   println("index.and(0):" + index.and(0))
   println("index.or(0):" + index.or(1))
   println("index.xor(0):" + index.xor(1))
   println("index.inv():" + index.inv())
}

执行结果：
index.shl:8
index.shr:2
index.and(0):0
index.or(0):5
index.xor(0):5
index.inv():-5
```

##### 3、字符类型
Char类型是用''包含起来的字符，比如'1'，'a'。字符字面值用单引号括起来: '1'。 特殊字符可以用反斜杠转义。 支持这几个转义序列：\t、 \b、\n、\r、\'、\"、\\ 和 \$。 编码其他字符要用 Unicode 转义序列语法：'\uFF00'。

##### 4、布尔类型
布尔用 Boolean 类型表示，它有两个值：true 和 false。内置的布尔运算有：
- || – 短路逻辑或
- && – 短路逻辑与
- ! - 逻辑非

##### 5、数组
数组用类 Array 实现，并且还有一个 size 属性及 get 和 set 方法，由于使用 [] 重载了 get 和 set 方法，所以我们可以通过下标很方便的获取或者设置数组对应位置的值。数组的创建方式：
- 一种是通过arrayOf()
- 一种是通过Array工厂函数

```java
package com.andoter.dsw
fun main(args: Array<String>) {
   var array1 = intArrayOf(1,2,3)
   var array2 = IntArray(3,{i -> i*3 })
   printArray(array1)
   printArray(array2)
}
fun printArray(array : IntArray){
   for(arr in array){
      System.out.print(arr)
   }
}
```

##### 6、字符串
和 Java 一样，String 是可不变的。方括号 [] 语法可以很方便的获取字符串中的某个字符，也可以通过 for 循环来遍历。
```java
for (c in str) {
    println(c)
}
```
Kotlin 支持三个引号 """ 扩起来的字符串，支持多行字符串，比如：
```java
fun main(args: Array<String>) {
    val text = """
    多行字符串
    多行字符串
    """
    println(text)   // 输出有一些前置空格
}
```

#### 五、条件控制与循环
##### 1、if表达式
一个 if 语句包含一个布尔表达式和一条或多条语句。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
   var max = 5
   if(max > 3){
      print(">3")
   }
}
```
IF表达式的结果也可以复制给一个变量。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
    var a = 1
    var b = 2
    val c = if (a>=b) a else b
    print(c)
}
```
**使用区间**
使用 in 运算符来检测某个数字是否在指定区间内，区间格式为 x..y 。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
    var x = 4
    if(x in 1..6){
        print("在区间内")
    }
}
```
##### 2、When表达式
when 将它的参数和所有的分支条件顺序比较，直到某个分支满足条件。

when 既可以被当做表达式使用也可以被当做语句使用。如果它被当做表达式，符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。when 类似其他语言的 switch 操作符。
```java
fun main(args: Array<String>) {
    var x = 5
    when(x){
        2-> print("2")
        4-> print("4")
        else-> print("else")
    }
}
```
在 when 中，else 同 switch 的 default。如果其他分支都不满足条件将会求值 else 分支。

如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔：
```java
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```
在Kotlin中通过in或!in来判断一个值是否在某个区间，同样可以配合when进行使用。
```java
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```
##### 3、For循环
for循环可以对任何提供迭代器的对象进行遍历。
```java
for (item in collection) print(item)
```
如果要遍历一个数组或者集合，也可以这样：
```java
for (i in array.indices) {
    print(array[i])
}
```
比如下面对集合的遍历：
```java
fun main(args: Array<String>) {
    var items = listOf<String>("apple","banana")
    for (item in items){
        print(item)
    }
}
```
##### 4、while与do...while循环
while是最基本的循环，它的结构为：
```java
while( 布尔表达式 ) {
  //循环内容
}
```
do…while 循环 对于 while 语句而言，如果不满足条件，则不能进入循环。但有时候我们需要即使不满足条件，也至少执行一次。

do…while 循环和 while 循环相似，不同的是，do…while 循环至少会执行一次。
```java
do {
       //代码语句
}while(布尔表达式);
```
##### 5、循环的返回与跳转
Java中有break、continue用于循环的跳转，同样Kotlin中也提供了三种结构化跳转表达式：
- return：直接返回保卫他的函数
- break：终止当前外围循环
- continue：结束当前此次循环，进行下一次。

```java
fun main(args: Array<String>) {
    for (i in 1..10) {
        if (i==3) continue  // i 为 3 时跳过当前循环，继续下一次循环
        println(i)
        if (i>5) break   // i 为 6 时 跳出循环
    }
}
```
#### 六、Kotlin类和对象
Java是面向对象编程，同样Kotlin也有类型，Kotlin类主要包含：构造函数和初始化代码块、函数、属性、对象声明。

Kotlin中通过关键字class声明类，后面紧跟类名：
```java
class Andoter{
    
}
```
**可见性修饰符**
类、对象、接口、构造函数、方法、属性和它们的 setter 都可以有_可见性修饰符_。 （getter 总是与属性有着相同的可见性。） 在 Kotlin 中有这四个可见性修饰符：private、 protected、 internal 和 public。 如果没有显式指定修饰符的话，默认可见性是 public。

- 如果你不指定任何可见性修饰符，默认为 public，这意味着你的声明 将随处可见；
- 如果你声明为 private，它只会在声明它的文件内可见；
- 如果你声明为 internal，它会在相同模块内随处可见；
- protected 不适用于顶层声明。

注意 对于Java用户：Kotlin 中外部类不能访问内部类的 private 成员。

**类的属性**
Kotlin中变量通过var或val定义，同样类的属性也属于变量一族，只需要将成员变量定义成一个变量，默认是 public 类型的。
```java
class Andoter{
    var name:String = "andoter"
    var city:String = "HeFei"
}
```
在这里如果不对属性进行实例化，编译器会给我们提示：
```java
Property must be initialized or be abstract
```
我们需要制定进行初始化，或者将该变量定义成abstract类型。
```java
abstract class Andoters{
    abstract var name :String
    abstract var city : String
    constructor()
}
```

**getter和setter**
属性的getter和setter声明语法：
```java
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```
比如：
```java
class Andoter{
    var name:String
        get() = name
        set(value){
            name = value
        }
    var city:String = "HeFei"
}
```
getter 和 setter 都是可选，并且getter、setter的访问限制级必须和变量是一致的，这点很让人费解，比如变量默认是public，你还定义setter和getter有啥用，直接对象就引用到了。

如果属性类型可以从初始化语句或者类的成员函数中推断出来，那就可以省去类型，val不允许设置setter函数，因为它是只读的。

**主构造器**

主构造函数是类头的一部分，类名的后面跟上构造函数的关键字constructor以及类型参数。主构造器中不能包含任何代码，初始化代码可以放在初始化代码段中，初始化代码段使用 init 关键字作为前缀。
```java
fun main(args: Array<String>) {
    var andoter = Andoter("andoter学习笔记")
    print(andoter.name)
}

class Andoter constructor(name : String){
    var name:String = "anodter"
    var city:String = "HeFei"

    init {
        this.name = name
    }
}
```
同时针对constructor关键字，也可以省略不写，那什么时候可以省略constructor关键字呢？
- 在构造函数不具有朱师傅或者默认的可见修饰符时，constructor关键字可以省略
- 默认的可见修饰符public，可以省略不写。

```java
// 类似下面两种情况的，都必须存在constructor关键字，并且在修饰符或者注释符后面。
class Test private constructor(num: Int){
}

class Test @Inject constructor(num: Int){
}
```
**辅助（二级）构造函数**

Kotlin中支持二级构造函数。它们以constructor关键字作为前缀。
```java
class Andoters{
    var name :String = "Andoter"
    var city : String = "HeFei"
    constructor(name: String){
        this.name = name
    }
}
```
如果类有主构造函数，每个次构造函数都要，或直接或间接通过另一个次构造函数代理主构造函数。在同一个类中代理另一个构造函数使用 this 关键字。
```java
package com.andoter.dsw
fun main(args: Array<String>) {
    var andoter = Andoter("andoter学习笔记")
    println(andoter.name)
    var andoters = Andoter("Andoter学习笔记","AnhuiHefei")
    print(andoters.name + andoters.city)
}

class Andoter constructor(name : String){
    var name:String = "anodter"
    var city:String = "HeFei"

    init {
        this.name = name
    }

    constructor(name :String, city : String): this(name){
        this.name = name
        this.city = city
    }
}
```
#### 七、抽象类
抽象是面向对象编程的特征之一，类本身，或类中的部分成员，都可以声明为abstract的。抽象成员在类中不存在具体的实现。
注意：无需对抽象类或抽象成员标注open注解。
```java
abstract class Developer{
    abstract var name:String
    abstract var developDirection : String
}

class Andoter(override var name: String, override var developDirection: String) : Developer(){
    var skill : String = "Android"

    init {
        this.name = name
        this.developDirection = developDirection
    }
}
```
#### 八、嵌套类
```java
class Outer {                  // 外部类
    private val bar: Int = 1
    class Nested {             // 嵌套类
        fun foo() = 2
    }
}

fun main(args: Array<String>) {
    val demo = Outer.Nested().foo() // 调用格式：外部类.嵌套类.嵌套类方法/属性
    println(demo)    // == 2
}
```
#### 九、内部类
内部类使用 inner 关键字来表示。

内部类会带有一个对外部类的对象的引用，所以内部类可以访问外部类成员属性和成员函数。
```java
class Outer{
    private var outermem :String = "Andoter"
    var v = "成员属性"
    inner class Inner{
        fun foo() = outermem
    }
}
```

#### 九、Kotlin继承
Kotlin 中所有类都继承该 Any 类，它是所有类的超类，对于没有超类型声明的类是默认超类。
```java
class Example // 从 Any 隐式继承
```
Any 默认提供了三个函数：
```java
equals()

hashCode()

toString()
```
注意：Any 不是 java.lang.Object。
如果一个类要被继承，可以使用 open 关键字进行修饰，不然该类是无法被别的Kotlin类继承的。

**如果子类有主构造函数， 则基类必须在主构造函数中立即初始化。**
```java
open class Person(var name : String, var age : Int){// 基类

}

class Student(name : String, age : Int, var no : String, var score : Int) : Person(name, age) {

}

// 测试
fun main(args: Array<String>) {
    val s =  Student("Runoob", 18, "S12346", 89)
    println("学生名： ${s.name}")
    println("年龄： ${s.age}")
    println("学生号： ${s.no}")
    println("成绩： ${s.score}")
}
```

**如果子类没有主构造函数，则必须在每一个二级构造函数中用 super 关键字初始化基类，或者在代理另一个构造函数。初始化基类时，可以调用基类的不同构造方法。**
```java
class Student : Person {

    constructor(ctx: Context) : super(ctx) {
    } 

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx,attrs) {
    }
}
```
在基类中，使用fun声明函数时，此函数默认为final修饰，不能被子类重写。如果允许子类重写该函数，那么就要手动添加 open 修饰它, 子类重写方法使用 override 关键词：
```java
/**用户基类**/
open class Person{
    open fun study(){       // 允许子类重写
        println("我毕业了")
    }
}

/**子类继承 Person 类**/
class Student : Person() {

    override fun study(){    // 重写方法
        println("我在读大学")
    }
}

fun main(args: Array<String>) {
    val s =  Student()
    s.study();

}
```
属性重写使用 override 关键字，属性必须具有兼容类型，每一个声明的属性都可以通过初始化程序或者getter方法被重写：
```java
open class Foo {
    open val x: Int get { …… }
}

class Bar1 : Foo() {
    override val x: Int = ……
}
```

清明节放假因为车票买错了，也没回家。这两天花了一天的时间在菜鸟教程中把Kotlin的基础给看了下，总体来说是一副Java的语法糖，更像脚本语言的形式。