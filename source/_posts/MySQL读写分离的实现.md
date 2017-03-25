---
title: MySQL读写分离的实现
date: 2017-03-25 18:12:24
categories: Java后台
tags:
- Java
- MySQL
---

### 实验环境
1. 主库:192.168.2.108:3306/kyg
2. 从库1:192.168.2.113:3306/kyg
3. 从库2:192.168.2.103:3306/kyg

<!-- more -->

### 实验目的
1. 写数据时使用主库
2. 读数据时使用从库

### 实现方案
使用MySQL自身提供的一个驱动`com.mysql.jdbc.ReplicationDriver`来实现读写分离。  
如果一个方法是只读的，那么ReplicationDriver就会为你选择从库读取数据，反之就会选择主库进行读写。  
可以结合SpringAop配置事物的读写：

```
<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="delete*" propagation="REQUIRED"></tx:method>
        <tx:method name="update*" propagation="REQUIRED"></tx:method>
        <tx:method name="save*" propagation="REQUIRED"></tx:method>
        <tx:method name="remove*" propagation="REQUIRED"></tx:method>

        <tx:method name="search*" read-only="true" propagation="REQUIRED"></tx:method>
        <tx:method name="find*" read-only="true" propagation="REQUIRED"></tx:method>
        <tx:method name="get*" read-only="true" propagation="REQUIRED"></tx:method>
        <tx:method name="*" read-only="true" propagation="REQUIRED"></tx:method>
    </tx:attributes>
</tx:advice>
```

jdbc的配置:

```
blog.jdbc.driver     = com.mysql.jdbc.Driver
blog.jdbc.password   = abc
blog.jdbc.url        = jdbc:mysql:replication://192.168.2.108:3306,192.168.2.113:3306,192.168.2.103:3306/kyg
blog.jdbc.username   = abc
```

### 观察日志

![走主库](/uploads/20170107152930664.png)

![走从库](/uploads/20170107152931258.png)

