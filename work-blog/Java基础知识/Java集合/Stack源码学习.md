### Stack源码学习
Stack，中文名称是栈。在数据结构中，栈是一种Last-In/First-Out(LIFO)先进后出的数据结构。在Java中，通过Stack类来实现栈这种结构，有以下特点：
1. 支持null值；
2. 栈的大小没限制
3. 通过push填充元素（进栈），通过pop取出元素（推栈）

#### 源码解读
栈的实现源码简短。
```java
 */
public class Stack<E> extends Vector<E> {

    private static final long serialVersionUID = 1224463164541339165L;

    /**
     * Constructs a stack with the default size of {@code Vector}.
     */
    public Stack() {
    }

    /**
     * Returns whether the stack is empty or not.
     *
     * @return {@code true} if the stack is empty, {@code false} otherwise.
     */
    public boolean empty() {
        return isEmpty();
    }

    /**
     * Returns the element at the top of the stack without removing it.
     *
     * @return the element at the top of the stack.
     * @throws EmptyStackException
     *             if the stack is empty.
     * @see #pop
     */
    @SuppressWarnings("unchecked")
    public synchronized E peek() {
        try {
            return (E) elementData[elementCount - 1];
        } catch (IndexOutOfBoundsException e) {
            throw new EmptyStackException();
        }
    }

    /**
     * Returns the element at the top of the stack and removes it.
     *
     * @return the element at the top of the stack.
     * @throws EmptyStackException
     *             if the stack is empty.
     * @see #peek
     * @see #push
     */
    @SuppressWarnings("unchecked")
    public synchronized E pop() {
        if (elementCount == 0) {
            throw new EmptyStackException();
        }
        final int index = --elementCount;
        final E obj = (E) elementData[index];
        elementData[index] = null;
        modCount++;
        return obj;
    }

    /**
     * Pushes the specified object onto the top of the stack.
     *
     * @param object
     *            The object to be added on top of the stack.
     * @return the object argument.
     * @see #peek
     * @see #pop
     */
    public E push(E object) {
        addElement(object);
        return object;
    }

    /**
     * Returns the index of the first occurrence of the object, starting from
     * the top of the stack.
     *
     * @return the index of the first occurrence of the object, assuming that
     *         the topmost object on the stack has a distance of one.
     * @param o
     *            the object to be searched.
     */
    public synchronized int search(Object o) {
        final Object[] dumpArray = elementData;
        final int size = elementCount;
        if (o != null) {
            for (int i = size - 1; i >= 0; i--) {
                if (o.equals(dumpArray[i])) {
                    return size - i;
                }
            }
        } else {
            for (int i = size - 1; i >= 0; i--) {
                if (dumpArray[i] == null) {
                    return size - i;
                }
            }
        }
        return -1;
    }
}
```
从上面可以看到，**Stack继承Vector实现。**这里为什么是继承Vector来实现栈结构呢？我猜想是因为Vector是线程安全的，结合栈的先进后出特点，所以务必要保证栈是线程安全的，才能确保进出栈的顺序正确。

**push进栈操作**
```java
/**
 * Pushes the specified object onto the top of the stack.
 *
 * @param object
 *            The object to be added on top of the stack.
 * @return the object argument.
 * @see #peek
 * @see #pop
 */
public E push(E object) {
    addElement(object);
    return object;
}
```
通过push方法向栈顶添加一个元素。push方法的本质还是通过vector的addElement方法进行实现。

**pop出栈操作**
```java
@SuppressWarnings("unchecked")
public synchronized E pop() {
    if (elementCount == 0) {
        throw new EmptyStackException();
    }
    final int index = --elementCount;
    final E obj = (E) elementData[index];
    elementData[index] = null;
    modCount++;
    return obj;
}
```
栈是一种先进后出的数据结构，同时栈继承Vector进行实现，Vector基于数组进行顺序存储实现。所以Stack也是基于数组进行实现的。由于先进后出的特点，所以进行数组的倒序输出即可。

**peek获取栈顶元素**
```java
    @SuppressWarnings("unchecked")
    public synchronized E peek() {
        try {
            return (E) elementData[elementCount - 1];
        } catch (IndexOutOfBoundsException e) {
            throw new EmptyStackException();
        }
    }
```
通过peek方法可以获取栈顶元素，但是并不将栈顶元素删除。

**search查找元素位置**
```java
    public synchronized int search(Object o) {
        final Object[] dumpArray = elementData;
        final int size = elementCount;
        if (o != null) {
            for (int i = size - 1; i >= 0; i--) {
                if (o.equals(dumpArray[i])) {
                    return size - i;
                }
            }
        } else {
            for (int i = size - 1; i >= 0; i--) {
                if (dumpArray[i] == null) {
                    return size - i;
                }
            }
        }
        return -1;
    }
```
查找元素第一次出现的位置index。本质就是数组的遍历进行排查，记住由于是先进后出，所以是倒序遍历。