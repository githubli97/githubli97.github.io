---
layout:       post
title:        "java中的引用类型"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - java
---

![img.png](/img/in-post/java/reference.png)
# 强引用(strong references)
- 如果有该引用关系,gc就不会回收该对象.
- 常见的赋值操作都是强引用
# 软引用(soft reference)
- 当内存不足时,会打破该引用,将对象回收
- 可用于缓存
# 弱引用(weak reference)
- 下次gc时就会打破该引用,将对象回收
# 虚引用(phantom reference)
- 甚至不能算作一种引用, 因为从reference中不能拿到对象
# final引用(final reference)
- 有默认实现