### ArrayList源码学习
ArrayList是我们常用的数据类型，是基于数组实现的非线程安全的动态数组结构。

先来引出常见的疑问：
1、ArrayList基于什么实现的。
2、ArrayList为什么是线程异步的，不安全的。
3、ArrayList的初始化大小是多少，如何增加的。

##### ArrayList的定义与创建
```java
public class ArrayList<E> extends AbstractList<E> implements Cloneable, Serializable, RandomAccess {
    /**
     * The minimum amount by which the capacity of an ArrayList will increase.
     * This tuning parameter controls a time-space tradeoff. This value (12)
     * gives empirically good results and is arguably consistent with the
     * RI's specified default initial capacity of 10: instead of 10, we start
     * with 0 (sans allocation) and jump to 12.
     */
    private static final int MIN_CAPACITY_INCREMENT = 12;

    /**
     * The number of elements in this list.
     */
    int size;

    /**
     * The elements in this list, followed by nulls.
     */
    transient Object[] array;

    /**
     * Constructs a new instance of {@code ArrayList} with the specified
     * initial capacity.
     *
     * @param capacity
     *            the initial capacity of this {@code ArrayList}.
     */
    public ArrayList(int capacity) {
        if (capacity < 0) {
            throw new IllegalArgumentException();
        }
        array = (capacity == 0 ? EmptyArray.OBJECT : new Object[capacity]);
    }

    /**
     * Constructs a new {@code ArrayList} instance with zero initial capacity.
     */
    public ArrayList() {
        array = EmptyArray.OBJECT;
    }

    /**
     * Constructs a new instance of {@code ArrayList} containing the elements of
     * the specified collection.
     *
     * @param collection
     *            the collection of elements to add.
     */
    public ArrayList(Collection<? extends E> collection) {
        Object[] a = collection.toArray();
        if (a.getClass() != Object[].class) {
            Object[] newArray = new Object[a.length];
            System.arraycopy(a, 0, newArray, 0, a.length);
            a = newArray;
        }
        array = a;
        size = a.length;
    }
    
......

}
```
**ArrayList的定义**
通过源码可以看到ArrayList集成AbstractList，并且实现了Cloneable、Serializable、RandomAcess接口，所以ArrayList是可复制、可序列化以及支持随机读取。同时可以看到ArrayList本质是通过Object[]数组来进行存储值。

通过源码我们可以看到ArrayList支持三种方式来创建：
- public ArrayList()：无参构造函数，创建一个大小为0的ArrayList对象
- public ArrayList(int capacity)：创建一个大小为capacity的ArrayList对象
- public ArrayList(Collection<? extends E> collection)：通过其它集合初始化元素创建

在介绍三种构造方法前，我们首先来看下EmptyArray这个类。
```java
public final class EmptyArray {
    private EmptyArray() {}

    public static final boolean[] BOOLEAN = new boolean[0];
    public static final byte[] BYTE = new byte[0];
    public static final char[] CHAR = new char[0];
    public static final double[] DOUBLE = new double[0];
    public static final int[] INT = new int[0];

    public static final Class<?>[] CLASS = new Class[0];
    public static final Object[] OBJECT = new Object[0];
    public static final String[] STRING = new String[0];
    public static final Throwable[] THROWABLE = new Throwable[0];
    public static final StackTraceElement[] STACK_TRACE_ELEMENT = new StackTraceElement[0];
}
```
我们可以理解EmptyArray是一个用来初始化常见数据类型的且大小为0的数组的工具类。

1. 无参构造函数的实质就是初始化一个大小为0的Object类型数组。

2. 指定大小的构造函数，首先会判断传入的capacity是否小于0，如果小于0则抛出IllegalArgumentException异常。如果等于0则EmptyArray.OBJECT，不等于0则创建new Object[capacity])大小的数组。

3. 指定集合的构造函数，如果集合不是Object类型，就通过System.arraycopy方法复制到临时数组中，然后在赋值给目标数组。

##### ArrayList添加元素
**public boolean add(E object)**
```java
    @Override public boolean add(E object) {
        Object[] a = array;
        int s = size;
        if (s == a.length) {
            Object[] newArray = new Object[s +
                    (s < (MIN_CAPACITY_INCREMENT / 2) ?
                     MIN_CAPACITY_INCREMENT : s >> 1)];
            System.arraycopy(a, 0, newArray, 0, s);
            array = a = newArray;
        }
        a[s] = object;
        size = s + 1;
        modCount++;
        return true;
    }
```
在add方法中，如果当前数组已经填满了，就会进行扩容。**扩容的规则是当前List的大小如果小于MIN_CAPACITY_INCREMENT / 2=6，则扩容数组大小为原大小+MIN_CAPACITY_INCREMENT，反之则原大小扩大1倍。**，然后创建新的Object数组，将旧元素先复制拷贝进去，最后赋值新元素。注意：返回值一直是true

**public void add(int index, E object)**
```java
@Override public void add(int index, E object) {
    Object[] a = array;
    int s = size;
    if (index > s || index < 0) {
        throwIndexOutOfBoundsException(index, s);
    }

    if (s < a.length) {
        System.arraycopy(a, index, a, index + 1, s - index);
    } else {
        // assert s == a.length;
        Object[] newArray = new Object[newCapacity(s)];
        System.arraycopy(a, 0, newArray, 0, index);
        System.arraycopy(a, index, newArray, index + 1, s - index);
        array = a = newArray;
    }
    a[index] = object;
    size = s + 1;
    modCount++;
}
```
可以看到首先判断index是否大于size或者<0，成立则抛出IndexOutOfBoundsException异常。然后判断当前size是否小于数组的长度，如果小于说明数组还没填满，则通过System.arraycopy方法进行拷贝到临时存储数组中，最后添加赋值的原数组。如果size已经等于数组大小，则通过newCapacity方法进行扩容。

```java
private static int newCapacity(int currentCapacity) {
    int increment = (currentCapacity < (MIN_CAPACITY_INCREMENT / 2) ?
            MIN_CAPACITY_INCREMENT : currentCapacity >> 1);
    return currentCapacity + increment;
}
```
**扩容的规则是当前List的大小如果小于MIN_CAPACITY_INCREMENT / 2=6，则扩容数组大小为原大小+MIN_CAPACITY_INCREMENT，反之则原大小扩大1倍。**


针对ArrayLis我们主要就看这么多，这里我们是基于OpenJDK进行的分析，不是Sun公司发布的JDK。二者在实现上还是有一定的差异。

#### Tip
在源码中，我们看到经常使用System.arraycopy进行数组的操作赋值。
System提供了一个静态方法arraycopy(),我们可以使用它来实现数组之间的复制。其函数原型是：

    public static void arraycopy(Object src, int srcPos, Object dest,  int destPos, int length)
    
- src:源数组；	
- srcPos:源数组要复制的起始位置；
- dest:目的数组；	
- destPos:目的数组放置的起始位置；	length:复制的长度。

注意：src and dest都必须是同类型或者可以进行转换类型的数组．
有趣的是这个函数可以实现自己到自己复制，比如：
int[] fun ={0,1,2,3,4,5,6}; 
System.arraycopy(fun,0,fun,3,3);
则结果为：{0,1,2,0,1,2,6};
实现过程是这样的，先生成一个长度为length的临时数组,将fun数组中srcPos 