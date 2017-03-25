---
title: 使用配置中心代替properties
date: 2017-03-25 18:40:49
categories: Java后台
tags:
- Java
---

## 生成一个项目`confogcenter`
用`cms-archetype`生成一个项目，然后在此基础上开发一个模块，用于管理配置信息（CRUD）

我写的配置中心在github上: [https://github.com/kangyonggan/configcenter.git](https://github.com/kangyonggan/configcenter.git)

<!-- more -->

## 原理
在spring把占位符替换之前，发送http get请求到配置中心读取配置，拿到此项目的所有配置信息后（json），把json数据解析成一对对的名值对，最后把所有的名值对全部放入系统配置中，即:`System.setProperty(name, value);`。

## 具体实现

在`applicationContext.xml`的最上方（也不一定最上，但一定要在所有占位符的上面）添加自定义的`bean`:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/aop
ohttp://www.springframework.org/schema/aop/spring-aop-4.0.xsd
ohttp://www.springframework.org/schema/beans
ohttp://www.springframework.org/schema/beans/spring-beans-4.0.xsd
ohttp://www.springframework.org/schema/tx
ohttp://www.springframework.org/schema/tx/spring-tx-4.0.xsd
ohttp://www.springframework.org/schema/context 
ohttp://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <!-- Activates annotation-based bean configuration -->
    <context:annotation-config></context:annotation>

    <!-- 读取属性文件，否则 java 类无法直接读取属性 -->
    <context:property-placeholder location="classpath:app.properties"></context:property>

    <!-- 读取配置中心 -->
    <bean class="com.kangyonggan.archetype.cmscc.biz.core.MyPropertyPlaceholderConfigurer" ></bean>
 
    ...
</beans>
```

`MyPropertyPlaceholderConfigurer.java`的实现:

```
package com.kangyonggan.archetype.cmscc.biz.core;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.kangyonggan.archetype.cmscc.biz.util.HttpUtil;
import com.kangyonggan.archetype.cmscc.biz.util.PropertiesUtil;
import lombok.extern.log4j.Log4j2;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.config.PropertyPlaceholderConfigurer;

/**
 * @author kangyonggan
 * @since 2017/1/25
 */
@Log4j2
public class MyPropertyPlaceholderConfigurer extends PropertyPlaceholderConfigurer {

    public MyPropertyPlaceholderConfigurer() {
        loadConfigs();
    }

    /**
     * 加载配置
     */
    private void loadConfigs() {
        String server = PropertiesUtil.getProperties("config.center.server");
        log.info("配置中心服务地址:{}", server);

        String data = HttpUtil.sendGet(server);

        if (StringUtils.isEmpty(data)) {
            throw new RuntimeException("读取配置中心异常");
        }

        log.info("已成功获取配置中心的配置");

        JSONObject jsonObject = JSON.parseObject(data);
        String errCode = (String) jsonObject.get("errCode");
        String errMsg = (String) jsonObject.get("errMsg");

        log.info("errCode:{}", errCode);
        log.info("errMsg:{}", errMsg);

        if (!"success".equals(errCode)) {
            throw new RuntimeException("读取配置中心失败");
        }

        JSONArray jsonArray = jsonObject.getJSONArray("configs");
        log.info("共有{}项配置!", jsonArray.size());

        load(jsonArray);
    }

    /**
     * 加载配置
     *
     * @param jsonArray
     */
    public static void load(JSONArray jsonArray) {
        for (int i = 0; i < jsonArray.size(); i++) {
            JSONObject object = jsonArray.getJSONObject(i);
            log.info("正在存储配置:{}", object);

            String name = object.getString("name");
            String value = object.getString("value");

            System.setProperty(name, value);
            PropertiesUtil.putProperties(name, value);
        }

        log.info("从配置中心加载配置完毕！！！");
    }

}
```

其中`public static void load(JSONArray jsonArray) `之所以写成了`static`是因为我其他地方也想使用, 比如：我发现ftp的ip配置错了，然后修改了ftp的ip，但是又不想重启服务器，这时候我就会在配置中心的控制台上点一下`推送配置`,然后客户端项目就能接收到配置了，然后就需要调用load方法刷新一下内存中的配置，但有些配置刷了也没用，比如jdbc的url等。

接收配置的代码`ConfigcenterController.java`:

```
package com.kangyonggan.archetype.cmscc.web.controller.web;

import com.alibaba.fastjson.JSONArray;
import com.kangyonggan.archetype.cmscc.biz.core.MyPropertyPlaceholderConfigurer;
import lombok.extern.log4j.Log4j2;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.net.URLDecoder;

/**
 * @author kangyonggan
 * @since 2017/1/27
 */
@Controller
@RequestMapping("configcenter")
@Log4j2
public class ConfigcenterController {

    /**
     * 接收配置中心推送过来的配置
     *
     * @param data
     */
    @RequestMapping(method = RequestMethod.POST)
    @ResponseBody
    public boolean receiver(@RequestParam("data") String data) {
        try {
            data = URLDecoder.decode(data, "UTF-8");
            MyPropertyPlaceholderConfigurer.load(JSONArray.parseArray(data));
        } catch (Exception e) {
            log.error("接收配置失败", e);
            return false;
        }
        return true;
    }

}
```

## 四、注意
#### 1. 日志的目录不能配置在配置中心，因为在读取配置中心的配置之前就使用到了日志，我的解决方案是写入了`pom.xml`, 如：`<log4j2.home>/Users/kyg/logs/cmscc</log4j2.home>`
#### 2. 配置中心服务器的地址配置在了`app.properties`，因为配置中心的地址是可能变的，所以不能硬编码，如：`app.proerperties`的内容:

```
config.center.server=http://localhost:7777/configuration?proj=${project.parent.artifactId}&env=${env}
```

其中`${project.parent.artifactId}`会取自`pom.xml`中父模块的artifactId,  
`${env}`指的是环境，比如开发环境，联调环境，生产环境等。我写在了pom.xml中:

```
<profiles>
    <profile>
        <id>local</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <env>local</env>
        </properties>
    </profile>
    <profile>
        <id>dev</id>
        <properties>
            <env>local</env>
        </properties>
    </profile>
    <profile>
        <id>uat</id>
        <properties>
            <env>local</env>
        </properties>
    </profile>
    <profile>
        <id>hd</id>
        <properties>
            <env>local</env>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <env>local</env>
        </properties>
    </profile>
</profiles>
```

我把使用配置中心的项目也写了一个项目原型,地址在:[https://github.com/kangyonggan/cmscc-archetype.git](https://github.com/kangyonggan/cmscc-archetype.git)

## 附配置中心推送配置的代码
```
/**
 * 推送配置
 *
 * @param id
 * @param env
 * @return
 */
@RequestMapping(value = "push", method = RequestMethod.POST)
@RequiresPermissions("CORE_PROJECT")
@ResponseBody
public Map<String, Object> push(@RequestParam("id") Long id, @RequestParam("env") String env) {
    Map<String, Object> resultMap = getResultMap();
    Project project = projectService.findProjectById(id);

    if (project != null && StringUtils.isNotEmpty(project.getPushUrl())) {
        List<Configuration> configurations = configurationService.findProjectConfigurations(project.getCode(), env);
        String json = JSON.toJSONString(configurations);
        try {
            String data = URLEncoder.encode(json, "UTF-8");
            String result = HttpUtil.sendPost(project.getPushUrl(), "data=" + data);
            if (!"true".equals(result)) {
                setResultMapFailure(resultMap, "推送失败，请稍后再试！");
            }
        } catch (Exception e) {
            log.error("推送配置失败", e);
            setResultMapFailure(resultMap);
        }
    } else {
        setResultMapFailure(resultMap);
    }

    return resultMap;
}
```


