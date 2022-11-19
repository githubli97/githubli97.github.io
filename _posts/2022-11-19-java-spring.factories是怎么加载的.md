---
layout:       post
title:        "spring.factories是怎么加载的"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - spring-boot
    - java
---
# 实例化META-INF/spring.factories
1. example
```java
org.springframework.boot.SpringApplication#getSpringFactoriesInstances(java.lang.Class<T>, java.lang.Class<?>[], java.lang.Object...)
```
2. 加载spring.factories
- 传入ClassLoader使得真正**所有包**下的META-INF/spring.factories
- 读取文件后按照ClassLoader缓存键值对
```java
org.springframework.core.io.support.SpringFactoriesLoader#loadSpringFactories
```
3. 使用反射初始化
---
4. 普通springboot项目
![img.png](/img/in-post/spring/get-spring.factories.png)

```java
0 = {URL@2329} "jar:file:/Users/edz/.m2/repository/org/springframework/boot/spring-boot/2.4.0/spring-boot-2.4.0.jar!/META-INF/spring.factories"
1 = {URL@2330} "jar:file:/Users/edz/.m2/repository/org/springframework/boot/spring-boot-autoconfigure/2.4.0/spring-boot-autoconfigure-2.4.0.jar!/META-INF/spring.factories"
2 = {URL@2331} "jar:file:/Users/edz/.m2/repository/org/springframework/spring-beans/5.3.1/spring-beans-5.3.1.jar!/META-INF/spring.factories"
```