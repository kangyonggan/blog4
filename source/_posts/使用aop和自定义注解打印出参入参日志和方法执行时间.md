---
title: 使用aop和自定义注解打印出参入参日志和方法执行时间
date: 2017-03-25 17:43:00
categories: Java后台
tags:
- Java
---

### 自定义注解
`LogTime.java`:

```
package com.kangyonggan.api.common.annotation;

import java.lang.annotation.*;

/**
 * 在方法上加此注解会打印入参和出参，会计算方法消耗的时间
 *
 * @author kangyonggan
 * @since 2016/12/8
 */
@Documented
@Inherited
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogTime {
}
```

<!-- more -->

### 主要逻辑：

```
package com.kangyonggan.api.common.aop;

import com.kangyonggan.api.common.annotation.LogTime;
import com.kangyonggan.api.common.util.AopUtil;
import com.kangyonggan.api.common.util.DateUtils;
import com.kangyonggan.api.common.util.PropertiesUtil;
import lombok.extern.log4j.Log4j2;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.reflect.MethodSignature;

import java.lang.reflect.Method;

/**
 * 切于内部service的实现方法上， 需要在方法上手动加上@LogTime注解， 打印入参和出参，打印方法执行时间, 慢方法打印error日志
 *
 * @author kangyonggan
 * @since 2016/11/30
 */
@Log4j2
public class LogAop {

    /**
     * 设定的方法最大执行时间
     */
    private Long slowMethodTime;

    public LogAop() {
        String val = PropertiesUtil.getPropertiesOrDefault("slow.method.time", "10");
        slowMethodTime = Long.parseLong(val);
    }

    /**
     * @param joinPoint
     * @return
     * @throws Throwable
     */
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        Object args[] = joinPoint.getArgs();
        Class clazz = joinPoint.getTarget().getClass();

        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        Method method = clazz.getMethod(methodSignature.getName(), methodSignature.getParameterTypes());
        String targetName = "[" + clazz.getName() + "." + method.getName() + "]";

        LogTime logTime = method.getAnnotation(LogTime.class);
        Object result;
        if (logTime != null) {
            log.info("进入方法:" + targetName + " - args:" + AopUtil.getStringFromRequest(args));

            long beginTime = DateUtils.getNow().getTime();
            result = joinPoint.proceed(args);
            long endTime = DateUtils.getNow().getTime();
            long time = endTime - beginTime;

            log.info("离开方法:" + targetName + " - return:" + AopUtil.getStringFromResponse(result));
            log.info("方法耗时:" + time + "ms - " + targetName);

            if (time > slowMethodTime * 1000) {
                log.error("方法执行超过设定时间" + slowMethodTime + "s," + targetName);
            }
        } else {
            result = joinPoint.proceed(args);
        }

        return result;
    }
}
```


其中用到的`AopUtil.java`:

```
package com.kangyonggan.api.common.util;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import org.apache.commons.lang3.StringUtils;

import java.math.BigDecimal;
import java.util.List;

/**
 * @author kangyonggan
 * @since 2016/11/30
 */
public class AopUtil {
    public static String getStringFromRequest(Object[] args) {
        String req = "";
        for (Object arg : args) {
            if (arg == null) {
                req = req + "null,";
                continue;
            } else if (arg instanceof List == true) {
                req = req + JSON.toJSONString(arg);
            } else if (arg.getClass().isArray()) {
                req = req + JSONArray.toJSONString(arg);
            } else if (arg instanceof Enum) {
                req = req + JSON.toJSONString(arg) + ",";
            } else if (!(arg instanceof String)
                    && !(arg instanceof BigDecimal)
                    && !(arg instanceof Boolean)
                    && !(arg instanceof Integer)
                    && (arg instanceof Object)) {
                req = req + JSON.toJSONString(arg) + ",";
            } else {
                req = req + arg.toString() + ",";
            }
        }

        if (StringUtils.isNotEmpty(req) && req.length() > 100) {
            return req.hashCode() + "";
        } else {
            return req;
        }
    }

    public static String getStringFromResponse(Object arg) {
        String rsp = "";
        if (arg == null) {
            rsp = rsp + "null,";
            return rsp;
        } else if (arg instanceof List) {
            rsp = rsp + JSON.toJSONString(arg);
            return rsp;
        } else if (arg instanceof Enum) {
            rsp = rsp + JSON.toJSONString(arg);
            return rsp;
        } else if (!(arg instanceof String)
                && !(arg instanceof BigDecimal)
                && !(arg instanceof Boolean)
                && !(arg instanceof Integer)
                && (arg instanceof Object)) {
            rsp = rsp + JSON.toJSONString(arg) + ",";
        } else {
            rsp = rsp + arg.toString() + ",";
        }
        return rsp;
    }
}
```
