---
layout:       post
title:        "垃圾收集器（三） Parallel收集器"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - gc
    - java
---
# 常见组合
- Parallel Scanvege + ServialOld(jdk14废弃此方案)
- Parallel Scanvege + Parallel Old(jdk8默认方案，jdk14现役)
# 特点
- 多线程并行收集
- Parallel Scanvege标记复制算法 ，Parallel Old标记整理算法
- 注重吞吐量的方案
- 自适应策略
# PP组合运行图
![img.png](/img/in-post/java/gc-parallel.png)
# 使用Parallel
- 使用Parallel: -XX:+UseParallelGC, -XX:+UseParallelOldGC,默认相互激活
- 并行垃圾收集的线程数： -XX:ParallelGCThreads(cpu核心数<8默认核心数，>8（3+（5*cpucount/8））)
- -XX:MaxGCPauseMillis 每次垃圾收集时间期望值，期望越小相应的情理到的垃圾也越少
- -XX:GCTimeRatio 垃圾收集时间占总时间的比例，（吞吐量的倒数）默认99
- -XX: -UseAdaptiveSizePolicy，参数自适应，因为Parallel更关注吞吐量，jvm可以通过自适应内存参数达到期望的垃圾收集收集时间（-XX:MaxGCPauseMillis ）和收集时间占比（-XX:GCTimeRatio）。自适应影响的值：新生代大小、eden与survivor比例、对象直接分配到老年代大小阈值等
 