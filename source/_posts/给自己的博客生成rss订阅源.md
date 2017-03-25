---
title: 给自己的博客生成rss订阅源
date: 2017-03-25 18:38:54
categories: Java后台
tags:
- Java
---

## 目的
1. 把我的所有博客全部变成rss订阅源，以便别人订阅。
2. 当我有博客更新时，会自定更新订阅源，或者省事一点每隔30分钟自动更新一次。

## 大致步骤
1. 查询所有博客。
2. 遍历所有博客，按`rss规范`组装成xml。
3. 把组装好的xml写入服务器某个位置，比如放在:`/rss/blog.xml`
4. 在rss阅读器中添加订阅源，url为:`http://cdn.kangyonggan.com/rss/blog.xml`
5. 订阅成功，阅读器会把所有博客缓存到本地，离线也可阅读。

<!-- more -->

## 按照rss规范组装xml
使用第三方jar包`rsslibj`，它依赖`exml`,如下:

```
<rsslibj.version>1.0RC2</rsslibj.version>
<exml.version>7.0</exml.version>

<dependency>
    <groupId>rsslibj</groupId>
    <artifactId>rsslibj</artifactId>
    <version>${rsslibj.version}</version>
</dependency>
<dependency>
    <groupId>exml</groupId>
    <artifactId>exml</artifactId>
    <version>${exml.version}</version>
</dependency>
```

代码如下:

```
Channel channel = new Channel();
channel.setTitle("朕的博客");
channel.setLink("http://kangyonggan.com");

String baseUrl = "http://kangyonggan.com/#article/";
String rssName = "blog.xml";

try {
    List<Article> list = articleService.findAllArticles();
    log.info("一共{}篇文章", list.size());

    for (int i = 0; i < list.size(); i++) {
        Article article = list.get(i);
        Item item = new Item();
        item.setTitle(article.getTitle());
        item.setLink(baseUrl + article.getId());
        item.setDcDate(article.getUpdatedTime());
        item.setDescription(MarkdownUtil.markdownToHtml(article.getContent()));

        channel.addItem(i, item);
    }

    File file = new File(PropertiesUtil.getProperties(AppConstants.FILE_PATH_ROOT) + rssName);

    if (!file.exists()) {
        file.createNewFile();
    }

    PrintWriter writer = new PrintWriter(new FileWriter(file));
    writer.write(channel.getFeed("rss"));
    writer.flush();
    writer.close();

    FtpUtil.upload(rssName, "rss/");

    log.info("rss刷新成功");
    return "success";
} catch (Exception e) {
    log.error("查询所有文章失败", e);
}
```

但是有个问题，那就是`Item`不能设置发布时间，订阅后在阅读器里看不到文章的发布时间或者更新时间，所以我就从网上重新找了一个，它不去用任何jar包。

## 解决发布时间问题
#### `Feed.java`:

```
package com.kangyonggan.blog.model.rss;

import lombok.Data;

import java.util.ArrayList;
import java.util.List;

/**
 * @author kangyonggan
 * @since 2017/1/24
 */
@Data
public class Feed {
    String title;
    String link;
    String description;
    String language;
    String copyright;
    String pubDate;
    List<FeedMessage> feedMessages = new ArrayList();

}
```

#### `FeedMessage.java`:

```
package com.kangyonggan.blog.model.rss;

import lombok.Data;

/**
 * @author kangyonggan
 * @since 2017/1/24
 */
@Data
public class FeedMessage {

    String title;

    String description;

    String link;

    String author;

    String guid;

    String pubDate;

}
```

#### `RSSFeedWriter.java`:

```
package com.kangyonggan.blog.biz.util;


import com.kangyonggan.blog.model.rss.Feed;
import com.kangyonggan.blog.model.rss.FeedMessage;

import javax.xml.stream.XMLEventFactory;
import javax.xml.stream.XMLEventWriter;
import javax.xml.stream.XMLOutputFactory;
import javax.xml.stream.XMLStreamException;
import javax.xml.stream.events.*;
import java.io.FileOutputStream;

/**
 * @author kangyonggan
 * @since 2017/1/24
 */
public class RSSFeedWriter {

    private String outputFile;
    private Feed feed;

    public RSSFeedWriter(Feed feed, String outputFile) {
        this.feed = feed;
        this.outputFile = outputFile;
    }

    public void write() throws Exception {
        XMLOutputFactory outputFactory = XMLOutputFactory.newInstance();

        XMLEventWriter eventWriter = outputFactory.createXMLEventWriter(new FileOutputStream(outputFile));

        XMLEventFactory eventFactory = XMLEventFactory.newInstance();
        XMLEvent end = eventFactory.createDTD("\n");

        // Create and write Start Tag

        StartDocument startDocument = eventFactory.createStartDocument();

        eventWriter.add(startDocument);

        // Create open tag
        eventWriter.add(end);

        StartElement rssStart = eventFactory.createStartElement("", "", "rss");
        eventWriter.add(rssStart);
        eventWriter.add(eventFactory.createAttribute("version", "2.0"));
        eventWriter.add(end);

        eventWriter.add(eventFactory.createStartElement("", "", "channel"));
        eventWriter.add(end);

        // Write the different nodes

        createNode(eventWriter, "title", feed.getTitle());

        createNode(eventWriter, "link", feed.getLink());

        createNode(eventWriter, "description", feed.getDescription());

        createNode(eventWriter, "language", feed.getLanguage());

        createNode(eventWriter, "copyright", feed.getCopyright());

        createNode(eventWriter, "pubdate", feed.getPubDate());

        for (FeedMessage entry : feed.getFeedMessages()) {
            eventWriter.add(eventFactory.createStartElement("", "", "item"));
            eventWriter.add(end);
            createNode(eventWriter, "title", entry.getTitle());
            createNode(eventWriter, "description", entry.getDescription());
            createNode(eventWriter, "link", entry.getLink());
            createNode(eventWriter, "author", entry.getAuthor());
            createNode(eventWriter, "guid", entry.getGuid());
            createNode(eventWriter, "pubDate", entry.getPubDate());
            eventWriter.add(end);
            eventWriter.add(eventFactory.createEndElement("", "", "item"));
            eventWriter.add(end);

        }

        eventWriter.add(end);
        eventWriter.add(eventFactory.createEndElement("", "", "channel"));
        eventWriter.add(end);
        eventWriter.add(eventFactory.createEndElement("", "", "rss"));

        eventWriter.add(end);

        eventWriter.add(eventFactory.createEndDocument());

        eventWriter.close();
    }

    private void createNode(XMLEventWriter eventWriter, String name,

                            String value) throws XMLStreamException {
        XMLEventFactory eventFactory = XMLEventFactory.newInstance();
        XMLEvent end = eventFactory.createDTD("\n");
        XMLEvent tab = eventFactory.createDTD("\t");
        // Create Start node
        StartElement sElement = eventFactory.createStartElement("", "", name);
        eventWriter.add(tab);
        eventWriter.add(sElement);
        // Create Content
        Characters characters = eventFactory.createCharacters(value);
        eventWriter.add(characters);
        // Create End node
        EndElement eElement = eventFactory.createEndElement("", "", name);
        eventWriter.add(eElement);
        eventWriter.add(end);
    }
}
```

使用如下：

```
Feed feed = new Feed();
feed.setDescription("记录生活、工作和学习时的笔记心得等");
feed.setLink("http://kangyonggan.com");
feed.setTitle("朕的博客");
List<FeedMessage> feedMessages = feed.getFeedMessages();

String baseUrl = "http://kangyonggan.com/#article/";
String rssName = "blog.xml";

try {
    List<Article> list = articleService.findAllArticles();
    log.info("一共{}篇文章", list.size());

    for (int i = 0; i < list.size(); i++) {
        Article article = list.get(i);
        FeedMessage feedMessage = new FeedMessage();

        feedMessage.setTitle(article.getTitle());
        feedMessage.setLink(baseUrl + article.getId());
        feedMessage.setDescription(MarkdownUtil.markdownToHtml(article.getContent()));
        Date date = article.getUpdatedTime();
        date.setTime(date.getTime() - 8 * 60 * 60 * 1000);
        feedMessage.setPubDate(format.format(date));
        feedMessages.add(feedMessage);
    }

    File file = new File(PropertiesUtil.getProperties(AppConstants.FILE_PATH_ROOT) + rssName);

    if (!file.exists()) {
        file.createNewFile();
    }

    new RSSFeedWriter(feed, file.getPath()).write();

    FtpUtil.upload(rssName, "rss/");

    log.info("rss刷新成功");
    return "success";
} catch (Exception e) {
    log.error("查询所有文章失败", e);
}
```

## 附rss订阅源和订阅效果

![rss](http://cdn.kangyonggan.com/upload/20170125103534094.png)

![reeder](http://cdn.kangyonggan.com/upload/20170125103943826.png)


