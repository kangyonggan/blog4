---
title: 使用Log4j2让项目输出info级别的日志和debug级别的sql
date: 2017-03-25 18:46:10
categories: Java后台
tags:
- Java
---

> log4j2的使用方法[http://kangyonggan.com/2017/03/25/log4j2的使用和配置/](http://kangyonggan.com/2017/03/25/log4j2的使用和配置/)

让项目输出info级别的日志，让项目输出debug级别的日志，`log4j2.xml`的配置如下:

<!-- more -->

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="300">
    <properties>
        <property name="LOG_HOME">/Users/kyg/logs/simconf</property>
    </properties>

    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%logger{36}.%t:%L] - %msg%n"></PatternLayout>
        </Console>
        <RollingRandomAccessFile name="AllFile" fileName="${LOG_HOME}/all.log"
                                 filePattern="${LOG_HOME}/all-%d{yyyy-MM-dd}-%i.log">
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%logger{36}.%t:%L] - %msg%n"></PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"></TimeBasedTriggeringPolicy>
                <SizeBasedTriggeringPolicy size="500 MB"></SizeBasedTriggeringPolicy>
            </Policies>
            <DefaultRolloverStrategy max="30"></DefaultRolloverStrategy>
            <Filters>
                <ThresholdFilter level="fatal" onMatch="DENY" onMismatch="NEUTRAL"></ThresholdFilter>
                <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"></ThresholdFilter>
            </Filters>
        </RollingRandomAccessFile>
        <RollingRandomAccessFile name="ErrorFile" fileName="${LOG_HOME}/error.log"
                                 filePattern="${LOG_HOME}/error-%d{yyyy-MM-dd}-%i.log">
            <PatternLayout pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%logger{36}.%t:%L] - %msg%n"></PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"></TimeBasedTriggeringPolicy>
                <SizeBasedTriggeringPolicy size="500 MB"></SizeBasedTriggeringPolicy>
            </Policies>
            <DefaultRolloverStrategy max="30"></DefaultRolloverStrategy>
            <Filters>
                <ThresholdFilter level="fatal" onMatch="DENY" onMismatch="NEUTRAL"></ThresholdFilter>
                <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"></ThresholdFilter>
            </Filters>
        </RollingRandomAccessFile>
        <Async name="AsyncAll">
            <AppenderRef ref="AllFile"></AppenderRef>
        </Async>
        <Async name="AsyncError">
            <AppenderRef ref="ErrorFile"></AppenderRef>
        </Async>
    </Appenders>
    <Loggers>
        <Logger name="com.kangyonggan.app.simconf.mapper" level="debug" additivity="false">
            <AppenderRef ref="AsyncAll"></AppenderRef>
            <AppenderRef ref="AsyncError"></AppenderRef>
        </Logger>

        <Root level="info" additivity="true">
            <AppenderRef ref="Console"></AppenderRef>
            <AppenderRef ref="AsyncAll"></AppenderRef>
            <AppenderRef ref="AsyncError"></AppenderRef>
        </Root>
    </Loggers>
</Configuration>
```

只配置log4j2.xml是不够的，因为Mybatis默认使用的不是log4j2，默认使用的顺序如下:

> SLF4J > Apache Commons Logging > Log4j2 > Log4j > JDK logging

所有我们还需要让Mybatis使用log4j2来输出日志:

```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    ...    
    <property name="configLocation" value="classpath:mybatis.xml"></property>
    ...
</bean>
```

`mybatis.xml`的内容:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!--使用log4j2输出日志-->
        <setting name="logImpl" value="LOG4J2"></setting>
    </settings>
</configuration>
```

小记：本人单线程测试log4j和log4j2的性能，结果如下:  
输出10000个debug+10000个info+10000个error到日志文件，log4j2是log4j速度的1.8倍左右。 
输出100000个debug+100000个info+100000个error到日志文件，log4j2是log4j速度的2.6倍左右。

