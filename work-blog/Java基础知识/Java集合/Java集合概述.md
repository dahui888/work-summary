### Java概述

Java中的集合可以分为Collection和Map两大类。

#### 一、Iterable和Iterator
在介绍集合类之前，我们先来介绍下Iterable和Iterator两个类。

**Iterable**

```java
/**
 * Instances of classes that implement this interface can be used with
 * the enhanced for loop.
 *
 * @since 1.5
 */
public interface Iterable<T> {

    /**
     * Returns an {@link Iterator} for the elements in this object.
     *
     * @return An {@code Iterator} instance.
     */
    Iterator<T> iterator();
}
```
从Iterable类的源码注释可以看到，只要实现该接口的类都可以使用Iterator.iterator方法。

**Iterator**

```java
/**
 * An iterator over a sequence of objects, such as a collection.
 *
 * <p>If a collection has been changed since the iterator was created,
 * methods {@code next} and {@code hasNext()} may throw a {@code ConcurrentModificationException}.
 * It is not possible to guarantee that this mechanism works in all cases of unsynchronized
 * concurrent modification. It should only be used for debugging purposes. Iterators with this
 * behavior are called fail-fast iterators.
 *
 * <p>Implementing {@link Iterable} and returning an {@code Iterator} allows your
 * class to be used as a collection with the enhanced for loop.
 *
 * @param <E>
 *            the type of object returned by the iterator.
 */
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
Iterator接口用来实现遍历使用，通过Iterator接口可以看到一个重要结论**如果一个集合在生成Iterator后发生改变，使用next()方法会跑异常ConcurrentModificationException，即fail-fast 机制。fail-fast 机制是java集合(Collection)中的一种错误机制。**

#### Collection集合类

##### 一、List集合
**1、ArrayList**
ArrayList通过数组来进行数据的存储。
```java
/**
 * The elements in this list, followed by nulls.
 */
transient Object[] array;
```
vector是线程同步的，所以它也是线程安全的，而arraylist是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用arraylist效率比较高。

要点：
- ArrayListk可以添加null类型的值。
- ArrayList中提供了size、get、set、isEmpty等方法，这些方法在时间复杂度上都是O(1)，而add方法在执行效率上，时间复杂度是O(n)。
- 每个ArrayList实例都就有一个初始大小为0，且大小会动态增加。
- ArrayList是异步的，如果多线程操作ArrayList实例，最好通过List list = Collections.synchronizedList(new ArrayList(...)); 进行创建实例。

Iterator和ListIterator主要区别有：
一、ListIterator有add()方法，可以向List中添加对象，而Iterator不能。
二、ListIterator和Iterator都有hasNext()和next()方法，可以实现顺序向后遍历。但是ListIterator有hasPrevious()和previous()方法，可以实现逆向（顺序向前）遍历。Iterator就不可以。
三、ListIterator可以定位当前的索引位置，nextIndex()和previousIndex()可以实现。Iterator 没有此功能。
四、都可实现删除对象，但是ListIterator可以实现对象的修改，set()方法可以实现。Iterator仅能遍历，不能修改。因为ListIterator的这些功能，可以实现对LinkedList等List数据结构的操作。

我们看下一个方法：
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
add方法中判断当前大小是否已满，然后判断size是否小于MIN_CAPACITY_INCREMENT / 2，如果小于，则ArrayList的大小自动扩容到MIN_CAPACITY_INCREMENT，反之则扩大一倍。

**2、LinkedList**
LinkedList是通过链表的数据结构，比较适合用于增删改，而ArrayList基于数组的数据结构，比较适合查找。LinkedList同样是异步的，当多个线程进行操作的时候，建议通过  创建实例。

	List list = Collections.synchronizedList(new LinkedList(...));

LinkedList实现了List接口和Queue接口，因此我们可以把LinkedList当成Queue来用。Queue使用时要尽量避免Collection的add()和remove()方法，而是要使用offer()来加入元素，使用poll()来获取并移出元素。它们的优点是通过返回值可以判断成功与否，add()和remove()方法在失败的时候会抛出异常。 如果要使用前端而不移出该元素，使用element()或者peek()方法。

作为List的子类，它集成List的常用方法，同时也具有自身特定的方法。
- addFirst(E e)
- addLast(E e)
- getFirst()
- getLast()
- removeFirst()
- removeLast()

作为Queue的子类，具有：
- add        增加一个元索                     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
- remove   移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException
- element  返回队列头部的元素             如果队列为空，则抛出一个NoSuchElementException异常
- offer       添加一个元素并返回true       如果队列已满，则返回false
- poll         移除并返问队列头部的元素    如果队列为空，则返回null
- peek       返回队列头部的元素             如果队列为空，则返回null

**3、AttributeList**
AttributeList属性集合，用来存放Attribute类的集合。
```java
public static void main(String []args){
    Attribute attribute = new Attribute("Name", "dsw");
    Attribute attribute2 = new Attribute("Name","Lucy");
    AttributeList attributeList = new AttributeList(3);
    attributeList.add(attribute2);
    attributeList.add(attribute);
}
```

**4、CopyOnWriteArrayList**
CopyOnWriteArrayList适合使用在读操作远远大于写操作的场景里，比如缓存。发生修改时候做copy，新老版本分离，保证读的高性能，适用于以读为主的情况。

**5、Vector**
类似于ArrayList，只不过ArrayList是线程异步的，Vector是线程同步的。Vector的默认大小是10.

**6、Stack**
Stack是Java中一种先进后出的数据结构-栈。它是Vector集合的子类，Vector是线程同步的，所以Stack也是线程同步的，同时Stack也具有List的相关操作方法，并且自身含有特有方法：
- peek：取出栈顶元素，但是并不从集合中删除；
- pop：删除栈顶元素
- push：向栈中添加元素。

疑点：
1、ArrayList如何实现顺序存储的；
2、LinkedList如何进行存储的；
3、Vector如何实现线程同步的；
4、Stack为什么集成Vector而不是ArrayList。

##### 二、Set集合
Set集合是一种集合，不能存储重复值。常见的实现类由：HashSet、TreeSet、EnumSet、LinkedHashSet。

**1、HashSet**
HashSet是一种无序集合，可以存储Null值，由于不存储重复值的特性，所以最多也就存储一个Null值。**HashSet是线程异步的，所以在多线程情况下： Set s = Collections.synchronizedSet(new HashSet(...));**。同样，在获取到Iterator后，如果对Set进行add、remove等操作会抛 ConcurrentModificationException异常。HashSet是存储在HashMap中。
```java
HashMap<E, HashSet<E>> backingMap
```

那么HashSet如何判断元素重复呢？
HashSet需要同时通过equals和HashCode来判断两个元素是否相等，具体规则是，如果两个元素通过equals为true，并且两个元素的hashCode相等，则这两个元素相等（即重复）。所以如果要重写保存在HashSet中的对象的equals方法，也要重写hashCode方法，重写前后hashCode返回的结果相等（即保证保存在同一个位置）。所有参与计算 hashCode() 返回值的关键属性，都应该用于作为 equals() 比较的标准。

**2、TreeSet**
TreeSet描述的是Set的一种变体——可以实现排序等功能的集合，它在讲对象元素添加到集合中时会自动按照某种比较规则将其插入到有序的对象序列中，并保证该集合元素组成的读写时刻按照“升序”排列。这里需要介绍下**如果添加到TreeSet中的类型没有实现Comparable接口，则会抛出java.lang.ClassCastException异常。**

   java提供了一个Comparable接口，该接口里定义了一个compareTo(Object obj)方法，该方法返回一个整数值，实现该接口的类必须实现该方法，实现了该接口的类的对象就可以比较大小。当一个对象调用该方法与另一个对象进行比较，例如obj1.comparTo(obj2),如果该方法返回0，则表明这两个对象相等；如果返回一个正整数，则表明obj1大于obj2；如果该方法返回一个负整数，则表明obj1小于obj2.

- TreeSet的add、remove、contains方法时间复杂度是log(n)
- TreeSet也是线程异步的，如果想同步，通过   SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...));创建

**3、LinkedHashSet**
LinkedHashSet是HashSet的子类，与HashSet的区别在于它是有序的排序。同样也是非线程安全的。

**4、ConcurrentSkipListSet**
跳跃链表是一种随机化数据结构，基于并联的链表，其效率可比拟于二叉查找树(对于大多数操作需要O(log n)平均时间)，并且对并发算法友好。**对并发性支持较好。**
[推荐文章](http://blog.csdn.net/bigtree_3721/article/details/51291974)

**5、CopyOnWriteArraySet**
它是线程安全的无序的集合，可以将它理解成线程安全的HashSet。有意思的是，CopyOnWriteArraySet和HashSet虽然都继承于共同的父类AbstractSet；但是，HashSet是通过“散列表(HashMap)”实现的，而CopyOnWriteArraySet则是通过“动态数组(CopyOnWriteArrayList)”实现的，并不是散列表。
1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。


疑点：
1、为什么HashSet使用HashMap<E, HashSet<E>> 结构来存储。
2、LinkedHashSet如何实现数据存储有序的。

##### 二、Map集合
Map是一种键值对集合。数据结构是Map< K, V>。一个Map集合不能包含重复的key值。Map接口可用来替代Dictionary类。Map接口提供了三种数据视图，分别是key的Set集合、value的Collection集合、key-value的Set键值集合。Map中item的顺序就是Iterator中元素的顺序，有些子类进行了实现了排序如TreeMap，使他们按照一定顺序进行存储，另一些没有实现，比如HashMap。

**1、HashMap**
HashMap是一种常见的Map集合，它实现了Map集合所有的方法，支持存储各种类型的值，包含Null值。但是HashMap是非线程同步，如果多线程使用，可以通过Map m = Collections.synchronizedMap(new HashMap(...));来创建线程同步的Map。HashMap通过get和put方法进行值的获取和存储。HashMap的性能由两个参数来影响，初始化大小和扩容因子，初始化大小就是用来存储HashMap值的hash table的大小，扩容因子是用来实现HashMap的自动扩容的，默认是0.75。 HashMap中是用HashMapEntry数组来存储Key-Value键值对的，HashMapEntry是HashMap中一个静态的内部类，实现了java.util.Map.Entry接口。
如果存储很多值，直接指定HashMap的大小值会提高很高的效率，不用经常计算HashMap的扩容。

更多详细参照[http://www.importnew.com/7099.html](http://www.importnew.com/7099.html)

**2、TreeMap**
基于红黑树进行实现。TreeMap实现SortMap接口，能够把它保存的记录根据键排序,默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator 遍历TreeMap时，得到的记录是排过序的。该映射根据其键的自然顺序进行排序，或者根据创建映射时提供的 Comparator 进行排序，具体取决于使用的构造方法。此实现为 containsKey、get、put 和 remove 操作提供受保证的 log(n) 时间开销。这些算法是 Cormen、Leiserson 和 Rivest 的 Introduction to Algorithms 中的算法的改编。
注意，如果要正确实现 Map 接口，则有序映射所保持的顺序（无论是否明确提供了比较器）都必须与 equals 一致。
注意，此实现不是同步的。如果多个线程同时访问一个映射，并且其中至少一个线程从结构上修改了该映射，则其必须 外部同步。
SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));

**3、LinkedHashMap**
LinkedHashMap 是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的.也可以在构造时用带参数，按照应用次数排序。在遍历的时候会比HashMap慢，不过有种情况例外，当HashMap容量很大，实际数据较少时，遍历起来可能会比 LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关。

**4、ConcurrentHashMap**


**5、WeakHashMap**

**6、Hashtable**

**7、Attributes**

**8、Properties**

疑点：
1. HashMap的put和get的原理，如何使用HashCode和equals方法，如何进行扩容。