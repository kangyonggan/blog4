---
title: 学习java8中java.time包下的几个类-LocalDate
date: 2017-03-25 19:29:59
categories: Java后台
tags:
- Java
---

> 官方文档:[http://docs.oracle.com/javase/8/docs/api/index.html](http://docs.oracle.com/javase/8/docs/api/index.html)

在我之前的一篇文章中提到[SimpleDateFormat日期格式化线程安全问题](http://kangyonggan.com/#article/84)，尽管有了解决方案，但依然不是很爽，接下来我就来学习一下java8中提供的新的日期时间类`java.time`，重新写一个合手的日期时间工具类，不！这些类本身就可以作为工具类了。

<!-- more -->

## LocalDate类
### 类的定义
```
public final class LocalDate extends Object implements 
    Temporal, TemporalAdjuster, ChronoLocalDate, Serializable
```

### 类的描述
这是一个`不可变`、`线程安全`的日期类，它可以存储年月日，但是不能存储时分秒。

> 常用术语: 日期-年月日，时间-时分秒，日期时间-年月日时分秒

## 获取当前日期
### 方法定义
```
public static LocalDate now();
```

### 方法描述
获取系统当前日期。

### 例子1：

在`pom.xml`中指定使用jdk1.8编译：

```
<!--compiler plugin -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.2</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
    </configuration>
</plugin>
```

`LocalDateNowDemo.java`:

```
package com.kangyonggan.demo.time;

import java.time.LocalDate;

/**
 * @author kangyonggan
 * @since 2017/3/22
 */
public class LocalDateNowDemo {

    public static void main(String[] args) {
        LocalDate today = LocalDate.now();
        System.out.println(today);
    }

}
```

输出：

```
2017-03-22
Process finished with exit code 0
```

可以看出它不像java.util.Date那样输出一串乱七八糟的时间，如果我们仅仅使用日期，那会非常爽。  
但是，有时候我们需要`yyyyMMdd`格式的日期。

## 日期格式化
### 方法定义
```
public String format(DateTimeFormatter formatter);
```

### 方法描述
把日期格式化指定的格式。

### 抛异常
可能会抛运行时异常`DateTimeException`

### 例子2：

```
public static void main(String[] args) {
    LocalDate today = LocalDate.now();
    System.out.println(today);
    System.out.println(today.format(DateTimeFormatter.ofPattern("yyyyMMdd")));
}
```

输出：

```
2017-03-22
20170322

Process finished with exit code 0
```

每次都写`yyyyMMdd`感觉很麻烦，而且容易错容易忘，再看下面的写法：

### 例子3：

```
public static void main(String[] args) {
    LocalDate today = LocalDate.now();
    System.out.println(today);
    System.out.println(today.format(DateTimeFormatter.BASIC_ISO_DATE));
}
```

输出：

```
2017-03-22
20170322

Process finished with exit code 0
```

输出结果是一样的，这是使用了DateTimeFormatter类中预定义的一些格式器，更多预定义格式器如下：

Formatter	| Description	 | Example
--------- | ----------- | ---------
ofLocalizedDate(dateStyle) | Formatter with date style from the locale | '2011-12-03'
ofLocalizedTime(timeStyle) | Formatter with time style from the locale | '10:15:30'
ofLocalizedDateTime(dateTimeStyle) | Formatter with a style for date and time from the locale | '3 Jun 2008 11:05:30'
ofLocalizedDateTime(dateStyle,timeStyle) | Formatter with date and time styles from the locale | '3 Jun 2008 11:05'
BASIC_ISO_DATE | Basic ISO date | '20111203'
ISO_LOCAL_DATE | ISO Local Date | '2011-12-03'
ISO_OFFSET_DATE | ISO Date with offset | '2011-12-03+01:00'
ISO_DATE | ISO Date with or without offset | '2011-12-03+01:00'; '2011-12-03'
ISO_LOCAL_TIME | Time without offset | '10:15:30'
ISO_OFFSET_TIME | Time with offset | '10:15:30+01:00'
ISO_TIME | Time with or without offset | '10:15:30+01:00'; '10:15:30'
ISO_LOCAL_DATE_TIME | ISO Local Date and Time | '2011-12-03T10:15:30'
ISO_OFFSET_DATE_TIME | Date Time with Offset | '2011-12-03T10:15:30+01:00'
ISO_ZONED_DATE_TIME | Zoned Date Time | '2011-12-03T10:15:30+01:00[Europe/Paris]'
ISO_DATE_TIME | Date and time with ZoneId | '2011-12-03T10:15:30+01:00[Europe/Paris]'
ISO_ORDINAL_DATE | Year and day of year | '2012-337'
ISO_WEEK_DATE | Year and Week | 2012-W48-6'
ISO_INSTANT | Date and Time of an Instant | '2011-12-03T10:15:30Z'
RFC_1123_DATE_TIME | RFC 1123 / RFC 822 | 'Tue, 3 Jun 2008 11:05:30 GMT'

刚刚是把日期格式化，下面方法是相反的操作，即把指定格式的字符串解析成日期。

## 日期解析
### 方法定义
```
public static LocalDate parse(CharSequence text);
```

### 方法描述
把`固定格式`的字符串解析成日期，字符串格式必须为:`yyyy-MM-dd`

### 抛异常
如果字符串不能被解析就会抛异常`DateTimeParseException`

### 例子4：

```
public static void main(String[] args) {
    String dateStr = "2017-03-22";
    LocalDate localDate = LocalDate.parse(dateStr);
    System.out.println(localDate);
}
```

输出：

```
2017-03-22

Process finished with exit code 0
```

### 例子5：

```
public static void main(String[] args) {
    String dateStr = "2017-3-22";
    LocalDate localDate = LocalDate.parse(dateStr);
    System.out.println(localDate);
}
```

输出：

```
Exception in thread "main" java.time.format.DateTimeParseException: Text '2017-3-22' could not be parsed at index 5
	at java.time.format.DateTimeFormatter.parseResolved0(DateTimeFormatter.java:1949)
	at java.time.format.DateTimeFormatter.parse(DateTimeFormatter.java:1851)
	at java.time.LocalDate.parse(LocalDate.java:400)
	at java.time.LocalDate.parse(LocalDate.java:385)
	at com.kangyonggan.demo.time.LocalDateParseDemo.main(LocalDateParseDemo.java:14)

Process finished with exit code 1
```

比较例4和例5，发现parse方法对字符串的要求还是比较严格的，一旦解析不了就会抛异常。  
在实际开发中，我们的字符串不可能都是yyyy-MM-dd型的，LocalDate还提供了另外一个同名方法。

## 日期解析（指定格式）
### 方法定义
```
public static LocalDate parse(CharSequence text, DateTimeFormatter formatter);
```

### 方法描述
把指定格式的字符串解析成日期。

### 抛异常
如果字符串不能被解析就会抛异常`DateTimeParseException`

### 例子6：

```
public static void main(String[] args) {
    String dateStr = "20170322";
    LocalDate localDate = LocalDate.parse(dateStr, DateTimeFormatter.BASIC_ISO_DATE);
    System.out.println(localDate);
}
```

输出：

```
2017-03-22

Process finished with exit code 0
```

这个例子中我使用的是预定义的格式器，当然你也可以自己定义格式。  
相比SimpleDateFormat，这个方法是线程安全的，且提供了一大批预定义的格式器。

## 其他常用方法
LocalDate类中还提供了一些其他方法，比如：

方法定义 | 描述
------- | ------
public int getYear() | 获取年
public int getMonthValue() | 获取月
public int getDayOfMonth() | 获取日
public boolean isAfter(ChronoLocalDate other) | 判断是否在other日期之后
public boolean isBefore(ChronoLocalDate other) | 判断是否在other日期之前
public boolean isEqual(ChronoLocalDate other) | 判断和other日期是否相等
public boolean isLeapYear() | 判断是否是闰年
public int lengthOfMonth() | 返回一个月有多少天
public int lengthOfYear() | 返回一年有多少天
public static LocalDate of(int year, int month, int dayOfMonth) | 返回一个日期
public LocalDate minusDays(long daysToSubtract) | 减去n天，返回一个新的LocalDate
public LocalDate minusWeeks(long weeksToSubtract) | 减去n周，返回一个新的LocalDate
public LocalDate minusMonths(long monthsToSubtract) | 减去n月，返回一个新的LocalDate
public LocalDate minusYears(long yearsToSubtract) | 减去n年，返回一个新的LocalDate
public LocalDate plusDays(long daysToSubtract) | 加上n天，返回一个新的LocalDate
public LocalDate plusWeeks(long weeksToSubtract) | 加上n周，返回一个新的LocalDate
public LocalDate plusMonths(long monthsToSubtract) | 加上n月，返回一个新的LocalDate
public LocalDate plusYears(long yearsToSubtract) | 加上n年，返回一个新的LocalDate

> LocalDate类中远不止这些方法，具体请查看官方文档。

