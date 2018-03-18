### 第五章：泛型
>泛型作为Java语言在JDK1.5中引入的一个重要特性，为我们的开发提供了极大的方便。在Effective Java中的第五章也单独把泛型拿出来单独总结，很有必要学习一波。

在前面的笔记中，我们针对泛型的基本使用进行了学习总结，泛型有泛型类、泛型接口、泛型方法。泛型只是在编译器进行类型检查，在运行期通过类型擦除本质上是一样的。

##### 第一条：不要在新代码中使用原生类型
Java在jdk1.5中引入了泛型，那么我们就应该避免使用原生类型来指定，如果使用原生态类型，就失掉了泛型在安全性和表属性方面的所有优势。
```java
static int matchSets(Set s1, Set s2){
    int result = 0;
    for(Object obj : s1){
        if(s2.contains(obj))result++;
    }
    return result;
}
```
在上面的一小段示例代码中，这里使用的就是原生类型Object，尽管是可行的，但是非常危险。Java就提供了一种安全的替代方法，称之为无限制的通配符类型。如果要使用泛型，单不确定或者不关心实际的类型参数，就可以使用一个？hao号替代，例如泛型Set< E >的无限制通配符类型为Set < ? >。

| 术语 | 示例 |
|--------|--------|
|参数化的类型        |     List<String>   |
|实际类型参数        |     String   |
|泛型        |     List<E>   |
|形式类型参数        |     E   |
|无限制通配符类型        |     List< ? >   |
|原生态类型        |     List   |
|有限制类型参数        |     List< E  extends Number >   |

##### 第二条：清除非受检查警告
在刚开始使用泛型的时候难免会出现很多警告，要清除这些警告，如果无法清除警告，要在确保引起警告的代码是安全的前提下使用@SuppressWarnings("unchecked")注解来禁止这条警告，同时添加一条注释来说明为什么是安全的。

##### 第三条：列表优先于数组
数组与泛型相比，有连个重要的不通电。首先数组是协变的，比如Sub是Super的子类型，那么对于数组类型Sub[]就是Super[]的子类型，反之对于列表任意两个不同的类型Type1和Type2，List< Type1 > 不是List< Type2 >的子类型 ，List< Type2 > 不是List< Type1 >的子类型 。

##### 第四条：优先考虑泛型
一般来说，将集合声明参数化，以及使用JDK所提供的泛型和泛型方法。在实际的开发工作中，如何设计实现一个泛型类是我们需要考虑的重点。

例如下面的简单堆栈的时间：
```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITAL_CAPACITY = 16;
	
	public Stack(){
		elements = new Object[DEFAULT_INITAL_CAPACITY];
	}
	
	public void push(Object obj){
		ensureCapacity();
		elements[size++] = obj;
	}
	
	public Object pop(){
		if(size ==0)throw new EmptyStackException();
		Object obj = elements[size--];
		elements[size] = null;
		return obj;
	}

	private void ensureCapacity() {
		if(elements.length >= DEFAULT_INITAL_CAPACITY){
			elements = Arrays.copyOf(elements, 2*size + 1);
		}
	}
}
```
在上面的例子上我们可以看到是泛型化的备选对象，这里使用的是Object类，我们可以使用泛型强化这个类的能力。
```java
public class Stack<T> {
	private T[] elements;
	private int size = 0;
	private static final int DEFAULT_INITAL_CAPACITY = 16;
	
	@SuppressWarnings("unchecked")
	public Stack(){
		elements = (T[]) new Object[DEFAULT_INITAL_CAPACITY];
	}
	
	public void push(T obj){
		ensureCapacity();
		elements[size++] = obj;
	}
	
	public T pop(){
		if(size ==0)throw new EmptyStackException();
		T obj = elements[size--];
		elements[size] = null;
		return obj;
	}

	private void ensureCapacity() {
		if(elements.length >= DEFAULT_INITAL_CAPACITY){
			elements = Arrays.copyOf(elements, 2*size + 1);
		}
	}
}
```
##### 第五条：优先考虑泛型方法
在上一条中，我们总结了类的泛型化，方法也是一样优先使用泛型化。静态工具方法特别适用于这种情况，比如Collections中的方法。
```java
/**
 * Returns a type-safe empty, immutable {@link List}.
 *
 * @return an empty {@link List}.
 * @since 1.5
 * @see #EMPTY_LIST
 */
@SuppressWarnings("unchecked")
public static final <T> List<T> emptyList() {
    return EMPTY_LIST;
}

/**
 * Returns a type-safe empty, immutable {@link Set}.
 *
 * @return an empty {@link Set}.
 * @since 1.5
 * @see #EMPTY_SET
 */
@SuppressWarnings("unchecked")
public static final <T> Set<T> emptySet() {
    return EMPTY_SET;
}
```

##### 第六条：利用有限制通配符提升API的灵活性
前面第三条中我们提及到对于列表任意两个不同的类型Type1和Type2，List< Type1 > 不是List< Type2 >的子类型 ，List< Type2 > 不是List< Type1 >的子类型 。

看下面的实例：
```java
interface Stacks<E>{
	void push(E e);
	boolean isEmpty();
	E pop();
}
```
在上面的API中，我们新增一个pushAll方法。
```java
void pushAll(Iterable<E> src);
```
这个方法在编译的时候能够通过，但是并不完善。如果src的类型跟堆栈的类型完全匹配就没问题，但是根据参数化类型是不可变的，即列表泛型不是XX的子类型，在使用的时候就会报错。好在这里就有了泛型的通配符概念。
```java
void pushAll(Iterable<? extends E> iterate);
```