---
title: 实现全文检索-基于MySQL
date: 2017-03-25 17:49:18
categories: Java后台
tags:
- Java
- MySQL
---

我们经常在网站上能看到搜索框， 可以用空格分割关键词、可以用拼音搜索等...

### 把`关键字`拆分为一组`词`
例子：搜索`懒人快乐`， 后台程序就会把`懒人快乐`拆分成两个词`懒人,快乐`。

说明：可以使用`jieba-analysis`结巴分词， 用法如下：

```
<dependency>
    <groupId>com.huaban</groupId>
    <artifactId>jieba-analysis</artifactId>
    <version>1.0.2</version>
</dependency>
```

<!-- more -->

```
package com.kangyonggan.blog.util;


import com.huaban.analysis.jieba.JiebaSegmenter;
import com.huaban.analysis.jieba.SegToken;
import org.apache.commons.lang3.StringUtils;

import java.util.List;

/**
 * @author kangyonggan
 * @since 16/7/22
 */
public class FenCi {

    /**
     * 分词器
     */
    private static final JiebaSegmenter segmenter = new JiebaSegmenter();

    /**
     * 中文分词, 并转成拼音
     *
     * @param data
     * @return
     */
    public static String process(String data) {
        if (StringUtils.isEmpty(data)) {
            return "";
        }

        data = data.replaceAll("\\s", " ");
        data = data.replaceAll("'", " ");

        List<SegToken> list = segmenter.process(data, JiebaSegmenter.SegMode.INDEX);

        StringBuilder sb = new StringBuilder();
        list.forEach(segToken -> sb.append(PinYin.converterToSpellWithMuti(segToken.word)).append(","));
        sb.deleteCharAt(sb.lastIndexOf(","));

        return sb.toString();
    }

}
```
        
### 把`词`转化为`拼音`
例子：`懒人,快乐`会被后台程序转化为`lanren,kuaile,kuaiyue`，之所以不是`lanren,kuaile`而是`lanren,kuaile,kuaiyue`， 是因为`乐`是多音字。

说明：可以使用`pinyin4j`把汉字转化为拼音， 用法如下：

```
<dependency>
     <groupId>com.belerweb</groupId>
     <artifactId>pinyin4j</artifactId>
     <version>2.5.1</version>
</dependency>
```

```
package com.kangyonggan.blog.util;

import net.sourceforge.pinyin4j.PinyinHelper;
import net.sourceforge.pinyin4j.format.HanyuPinyinCaseType;
import net.sourceforge.pinyin4j.format.HanyuPinyinOutputFormat;
import net.sourceforge.pinyin4j.format.HanyuPinyinToneType;
import net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination;

import java.util.ArrayList;
import java.util.Hashtable;
import java.util.List;
import java.util.Map;

/**
 * @author kangyonggan
 * @since 16/7/22
 */
public class PinYin {

    /**
     * 汉字转换位汉语拼音首字母，英文字符不变，特殊字符丢失 支持多音字，生成方式如（长沙市长:cssc,zssz,zssc,cssz）
     *
     * @param chines 汉字
     * @return 拼音
     */
    public static String converterToFirstSpell(String chines) {
        StringBuffer pinyinName = new StringBuffer();
        char[] nameChar = chines.toCharArray();
        HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
        defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
        defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
        for (int i = 0; i < nameChar.length; i++) {
            if (nameChar[i] > 128) {
                try {
                    // 取得当前汉字的所有全拼
                    String[] strs = PinyinHelper.toHanyuPinyinStringArray(
                            nameChar[i], defaultFormat);
                    if (strs != null) {
                        for (int j = 0; j < strs.length; j++) {
                            // 取首字母
                            pinyinName.append(strs[j].charAt(0));
                            if (j != strs.length - 1) {
                                pinyinName.append(",");
                            }
                        }
                    }
                    // else {
                    // pinyinName.append(nameChar[i]);
                    // }
                } catch (BadHanyuPinyinOutputFormatCombination e) {
                    e.printStackTrace();
                }
            } else {
                pinyinName.append(nameChar[i]);
            }
            pinyinName.append(" ");
        }
        // return pinyinName.toString();
        return parseTheChineseByObject(discountTheChinese(pinyinName.toString()));
    }

    /**
     * 汉字转换位汉语全拼，英文字符不变，特殊字符丢失
     * 不支持多音字，生成方式如（重当参:zhongdangcen）
     *
     * @param chines 汉字
     * @return 拼音
     */
    public static String converterToSpell(String chines) {
        StringBuffer pinyinName = new StringBuffer();
        char[] nameChar = chines.toCharArray();
        HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
        defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
        defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
        for (int i = 0; i < nameChar.length; i++) {
            if (nameChar[i] > 128) {
                try {
                    // 取得当前汉字的所有全拼
                    String[] strs = PinyinHelper.toHanyuPinyinStringArray(
                            nameChar[i], defaultFormat);
                    if (strs != null && strs.length > 0) {
                        pinyinName.append(strs[0]);
                    }
                } catch (BadHanyuPinyinOutputFormatCombination e) {
                    e.printStackTrace();
                }
            } else {
                pinyinName.append(nameChar[i]);
            }
            pinyinName.append(" ");
        }
        // return pinyinName.toString();
        return parseTheChineseByObject(discountTheChinese(pinyinName.toString()));
    }

    /**
     * 汉字转换位汉语全拼，英文字符不变，特殊字符丢失
     * 支持多音字，生成方式如（重当参:zhongdangcen,zhongdangcan,chongdangcen
     * ,chongdangshen,zhongdangshen,chongdangcan）
     *
     * @param chines 汉字
     * @return 拼音
     */
    public static String converterToSpellWithMuti(String chines) {
        StringBuffer pinyinName = new StringBuffer();
        char[] nameChar = chines.toCharArray();
        HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
        defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
        defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
        for (int i = 0; i < nameChar.length; i++) {
            if (nameChar[i] > 128) {
                try {
                    // 取得当前汉字的所有全拼
                    String[] strs = PinyinHelper.toHanyuPinyinStringArray(
                            nameChar[i], defaultFormat);
                    if (strs != null) {
                        for (int j = 0; j < strs.length; j++) {
                            pinyinName.append(strs[j]);
                            if (j != strs.length - 1) {
                                pinyinName.append(",");
                            }
                        }
                    }
                } catch (BadHanyuPinyinOutputFormatCombination e) {
                    e.printStackTrace();
                }
            } else {
                pinyinName.append(nameChar[i]);
            }
            pinyinName.append(" ");
        }
        // return pinyinName.toString();
        return parseTheChineseByObject(discountTheChinese(pinyinName.toString()));
    }

    /**
     * 去除多音字重复数据
     *
     * @param theStr
     * @return
     */
    private static List<Map<String, Integer>> discountTheChinese(String theStr) {
        // 去除重复拼音后的拼音列表
        List<Map<String, Integer>> mapList = new ArrayList<Map<String, Integer>>();
        // 用于处理每个字的多音字，去掉重复
        Map<String, Integer> onlyOne = null;
        String[] firsts = theStr.split(" ");
        // 读出每个汉字的拼音
        for (String str : firsts) {
            onlyOne = new Hashtable();
            String[] china = str.split(",");
            // 多音字处理
            for (String s : china) {
                Integer count = onlyOne.get(s);
                if (count == null) {
                    onlyOne.put(s, new Integer(1));
                } else {
                    onlyOne.remove(s);
                    count++;
                    onlyOne.put(s, count);
                }
            }
            mapList.add(onlyOne);
        }
        return mapList;
    }

    /**
     * 解析并组合拼音，对象合并方案(推荐使用)
     *
     * @return
     */
    private static String parseTheChineseByObject(
            List<Map<String, Integer>> list) {
        Map<String, Integer> first = null; // 用于统计每一次,集合组合数据
        // 遍历每一组集合
        for (int i = 0; i < list.size(); i++) {
            // 每一组集合与上一次组合的Map
            Map<String, Integer> temp = new Hashtable<String, Integer>();
            // 第一次循环，first为空
            if (first != null) {
                // 取出上次组合与此次集合的字符，并保存
                for (String s : first.keySet()) {
                    for (String s1 : list.get(i).keySet()) {
                        String str = s + s1;
                        temp.put(str, 1);
                    }
                }
                // 清理上一次组合数据
                if (temp != null && temp.size() > 0) {
                    first.clear();
                }
            } else {
                for (String s : list.get(i).keySet()) {
                    String str = s;
                    temp.put(str, 1);
                }
            }
            // 保存组合数据以便下次循环使用
            if (temp != null && temp.size() > 0) {
                first = temp;
            }
        }
        String returnStr = "";
        if (first != null) {
            // 遍历取出组合字符串
            for (String str : first.keySet()) {
                returnStr += (str + ",");
            }
        }
        if (returnStr.length() > 0) {
            returnStr = returnStr.substring(0, returnStr.length() - 1);
        }
        return returnStr;
    }
}
```

### 单独创建一个表用于全文检索
例子：现在有一个文章表`article`, 它有`title`和`body`等字段，我们想要实现的效果是~如果我们检索的关键字在title或body中， 那么我们就把这条文章记录检索出来。

- 创建一个用于检索的表`article_index`， 它有`article_id`，`title`，`body`等字段， 其中title和body字段要给的大一点， 因为这两个字段将要存储文章表的title和body所对应的拼音，具体给多大请自行分析。
- ALTER TABLE article_index ENGINE = MyISAM; 默认是InnoDB。MyISAM：支持全文索引， 但不支持事务。InnoDB：支持事务， 但不支持全文索引。
- ALTER TABLE `article_index` ADD FULLTEXT INDEX (`title`, `body`); 给title和body字段添加全文索引。
- 在发表一篇新的文章时， 把文章的`title`用`结巴分词`分成多个词， 再用`pinyin4j`转化为拼音， `body`字段也做相同处理，最后把转化后的article_id、title和body存储到`article_index`表中，用于全文检索。

说明：关于全文索引的一些常用知识`SHOW VARIABLES LIKE \'ft_min_word_len\';`， `REPAIR TABLE article_index QUICK;` 作用以及用法请自行学习。

### 全文检索
例子：检索的关键字为`懒人快乐`

- 先把分词， 变为`懒人,快乐`。
- 再把汉字变拼音`lanren,kuaile,kuaiyue`。
- 检索：`SELECT * FROM article_index WHERE match(title, body) against(\'lanren,kuaile,kuaiyue\' IN BOOLEAN MODE)`
- 根据检索到article_id去文章表查询对应的文章

说明：此检索是基于MySQL的， 其他数据库的检索请自行学习， 检索语句还有很多模式， 请自行学习。

> 结巴分词的时候会返回偏移量，所以你可以根据偏移量定位到文章中的关键词， 然后变成红色。
