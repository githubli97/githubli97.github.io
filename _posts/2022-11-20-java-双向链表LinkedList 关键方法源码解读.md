---
layout:       post
title:        "双向链表LinkedList 关键方法源码解读"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - list
    - java
---

## 1.LinkedList 头结点，尾结点

```java
    /**
     * 首个节点
	 * 恒定：first节点是null，last节点必是null
	 * 恒定：如果first节点不是null，first的前驱节点必是null
	 * 所有的添加修改删除操作都要保持恒定
     */
    transient Node<E> first;
    /**
     * 最后个节点
	 * 恒定：last节点是null，first节点必是null
	 * 恒定：如果last节点不是null，last后置节点必是null
	 * 所有的添加修改删除操作都要保持恒定
     */
    transient Node<E> last;
```

## 2. 链表节点数据模型

```java
	/**
	 * Node节点，说明是双向链表
	 */
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
## 3.链表批量插入

```java
    /**
     * 1. 传入的集合c转为数组a，判断是否为空集合
	 * 2. 将原有的集合从index处一分为二（index在后边）
	 * 3. succ为后半个集合的第一个节点， pred为前半个集合的最后一个节点
	 * 4. 遍历数组a，每一个元素构造成一个Node，将新Node挂到pred.next上，将pred指向原pred.next
	 * 5. a中的元素全加到前半个集合上后，将之前拆开的集合再合到一起
	 * 6. 修改集合长度
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```
## 4.clear方法，对GC的影响

```java
    /**
     * clear，每一个Node的每一个值置null
     * 处理恒定
     * 修改大小
     * 清除节点之间的所有链接是“不必要的”，但是:
     * -如果丢弃的节点占用多个代，则帮助分代GC
     * -即使存在可达迭代器，也一定会释放内存
     */
    public void clear() {
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```
## 5.头插方法

```java
    /**
     * 在集合开头插入元素
     * 1. 创建新节点前驱null，后置为first
     * 2. first指向新节点
     * 3. 原first.prev指向新Node
     * 4. 维护恒定
     */
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
```
## 6.尾插方法

```java
    /**
     * 追加元素
     * 1. 创建新节点前驱last，后置为null
     * 2. last指向新节点
     * 3. 原last.next指向新Node
     * 4. 维护恒定
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```
## 7.插入到指定元素前

```java
    /**
     * 插入到succ节点之前
	 * 1. 从succ处将集合一分为二，succ为后半个集合的首节点，pred为前半个的尾结点
	 * 2. 创建Node前驱为pred，后置为succ
	 * 3. 处理恒定
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```
## 8.批量出队，批量出栈原理相似

```java
    /**
     * f及其以前集合出队，返回f的值
     * 1. 记录f的值，f的next
     * 2. 将element与f.item，next与f.next关系断掉，方便回收剩下的集合空间
     * 3. 处理恒定
     * 4. 处理长度
     * 5. 返回element
     */
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC（虽然此操作看起来f更像是一个垃圾，但是确实是为了方便回收出队后剩下的集合或item中的对象，此处防止内存泄露）
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```
## 9.删除指定节点

```java
    /**
     * 删除x节点
     * 1. 按照x一分为3，prev连接next
     * 2. 方便gc回收
     * 3. 处理恒定
     * 4. 处理长度
     * 5. 返回element
     */
    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```
