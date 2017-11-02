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
从这里可以看到类Link的结构，是不是跟当初学习链表的时候Node结构体特别相似，确实这个就是LinkedList实现的基础，也可以侧面证实LinkedList是给予链表实现的。通过上述可以看到LinkedList初始化大小size=0；然后创建一个Link节点。

**public LinkedList()**

在无参函数中，首先创建一个空的Link对象，然后将Link的previous和next节点指向自身。

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
在add方法中，添加的位置必须location>=0&&location<=size。如果不符合则跑出IndexOutOfBoundsException异常。在插入的时候，先判断插入的位置是在size的前半段还是后半段，如果在前判断，则只需要变动0-location之间元素，如果在后半段，则变动location-size之间的元素位置。
