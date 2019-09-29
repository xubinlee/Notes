# 集合

## List：

ArrayList、LinkedList、Vector与java.util.Collections.synchronizedList、CopyOnWriteArrayList

1. ArrayList：数组实现，扩容：每次对size增长50%空间

   **ArrayList高效操作：**

   &emsp;&emsp;在个数固定的情况下，ArrayList避免了开辟空间的问题，所以当确定数据个数的时候就使用ArrayList，如果不确定的时候就使用LinkedList（链表实现，O（n））。

   &emsp;&emsp;因为ArrayList底层原理就是一个数组的动态操作，数组的容量是固定的，ArrayList通过一些算法实现动态扩容。

   int initialCapacity = (int)Math.round(list.size() / 0.75 + 1);

2. LinkedList：双向链表实现，只能顺序访问，但可以快速地在链表中间插入和删除元素。LinkedList还可以用作栈、队列和双向队列。

3. Vector【'vektə】（不推荐使用）：数组实现，使用同步方法实现线程安全，扩容：每次对size增长2倍空间

   &emsp;&emsp;Vector是增删改查都加synchronized；CopyOnWriteArrayList增删改加锁，读不加锁，读方面就好于Vector，CopyOnWriteArrayList支持读多写少的并发情况

4. synchronizedList：使用同步代码块实现，多线程环境下，写操作性能较好，读操作性能较差

5. CopyOnWriteArrayList：多线程下，写操作性能较差，读操作性能较好

Enumeration [ɪ,njuːmə'reɪʃən] 只能读，Iterator[ɪtə'reɪtə]可以读和删

&emsp;&emsp;禁止在foreach里进行元素的remove/add操作（触发fail-fast机制，modCount[实际被修改次数]和expectedModCount[预期被修改次数]不相等），remove元素使用Iterator对象，如果是并发操作，需要对Iterator加锁。

## Map：

HashMap、HashTable、LinkedHashMap、synchronizedMap、synchronizedSortedMap、ConcurrentHashMap与ConcurrentSkipListMap                （key不能重复）

1. HashTable：

   同步的（线程安全）

   key和value不允许null值

   扩容：初始11，按old*2+1扩容

2. HashMap（原理：链表长度>8，转成红黑树）：

   非同步（非线程安全）

   Key允许一个null值，value允许多个null值

   扩容：初始16，按2的指数扩容（即移位处理），然后重新计算每个元素在数组中的位置。

3. TreeMap：支持排序

4. ConcurrentHashMap：（线程安全）对桶数组进行分段，每一段都用锁进项保护，不支持排序

5. ConcurrentSkipListMap：（线程安全）基于跳表（空间换取时间），支持排序操作

## Set：

TreeSet、HashSet、synchronizedSet、synchronizedSortedSet   （都不可重复）

1. HashSet：hash表实现、无序的、允许null值
2. TreeSet：二叉树实现、有序的、不允许null值
3. LinkedHashSet：链表实现、有序的、允许null值

**集合对象是否重复比较：**

&emsp;&emsp;将对象放入到集合中时，首先判断要放入对象的hashcode值与集合中的任意一个元素的hashcode值是否相等，如果不相等直接将该对象放入集合中。如果hashcode值相等，然后再通过equals方法判断要放入对象与集合中的任意一个对象是否相等，如果equals判断不相等，直接将该元素放入到集合中，否则不放入。

**注意避免集合的扩容，它很耗性能，根据元素的数量给它一个初始消息的值。**