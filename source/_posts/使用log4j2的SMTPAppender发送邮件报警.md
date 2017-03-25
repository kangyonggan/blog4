---
title: 使用log4j2的SMTPAppender发送邮件报警
date: 2017-03-25 18:36:33
categories: Java后台
tags:
- Java
---

## 目的
1. 当项目中有`报错时`，要能`自动的`、`及时的`发邮件通知`指定人员`。
2. 邮件中的错误日志要全面，最好能把当前线程的全部日志输出，不论日志级别。
3. 仅当日志为error级别时，才发邮件通知。
4. 可以配置邮件抄送给其他人。

<!-- more -->

## 引入依赖
```
<log4j2.api.version>2.5</log4j2.api.version>

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

## 配置`log4j2.xml` 
```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="300">
    <properties>
        <property name="LOG_HOME">/Users/kyg/logs/cms</property>
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
        <SMTP name="Mail" subject="内容管理系统 - 报警通知" to="kangyonggan@gmail.com" from="kangyg2017@163.com"
              smtpHost="smtp.163.com" smtpUsername="kangyg2017@163.com" smtpPassword="xxxxxxxx" bufferSize="50" >
        </SMTP>
        <Async name="AsyncAll">
            <AppenderRef ref="AllFile"></AppenderRef>
        </Async>
        <Async name="AsyncError">
            <AppenderRef ref="ErrorFile"></AppenderRef>
            <AppenderRef ref="Mail" ></AppenderRef>
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

上面的配置中，关于报警的配置有两点:

```
<AppenderRef ref="Mail" ></AppenderRef>
```

```
<SMTP name="Mail" subject="内容管理系统 - 报警通知" to="kangyonggan@gmail.com" from="kangyg2017@163.com"
      smtpHost="smtp.163.com" smtpUsername="kangyg2017@163.com" smtpPassword="xxxxxxxx" bufferSize="50" >
</SMTP>
```

## 附加说明
- 异步输出日志，不会影响主线程性能
- error日志单独输出到error.log文件
- all.log包含所有级别的日志
- 每天会备份旧的日志文件，产生新的日志文件
- 如果一天之中有日志文件超出大小限制(上面配的500M)，会存档当前文件，另外再创建一个新的文件
- 日志文件只保存近期的（上面配置的30天）

> 温馨提示：由于是个人项目，所以没配置多个邮件接收者，如有需要，请参考[官方文档](http://logging.apache.org/log4j/2.x/manual/appenders.html#SMTPAppender)

## 五、收到的邮件截图
![mail](/uploads/20170123163536603.png)
![mail](/uploads/20170123163703359.png)
![mail](/uploads/20170123163703945.png)
![mail](/uploads/20170123163704569.png)
