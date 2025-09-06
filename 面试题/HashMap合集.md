# HashMap合集

## 你项目中常用哪些集合?

ArrayList、HashMap、LinkedList、ConcurrentHashMap





## 说一下list、set和map的区别

List：有序，不可重复，通过索引访问，底层是基于数组和链表来实现的

Set：无序，元素唯一，快速查找，适合去重，底层基于哈希表和红黑树实现

Map：无序，键唯一，键找值，基于哈希表和红黑树实现

选择建议：
**有序且允许重复**用`List`
**唯一且需要快速查找**用`Set`
需要**键值映射**用`Map`



## 说一下hashmap的底层原理

HashMap的数据结构是一个“链表散列”来实现的，也就是数组和链表的结合体，在JDK8以后是链表加数组加红黑树。

// TODO









## hashmap的key和value可以为空吗，如果可以允许key为null，那么null的这个元素是存在哪个桶？

Java HashMap 的 key 和 value 都可以为 null。其中，key 只允许一个 null（因为 null 被视为一个唯一的键），而 value 可以有多个 null。 如果将 null 作为 key 插入 HashMap 时，不会调用 hashCode() 方法，而是直接将它放置在桶索引为 0 的位置（bucket 0）。

> [!tip]
>
> 为什么key可以为空？
>
> HashMap 允许 key 为 null 是为了提供更高的灵活性和通用性，同时通过特殊处理（将 null key 放入 bucket 0）确保了实现的正确性。这种设计在非并发场景下是安全的，但在需要线程安全的场景下（如 ConcurrentHashMap），则会禁止 null key 以避免复杂性。





## hashmap是线程安全的吗,那想要线程安全怎么办？

不是线程安全的。

有四种方法。第一种是手动加锁，使用ReentrantLock，调用lock()方法（要记得释放锁unlock()）;

> [!tip]
>
> 也可以使用synchronized锁，引出synchronized和ReentrantLock的区别

第二种是使用`ConcurrentHashMap`，它是线程安全的，并且性能更好，特点是使用了~~分段锁~~或CAS机制锁粒度更细，只锁住部分数据通，读操作通常无锁，写操作高效，支持高并发，性能优于Colletions.synchronizedMap。

第三种是使用`Colletions.synchronizedMap`来包装HashMap，使其变成线程安全，性能比较低，每次操作都会上锁，锁粒度较大。

第四种是适用于Hashtable，它也是线程安全的，它不支持键值为null，性能也是比较低的，已经被淘汰了

更推荐使用`ConcurrentHashMap`或者手动加锁

> [!tip]
>
> | 特性   | Hashtable                       | Collections.synchronizedMap    |
> | ------ | :------------------------------ | ------------------------------ |
> | 锁粒度 | 对象级别（整个 Hashtable 对象） | 对象级别（包装的 mutex 对象）  |
> | 锁实现 | 方法上直接加 synchronized       | 包装方法中使用 synchronized 块 |





## hashmap的时间复杂度是多少

O(1)











































