---
title: log4j2的使用和配置
date: 2017-03-25 17:12:26
categories: Java后台
tags:
- Java
---

### 引入依赖
在`pom.xml`中添加依赖:

```
<log4j2.api.version>2.5</log4j2.api.version>

...

<!--Log4j2-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j2.api.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j2.api.version}</version>
</dependency>            
```

<!-- more -->

### 配置`log4j2.xml`

1. 异步输出日志，不会影响主线程性能
2. error日志单独输出到error.log文件
3. all.log包含所有级别的日志
4. 每天会备份旧的日志文件，产生新的日志文件
5. 如果一天之中有日志文件超出大小限制(下面配的500M)，会存档当前文件，另外再创建一个新的文件
6. 日志文件只保存近期的（下面配置的30天）

这已经可以满足大部分需求了，如果还不满足，那就再改造吧

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="300">
    <properties>
        <property name="LOG_HOME">/Users/kyg/logs/test</property>
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
        <Root level="debug" additivity="true">
            <AppenderRef ref="Console"></AppenderRef>
            <AppenderRef ref="AsyncAll"></AppenderRef>
            <AppenderRef ref="AsyncError"></AppenderRef>
        </Root>
    </Loggers>
</Configuration>
```
