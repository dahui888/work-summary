### LinkedList源码学习
前面我们对ArrayList的学习，知道ArrayList是通过数组进行实现。通过平常的了解，我们也知道LinkedList是基于链表进行实现的，带着几个疑惑来学习下。

1. LinkedList的实现原理
2. LinkedList是否线程安全
3. LinkedList常见方法的时间复杂度和空间复杂的

#### LinkedList的创建
```java
public class LinkedList<E> extends AbstractSequentialList<E> implements
        List<E>, Deque<E>, Queue<E>, Cloneable, Serializable {
 	private static final long serialVersionUID = 876323262645176354L;

    transient int size = 0;

    transient Link<E> voidLink;
    
    private static final class Link<ET> {
        ET data;

        Link<ET> previous, next;

        Link(ET o, Link<ET> p, Link<ET> n) {
            data = o;
            previous = p;
            next = n;
        }
    }
    
      /**
     * Constructs a new empty instance of {@code LinkedList}.
     */
    public LinkedList() {
        voidLink = new Link<E>(null, null, null);
        voidLink.previous = voidLink;
        voidLink.next = voidLink;
    }

    /**
     * Constructs a new instance of {@code LinkedList} that holds all of the
     * elements contained in the specified {@code collection}. The order of the
     * elements in this new {@code LinkedList} will be determined by the
     * iteration order of {@code collection}.
     *
     * @param collection
     *            the collection of elements to add.
     */
    public LinkedList(Collection<? extends E> collection) {
        this();
        addAll(collection);
    }
}
```
从这里可以看到类Link的结构，是不是跟当初学习链表的时候Node结构体特别相似，确实这个就是LinkedList实现的基础，也可以侧面证实LinkedList是给予链表实现的。通过上述可以看到LinkedList初始化大小size=0；然后创建一个Link节点。通过上面我们也可以看到，这里是**通过循环双向链表进行实现。**

**public LinkedList()**
```java
public LinkedList() {
    voidLink = new Link<E>(null, null, null);
    voidLink.previous = voidLink;
    voidLink.next = voidLink;
}
```

在无参构造方法中，首先创建一个空的Link对象，然后将Link的previous和next节点指向自身。最终完成创建一个空节点。

#### LinkedList常见方法

** public boolean add(E object)**：添加元素
```java
@Override
public boolean add(E object) {
    return addLastImpl(object);
}

private boolean addLastImpl(E object) {
    Link<E> oldLast = voidLink.previous;
    Link<E> newLink = new Link<E>(object, oldLast, voidLink);
    voidLink.previous = newLink;
    oldLast.next = newLink;
    size++;
    modCount++;
    return true;
}
```
在add方法中插入一个元素，操作步骤如下：
1. 先保存节点voidLink.previous节点，即断开连接；
2. 将object构建新节点；
3. 将新节点链接到voidLink.previous几点上；
4. 在将新节点连接到oldLink节点上。

这里的插入方式就是链表中的尾插法。链表中的插入方法有前插法和尾插法。


**public void add(int location, E object)**
```java
@Override
public void add(int location, E object) {
    if (location >= 0 && location <= size) {
        Link<E> link = voidLink;
        if (location < (size / 2)) {
            for (int i = 0; i <= location; i++) {
                link = link.next;
            }
        } else {
            for (int i = size; i > location; i--) {
                link = link.previous;
            }
        }
        Link<E> previous = link.previous;
        Link<E> newLink = new Link<E>(object, previous, link);
        previous.next = newLink;
        link.previous = newLink;
        size++;
        modCount++;
    } else {
        throw new IndexOutOfBoundsException();
    }
}
```
在add方法中，添加的位置必须location>=0&&location<=size。如果不符合则跑出IndexOutOfBoundsException异常。由于是双向循环链表，在插入的时候，先判断插入的位置是在size的前半段还是后半段，如果在前判断，则只需要变动0-location之间元素，如果在后半段，则变动location-size之间的元素位置。

** public boolean contains(Object object)**
```java
public boolean contains(Object object) {
    Link<E> link = voidLink.next;
    if (object != null) {
        while (link != voidLink) {
            if (object.equals(link.data)) {
                return true;
            }
            link = link.next;
        }
    } else {
        while (link != voidLink) {
            if (link.data == null) {
                return true;
            }
            link = link.next;
        }
    }
    return false;
}
```
在contains方法中，我们看到LinkedList在判断的时候也是通过遍历节点，然后判断，如果这个节点在最后，那么久会遍历到最后一个节点才判断出来结果，所以是非常费时的操作。

**public E get(int location)**
```java
public E get(int location) {
    if (location >= 0 && location < size) {
        Link<E> link = voidLink;
        if (location < (size / 2)) {
            for (int i = 0; i <= location; i++) {
                link = link.next;
            }
        } else {
            for (int i = size; i > location; i--) {
                link = link.previous;
            }
        }
        return link.data;
    }
    throw new IndexOutOfBoundsException();
}
```
在get方法中，LinkedList的实现也是通过逐节点的遍历，然后找到指定的节点值，进行返回。

**public E getFirst()和public E getLast()**
```java
public E getFirst() {
    return getFirstImpl();
}

private E getFirstImpl() {
    Link<E> first = voidLink.next;
    if (first != voidLink) {
        return first.data;
    }
    throw new NoSuchElementException();
}

/**
 * Returns the last element in this {@code LinkedList}.
 *
 * @return the last element
 * @throws NoSuchElementException
 *             if this {@code LinkedList} is empty
 */
public E getLast() {
    Link<E> last = voidLink.previous;
    if (last != voidLink) {
        return last.data;
    }
    throw new NoSuchElementException();
}
```
因为是双向循环链表，所以getFirst返回的就是头结点voidLikn.next.data的值，getLast返回的就是voidLink.previous.data的值。

#### LinkedList实现Queue接口方法
**offerFirst和offerLast**
```java
/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#offerFirst(java.lang.Object)
 * @since 1.6
 */
public boolean offerFirst(E e) {
    return addFirstImpl(e);
}

/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#offerLast(java.lang.Object)
 * @since 1.6
 */
public boolean offerLast(E e) {
    return addLastImpl(e);
}

private boolean addFirstImpl(E object) {
    Link<E> oldFirst = voidLink.next;
    Link<E> newLink = new Link<E>(object, voidLink, oldFirst);
    voidLink.next = newLink;
    oldFirst.previous = newLink;
    size++;
    modCount++;
    return true;
}
```
通过查看addFirstImpl和addLastImpl的实现，可以看到实现的方式是往链表的头部或尾部添加元素。

**peek类方法**
```java
/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#peekFirst()
 * @since 1.6
 */
public E peekFirst() {
    return peekFirstImpl();
}

/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#peekLast()
 * @since 1.6
 */
public E peekLast() {
    Link<E> last = voidLink.previous;
    return (last == voidLink) ? null : last.data;
}

/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#pollFirst()
 * @since 1.6
 */
public E pollFirst() {
    return (size == 0) ? null : removeFirstImpl();
}

/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#pollLast()
 * @since 1.6
 */
public E pollLast() {
    return (size == 0) ? null : removeLastImpl();
}

/**
 * {@inheritDoc}
 *
 * @see java.util.Deque#pop()
 * @since 1.6
 */
public E pop() {
    return removeFirstImpl();
}
```
通过上面源码可以看出，通过peek方法获取元素，但是不删除元素，poll方法会删除元素。