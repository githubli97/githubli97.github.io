---
layout:       post
title:        "线性表"
author:       "hang.li"
header-style: text
catalog:      data
tags:
    - data
---

## 栈
- 后进先出
### 应用
1. 函数执行模型
2. 任意树的深度遍历算法
3. ...
## 队列
- 先进先出
### 应用
1. 有序的任务列表(消息队列, Lock线程执行队列, 线程池任务队列)
2. 任意树的广度遍历算法
3. ...
# 数组
- 可以对,栈,队列都可以进行实现
- 内存连续, 能够根据首地址,计算指定下标处的值.
- 查询效果好,
- 频繁插入删除, 涉及挪动操作位置后所有元素的位置,性能差
# 链表
- 可以对,栈,队列都可以进行实现
- 每个元素都是一个节点, 每个节点都有指向上一个或下一个元素的地址
- 插入,删除时,只需把操作元素两侧的元素相关联即可
- 查询只能依靠从前向后遍历, 比对,效率低