---
title: 多环境下autoconfig使用的小例子
date: 2017-03-25 17:37:30
categories: Java后台
tags:
- Java
---

### 为什么使用Autoconfig
在一个应用中，我们总是会遇到一些参数，例如：

- 数据库服务器IP地址、端口、用户名；
- 用来保存上传资料的目录。
- 一些参数，诸如是否打开cache、加密所用的密钥名称等等。

这些参数有一个共性，那就是：它们和应用的逻辑无关，只和当前环境、当前系统用户相关。以下场景很常见：

- 在开发、测试、发布阶段，使用不同的数据库服务器；
- 在开发阶段，使用Windows的A开发者将用户上传的文件存放在d:\my_upload目录中，而使用Linux的B开发者将同样的文件存放在/home/myname/my_upload目录中。
- 在开发阶段设置cache=off，在生产环境中设置cache=on。
- 很明显，这些参数不适合被“硬编码”在配置文件或代码中。因为每一个从源码库中取得它们的人，都有可能需要修改它们，使之与自己的环境相匹配, autoconfig就是很好的选择。

<!-- more -->

### 引入插件
在`pom.xml`中引入插件，下面为多环境配置

```
<plugin.autoconfig.version>1.2</plugin.autoconfig.version>

...

<!-- autoconfig -->
<plugin>
    <groupId>com.alibaba.citrus.tool</groupId>
    <artifactId>autoconfig-maven-plugin</artifactId>
    <version>${plugin.autoconfig.version}</version>
    <executions>
        <execution>
            <id>compile</id>
            <phase>compile</phase>
            <goals>
                <goal>autoconfig</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <userProperties>${antx.path}</userProperties>
    </configuration>
</plugin>

...

<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <antx.path>${user.home}/antx-dev.properties</antx.path>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <antx.path>${user.home}/antx-prod.properties</antx.path>
        </properties>
    </profile>
</profiles>
```

### 配置
在`src/main/resources`下创建目录`Meta-INF/autoconfig`，在目录下创建文件`auto-config.xml`, 下面是我的配置（依个人而定）:

```
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <group>
        <!-- dubbo -->
        <property name="api.dubbo.name" description="项目名字" defaultValue="api"></property>
        <property name="api.dubbo.registry.address" description="dubbo注册地址"
                  defaultValue="zookeeper://127.0.0.1:2181?backup=127.0.0.1:2182,127.0.0.1:2183"></property>
        <property name="api.dubbo.protocol.port" description="dubbo注册端口" defaultValue="-1"></property>
        <property name="api.dubbo.timeout" description="dubbo超时时间" defaultValue="10000"></property>
        <property name="api.dubbo.api.version" description="api提供接口的版本" defaultValue="1.0.0"></property>

        <!-- redis -->
        <property name="api.redis.maxTotal" description="redis最大连接数" defaultValue="1000"></property>
        <property name="api.redis.minIdle" description="redis最小等待数" defaultValue="50"></property>
        <property name="api.redis.maxIdle" description="redis最大等待数" defaultValue="100"></property>
        <property name="api.redis.testOnBorrow" description="redis测试支持" defaultValue="true"></property>
        <property name="api.redis.host" description="redis主机ip" defaultValue="127.0.0.1"></property>
        <property name="api.redis.port" description="redis主机端口" defaultValue="6379"></property>
        <property name="api.redis.password" description="redis密码" defaultValue="123456"></property>

        <!--jdbc-->
        <property name="api.jdbc.driver" description="jdbc驱动" defaultValue="com.mysql.jdbc.Driver"></property>
        <property name="api.jdbc.url" description="jdbc地址" defaultValue="jdbc:mysql://127.0.0.1:3306/api"></property>
        <property name="api.jdbc.username" description="jdbc用户名" defaultValue="root"></property>
        <property name="api.jdbc.password" description="jdbc密码" defaultValue="123456"></property>

        <!-- log4j2 -->
        <property name="api.log4j2.home" description="log4j2日志的文件主目录" defaultValue="/root/logs/api"></property>

        <property name="api.slow.method.time" description="慢方法时间(秒)" defaultValue="10"></property>
        <property name="api.slow.interface.time" description="慢接口时间(秒)" defaultValue="10"></property>
        <property name="api.redis.prefix.key" description="redis键的前缀" defaultValue="api"></property>
        <property name="api.cache.open" description="是否开启缓存,Y:开启,N:不开启" defaultValue="Y"></property>
    </group>
    <script>
        <generate template="app.properties"></generate>
        <generate template="applicationContext-datasource.xml"></generate>
        <generate template="applicationContext-provider.xml"></generate>
        <generate template="applicationContext-redis.xml"></generate>
        <generate template="log4j2.xml"></generate>
    </script>
</config>
```

### 使用
在上一步中可以看到antx将会替换五个文件，下面是其中一个文件的配置`applicationContext-redis.xml`:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
ohttp://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${api.redis.maxTotal}"></property>
        <property name="minIdle" value="${api.redis.minIdle}"></property>
        <property name="maxIdle" value="${api.redis.maxIdle}"></property>
        <property name="testOnBorrow" value="${api.redis.testOnBorrow}"></property>
    </bean>

    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:hostName="${api.redis.host}" p:port="${api.redis.port}" p:password="${api.redis.password}" p:poolConfig-ref="poolConfig"></bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnectionFactory"></property>
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" ></bean>
        </property>
        <property name="valueSerializer">
            <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"></bean>
        </property>
    </bean>
</beans>
```

> 在项目`mvn clean install`的时候，antx会自动替换它所包含的文件，可以再`target`目录下查看指定的文件是否被替换成功