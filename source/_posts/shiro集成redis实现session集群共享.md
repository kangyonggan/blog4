---
title: shiro集成redis实现session集群共享
date: 2017-03-25 17:52:51
categories: Java后台
tags:
- Java
- Shiro
---

### 好处
1. session在tomcat集群中共享（单点登录）
2. tomcat重启后会话不丢失

### 实现
覆写`EnterpriseCacheSessionDAO`

```java
package com.kangyonggan.blog.web.shiro;

import com.kangyonggan.api.common.service.RedisService;
import org.apache.shiro.session.Session;
import org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO;
import org.springframework.beans.factory.annotation.Autowired;

import java.io.Serializable;

/**
 * @author kangyonggan
 * @since 2016/12/31
 */
public class MyEnterpriseCacheSessionDAO extends EnterpriseCacheSessionDAO {

    @Autowired
    private RedisService redisService;

    /**
     * 创建session，保存到redis数据库
     *
     * @param session
     * @return
     */
    @Override
    protected Serializable doCreate(Session session) {
        Serializable sessionId = super.doCreate(session);
        redisService.set(sessionId.toString(), session);

        return sessionId;
    }

    /**
     * 获取session
     *
     * @param sessionId
     * @return
     */
    @Override
    protected Session doReadSession(Serializable sessionId) {
        // 先从缓存中获取session，如果没有再去数据库中获取
        Session session = super.doReadSession(sessionId);
        if (session == null) {
            session = (Session) redisService.get(sessionId.toString());
        }
        return session;
    }

    /**
     * 更新session的最后一次访问时间
     *
     * @param session
     */
    @Override
    protected void doUpdate(Session session) {
        super.doUpdate(session);
        redisService.set(session.getId().toString(), session);
    }

    /**
     * 删除session
     *
     * @param session
     */
    @Override
    protected void doDelete(Session session) {
        super.doDelete(session);
        redisService.delete(session.getId().toString());
    }

}
```

> 但是一般还是别在集群中使用session。