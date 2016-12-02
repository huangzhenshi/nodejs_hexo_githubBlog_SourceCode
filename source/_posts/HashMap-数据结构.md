---
title: HashMap 数据结构
date: 2016-11-24 18:14:43
tags:
---
## 1.设计结构
![hashmap数据结构](/img/1.png)

结合了数组结构（查询快）和链表结构（插入和删除快1）的特点。

第一层是数组  Entry(K,V) 的一个数组  table，根据key的hashcode值，对当前数组长度-1进行 &运算，得出该键值对 在数组中的存储位置。然后再判断 数组的该位置是否有值，如果该数组位置没有值（null），那么这个键值对的位置就是入住。如果该数组位置有值，那么老主人 就作为新主人的一部分，新进来的键值对占据该位置，  Entry  current.next= oldEntry. 也就形成了链表的结构，上线找下线，下线下面可能还有下线也有可能没有

## 2.1数组的长度

初始化的时候是16（ 2的4次方），每次扩容都是 2的N次方

```bash
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
}
```

为什么取2的N次方，因为 在键值对插入的时候，会对index求hashcode值，然后将hashcode值和 数组长度-1 进行 &运算（后面会重点讲 &运算）

```bash
public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
}

static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }

```

## 2.2 什么是 &运算?
就是把前后2个值转换为 2进制，相同位置上都为1 则为1，其他的都为0

例如左边图是 16长度数组，也就是  9&15   8&15 运算结果，一个1001(9 十进制) table[9]的位置， 一个1000（8 十进制）table[8]的位置

![&运算示例](/img/2.png)

如果不是2的N次方，那么总数-1 转换为2进制的时候，肯定有一个位置上为0

例如如果数组长度为15，那么 N-1=14， 14转换为二进制就是 1110

进行&运算的时候，因为1110与任何数进行&运算的结果都不可能是 ***1，所以例如 0001 也就是 table[1]这个位置上始终都不可能有键值对可以插入进去，所以数组的长度只能是2的N次方。

## 2.3 数组长度和实际键值对数量关系
默认初始长度是16，扩容时机的判断2个，所以理论上长度  length=0.75size - size 之间。当loadFactor为0.75的时候。
因为当size达到 length的时候，一定会触发 size>=0.75*length  和 table[index]！=null。也就是当size超过0.75*length的时候就有概率触发扩容，而且这种触发是随机的。

所以合理的 hashmap的长度应该是 4*size / 3  也就是 1.33*size 这样就不可能触发 hashmap的重构。特别是数量比较多的时候，几万个键值对的时候，初始化 hashmap的时候设置长度，非常有意义。

## 2.4 loadFactor的设置
当然也可以根据实际的需求设置 loadFactor，来设置适合业务规则的 hashmap。
如果内存富余，那么建议把loadFactor设置的小一点，但是要注意 初始size的设置，如果不合适会导致频繁的 resize 严重影响插入的效率。
如果内存比较吃紧，就可以把loadFactor设置的大一些，但是loadFactor设置大的话，键值对以链表的形式存储的概率就提高，平均的查询时间变慢，但是对于插入而言，虽然没有直接的影响，但是loadFactor提高，
需要插入更多的数据才会触发 resize，这样某种程度上是提升了 插入的效率

插入到某个有值的位置，挨个对比是否有 key值相同的对象
	

2.4.1）如果有key值一样(hashcode值相同，而且key==oldKey|| key.equals(k)），则替换并返回老的key的value值
	
2.4.2）如果没有key值一样的，那么该位置会被最后一个进来的Entry 占据，并且Entry的next属性，指向 之前第一个位置的Entry，也就是链表了，一个找一个

``` bash
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

初始化hashmap 2种 构造函数，

``` bash
 public HashMap(int initialCapacity, float loadFactor) {
  public HashMap(int initialCapacity) {

/**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
        init();
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

```

## 2.5 初始化hashmap length的设置
如果不修改 loadFactor的话（默认0.75），那么初始化的length应该为 1.34*size，比如1万个键值对，那么初始化的时候长度给13400比较合适，不会导致resize()，当然也要结合实际的内存情况做权衡

## 3.数据的put（插入一个元素）
3.1）key=null

会直接放在数组的table[0]位置

3.2）插入的位置没有值

直接返回null

3.3）插入的位置已经有值了

	3.31）遍历该key是否存在
![key值遍历](/img/3.png)

判断该key是否已经存在有2个条件 &&，hashcode值相同 并且 equals为true，一般实际开发不太可能出现这种情况，除非自己故意设置成这样。

## 4.hashmap的重构 resize()
会遍历之前所有的 Entry

计算新的 table.index  

---   如果该Entry的 next不为空，则next占据原位置，并且下一个处理这个next     
----  如果待插入的位置已经有Entry，则按照之前的规则，把老的Entry 作为 next存在新的 Entry里面。

```bash

void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```










