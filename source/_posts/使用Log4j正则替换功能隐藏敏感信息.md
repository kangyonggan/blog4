---
title: 使用Log4j正则替换功能隐藏敏感信息
date: 2017-03-25 19:29:01
categories: Java后台
tags:
- Java
---

今天工作中遇到一个问题，需要把一些Log4j日志中的敏感信息给隐藏了，比如：手机号，身份证号等。

## `pom.xml`
```
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.16</version>
</dependency>
```

<!-- more -->

## `log4j.xml`
```
<?xml version="1.0" encoding="GBK" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration>

    <appender name="console" class="org.apache.log4j.ConsoleAppender">
        <!--<layout class="org.apache.log4j.PatternLayout">-->
        <!--<param name="ConversionPattern" value="%d %t [%F:%L] %-5p : %m%n" />-->
        <!--</layout>-->
        <layout class="com.kangyonggan.demo.MyPatternLayout">
            <param name="ConversionPattern" value="%d %t [%F:%L] %-5p : %m%n"/>
        </layout>
    </appender>

    <root>
        <level value="debug"></level>
        <appender-ref ref="console"></appender>
    </root>
</log4j:configuration>
```

## `MyPatternLayout.java`
```
package com.kangyonggan.demo;

import org.apache.log4j.Logger;
import org.apache.log4j.PatternLayout;
import org.apache.log4j.spi.LoggingEvent;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * @author kangyonggan
 * @since 2017/3/20
 */
public class MyPatternLayout extends PatternLayout {

    private static final String HIDDEN = "$1$2$3****$5$6";
    private static final Pattern MOBILE_PATTERN = Pattern.compile("(\\D|^)(1)(3\\d|4[57]|5[^4]|7[0137]|8\\d)(\\d{4})(\\d{4})(\\D|$)");

    @Override
    public String format(LoggingEvent event) {
        if (event.getMessage() instanceof String) {
            String message = event.getRenderedMessage();

            Matcher matcher = MOBILE_PATTERN.matcher(message);

            if (matcher.find()) {
                String maskedMessage = matcher.replaceAll(HIDDEN);

                Throwable throwable = event.getThrowableInformation() != null ? event.getThrowableInformation().getThrowable() : null;

                LoggingEvent maskedEvent = new LoggingEvent(
                        event.fqnOfCategoryClass,
                        Logger.getLogger(event.getLoggerName()),
                        event.timeStamp,
                        event.getLevel(),
                        maskedMessage,
                        throwable);

                return super.format(maskedEvent);
            }
        }

        return super.format(event);
    }
}
```

## `Test.java`
```
package com.kangyonggan.demo;

import org.apache.log4j.Logger;

/**
 * @author kangyonggan
 * @since 2017/3/20
 */
public class Test {

    private static final Logger logger = Logger.getLogger(Test.class);

    public static void main(String[] args) {
        logger.info("手机号1: 13911119999，姓名：qweqe");
        logger.info("手机号2: asd13911119999asd");
        logger.info("假手机号1: 139a11119999");
        logger.info("假手机号2: 139111199991");
        logger.info("假手机号3: 139111199991");
        logger.info("假手机号4: 444139111199991");
        logger.info("假手机号5: 44413911119999");

        try {
            int a = 1 / 0;
        } catch (Exception e) {
            logger.error("13911110000", e);
        }
    }

}
```

## 输出
```
2017-03-21 15:32:50,163 main [Test.java:14] INFO  : 手机号1: 139****9999，姓名：qweqe
2017-03-21 15:32:50,166 main [Test.java:15] INFO  : 手机号2: asd139****9999asd
2017-03-21 15:32:50,166 main [Test.java:16] INFO  : 假手机号1: 139a11119999
2017-03-21 15:32:50,166 main [Test.java:17] INFO  : 假手机号2: 139111199991
2017-03-21 15:32:50,166 main [Test.java:18] INFO  : 假手机号3: 139111199991
2017-03-21 15:32:50,167 main [Test.java:19] INFO  : 假手机号4: 444139111199991
2017-03-21 15:32:50,167 main [Test.java:20] INFO  : 假手机号5: 44413911119999
2017-03-21 15:32:50,167 main [Test.java:25] ERROR : 13911110000
java.lang.ArithmeticException: / by zero
	at com.kangyonggan.demo.Test.main(Test.java:23)

Process finished with exit code 0
```

> 例子中的正则表达式写的比较弱，请根据实际情况自己写。
