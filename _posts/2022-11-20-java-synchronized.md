---
layout:       post
title:        "synchronized锁"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - java
---

## 锁升级过程
1. 无锁状态
2. 偏向锁: 一旦有线程对该对象加锁,markword区前54位指向当前线程指针
3. 自旋锁(无锁,轻量级): CAS一旦有多个线程竞争锁,在每个线程栈帧内生成Lock Record, 前62位保存拿到当前执行权限的线程的Lock Record
   Lock Record中保存之前对象头中被替换掉的内容
4. 重量级: 自旋10次,升级为重量级锁, 其他线程不再自旋,进入队列等待状态, 借用操作系统资源加锁
5. 锁降级: gc时

## markdown头中的信息
![img.png](/img/in-post/java/objecthead.png)
## 字节码命令
[monitorenter   monitorexit](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.monitorenter)
## 锁消除
- 如果运行时发现synchronized锁住的是线程私有的对象,去掉加锁的过程.

```java
    public static void main(String[] args) {
        Object o = new Object();
        synchronized (o) {
            System.out.println(o);
        }
    }
```

## 锁粗化
- 循环体中多次对同一个对象进行synchronized,将synchronized锁到外部.

```java
    private static StringBuilder sb = new StringBuilder();
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            synchronized (sb){
                sb.append(i);
            }
        }
    }
```

优化为:

```java
    private static StringBuilder sb = new StringBuilder();
    public static void main(String[] args) {
        synchronized (sb){
            for (int i = 0; i < 100; i++) {
                    sb.append(i);
            }
        }
    }
```
