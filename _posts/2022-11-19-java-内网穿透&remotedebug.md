---
layout:       post
title:        "内网穿透&remotedebug"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - working
    - java
---

@[toc]
# 背景
- 本地没问题，上线有问题
- 支付、各种回调、没法调试
# 内网穿透工具
## frp
### 需要公网机器
### 官网
[文档](https://gofrp.org/docs/)
### 下载地址
[下载地址](https://github.com/fatedier/frp/releases/download/v0.36.2/frp_0.36.2_linux_amd64.tar.gz)
## 花生壳
## ngrok


# arthas
1. jad：支持jvm中类的查看
2. watch：查看某次调用的入参、返回、异常抛出、成员变量
3. 热部署、反编译。。。。。
## 官网
[官网](https://arthas.aliyun.com/doc/)
# 远程调试（代码要一致）
## resin
```xml
<jvm-arg>-Xdebug</jvm-arg>
<jvm-arg>-Xnoagent</jvm-arg>
<jvm-arg>-Djava.compiler=NONE</jvm-arg>
<jvm-arg>-Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9114</jvm-arg>

```

## java ... -jar ***.jar
```java 
-Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9114
```


##  Idea增加remote启动方式
![img.png](/img/in-post/ide/idea-remotedebug-config.png)

