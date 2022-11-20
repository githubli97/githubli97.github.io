---
layout:       post
title:        "ArrayList常见方法解读（源码，以及对内存使用，gc的影响）"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - list
    - java
---

1. 扩容方法 ensureCapacity， 普通扩容类似只是截取了此方法中的5，6，7步。
   MAX_ARRAY_SIZE = Integer.MAX_VALUE-8；
   Integer.MAX_VALUE = 0x7fffffff；
   
```
   /**
     * 对elementData扩容
	 * 1. 原elementData=[], minCapacity<=10,无效操作
	 * 2. 原elementData=[], MAX_ARRAY_SIZE>=minCapacity>10, modCount++, elementData长度改为minCapacity
	 * 3. 原elementData=[], MAX_ARRAY_SIZE<minCapacity, modCount++, elementData长度改为Integer.MAX_VALUE
	 * 原elementData<>[], elementData长度最小是10
	 * 4. 原elementData<>[], minCapacity<=elementData长度, modCount++
	 * 5. 原elementData<>[], (elementData长度 * 1.5)>=minCapacity>elementData长度, modCount++ , elementData长度为(原elementData长度 * 1.5)
	 * 6. 原elementData<>[], MAX_ARRAY_SIZE>=minCapacity>(elementData长度 * 1.5), modCount++ , elementData长度为(原elementData长度 * 1.5)
	 * 7. 原elementData<>[], MAX_ARRAY_SIZE<minCapacity, modCount++ , elementData长度为Integer.MAX_VALUE
     *
     * @param   minCapacity   the desired minimum capacity
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
```
2. System.arraycopy方法

```java
// elementData从下标index至下标size-1复制到index+1至size位置（size - index为数组长度）
System.arraycopy(elementData, index, elementData, index + 1, size - index);
```
3. remove方法此处有内存泄露的例子，和在此处的解决办法

```java
    /**
     * 1. elementData从下标index+1至下标size-1复制到index至size-2位置
	 * 2. 此时下标size-1和size-2为相同地址，指向同一对象，size-1置null防止内存泄露（size-2指向别的位置时，size-1还有引用，既无法访问，又是垃圾数据）
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
4. fastRemove 不返回删除的对象，防止对象逃逸，增加了效率，所以称为fastRemove

```java
    /*
     * 跳过边界检查，不返回删除的值（避免对象逃逸）
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```