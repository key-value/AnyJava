

# 内部代码
```
static final class Segment<K,V> extends ReentrantLock implements Serializable { 
transient volatile int count;
transient int modCount;
transient int threshold;
transient volatile HashEntry<K,V>[] table;
final float loadFactor; }
```

>作为锁的界定边界，同时又是链表结构。所以继承了ReentrantLock的同时有哈希map 需要的字段


-   count：Segment中元素的数量
-   modCount：对table的大小造成影响的操作的数量（比如put或者remove操作）
-   threshold：阈值，Segment里面元素的数量超过这个值依旧就会对Segment进行扩容
-   table：链表数组，数组中的每一个元素代表了一个链表的头部
-   loadFactor：负载因子，用于确定threshold