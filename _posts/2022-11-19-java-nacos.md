---
layout:       post
title:        "nacos"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - micro service
    - java
---

# nacos安装
1. 传包
2. 解压
```
tar -xvf nacos-server-1.4.1.tar.gz
  cd nacos/bin
```
3. 在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip:port。（请配置3个或3个以上节点）
4. 制定jdk环境
```
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/home/work/software/jdk1.8.0_191
```
5. 启动服务器（本地内存集群）
```
sh bin/startup.sh -p embedded
```

# 官网
[https://nacos.io/zh-cn/](https://nacos.io/zh-cn/)
# 主要功能
## 注册中心
- eureka2.0+ 不再开源
- 中国人开发，友好的控制台
## 配置中心
- 可以把配置文件上线
- 如果修改配置、不用打包、甚至不用重启即可实现
- 同类spring cloud config(全家桶) 、 apollo（携程）

# 控制台
![img.png](/img/in-post/microservice/nacos-console.png)
