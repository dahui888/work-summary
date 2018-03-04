### Java泛型浅谈
>泛型作为Java进阶必备知识，本文进行整理的学习笔记。

泛型作为Java1.5以后引入的一种编译时泛化机制给Java语言提供了很大的灵活性，泛型的表现是参数化类型，通过参数化类型提高代码的灵活性。

泛型的有点：
- 编译时检查，保证类型安全
- 消除类型检查，提高代码的可读性
- 提高了代码的通用性
- 提高了性能

#### 泛型的基本使用
在Java中，泛型可以用于类、接口和方法的修饰，分别对应三种方式：泛型类、泛型接口和泛型方法。

##### 泛型类
在Java的Api中最典型的泛型类代表就是List、Map集合。

**泛型类格式**
```java
class 类名称 <泛型标识>{
}
```
注意：泛型的类型参数只能是类对象型，不能是简单类型。

下面我们看一个简单的泛型类：
```java
public class GenericClass<T> {
	private T name;
	
	public GenericClass(T name){
		this.name = name;
	}
	
	public void printString(T str){
		System.out.println(str);
	}

	public T getName() {
		return name;
	}

	public void setName(T name) {
		this.name = name;
	}
}
```
##### 泛型接口
泛型接口的形式跟泛型类基本相同，在Java中典型的泛型接口就是Iterator。
```java
public interface Iterator<E> {
    /**
     * Returns true if there is at least one more element, false otherwise.
     * @see #next
     */
    public boolean hasNext();

    /**
     * Returns the next object and advances the iterator.
     *
     * @return the next object.
     * @throws NoSuchElementException
     *             if there are no more elements.
     * @see #hasNext
     */
    public E next();

    /**
     * Removes the last object returned by {@code next} from the collection.
     * This method can only be called once between each call to {@code next}.
     *
     * @throws UnsupportedOperationException
     *             if removing is not supported by the collection being
     *             iterated.
     * @throws IllegalStateException
     *             if {@code next} has not been called, or {@code remove} has
     *             already been called after the last call to {@code next}.
     */
    public void remove();
}
```
**泛型接口格式**
```java
interface 接口名称 <泛型标识>{
}
```
既然是接口，就少不了让别的类来实现。所以这里又会涉及到两种形态：
- 当实现泛型接口的类，未传入泛型实参，即形式：class ConcreteGeneric<T> implements Generic<T>{}

```java
interface Generic<T>{
	public T next();
}

class ConcreteGeneric<T> implements Generic<T>{

	@Override
	public T next() {
		return null;
	}
}
```
当在实现泛型接口没有指定具体的实参类型时，我们定义的类就是与泛型类的定义相同，在声明类的声明相同，需将泛型的声明也一起加到类中
- 当实现泛型接口的类，传入泛型实参，即形式：class ConcreteGeneric implements Generic<String>{}

```java
interface Generic<T>{
	public T next();
}

class ConcreteGeneric implements Generic<String>{

	@Override
	public String next() {
		// TODO Auto-generated method stub
		return null;
	}
}
```
在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型

##### 泛型方法
泛型方法是使用最多的一种泛型，使用的场景也比较多，比如泛型类中的泛型方法、普通类中的泛型方法、泛型方法与可变参数、静态方法与泛型等。

**泛型方法的格式**
泛型方法需要使用< T >来指明泛型格式。比如：
```java
public <K> void add(K name){
    System.out.print(name);
}
```
**泛型方法与可变参数**
```java
public <K> void printName(K... name){
    for(K k : name){
        System.out.println(k);
    }
}
```
**静态方法与泛型**
```java
class Generics<T>{	
	public static <T> void print(T msg){
		System.out.println(msg);
	}
}
```
#### 泛型的边界
Java中的泛型是在编译期起到类型检查作用，在运行期是不可见的。所以在使用泛型的时候，我们可以通过边界限定来限制实参的类型。

**泛型上边界**
泛型上边界规定传入的实参比继续是指定类型的字类型。
格式：*？extends 类型*

<? extends T> 表示类型的上界，表示参数化类型的可能是T 或是 T的子类

```java
public class Generic<T extends BoundingType>
```
如上面所示，传入的实参类型必须是BoundingType类型的子类。（BoundingType是一个类或者接口）。其中的BoundingType可以多于1个，用“&”连接即可。

**上界类型通配符add方法受限，但可以获取列表中的各种类型的数据，并赋值给父类型（extends Number）的引用。因此如果你想从一个数据类型里获取数据，使用 ? extends 通配符。限定通配符总是包括自己。**

**泛型下边界**

格式：*？supper 类型*

<? super T> 表示类型下界（Java Core中叫超类型限定），表示参数化类型是此类型的超类型（父类型），直至Object

**下界类型通配符get方法受限，但可以往列表中添加各种数据类型的对象。因此如果你想把对象写入一个数据结构里，使用 ? super 通配符。限定通配符总是包括自己。**

#### 通配符
在java泛型中由于针对实参类型有很多不确定性，这里就引出了通配符的概念。
- ? 通配符类型
- <? extends T> 表示类型的上界，表示参数化类型的可能是T 或是 T的子类
- <? super T> 表示类型下界（Java Core中叫超类型限定），表示参数化类型是此类型的超类型（父类型），直至Object

**通配符与T的区别**
- T：作用于模板上，用于将数据类型进行参数化，不能用于实例化对象。
- ?：在实例化对象的时候，不确定泛型参数的具体类型时，可以使用通配符进行对象定义。

**案例一：定义一个泛型类**
```java
class Generics<T extends BoundingType>{
	public void print(T msg){
		System.out.println(msg);
	}
}
```
在定义泛型类的模版中，就必须使用T，不能用通配符来代替，通配符用于实例化对象中。

**案例二：实例化泛型对象**

```java
List<? extends Number> list = null;
list = new ArrayList<Integer>();
list = new ArrayList<Long>();
```
我们不能够确定list存储的数据类型是Integer还是Long，因此我们使用List<? extends Number>定义变量的类型。

#### 泛型的原理
**Java会在编辑期把泛型擦除掉**
在JAVA的虚拟机中并不存在泛型，泛型只是为了完善java体系，增加程序员编程的便捷性以及安全性而创建的一种机制，在JAVA虚拟机中对应泛型的都是确定的类型，在编写泛型代码后，java虚拟机中会把这些泛型参数类型都擦除，用相应的确定类型来代替，代替的这一动作叫做类型擦除，而用于替代的类型称为原始类型，在类型擦除过程中，一般使用第一个限定的类型来替换，若无限定，则使用Object.

我们通过javap -c指令来查看一个.class文件
```java
public class com.dsw.test.GenericClass<T> {
  public com.dsw.test.GenericClass(T);
    Code:
       0: aload_0
       1: invokespecial #13                 // Method java/lang/Object."<init>":()V
       4: aload_0
       5: aload_1
       6: putfield      #16                 // Field name:Ljava/lang/Object;
       9: return

  public void printString(T);
    Code:
       0: getstatic     #25                 // Field java/lang/System.out:Ljava/io/PrintStream;
       3: aload_1
       4: invokevirtual #31                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
       7: return

  public T getName();
    Code:
       0: aload_0
       1: getfield      #16                 // Field name:Ljava/lang/Object;
       4: areturn

  public void setName(T);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #16                 // Field name:Ljava/lang/Object;
       5: return
}
```
可以看到泛型方法中的实际泛型参数在虚拟机中都是Object类型。这也印证了Java中的泛型是伪泛型。

#### 总结
在使用泛型的时候可以遵循一些基本的原则，从而避免一些常见的问题。

1. 在代码中避免泛型类和原始类型的混用。比如List<String>和List不应该共同使用。这样会产生一些编译器警告和潜在的运行时异常。当需要利用JDK 5之前开发的遗留代码，而不得不这么做时，也尽可能的隔离相关的代码。
1. 在使用带通配符的泛型类的时候，需要明确通配符所代表的一组类型的概念。由于具体的类型是未知的，很多操作是不允许的。
1. 泛型类最好不要同数组一块使用。你只能创建new List<?>[10]这样的数组，无法创建new List<String>[10]这样的。这限制了数组的使用能力，而且会带来很多费解的问题。因此，当需要类似数组的功能时候，使用集合类即可。
1. 不要忽视编译器给出的警告信息。

**参考资料**
1、[Java中泛型 类型擦除](https://www.cnblogs.com/drizzlewithwind/p/6101081.html)
2、[深入理解Java泛型](https://www.cnblogs.com/lucky_dai/p/5589317.html)