---
title: SimpleDateFormat日期格式化线程安全问题
date: 2017-03-25 19:23:30
categories: Java后台
tags:
- Java
---

## 常用的日期工具类
我们在开发中通常需要对日期进行格式化，就想到了写一个工具类，如下：

<!-- more -->

```
package com.kangyonggan.demo.util;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 日期工具类
 *
 * @author kangyonggan
 * @since 2017/3/10
 */
public class DateUtil {

    private static final String DATE_TIME = "yyyy-MM-dd HH:mm:ss";

    /**
     * 格式化成日期时间型字符串
     *
     * @param date 日期
     * @return 返回日期时间型字符串
     */
    public static String format2DateTime(Date date) {
        return new SimpleDateFormat(DATE_TIME).format(date);
    }

    /**
     * 把字符串成解析日期
     *
     * @param source 日期字符串
     * @return 解析后的日期
     * @throws ParseException 解析异常【字符串的格式不正确】
     */
    public static Date parse(String source) throws ParseException {
        return new SimpleDateFormat(DATE_TIME).parse(source);
    }
}
```

## 追求性能的日期工具类
追求性能的同学可能会说：每次都new一个SimpleDateFormat太浪费了，于是有了下面一版：

```
package com.kangyonggan.demo.util;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 日期工具类
 *
 * @author kangyonggan
 * @since 2017/3/10
 */
public class DateUtil {

    private static final String DATE_TIME = "yyyy-MM-dd HH:mm:ss";
    private static final SimpleDateFormat simpleDateFormat = new SimpleDateFormat(DATE_TIME);

    /**
     * 格式化成日期时间型字符串
     *
     * @param date 日期
     * @return 返回日期时间型字符串
     */
    public static String format2DateTime(Date date) {
        return simpleDateFormat.format(date);
    }

    /**
     * 把字符串成解析日期
     *
     * @param source 日期字符串
     * @return 解析后的日期
     * @throws ParseException 解析异常【字符串的格式不正确或多线程临界资源问题】
     */
    public static Date parse(String source) throws ParseException {
        return simpleDateFormat.parse(source);
    }

}
```

## 测试临界资源问题
这个工具类在大多数的时候都是ok的，但是在高并发的时候就不是那么好使了，看看测试代码：

```
package com.kangyonggan.demo.util;

import java.text.ParseException;

/**
 * 测试DateUtil高并发
 *
 * @author kangyonggan
 * @since 2017/3/10
 */
public class TestDateUtil {

    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            new Thread() {
                public void run() {
                    while (true) {
                        try {
                            sleep(1000);
                            System.out.println(DateUtil.parse("2017-03-10 11:24:35"));
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        } catch (ParseException e) {
                            e.printStackTrace();
                            return;
                        }
                    }
                }
            }.start();
        }
    }

}
```

输出信息如下：

```
Exception in thread "Thread-0" Exception in thread "Thread-1" java.lang.NumberFormatException: For input string: "101.E1012E"
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:2043)
	at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
	at java.lang.Double.parseDouble(Double.java:538)
	at java.text.DigitList.getDouble(DigitList.java:169)
	at java.text.DecimalFormat.parse(DecimalFormat.java:2056)
	at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:2162)
	at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
	at java.text.DateFormat.parse(DateFormat.java:364)
	at com.kangyonggan.demo.util.DateUtil.parse(DateUtil.java:36)
	at com.kangyonggan.demo.util.TestDateUtil$1.run(TestDateUtil.java:20)
java.lang.NumberFormatException: For input string: "101.E1012E2"
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:2043)
	at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
	at java.lang.Double.parseDouble(Double.java:538)
	at java.text.DigitList.getDouble(DigitList.java:169)
	at java.text.DecimalFormat.parse(DecimalFormat.java:2056)
	at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:2162)
	at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
	at java.text.DateFormat.parse(DateFormat.java:364)
	at com.kangyonggan.demo.util.DateUtil.parse(DateUtil.java:36)
	at com.kangyonggan.demo.util.TestDateUtil$1.run(TestDateUtil.java:20)

Process finished with exit code 0
```

从错误信息来看是字符串的格式有误，但我们的入参时没错的，于是我稍微修改代码:`i < 1`，即只有一个线程，这时候是不会报错的。    
报错显然是多线程中临界资源`simpleDateFormat`引起的，说明`parse`方法不是线程安全的。查看SimpleDateFormat源码可以看到

```
Date parsedDate;
try {
    parsedDate = calb.establish(calendar).getTime();
    // If the year value is ambiguous,
    // then the two-digit year == the default start year
    if (ambiguousYear[0]) {
        if (parsedDate.before(defaultCenturyStart)) {
            parsedDate = calb.addYear(100).establish(calendar).getTime();
        }
    }
}
// An IllegalArgumentException will be thrown by Calendar.getTime()
// if any fields are out of range, e.g., MONTH == 17.
catch (IllegalArgumentException e) {
    pos.errorIndex = start;
    pos.index = oldStart;
    return null;
}
```

其中`calendar `是成员变量：

```
protected Calendar calendar;
```

也就是说在多线程并发时，这个临界资源是可能被多个线程同时使用的。

## 解决方案
### 1. 在每次调用的时候去new一个
虽然可能会占用一些内存（一般不是很明显，忽略不计），但比较安全。
### 2. 访问临界资源时，使用同步
```
package com.kangyonggan.demo.util;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 日期工具类
 *
 * @author kangyonggan
 * @since 2017/3/10
 */
public class DateUtil {

    private static final String DATE_TIME = "yyyy-MM-dd HH:mm:ss";
    private static final SimpleDateFormat simpleDateFormat = new SimpleDateFormat(DATE_TIME);

    /**
     * 格式化成日期时间型字符串
     *
     * @param date 日期
     * @return 返回日期时间型字符串
     */
    public static String format2DateTime(Date date) {
        synchronized (simpleDateFormat) {
            return simpleDateFormat.format(source);
        }
    }

    /**
     * 把字符串成解析日期
     *
     * @param source 日期字符串
     * @return 解析后的日期
     * @throws ParseException 解析异常【字符串的格式不正确】
     */
    public static Date parse(String source) throws ParseException {
        synchronized (simpleDateFormat) {
            return simpleDateFormat.parse(source);
        }
    }
}
```

但是，当调用太过频繁时，会有阻塞，对性能有一定的影响。

### 3. 使用ThreadLocal
```
package com.kangyonggan.demo.util;

import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 日期工具类
 *
 * @author kangyonggan
 * @since 2017/3/10
 */
public class DateUtil {

    private static final String DATE_TIME = "yyyy-MM-dd HH:mm:ss";
    private static final ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>();

    /**
     * 格式化成日期时间型字符串
     *
     * @param date 日期
     * @return 返回日期时间型字符串
     */
    public static String format2DateTime(Date date) {
        return getDateFormat().format(date);
    }

    /**
     * 把字符串成解析日期
     *
     * @param source 日期字符串
     * @return 解析后的日期
     * @throws ParseException 解析异常【字符串的格式不正确】
     */
    public static Date parse(String source) throws ParseException {
        return getDateFormat().parse(source);
    }

    /**
     * 获取一个线程独享的dateFormat，如果没有则创建一个
     *
     * @return 返回一个线程独享的dateFormat
     */
    private static DateFormat getDateFormat() {
        DateFormat dateFormat = threadLocal.get();
        if (dateFormat == null) {
            dateFormat = new SimpleDateFormat(DATE_TIME);
            threadLocal.set(dateFormat);
        }
        return dateFormat;
    }
}
```

使用ThreadLocal有些情况下并不能减少对象的创建（如果一个线程只调用一次DateUtil），但是有些时候还是有效果的（一个线程多次调用DateUtil）。



