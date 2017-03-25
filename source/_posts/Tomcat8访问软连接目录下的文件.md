---
title: Tomcat8访问软连接目录下的文件
date: 2017-03-25 17:31:39
categories: 综合
tags:
- Tomcat
---

### Tomcat 7 修改`context.xml`:
```
<Context allowLinking="true" ></Context>
```

### Tomcat 8 修改`context.xml`:
```
<Context>
  <Resources allowLinking="true" ></Resources>
</Context>
```

### 使用软链接

```
ln -s /home/kyg/data/blog/upload/ /home/kyg/install/apache-tomcat-8.5.6-blog/webapps/ROOT/WEB-INF/
```