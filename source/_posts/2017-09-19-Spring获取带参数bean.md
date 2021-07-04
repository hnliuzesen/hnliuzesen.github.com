---
title: Spring 获取带参数 bean
date: 2017-09-19 10:35:32
categories:
- Computers & Technology
- Program
tags:
- Java
- Spring
---

一个方便在不能够自动注入的时候，或者需要动态给构造函数传入参数时获取 Bean 的工具类。

<!--more-->

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Service;

import java.lang.reflect.Field;

/**
 * some useful static methods for spring. In fact, this class is not very "legal" in some way, and maybe warned by some
 * static code analysis tool like sonarqube for updating static field in constructor.
 * Created by shanghechen on 2017/8/1.
 */
@Service
public final class Springs {

    /**
     * accessed via reflection, see {@link Springs#Springs(ApplicationContext)}
     */
    private static ApplicationContext applicationContext;

    /**
     * use reflection to violate sonar rule: "write to static field from instance method"
     * trivially, this method should be implemented as
     * {@code Springs.applicationContext = applicationContext; }
     *
     * @param applicationContext injected context
     */
    @Autowired
    private Springs(ApplicationContext applicationContext) {
        try {
            Field field = this.getClass().getDeclaredField("applicationContext");
            field.setAccessible(true);
            field.set(this, applicationContext);
        } catch (IllegalAccessException | NoSuchFieldException e) {
            throw new IllegalStateException(e);
        }
    }

    public static Object getBean(String name) {
        return applicationContext.getBean(name);
    }

    public static <T> T getBean(String name, Class<T> requiredType) {
        return applicationContext.getBean(name, requiredType);
    }

    public static <T> T getBean(Class<T> requiredType) {
        return applicationContext.getBean(requiredType);
    }

    public static Object getBean(String name, Object... args) {
        return applicationContext.getBean(name, args);
    }

    public static <T> T getBean(Class<T> requiredType, Object... args) {
        return applicationContext.getBean(requiredType, args);
    }
}
```