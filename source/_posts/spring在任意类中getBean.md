---
title: spring在任意类中getBean
date: 2018-07-25 15:56:55
tags: [java,spring]
categories: spring
---

# ApplicationContextAware接口
ApplicationContextAware接口的bean在被初始化之后，可以在任意类中拿到容器中的bean


## 实现这个接口的代码：
```java
package web.utils;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class SpringContextUtil implements ApplicationContextAware {
	private static ApplicationContext context = null;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		context = applicationContext;
	}

	public static ApplicationContext getApplicationContext() {
		assertApplicationContext();
		return context;
	}

	@SuppressWarnings("unchecked")
	public static <T> T getBean(String beanName) {
		assertApplicationContext();
		return (T) context.getBean(beanName);
	}

	public static <T> T getBean(Class<T> requiredType) {
		assertApplicationContext();
		return context.getBean(requiredType);
	}


	private static void assertApplicationContext() {
		if (SpringContextUtil.context == null) {
			throw new RuntimeException("applicaitonContext属性为null,请检查是否注入了SpringContextUtil!");
		}
	}
}
```

## 在application.xml文件中定义对应的bean,或者通过注解@Component标注
```xml
  <bean class="web.utils.SpringContextUtil" lazy-init="false" ></bean>
```

## 配置web.xml
```xml
    <!-- 全局spring定义 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring-mvc.xml</param-value>
    </context-param>
    <!-- ContextLoaderListener载入 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
```

## 使用
在任意类中，可以直接拿到bean了
```java
LoggerDao loggerDao = (LoggerDao) SpringContextUtil.getBean( LoggerDao.class);
```
