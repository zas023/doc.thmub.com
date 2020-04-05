### 1. 概述

容器主要包含Collection和Map两种，Collection是存储对象的集合，而Map是存储键值对的映像表。

#### 1.1 Collection

![java collection framework image1](https://www.w3resource.com/w3r_images/java-collections.png)

- **List**：有序可重复

| 实现类       | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `ArrayList`  | 基于动态数组实现，支持随机访问                               |
| `Vector`     | 与ArrayList 类似，但它是线程安全的，但已过时，若需要实现同步可以调用Collections工具类的synchronizedList方法 |
| `LinkedList` | 基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列 |

1. ArrayList没有定义增长算法，当需要插入大量元素是，可调用ensureCapacity方法提高添加效率；
2. Vector类似与ArrayList，但是是同步的，多线程安全（另外一点区别是ArrayList扩容时默认增长一半，Vector增长一倍）



- **Set**：无序不重复，可最多含一个null元素（无序因而不能通过索引操作对象）

| 实现类          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `HashSet`       | 基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的 |
| `LinkedHashSet` | 具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序 |
| `TreeSet`       | 基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN) |

#### 1.2 Map

- **Map**：映射不能包含重复键

![](https://www.w3resource.com/w3r_images/java-collections1.png)

| 实现类          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `HashTable`     | 类似HashMap，但是线程安全的。它是遗留类，不宜使用。可用ConcurrentHashMap 来支持线程安全，并且效率更高，因为 ConcurrentHashMap 引入了分段锁 |
| `HashMap`       | 基于哈希表实现                                               |
| `LinkedHashMap` | 使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序 |
| `TreeMap`       | 基于红黑树实现                                               |



### 2. 源码分析

源码基于 JDK 1.8，使用Idea快捷键`Ctrl+B`调出。

#### 2.1 ArrayList

**概览**

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>,
         RandomAccess, Cloneable, java.io.Serializable{
     //初始容量
     private static final int DEFAULT_CAPACITY = 10;
     //基于数组实现，支持快速的随机访问
     transient Object[] elementData; // non-private to simplify nested class access
     //transient 关键字被用来表示变量将不被序列化处理
}
```

**扩容**

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
    //扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作的代价较高
}
```

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，即旧容量的 1.5 倍。

**删除**

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)。

**序列化**

ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

**Fail-Fast（快速失败）**

ArrayList使用`modCount`来记录结构发生变化的次数。在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，若产生变化需要抛出 ConcurrentModificationException（如某个线程对该 collection 在结构上对其做了修改）。

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

**Fail-Safe（安全失败）**

对于采用fail-safe机制来说，就不会抛出异常。因为fail-safe机制会在复制原集合的一份数据出来，然后在复制的那份数据遍历。所以fail-safe机制存在两个缺点：

1. 复制时需要额外的空间和时间上的开销。
2. 不能保证遍历的是最新内容。

java.util包下的集合类都是快速失败机制的，不能在多线程下发生并发修改（迭代过程中被修改）；

java.util.concurrent包下的容器都是安全失败的，可以在多线程下并发使用,并发修改。

#### 2.2 Vector

**同步**

实现与 ArrayList 类似，但是在所有修改结构的方法使用 synchronized 内置锁进行同步。

```java
//初始大小为10
public Vector() {
     this(10);
}
```

**扩容**

Vector 的构造函数可以传入 capacityIncrement （增长量）参数，它的作用是在扩容时使容量 capacity 增长 capacityIncrement。如果这个参数的值小于等于 0，扩容时每次都令 capacity 为原来的两倍。

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**代替方案**

- 在遍历过程中所有涉及到改变modCount值得地方全部加上synchronized或者直接使用Collections.synchronizedList。但不推荐，因为增删造成的同步锁可能会阻塞遍历操作。
- 使用CopyOnWriteArrayList来替换ArrayList。推荐使用该方案

#### 2.3 CopyOnWriteArrayList

CopyOnWriteArrayList会在一个复制的新数组上进行写操作，同时写操作使用ReentrantLock加锁。而读操作还是在原始数组中进行。写操作结束之后把原始数，将新数组的索引赋值给原始数组。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;
}
```

CopyOnWriteArrayList 在写操作的同时允许读操作，读写分离，互不影响，大大提高了读操作的性能。并实现**fail-safe**机制。

#### 2.4 LinkedList

```java
//基于双向链表实现，使用 Node 存储链表节点信息。
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}

transient Node<E> first;    //Pointer to first node.
transient Node<E> last;    //Pointer to last node.
```

- ArrayList 基于动态数组实现，LinkedList 基于双向链表实现；
- ArrayList 支持随机访问，LinkedList 不支持；
- LinkedList 快速的在任意位置添加删除元素。

#### 2.5 HashMap

**实现**：数组+链表

```java
transient Node<K,V>[] table;
//Node存储着键值对。从next字段可以看出Node是一个链表。
//即数组中的每个位置被当成一个桶，一个桶存放一个链表。
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
}

//默认初始大小为16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

**Put**

HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //(n - 1) & hash 计算桶的位置
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    //进入这里说明桶的位置已经放入过数据了
    else {
        Node<K,V> e; K k;
        //判断put的数据和之前的数据是否重复
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            //覆盖原有值
            e = p;
        //判断是否是红黑树，如果是红黑树就直接插入树中
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //如果不是红黑树，就遍历每个节点
            for (int binCount = 0; ; ++binCount) {
                //判断链表长度是否大于8，如果大于就转换为红黑树
               if ((e = p.next) == null) {
                   p.next = newNode(hash, key, value, null);
                   if (binCount >= TREEIFY_THRESHOLD - 1) //TREEIFY_THRESHOLD = 8
                       treeifyBin(tab, hash);
                   break;
               }
               //判断索引每个元素的key是否可要插入的key相同，如果相同就直接覆盖
               if (e.hash == hash &&
                   ((k = e.key) == key || (key != null && key.equals(k))))
                   break;
               p = e;
            }
       }
        //如果e不是null，说明跳出了循环，链表中有相同的key，因此只需要将value覆盖，返回oldValue即可
       if (e != null) { // existing mapping for key
           V oldValue = e.value;
           if (!onlyIfAbsent || oldValue == null)
               e.value = value;
           afterNodeAccess(e);
           return oldValue;
       }
   }
   //说明没有key相同，因此要插入一个key-value，并记录内部结构变化次数
   ++modCount;
   if (++size > threshold)
      resize();
   afterNodeInsertion(evict);
   return null;
}

//HashMap使用第0个桶存放键为null的键值对。
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

计算的hash值右移16位，即自己的高半区和低半位区做异或，混合原始哈希码的高位和低位，以此来加大低位的随机性。而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来。

![](https://pic4.zhimg.com/80/4acf898694b8fb53498542dc0c5f765a_hd.jpg)

**Get**

```java
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //如果表不是空的，并且要查找索引处有值，就判断位于第一个的key是否是要查找的key
    if ((tab = table) != null && (n = tab.length) > 0 &&
       (first = tab[(n - 1) & hash]) != null) {
       if (first.hash == hash && // always check first node
           ((k = first.key) == key || (key != null && key.equals(k))))
           //如果是，就直接返回
           return first;
       //如果不是就判断链表是否是红黑二叉树，如果是，就从树中取值
       if ((e = first.next) != null) {
           if (first instanceof TreeNode)
               return ((TreeNode<K,V>)first).getTreeNode(hash, key);
           //如果不是树，就遍历链表
           do {
               if (e.hash == hash &&
                   ((k = e.key) == key || (key != null && key.equals(k))))
                   return e;
           } while ((e = e.next) != null);
       }
    }
    return null;
}
```

**扩容**

```java
/**
* Initializes or doubles table size.  If null, allocates in
* accord with initial capacity target held in field threshold.
* Otherwise, because we are using power-of-two expansion, the
* elements from each bin must either stay at same index, or move
* with a power of two offset in the new table.
*
* @return the table
*/
final Node<K,V>[] resize() {
}
```

- HashTable 使用 synchronized 来进行同步。
- HashMap 可以插入键为 null 的 Entry。
- HashMap 的迭代器是 fail-fast 迭代器。
- HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

#### 2.6 ConcurrentHashMap

ConcurrentHashMap和HashMap实现上类似，最主要的差别是ConcurrentHashMap采用了分段锁Segment，每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

```java
//Segment 继承自 ReentrantLock
static class Segment<K,V> extends ReentrantLock implements Serializable {
    private static final long serialVersionUID = 2249069246763182397L;
    final float loadFactor;
    Segment(float lf) { this.loadFactor = lf; }
}

//默认的并发级别为 16，也就是说默认创建 16 个 Segment
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

#### 2.7 LinkedHashMap

继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```

内部维护了一个双向链表，用来维护插入顺序或者 LRU 顺序。

```java
//The head (eldest) of the doubly linked list.
transient LinkedHashMap.Entry<K,V> head;

//The tail (youngest) of the doubly linked list.
transient LinkedHashMap.Entry<K,V> tail;

//accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序
final boolean accessOrder;
```

LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。

```java
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

**afterNodeAccess**

当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。

**afterNodeInsertion**

在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。evict 只有在构建 Map 的时候才为 false，在这里为 true。

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

#### 2.8 WeakHashMap

WeakHashMap的Entry继承自WeakReference，被WeakReference关联的对象在下一次垃圾回收时会被回收。

WeakHashMap主要用来实现缓存，通过使用WeakHashMap来引用缓存对象，由 JVM 对这部分缓存进行回收。

```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
```

**ConcurrentCache **

Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能。

```java
public final class ConcurrentCache<K, V> {

    private final int size;

    private final Map<K, V> eden;

    private final Map<K, V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            v = this.longterm.get(k);
            if (v != null)
                this.eden.put(k, v);
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            this.longterm.putAll(this.eden);
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```

- 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；
- 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。
- 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。
- 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。


