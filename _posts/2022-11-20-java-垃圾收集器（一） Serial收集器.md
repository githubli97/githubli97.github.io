---
layout:       post
title:        "垃圾收集器（一） Serial收集器"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - gc
    - java
---
# 常见组合
1. Serial+SerialOld(jdk14现役)
2. Serial+CMS
# 特点
1. 单线程（进行gc时只有gc线程运行，所有其他线程停在安全点，STW）
2. java client模式下的默认垃圾收集器（windows 32位默认模式）
3. 对于单核处理 器或处理器核心数较少的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以 获得最高的单线程收集效率。
4. 管理空间在几十兆甚至一两百兆。
5. Serial使用标记复制算法，SerialOld老年代使用标记整理算法

# 运行图
![img.png](/img/in-post/java/gc-serial.png)
# 使用Serial
- -XX:+UseSerialGC
