---
title: Mybatis Generator集成MapperPlugin、LombokPlugin和SerializablePlugin
date: 2017-03-25 17:38:59
categories: Java后台
tags:
- Java
- MyBatis
---

### `MBG`集成`MapperPlugin`、`LombokPlugin`和`SerializablePlugin`
可以生成的持久层代码:  

- 生成BeanMapper.java
- 生成BeanMapper.xml
- 生成Bean.java

### 特点
- 生成的Mapper.java继承了MyMapper.java(所有单表的crud全部不用写SQL)
- 实体bean拥有@Data注解（免去写getter、setter和toString的烦恼）
- 实体bean实现Serializable接口，可以放心的放入Redis缓存或传输如分布式系统间（如:dubbo）

<!-- more -->

> 坑：生成的Mapper.java，默认不带注解`@Repository`，如果你又没用spring扫描mapper包， 在运行时会报错，是运行时而不是启动时。

### 引入依赖和插件
`pom.xml`中的配置:

```
<plugin.mybatis-generator.version>1.3.2</plugin.mybatis-generator.version>
<mybatis-generator.version>1.3.2</mybatis-generator.version>
<mybatis-mapper.version>3.3.8</mybatis-mapper.version>
<lombok.version>1.16.8</lombok.version>
         
...

<!--MBG plugin-->
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>${plugin.mybatis-generator.version}</version>
    <configuration>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>${project.groupId}</groupId>
            <artifactId>api-dao</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</plugin>

<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>${mybatis-generator.version}</version>
    <scope>compile</scope>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>${mybatis-mapper.version}</version>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>${lombok.version}</version>
    <scope>provided</scope>
</dependency>
```

> 提示：MBG插件之所以依赖dao, 是因为我在插件中使用了dao中的两个类,下面有。

### 代码和插件配置

`MyMapper.java`

```
package com.kangyonggan.api.mapper;

import org.springframework.stereotype.Component;
import tk.mybatis.mapper.common.ConditionMapper;
import tk.mybatis.mapper.common.ExampleMapper;
import tk.mybatis.mapper.common.SqlServerMapper;
import tk.mybatis.mapper.common.base.BaseDeleteMapper;
import tk.mybatis.mapper.common.base.BaseSelectMapper;
import tk.mybatis.mapper.common.base.BaseUpdateMapper;

/**
 * @author kangyonggan
 * @since 16/5/12
 */
@Component
public interface MyMapper<T> extends
        BaseSelectMapper<T>,
        BaseUpdateMapper<T>,
        BaseDeleteMapper<T>,
        ExampleMapper<T>,
        ConditionMapper<T>,
        SqlServerMapper<T> {
}
```

`LombokPlugin.java`

```
package com.kangyonggan.api.mapper.util;

import org.mybatis.generator.api.IntrospectedColumn;
import org.mybatis.generator.api.IntrospectedTable;
import org.mybatis.generator.api.PluginAdapter;
import org.mybatis.generator.api.dom.java.FullyQualifiedJavaType;
import org.mybatis.generator.api.dom.java.Method;
import org.mybatis.generator.api.dom.java.TopLevelClass;

import java.util.List;

/**
 * @author kangyonggan
 * @since 16/5/12
 */
public class LombokPlugin extends PluginAdapter {
    private FullyQualifiedJavaType dataAnnotation = new FullyQualifiedJavaType("lombok.Data");

    public boolean validate(List<String> warnings) {
        return true;
    }

    public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        this.addDataAnnotation(topLevelClass);
        return true;
    }

    public boolean modelPrimaryKeyClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        this.addDataAnnotation(topLevelClass);
        return true;
    }

    public boolean modelRecordWithBLOBsClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
        this.addDataAnnotation(topLevelClass);
        return true;
    }

    public boolean modelGetterMethodGenerated(Method method, TopLevelClass topLevelClass, IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        return false;
    }

    public boolean modelSetterMethodGenerated(Method method, TopLevelClass topLevelClass, IntrospectedColumn introspectedColumn, IntrospectedTable introspectedTable, ModelClassType modelClassType) {
        return false;
    }

    protected void addDataAnnotation(TopLevelClass topLevelClass) {
        topLevelClass.addImportedType(this.dataAnnotation);
        topLevelClass.addAnnotation("@Data");
    }
}
```

配置插件`generatorConfig.xml`, 插件放在dao模块的`src/main/resources`目录下即可

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="sqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">

        <property name="beginningDelimiter" value="`"></property>
        <property name="endingDelimiter" value="`"></property>

        <!--base mapper-->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="com.kangyonggan.api.mapper.MyMapper"></property>
        </plugin>

        <!--serialize plugin-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"></plugin>

        <!--lombok plugin-->
        <plugin type="com.kangyonggan.api.mapper.util.LombokPlugin"></plugin>

        <!--jdbc driver-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/api"
                        userId="root" password="123456"></jdbcConnection>

        <!--Xxx.java-->
        <javaModelGenerator targetPackage="com.kangyonggan.api.model.vo"
                            targetProject="../api-model/src/main/java">
            <property name="enableSubPackages" value="true"></property>
            <property name="trimStrings" value="true"></property>
        </javaModelGenerator>

        <!--XxxMapper.xml-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"></property>
        </sqlMapGenerator>

        <!--XxxMapper.java-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.kangyonggan.api.mapper"
                             targetProject="src/main/java">
            <property name="enableSubPackages" value="true"></property>
        </javaClientGenerator>

        <!--table name-->
        <table tableName="table_name">
            <generatedKey column="id" sqlStatement="Mysql" identity="true"></generatedKey>
        </table>
    </context>
</generatorConfiguration>
```

### 使用
1. 在项目跟目录下`mvn clean install`, 目的是打包`xxx-dao.jar`，好让插件去依赖
2. 使用 IntelliJ IDEA的，请参考下图(两步), 其他工具我不用，请自行摸索

![使用说明](/uploads/20170105191303945.png)

> 温馨提示：以上配置是我个人配置，请勿直接使用，使用前请自行改造。



