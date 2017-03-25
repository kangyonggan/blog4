---
title: 学习java8中java.time包下的几个类-LocalDateTime
date: 2017-03-25 19:32:46
categories: Java后台
tags:
- Java
---

> 官方文档：[http://docs.oracle.com/javase/8/docs/api/index.html](http://docs.oracle.com/javase/8/docs/api/index.html)

前两篇文章中学习了`LocalDate`和`LocalTime`类，学到了一些关于日期和时间的一些用法，这一章中学习一下`LocalDateTime`的用法。

<!-- more -->

## LocalDateTime类
### 类的定义
```
public final class LocalDateTime extends Object implements 
		Temporal, TemporalAdjuster, ChronoLocalDateTime<LocalDate>, Serializable
```

### 类的描述
这是一个`不可变`、`线程安全`的时间类，它可以存储年月日时分秒（毫秒）。

## 获取当前日期时间
### 方法定义
```
public static LocalDateTime now();
```

### 方法描述
获取系统当前日期时间。

### 例子1：

```
public static void main(String[] args) {
    LocalDateTime today = LocalDateTime.now();
    System.out.println(today);
}
```

输出：

```
2017-03-22T16:10:02.586

Process finished with exit code 0
```

## 日期时间格式化
### 方法定义
```
public String format(DateTimeFormatter formatter);
```

### 方法描述
把日期时间格式化指定的格式。

### 抛异常
可能会抛运行时异常DateTimeException

### 例子2：
```
public static void main(String[] args) {
    LocalDateTime today = LocalDateTime.now();
    System.out.println(today);
    System.out.println(today.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS")));
    System.out.println(today.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
}
```

输出：

```
2017-03-22T16:12:51.661
2017-03-22 16:12:51.661
2017-03-22T16:12:51.661

Process finished with exit code 0
```

其中，`DateTimeFormatter`类中一些预定义的格式器，请参考之前的一篇文章:  
[学习java8中java.time包下的几个类 - LocalDate](http://kangyonggan.com/#article/91)

## 日期时间解析
### 方法定义
```
public static LocalDateTime parse(CharSequence text);
```

### 方法描述
把固定格式的字符串解析成日期时间，字符串格式必须为`2017-03-22T15:57:04`。

### 抛异常
如果字符串不能被解析就会抛异常DateTimeParseException

### 例子4：
```
public static void main(String[] args) {
    String dateStr = "2017-03-22T15:57:04";
    LocalDateTime localDateTime = LocalDateTime.parse(dateStr);
    System.out.println(localDateTime);
}
```

输出：

```
2017-03-22T15:57:04

Process finished with exit code 0
```

### 例子5：
```
public static void main(String[] args) {
    String dateStr = "2017-03-22 15:57:04";
    LocalDateTime localDateTime = LocalDateTime.parse(dateStr, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
    System.out.println(localDateTime);
}
```

输出：

```
2017-03-22T15:57:04

Process finished with exit code 0
```

## 其他常用方法
请参考[学习java8中java.time包下的几个类-LocalDate](http://kangyonggan.com/2017/03/25/学习java8中java-time包下的几个类-LocalDate/)或者官方文档。



