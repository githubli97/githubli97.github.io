---
layout:       post
title:        "B+Tree"
author:       "hang.li"
header-style: text
catalog:      data
tags:
    - data
---

# 演示地址
[https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html)
![img.png](img/in-post/data/b+tree.png)
- 能减少树查询时的越级寻址
- 每个树非叶子节点上,可能有多个数据, 并且有子节点地址
- 可以指定每个树节点上最大的数据长度,如果根节点超过最大长度, 树会加深
- 叶子节点维护了有序链表
# 应用
- mysql的索引
  Innodb索引和数据都在一起(聚簇索引)的形式
  MyISAM 索引指向磁盘文件的地址, 索引和数据分开
- 虽然hash索引不需要寻址(不考虑hash冲突), 但是在范围查找方面不如B+Tree索引效果好