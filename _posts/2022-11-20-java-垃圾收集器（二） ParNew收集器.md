---
layout:       post
title:        "垃圾收集器（二） ParNew收集器"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - gc
    - java
---
# 常见组合

1.  ParNew + CMS(jdk9不推荐使用，随着jdk14CMS的废弃，此方案废弃)
2. ParNew + SerialOld(jdk8废弃)
# 特点
1.  Serial的多线程并行版本（在垃圾收集时间有多个线程进行垃圾收集）
2. 作用在新生代
3. 采用标记复制算法
4. 在单核心机器上效率低（涉及到多个线程争抢cpu，切换线程上下文降低效率）
# 运行图
![img.png](/img/in-post/java/gc-parnew.png)

# 使用ParNew
- 使用ParNew: -XX:+UseParNewGC
- 指定垃圾收集的线程数：-XX:ParallelGCThreads（默认为cpu核心数）
- 当开启CMS老年代收集器时，新生代默认采用ParNew收集器
