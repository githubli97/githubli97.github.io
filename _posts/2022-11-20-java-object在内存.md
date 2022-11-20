---
layout:       post
title:        "object在内存"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - java
---

# 测试代码
## 引入maven环境

```java
        <dependency>
            <groupId>org.openjdk.jol</groupId>
            <artifactId>jol-core</artifactId>
            <version>0.9</version>
        </dependency>
```
## 测试代码
```java
public class ObjectRAM {
    public static void main(String[] args) {
       
        System.out.println("**初始化Object**");
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());

        System.out.println("**初始化Object[]**");
        Object[] oa = new Object[15];
        System.out.println(ClassLayout.parseInstance(oa).toPrintable());
        
    }
}
```
## 测试结果
![img.png](/img/in-post/java/objectonmemory.png)
