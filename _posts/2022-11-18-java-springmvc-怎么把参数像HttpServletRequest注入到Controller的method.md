---
layout:       post
title:        "怎么把参数像HttpServletRequest注入到Controller的method"
author:       "hang.li"
header-style: text
catalog:      java
tags:
    - springmvc
    - spring
    - java
---
# spring-boot
# 1. 定义 HandlerMethodArgumentResolver 


```java
public class ArkMethodArgumentResolver implements HandlerMethodArgumentResolver {
    /**
     * 添加判断逻辑, 可以引入annotation, instanceof, {@link org.springframework.util.ClassUtils}.
     */
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return true;
    }

    /**
     * 从其他上下文中获取到的参数
     */
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return null;
    }
}
```

# 2. 添加到spring-boot提供的扩展mvc的方法中

```java
@Configuration
public class ArkWebMvcConfigurer implements WebMvcConfigurer {
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new ArkMethodArgumentResolver());
    }
}
```