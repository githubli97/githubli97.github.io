---
layout:       post
title:        "怎么快速构建spring源码"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - spring
    - java
---


# github加速
1. 使用gitee克隆github中的仓库,然后使用gitee克隆(不推荐, 如果需要和github保持一致需要一直复制)
2. 下载油猴插件, 安装链接脚本, [url](https://greasyfork.org/zh-CN/scripts/412245-github-%E5%A2%9E%E5%BC%BA-%E9%AB%98%E9%80%9F%E4%B8%8B%E8%BD%BD)
   ![img.png](/img/in-post/broswer/github-plugins.png)

# spring-framework
1. 克隆代码到本地, [github](https://github.com/spring-projects/spring-framework)
2. gradle可以指定gradle版本编译,最好下载源工程指定的版本
3. 可修改maven仓库,找到目录下**settings.gradle**,在原有的基础上添加阿里仓库
```
pluginManagement {
	repositories {
		maven { url "https://maven.aliyun.com/repository/public" }
		maven { url "https://maven.aliyun.com/repository/jcenter" }
		maven { url "https://maven.aliyun.com/repository/gradle-plugin" }
		maven { url "https://maven.aliyun.com/repository/central" }
		...
```
5. 找到根路径下的**import-into-idea.md**文件
6. 按照步骤操作
```java
_Within your locally cloned spring-framework working directory:_

7. Precompile `spring-oxm` with `./gradlew :spring-oxm:compileTestJava`
# 在window当前目录运行
./gradlew.bat :spring-oxm:compileTestJava

2. Import into IntelliJ (File -> New -> Project from Existing Sources -> Navigate to directory -> Select build.gradle)
# 使用idea导入当前项目
 
3. When prompted exclude the `spring-aspects` module (or after the import via File-> Project Structure -> Modules)
# 排除spring-aspects模块

4. Code away
# 编译整个工程
./gradlew.bat
```
# spring-boot
- 修改maven仓库后直接用idea打开构建即可