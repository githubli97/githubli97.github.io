---
layout:       post
title:        "hash table"
author:       "hang.li"
header-style: text
catalog:      data
tags:
    - data
---

# 演示动画
[https://www.cs.usfca.edu/~galles/visualization/OpenHash.html](https://www.cs.usfca.edu/~galles/visualization/OpenHash.html)

# 哈希表
- 由一个数组实现, 因为数组有下标, 而且连续 ,方便拿到哈希索引直接找到对应位置
# 哈希算法
## java HashMap
- 哈希表为特殊的哈希表,长度为2的整数幂
- 每个value根据hashCode处理后,与哈希表长度按位&, 正好能得到0 -> 哈希表长度减一几种类型的数据
## 除留余数(实现简单)
- 根据固定的规则拿到代表value的数字除以哈希表的长度, 得到余数正好是: 0 -> 哈希表长度减一
# 哈希冲突
- 将value计算得到的哈希索引冲突的地址, 以链表的形式排列==java8中如果达到一定的条件将链表转成红黑树==