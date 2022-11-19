---
layout:       post
title:        "springboot加载流程"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - spring-boot
    - java
---

# SpringApplication构造方法
- 加载并初始化META-INF/spring.factories，保存到SpringAppliction对象中。
1. 实例化Bootstrapper
2. 实例化ApplicationContextInitializer
3. 实例化ApplicationListener


# 版本
```java
spring: 5.3.1
spring-boot: 2.4.0
```
# run方法
## 1. 计时
## 2. 根据META-INF/spring.factories初始化DefaultBootstrapContext
## 3. 实例化SpringApplicationRunListener
```java
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```
## 4. 将ApplicationListener全都装入到SpringApplication.listeners
1. 执行SpringApplicationRunListener.starting()
## 5. 准备环境,配置
### 1. 加载常规环境
### 2. 执行SpringApplicationRunListener.environmentPrepared()
### 3. load factories EnvironmentPostProcessor.class,
1. 创建ReflectionEnvironmentPostProcessorsFactory
2. 对factories中定义的EnvironmentPostProcessor进行初始化
3. 执行postProcessEnvironment
4. 添加随机数据源
5. 替换封装类OriginAwareSystemEnvironmentPropertySource，启动参数，jvm参数，系统环境变量
6. 解析spring.application.json中的配置
7. 解析vcap.application属性
8. 解析配置文件中的配置属性
9. 响应式调试配置中的属性
### 4. 根据配置初始化日志系统的配置，注册ShutdownHook
### 5. 对一些耗时的对象进行提前初始化(可选)
![img.png](/img/in-post/spring/lazyloaditem.png)
## 6.配置spring.beaninfo.ignore
## 7.打印banner
## 8. 创建AnnotationConfigServletWebServerApplicationContext
![img.png](/img/in-post/spring/applicationContext.png)

-  执行子类构造方法前，先执行父类的构造方法
### 1. AbstractApplicationContext
```java
	public AbstractApplicationContext() {
		this.resourcePatternResolver = getResourcePatternResolver();
	}
```
### 2. GenericApplicationContext
```java
	public GenericApplicationContext() {
		this.beanFactory = new DefaultListableBeanFactory();
	}
```

#### 1. 重要，创建beanFactory
![Alt](/img/in-post/spring/beanfactory.png)

#### 2. AbstractAutowireCapableBeanFactory
- 添加一些忽略的依赖，默认使用cglib子类初始化策略
```java
	public AbstractAutowireCapableBeanFactory() {
		super();
		ignoreDependencyInterface(BeanNameAware.class);
		ignoreDependencyInterface(BeanFactoryAware.class);
		ignoreDependencyInterface(BeanClassLoaderAware.class);
		if (IN_NATIVE_IMAGE) {
			this.instantiationStrategy = new SimpleInstantiationStrategy();
		}
		else {
			this.instantiationStrategy = new CglibSubclassingInstantiationStrategy();
		}
	}
```

### 3. 最后回到当前类AnnotationConfigServletWebServerApplicationContext
```java
	public AnnotationConfigServletWebServerApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
```

## 9. 记录applicationcontext启动时的情况

## 10. 准备context

### 1. 将springboot准备好的environment覆盖到context中的environment
```java
super.environment = environment;

this.reader.setEnvironment(environment);
this.scanner.setEnvironment(environment);
```
### 2. ApplicationContext执行后置处理器

### 3. 执行ApplicationContextInitializer
1. 获取spring.factores中配置的ApplicationContextInitializer
2. 执行environment中要配置的context.initializer.classes，实例化后执行initialize
3. 添加CachingMetadataReaderFactoryPostProcessor
4. environment配置spring.application.name写到applicationContext中的id属性中，并注册到beanFactory
5. 添加BeanFactory ，ConfigurationWarningsPostProcessor
6. 添加一些记录日志报告的listener

### 4. 执行contextPrepared事件

### 5. bootstrapContext执行关闭事件

### 6. 打印启动信息

### 7. 获取对应环境下的日志配置

### 8. 向beanFactory中注册引导期创建的一些特殊的对象
```java
beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
if (printedBanner != null) {
	beanFactory.registerSingleton("springBootBanner", printedBanner);
}
if (beanFactory instanceof DefaultListableBeanFactory) {
	((DefaultListableBeanFactory) beanFactory)
			.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
}
if (this.lazyInitialization) {
	context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
}
```

### 9. 将启动类注册到BeanFactory中

#### 1. createBeanDefinitionLoader
```java
	/**
	 * BeanDefinitionLoader 能将sources中的bean通过以下的几种方式注册到registry中
	 */
	BeanDefinitionLoader(BeanDefinitionRegistry registry, Object... sources) {
		Assert.notNull(registry, "Registry must not be null");
		Assert.notEmpty(sources, "Sources must not be empty");
		this.sources = sources;
		this.annotatedReader = new AnnotatedBeanDefinitionReader(registry);
		this.xmlReader = (XML_ENABLED ? new XmlBeanDefinitionReader(registry) : null);
		this.groovyReader = (isGroovyPresent() ? new GroovyBeanDefinitionReader(registry) : null);
		this.scanner = new ClassPathBeanDefinitionScanner(registry);
		this.scanner.addExcludeFilter(new ClassExcludeFilter(sources));
	}
```

#### 2. 通过启动类，通过启动类注册bean

### 10. 执行ApplicationPreparedEvent事件
-  EnvironmentPostProcessorApplicationListener
   将所有延迟日志切换到其提供的目的地。
- LoggingApplicationListener
  将一些日志相关的bean对象注册到beanFactory中
- BackgroundPreinitializer
- DelegatingApplicationListener
  执行用户自定义事件
## 11. refreshContent

### 1. 注册应用shutdown钩子
### 2. AbstractApplicationContext执行refresh方法
- 注意此处为父类中定义的刷新context步骤的模板，查看执行方法优先查看子类（AnnotationConfigServletWebServerApplicationContext）的实现。
#### 1. 设置启动步骤日志
```java
StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
```
#### 2. 准备刷新
```java
prepareRefresh();
```
- AnnotationConfigServletWebServerApplicationContext
1. 清空元数据缓存
2. 调用父类prepareRefresh方法
- AbstractApplicationContext
3. 计时
4. 修改context状态
5. 打印刷新日志
6. 在上下文环境中初始化任何占位符属性源。
```java
	public static void initServletPropertySources(MutablePropertySources sources,
			@Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {

		Assert.notNull(sources, "'propertySources' must not be null");
		// "servletContextInitParams"
		String name = StandardServletEnvironment.SERVLET_CONTEXT_PROPERTY_SOURCE_NAME;
		if (servletContext != null && sources.get(name) instanceof StubPropertySource) {
			sources.replace(name, new ServletContextPropertySource(name, servletContext));
		}
		// "servletConfigInitParams"
		name = StandardServletEnvironment.SERVLET_CONFIG_PROPERTY_SOURCE_NAME;
		if (servletConfig != null && sources.get(name) instanceof StubPropertySource) {
			sources.replace(name, new ServletConfigPropertySource(name, servletConfig));
		}
	}
```
7. 验证environment中必须项
8. 刷新前注册监听器
9. 储存上下文刷新前的监听器
10. earlyApplicationEvents（null -> new LinkedHashSet<>();）
### 3. 刷新上下文内部的beanFactory,并返回
- GenericApplicationContext
```java
	/**
	 * 因为已经执行构造方法时，已经创建了内置的beanFactory，这里不再做刷新操作，将ApplicationContext的id写到beanFactory中
	 */
	@Override
	protected final void refreshBeanFactory() throws IllegalStateException {
		if (!this.refreshed.compareAndSet(false, true)) {
			throw new IllegalStateException(
					"GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
		}
		this.beanFactory.setSerializationId(getId());
	}
```
### 4. 准备beanFactory
1. 将context的类加载器给beanFactory
2. 配置是否忽略el配置
3. 添加能织入bean中常量的，属性赋值？
```java
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

```
4. 注入ApplicationContext相关的接口
- ApplicationContextAwareProcessor能够扫描类中是否实现ApplicationContext，如果实现能够通过BeanPostProcessor的方式实现对ApplicationContext中相关的值作为注入，从而可以忽略ApplicationContext相关的接口进行依赖注入。
```java
beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
beanFactory.ignoreDependencyInterface(ApplicationStartup.class);
```

5. 注册一些上下文相关的依赖
```java
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
beanFactory.registerResolvableDependency(ResourceLoader.class, this);
beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```

6. 对ApplicationListener类的BeanPostProcessor进行实现
   
7. 对environment、systemProperties、systemEnvironment、applicationStartup类进行注册
### 5. 执行子类的postProcessBeanFactory
- ServletWebServerApplicationContext
  用BeanPostProcessor的方式让applicationContext能够实现ServletContext和ServletConfig的自动注入
  ，并且停止ServletContextAware类型的依赖注入。
- 注册web类型的applicationContext的bean的作用空间，如：request，session。
### 6.记录启动过程

### 7. invokeBeanFactoryPostProcessors

#### 1. 处理invokeBeanDefinitionRegistryPostProcessors

- 调用
```java
/**
 *在标准初始化之后修改应用程序上下文的内部 bean 定义注册表。 所有  **常规**    bean 定义都将被加载，但尚未实例化任何 bean。 ***这允许在BeanFactoryPostProcessor 开始之前添加更多的 bean 定义。***
 */
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```

1. 首先，调用实现 Priority Ordered 的 Bean 定义注册表后处理器。
```java
beanFactory.isTypeMatch(ppName, PriorityOrdered.class
```

2. 接下来，调用实现 Ordered 的 Bean 定义注册表后处理器。
```java
!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)
```

3. 最后，调用所有其他 Bean 定义注册表后处理器，直到不再出现其他 Bean 定义注册表后处理器。
#### 2. 处理invokeBeanFactoryPostProcessors

- 调用，和BeanDefinitionRegistryPostProcessors时机相同，但是此时执行时说明常规的不常规的bean定义都已经加载完成了。
```java
public interface BeanFactoryPostProcessor {

	/**
	 * 所有 bean 定义都将被加载，但没有 bean将已经被实例化。这允许覆盖或添加属性甚至提前初始化bean。
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

- 同样的，按照优先order， order，常规的postProcessors顺序进行执行。


# 关键类
## 1. DelegatingApplicationListener
- 读取environment 中context.listener.classes配置的applictionlister

## 2. DelegatingApplicationContextInitializer
- 读取environment 中context.initializer.classes

# 监听器触发事件
```java
package org.springframework.boot;

public interface SpringApplicationRunListener {

	/**
	 * 当 run 方法第一次启动时立即调用。 可用于非常早的初始化。
	 */
	default void starting(ConfigurableBootstrapContext bootstrapContext) {
		starting();
	}

	@Deprecated
	default void starting() {
	}

	/**
	 * environment准备好，但在ApplicationContext被创建之前调用。
	 */
	default void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
			ConfigurableEnvironment environment) {
		environmentPrepared(environment);
	}

	@Deprecated
	default void environmentPrepared(ConfigurableEnvironment environment) {
	}

	/**
	 * 在创建并准备好ApplicationContext ，但在加载源之前调用。
	 */
	default void contextPrepared(ConfigurableApplicationContext context) {
	}

	/**
	 * 在加载应用程序上下文后但在刷新之前调用。
	 */
	default void contextLoaded(ConfigurableApplicationContext context) {
	}

	/**
	 * 上下文已刷新且应用程序已启动，但尚未调用CommandLineRunners和ApplicationRunners 。
	 */
	default void started(ConfigurableApplicationContext context) {
	}

	/**
	 * 在 run 方法完成之前立即调用，此时应用程序上下文已刷新并且所有CommandLineRunners和ApplicationRunners已被调用。
	 */
	default void running(ConfigurableApplicationContext context) {
	}

	/**
	 * 在运行应用程序时发生故障时调用。
	 */
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}

}

```