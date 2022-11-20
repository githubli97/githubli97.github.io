---
layout:       post
title:        "HashMap关键方法"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - hashmap
    - java
---

# 1. 类介绍
[官方api地址](https://docs.oracle.com/javase/8/docs/api/index.html)
该类是基于哈希表对Map接口进行的实现。该实现提供了所有可重写的map操作，并允许null值和null键。（HashMap与Hashtable大致相同，区别是Hashtable是同步的，并且Hashtable的key和value都不允许为null。）这个类不能保证map的顺序;不同时间相同的键值对也可能顺序不一样。
# 2.成员变量含义

```java
    /**
     * 空参hashmap，hash表长度
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * hash表最大长度
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认负载因子
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 当hash值对应链表转换成红黑树阈值之一，链表深度
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * resize()时，相同hash索引下的节点可能会分散到两个索引下，分散后的节点个数小于6将红黑树转为链表
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 当hash值对应链表转换成红黑树阈值之一，hash表长度
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    
 	/**
     * hash表具体数据结构
     */
    transient Node<K,V>[] table;

    /**
     * 缓存，entrySet
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * 节点数量
     */
    transient int size;

    /**
     * 修改次数
     */
    transient int modCount;

    /**
     * 当threshold<size时调用resize方法对table扩容
     */
    int threshold;

    /**
     * 负载因子
     */
    final float loadFactor;
```

# 3.hash算法
## 计算key的hash

```java
    /**
     * 计算key的hash
     * 1. key为null，hash索引为0
     * 2. key的hashCode与hashCode右移16位进行异或的值
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
	//hash索引
	table.length-1 & (h = hash(key) ^ (h >>> 16))
    
```

## 计算hash表容量

```java
    /**
     * 计算threshold或table.length的大小，如果是3return4，如果是4return4,最大2的30次幂
     * 为什么只能是2的整数次幂？
     * hash算法为table.length-1 & (key.hashCode() ^ (key.hashCode() >>> 16))
     * 如果table的长度为4，table.length-1的二进制数为00000000000000000000000000000011，任何Ineger与此数按位与都只能得到
     * 0，00000000000000000000000000000000
     * 1，00000000000000000000000000000001
     * 2，00000000000000000000000000000010
     * 3，00000000000000000000000000000011
     * 这4个值正好对应table的四个索引
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
# 4.put方法

```java
    /**
     * 1. 如果table没有初始化，初始化table
     * 2. 计算hash索引，如果索引位置为null，new一个Node放到数组中
     * 3. 索引位置不为null，与放入hash相等，并且key相同，拿到当前取到的节点
     * 4. 如果当前节点是树节点，调用树的putTreeVal
     * 5. 使用链表保存这个hash索引的所有键值对，找到链表的最后位置，添加新节点
     *  1）如果链表深度>8,hash表长度小于64扩容,大于64转成树
     *  2）如果在遍历链表时发现相同的key直接替换掉
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

```java
    /**
     * 给 Map.putAll 和 Map constructor使用的方法
     * 1. table没有初始化，
     *  1）如果当前map中没有数据，重新计算threshold
     *  2）判断map是否达到增长条件，如果达到扩充一次table
     * 2.循环赋值
     */
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) { // pre-size
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                        (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
            else if (s > threshold)
                resize();
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

# 5.get方法

```java
    /**
     * 根据key的hash，key的值拿到
     * 1. 计算hash索引拿到首个节点，如果key匹配则返回当前节点
     * 2. 判断当前hash索引下是链表还是红黑树
     *  1） 链表：从前向后遍历
     *  2） 红黑树：找到根节点，树搜索算法
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                    ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
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

# 6.扩容

```java
    /**
     * 恒定，如果table.length>=2的30次幂或者threshold>=2的30次幂,threshold=Integer.MAX_VALUE
     * 1. 扩容前table初始化过
     *  1) 如果table.length>=2的30次幂(到达hash表的最大值)，不对table扩容
     *  2）否则table扩容为原来hash表容量的2倍，如果newCap = table.length，2的29次幂>table.length>=16，
     * 2. (两个参数的构造方法)扩容前table没有初始化过，table按照threshold初始化
     * 3. （空参构造方法）table.length=16,threshold=12
     * 4. 创建新的数组，重新计算hash值添加到新table中
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

# 7.链表何时转为红黑树

```java
    /**
     * 链表转红黑树
     * 条件：链表深度>=8(原本hash索引处有一个节点，next操作7次，也就是8个节点)并且hash表长度>=64
     * 1. 不满足条件对hash表扩容
     * 2. 满足条件
     *  1) 将node节点换成treeNode节点
     *  2）将链表转为树
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

# 8.romove方法

```java
    /**
     * 1. 判断hash表对应位置有值
     * 2. 找到当前索引下要删除的key对应的node
     * 3. 判断是否等值才删除
     */
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
                (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                                ((k = e.key) == key ||
                                        (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                    (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```