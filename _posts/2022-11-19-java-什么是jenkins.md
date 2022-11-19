---
layout:       post
title:        "jenkins"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - CI/CD
    - java
---

# 背景
现在上线流程：
## 后端
1. 叫上n个人
2. 拉代码（git checkout 。。。 ）
3. maven -> 指定环境 -> 打包(mvn clean package -Pdev)
4. 传包（ssh .............）
5. ssh 到服务器上。。。。。
6. 执行一堆命令（cd。。。stop。。。mv。。。start。。。）


## 前端
1. npm build 。。。。。。。。。。。。。。。。。
2. 。。。。。。。。
3. 。。。。。。。。
## 问题
- 出问题了（谁上线了？包上错了？环境错了？咋回事？上成什么代码了？）
- 。。。。
# CI/CD（持续集成）

![img.png](/img/in-post/jenkins/introduce-from-baidu.png)
- 简单说就是把上边的流程交给**机器解决**。
- 现在我们只用到了通知、拉代码、打包、重启、通知
- 未来：+ 构建镜像、滚动发布、金丝雀发布、脚本化测试。。。
# 需求组实现
## 上线流程
- 我们在00机器上部署了一套jenkins
- 每个项目作为一个工程，每个环境在一个tab页下。
- 上线模块只需点击![img.png](/img/in-post/jenkins/deployment-start-icon.png)
- 企业微信中机器人会发送开始、结束通知
![img.png](/img/in-post/jenkins/qw-notice.png)

-
## 问题
1. 谁上线了？
- 点击企业微信中的控制台
  ![img.png](/img/in-post/jenkins/createby.png)
  （可以不愧是你啊）
2. 包上错了？环境错了？
- 每个配置都是写死的，每个工程都可以回溯上线日志
- 每次执行构建都有最后一次提交的备注 
![img.png](/img/in-post/jenkins/commit-log.png)
# 简单配置
1. 指定分支参数
   ![img.png](/img/in-post/jenkins/add-build-param.png)
2. 指定git仓库
   ![img.png](/img/in-post/jenkins/git-respository.png)
3. 可以选择构建前流程，指定构建命令
   ![img.png](/img/in-post/jenkins/build-cmd-config.png)
4. 构建成功后把指定包发送到指定服务器，执行重启命令（shell），等
   ![img.png](/img/in-post/jenkins/deploy-to-server.png)

# 为什么jenkins
- 个人感觉。jenkins拥有非常丰富的插件市场，包括用到的企业微信机器人插件、maven、node、git、ssh支持。