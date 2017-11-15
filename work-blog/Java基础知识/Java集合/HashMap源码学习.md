### HashMap源码学习
HashMap是我们常见的Map接口实现，用于存储键值对。HashMap是线程异步的，并且存储数据是无序的，所以我们可以通过Map m = Collections.synchronizedMap(new HashMap(...))来创建线程同步的Map对象。HashMap的性能由两个参数来影响，初始化大小和扩容因子，初始化大小就是用来存储HashMap值的hash table的大小，扩容因子是用来实现HashMap的自动扩容的，默认是0.75。

#### HashMap的定义
```java
public class HashMap<K, V> extends AbstractMap<K, V> implements Cloneable, Serializable {
    /**
     * Min capacity (other than zero) for a HashMap. Must be a power of two
     * greater than 1 (and less than 1 << 30).
     */
    private static final int MINIMUM_CAPACITY = 4;

    /**
     * Max capacity for a HashMap. Must be a power of two >= MINIMUM_CAPACITY.
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * An empty table shared by all zero-capacity maps (typically from default
     * constructor). It is never written to, and replaced on first put. Its size
     * is set to half the minimum, so that the first resize will create a
     * minimum-sized table.
     */
    private static final Entry[] EMPTY_TABLE
            = new HashMapEntry[MINIMUM_CAPACITY >>> 1];

    /**
     * The default load factor. Note that this implementation ignores the
     * load factor, but cannot do away with it entirely because it's
     * mentioned in the API.
     *
     * <p>Note that this constant has no impact on the behavior of the program,
     * but it is emitted as part of the serialized form. The load factor of
     * .75 is hardwired into the program, which uses cheap shifts in place of
     * expensive division.
     */
    static final float DEFAULT_LOAD_FACTOR = .75F;

    /**
     * The hash table. If this hash map contains a mapping for null, it is
     * not represented this hash table.
     */
    transient HashMapEntry<K, V>[] table;

    /**
     * The entry representing the null key, or null if there's no such mapping.
     */
    transient HashMapEntry<K, V> entryForNullKey;

    /**
     * The number of mappings in this hash map.
     */
    transient int size;

    /**
     * Incremented by "structural modifications" to allow (best effort)
     * detection of concurrent modification.
     */
    transient int modCount;

    /**
     * The table is rehashed when its size exceeds this threshold.
     * The value of this field is generally .75 * capacity, except when
     * the capacity is zero, as described in the EMPTY_TABLE declaration
     * above.
     */
    private transient int threshold;

    // Views - lazily initialized
    private transient Set<K> keySet;
    private transient Set<Entry<K, V>> entrySet;
    private transient Collection<V> values;

    /**
     * Constructs a new empty {@code HashMap} instance.
     */
    @SuppressWarnings("unchecked")
    public HashMap() {
        table = (HashMapEntry<K, V>[]) EMPTY_TABLE;
        threshold = -1; // Forces first put invocation to replace EMPTY_TABLE
    }

    /**
     * Constructs a new {@code HashMap} instance with the specified capacity.
     *
     * @param capacity
     *            the initial capacity of this hash map.
     * @throws IllegalArgumentException
     *                when the capacity is less than zero.
     */
    public HashMap(int capacity) {
        if (capacity < 0) {
            throw new IllegalArgumentException("Capacity: " + capacity);
        }

        if (capacity == 0) {
            @SuppressWarnings("unchecked")
            HashMapEntry<K, V>[] tab = (HashMapEntry<K, V>[]) EMPTY_TABLE;
            table = tab;
            threshold = -1; // Forces first put() to replace EMPTY_TABLE
            return;
        }

        if (capacity < MINIMUM_CAPACITY) {
            capacity = MINIMUM_CAPACITY;
        } else if (capacity > MAXIMUM_CAPACITY) {
            capacity = MAXIMUM_CAPACITY;
        } else {
            capacity = roundUpToPowerOfTwo(capacity);
        }
        makeTable(capacity);
    }

    /**
     * Constructs a new {@code HashMap} instance with the specified capacity and
     * load factor.
     *
     * @param capacity
     *            the initial capacity of this hash map.
     * @param loadFactor
     *            the initial load factor.
     * @throws IllegalArgumentException
     *                when the capacity is less than zero or the load factor is
     *                less or equal to zero or NaN.
     */
    public HashMap(int capacity, float loadFactor) {
        this(capacity);

        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new IllegalArgumentException("Load factor: " + loadFactor);
        }

        /*
         * Note that this implementation ignores loadFactor; it always uses
         * a load factor of 3/4. This simplifies the code and generally
         * improves performance.
         */
    }

    /**
     * Constructs a new {@code HashMap} instance containing the mappings from
     * the specified map.
     *
     * @param map
     *            the mappings to add.
     */
    public HashMap(Map<? extends K, ? extends V> map) {
        this(capacityForInitSize(map.size()));
        constructorPutAll(map);
    }
    /**
     * Inserts all of the elements of map into this HashMap in a manner
     * suitable for use by constructors and pseudo-constructors (i.e., clone,
     * readObject). Also used by LinkedHashMap.
     */
    final void constructorPutAll(Map<? extends K, ? extends V> map) {
        for (Entry<? extends K, ? extends V> e : map.entrySet()) {
            constructorPut(e.getKey(), e.getValue());
        }
    }
    ....
```
通过上面的源码，我们来分析下HashMap的结构。
**大小**
HashMap中定义了两个大小的常量。最小大小是4，最大的大小是1<<30。HashMap满的时候也会自动进行扩容，扩容大小因子是0.75。

**存储**
HashMap中的值存储在table和entryForNullKey中。table是类型HashMapEntry< K, V>的数组，entryForNullKey是HashMapEntry< K, V>类型。table中用于存储我们的值，entryForNullKey用于存储null值。

**扩容**
源码中介绍的扩容因子是DEFAULT_LOAD_FACTOR=0.75，那么什么时候触发扩容操作呢？这里就需要注意到threshold门限。当HashMap的大小超过阈值的时候，table会重新计算hashcode。threshold大小等于0.75*capacity。


**构造函数**
```java
public HashMap(int capacity) {
    if (capacity < 0) {
        throw new IllegalArgumentException("Capacity: " + capacity);
    }

    if (capacity == 0) {
        @SuppressWarnings("unchecked")
        HashMapEntry<K, V>[] tab = (HashMapEntry<K, V>[]) EMPTY_TABLE;
        table = tab;
        threshold = -1; // Forces first put() to replace EMPTY_TABLE
        return;
    }

    if (capacity < MINIMUM_CAPACITY) {
        capacity = MINIMUM_CAPACITY;
    } else if (capacity > MAXIMUM_CAPACITY) {
        capacity = MAXIMUM_CAPACITY;
    } else {
        capacity = roundUpToPowerOfTwo(capacity);
    }
    makeTable(capacity);
}
```
该构造函数中，传入指定大小的capacity。如果大小小于0，则抛出IllegalArgumentException异常。如果大小=0，则构建一个EMPTY_TABLE,并将该值赋值给table，同时初始化threshold值=-1。接着就是判断capacity的大小，然后通过makeTable初始化一个table和threshold的值。

在上面多次出现HashMapEntry类。我们来看下：
```java
static class HashMapEntry<K, V> implements Entry<K, V> {
    final K key;
    V value;
    final int hash;
    HashMapEntry<K, V> next;

    HashMapEntry(K key, V value, int hash, HashMapEntry<K, V> next) {
        this.key = key;
        this.value = value;
        this.hash = hash;
        this.next = next;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    @Override public final boolean equals(Object o) {
        if (!(o instanceof Entry)) {
            return false;
        }
        Entry<?, ?> e = (Entry<?, ?>) o;
        return Objects.equal(e.getKey(), key)
                && Objects.equal(e.getValue(), value);
    }

    @Override public final int hashCode() {
        return (key == null ? 0 : key.hashCode()) ^
                (value == null ? 0 : value.hashCode());
    }

    @Override public final String toString() {
        return key + "=" + value;
    }
}
```
它的定义就类似我们数据库中的Node节点，用于存储当前节点的key和value信息。同时存储next节点的信息。

#### HashMap添加数据
hashMap通过put方法添加值。
```java
@Override public V put(K key, V value) {
    if (key == null) {
        return putValueForNullKey(value);
    }

    int hash = secondaryHash(key.hashCode());
    HashMapEntry<K, V>[] tab = table;
    int index = hash & (tab.length - 1);
    for (HashMapEntry<K, V> e = tab[index]; e != null; e = e.next) {
        if (e.hash == hash && key.equals(e.key)) {
            preModify(e);
            V oldValue = e.value;
            e.value = value;
            return oldValue;
        }
    }

    // No entry for (non-null) key is present; create one
    modCount++;
    if (size++ > threshold) {
        tab = doubleCapacity();
        index = hash & (tab.length - 1);
    }
    addNewEntry(key, value, hash, index);
    return null;
}

private V putValueForNullKey(V value) {
    HashMapEntry<K, V> entry = entryForNullKey;
    if (entry == null) {
        addNewEntryForNullKey(value);
        size++;
        modCount++;
        return null;
    } else {
        preModify(entry);
        V oldValue = entry.value;
        entry.value = value;
        return oldValue;
    }
}


    /**
     * Applies a supplemental hash function to a given hashCode, which defends
     * against poor quality hash functions. This is critical because HashMap
     * uses power-of-two length hash tables, that otherwise encounter collisions
     * for hashCodes that do not differ in lower or upper bits.
     */
    private static int secondaryHash(int h) {
        // Doug Lea's supplemental hash function
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
    
        /**
     * Creates a new entry for the given key, value, hash, and index and
     * inserts it into the hash table. This method is called by put
     * (and indirectly, putAll), and overridden by LinkedHashMap. The hash
     * must incorporate the secondary hash function.
     */
    void addNewEntry(K key, V value, int hash, int index) {
        table[index] = new HashMapEntry<K, V>(key, value, hash, table[index]);
    }

```
首先我们看到如果key=null值，则将key插入到entryForNullKey中。反之则通过secondaryHash方法二次计算key的hashcode，然后将hashcode与table的大小进行&运算，计算出该key、value在数组中应该存储的index。接着就是遍历table，判断当前index位置上的HashMapEntry值是是否有值，这里的判断条件是该位置上元素的hashCode和key是否跟插入元素的相同。如果相同，替换原来的元素。**如果当前位置上元素不与此值相同，则将元素构建插入。**ps：其实这里的本质就是哈希表中处理碰撞方法的一种，叫链地址法。

#### HashMap获取元素
```java
/**
 * Returns the value of the mapping with the specified key.
 *
 * @param key
 *            the key.
 * @return the value of the mapping with the specified key, or {@code null}
 *         if no mapping for the specified key is found.
 */
public V get(Object key) {
    if (key == null) {
        HashMapEntry<K, V> e = entryForNullKey;
        return e == null ? null : e.value;
    }

    // Doug Lea's supplemental secondaryHash function (inlined)
    int hash = key.hashCode();
    hash ^= (hash >>> 20) ^ (hash >>> 12);
    hash ^= (hash >>> 7) ^ (hash >>> 4);

    HashMapEntry<K, V>[] tab = table;
    for (HashMapEntry<K, V> e = tab[hash & (tab.length - 1)];
            e != null; e = e.next) {
        K eKey = e.key;
        if (eKey == key || (e.hash == hash && key.equals(eKey))) {
            return e.value;
        }
    }
    return null;
}
```
首先根据key的hashCode计算出在table中的index。当找到与指定key相同的位置返回该位置的value，或者当找到hash值与key的相同并且key.dquals与之相等，则进行数据反馈。

通过对上面方法的了解，我们知道针对在Map中添加元素，需要进行HashCode和equals方法的判断。

因为Map这个类没有继承Iterable接口所以不能直接通过map.iterator来遍历(list，set就是实现了这个接口，所以可以直接这样遍历),所以就只能先转化为set类型，用entrySet()方法，其中set中的每一个元素值就是map中的一个键值对，也就是Map.Entry<K,V>了，然后就可以遍历了。
基本上 就是遍历map的时候才用得着它吧。