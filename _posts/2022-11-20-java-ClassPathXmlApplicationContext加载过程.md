---
layout:       post
title:        "ClassPathXmlApplicationContext加载过程"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - spring
    - java
---

# ClassPathXmlApplicationContext创建过程

1. 调用父类中的方法创建一个上下文对象

```
org.springframework.context.support.AbstractApplicationContext#AbstractApplicationContext(org.springframework.context.ApplicationContext)
```

2.  加载applicationcontext.xml中的配置

1. 生成一个标准的环境,

   ```
   org.springframework.core.env.AbstractEnvironment#AbstractEnvironment
   ```

2. 并且用环境中的变量替换${}符号

   ```
   org.springframework.core.env.PropertyResolver#resolveRequiredPlaceholders
   ```



3. 加载或刷新Bean的配置, refresh()方法结束时应该已经加载了所有的Bean配置,执行失败时应该一个配置都没有加载

    1. 销毁之前加载过的所有bean, 关闭之前的BeanFactory

    2. 创建的ClassPathXmlApplicationContext对象依赖的DefaultListableBeanFactory

    3. 以ClassPathXmlApplicationContext的id为key和2中创建DefaultListableBeanFactory对象的虚引用保存到beanFactory的serializableFactories中.

    4. 此时的beanFactory允许后定义的Bean覆盖相同id的Bean, 并且允许循环依赖(但是不建议)

    5. **加载bean的定义**

        1. loadBeanDefinitions方法调用,一些相同方法名的调用

           <!--ARAC: AbstractRefreshableApplicationContext-->

       | 加载bean定义 |         调用者          | 参数                        | 用途                                                         |
                | :----------: | :---------------------: | --------------------------- | ------------------------------------------------------------ |
       |      1       |          ARAC           | 依赖的beanFactory           | 创建一个以xml形式读取Bean配置的Reader,1. beanFactory写入到Reader中作为Bean定义的注册机, 2. 保存环境配置信息,3. 保存资源加载器,默认是按前缀匹配的资源加载器, 4. 创建一个解析xml文件的解释器. 衔接2 |
       |      2       |          ARAC           | 创建的xmlReader             | 拿到传入的bean所在的配置文件解析后的地址. 衔接3              |
       |      3       | XmlBeanDefinitionReader | 拿到的配置文件              | 因为是配置传入的配置文件可能是数组,这里做了循环遍历, 衔接4   |
       |      4       | XmlBeanDefinitionReader | 步骤3中的一个配置(location) | 按照前缀解析一个配置,因为是按前缀匹配,可能找到多个Resource, 衔接5 |
       |      5       | XmlBeanDefinitionReader | 解析到的多个资源文件        | 循环遍历,加载每一个资源,衔接6                                |
       |      6       | XmlBeanDefinitionReader | 循环中的一个Resource        | 将Resource包装成一个EncodedResource,(这里也是没有编码的), 衔接7 |
       |      7       | XmlBeanDefinitionReader | 包装后的EncodedResource     | 拿到一个输入流,调用doLoadBeanDefinitions方法读取Resource中的Bean的定义信息 |

        2. 读取Bean的定义信息

           ```
           org.springframework.beans.factory.xml.XmlBeanDefinitionReader#doLoadBeanDefinitions
           ```

        3. 使用dom的方式读取xml文件

        4. 注册bean定义

           ```
           org.springframework.beans.factory.xml.DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions
           ```

    4. 初始化BeanFactory

        1. 将当前类加载器写到BeanFactory中
        2. 将能够处理#{}表达式的对象写到BeanFactory中
        3. PropertyEditorRegistrar对象写到BeanFactory中
        4. 将上下文中已知的创建bean的前置程序添加到BeanFactory中
        5. 排除部分类型的依赖
        6. 如果Bean中依赖的是以下几个类型,将当前的环境写进去BeanFactory,ResourceLoader,ApplicationEventPublisher,ApplicationContext\
        7. 将要执行的监听方法添加到BeanFactory中
        8. 是否需要运行时织入代码
        9. 默认创建environment,systemProperties,systemEnvironment三个bean

    5. (空实现,可扩展)加载BeanFactory的前置扩展,

    6. 注册Bean的前置扩展程序处理流程

    7. 初始化消息处理

    8. 初始化事件处理器

    9. 调用子类扩展的onRefresh加载特殊的Bean定义

    10. 注册监听器

    11. 完成BeanFactory最后的初始化, 初始化一些要用的bean

        1. 为上下文初始化转换服务
        2. 注册默认的创建bean的后置执行方法
        3. 初始化代码织入要使用的Bean

        4. 去掉临时的类加载器
        5. 冻结(不允许修改)所有已经装入的类的定义
        6. 对于不需要懒加载的所有Bean 由BeanFactory调用getBean()方法进行初始化

    12. 发布上下文创建完成事件