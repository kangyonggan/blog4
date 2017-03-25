---
title: 学习java8中java.time包下的几个类-LocalTime
date: 2017-03-25 19:31:09
categories: Java后台
tags:
- Java
---

> 官方文档：[http://docs.oracle.com/javase/8/docs/api/index.html](http://docs.oracle.com/javase/8/docs/api/index.html)

上一篇文章中学习了`LocalDate`类，学到了一些关于日期的一些用法，这一章中学习一下`LocalTime`的用法。

<!-- more -->

## LocalTime类
### 类的定义
```
public final class LocalTime extends Object implements 
      Temporal, TemporalAdjuster, Comparable<LocalTime>, Serializable
```

### 类的描述
这是一个`不可变`、`线程安全`的时间类，它可以存储时分秒（毫秒），但是不能存储年月日。

## 获取当前时间
### 方法定义
```
public static LocalTime now();
```

### 方法描述
获取系统当前时间。

### 例子1：

```
public static void main(String[] args) {
    LocalDate today = LocalDate.now();
    System.out.println(today);
}
```

输出：

```
15:44:38.220

Process finished with exit code 0
```

## 时间格式化
### 方法定义
```
public String format(DateTimeFormatter formatter);
```

### 方法描述
把时间格式化指定的格式。

### 抛异常
可能会抛运行时异常DateTimeException

### 例子2：
```
public static void main(String[] args) {
    LocalTime today = LocalTime.now();
    System.out.println(today);
    System.out.println(today.format(DateTimeFormatter.ofPattern("HHmmss")));
    System.out.println(today.format(DateTimeFormatter.ISO_LOCAL_TIME));
}
```

输出：

```
15:51:29.885
155129
15:51:29.885

Process finished with exit code 0
```

其中，`DateTimeFormatter`类中一些预定义的格式器，请参考上一篇文章:  
[学习java8中java.time包下的几个类-LocalDate](http://kangyonggan.com/2017/03/25/学习java8中java-time包下的几个类-LocalDate/)

## 时间解析
### 方法定义
```
public static LocalTime parse(CharSequence text);
```

### 方法描述
把固定格式的字符串解析成时间，字符串格式必须为一下几种之一（亲测）:

- HH:mm
- HH:mm:ss
- HH:mm:ss.S
- HH:mm:ss.SS
- HH:mm:ss.SSS

不能为：

- HH
- HH:m:ss

### 抛异常
如果字符串不能被解析就会抛异常DateTimeParseException

### 例子4：
```
public static void main(String[] args) {
    String dateStr = "15:57:04";
    LocalTime localTime = LocalTime.parse(dateStr);
    System.out.println(localTime);
}
```

输出：

```
15:57:04

Process finished with exit code 0
```

### 例子5：
```
public static void main(String[] args) {
    String dateStr = "155704";
    LocalTime localTime = LocalTime.parse(dateStr, DateTimeFormatter.ofPattern("HHmmss"));
    System.out.println(localTime);
}
```

输出：

```
15:57:04

Process finished with exit code 0
```

## 其他常用方法
请参考上一篇文章或者官方文档。

