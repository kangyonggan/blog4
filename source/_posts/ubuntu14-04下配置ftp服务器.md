---
title: ubuntu14.04下配置ftp服务器
date: 2017-03-25 17:33:15
categories: 系统运维
tags:
- Linux
---

### 下载并安装
```
apt-get install vsftpd
```

### ftp服务器启动和重启
- 启动:`/etc/init.d/vsftpd start`  
- 重启:`/etc/init.d/vsftpd restart`  

<!-- more -->

### 创建ftp用户
ftp用户需满足下面条件：  

- 此用户只是用来使用ftp服务的
- 此用户不可登录服务器
- 此用户不能访问ftp指定文件夹之外的文件

### 创建用户
```
# 创建一个用户ftp0
useradd -d /home/ftp0 -m ftp0
# 修改ftp0的密码
passwd ftp0 
```

### 修改ftp配置
修改`/etc/vsftpd.conf`的几个关键配置:

```
anonymous_enable=NO cal_user=YES# 只能访问自身的目录，此处有坑，需加上下面一行
allow_writeable_chroot=YES# 允许写自身发目录
```

### 让用户不能登录
```
usermod -s /sbin/nologin ftp0
```

注意，ubuntu下还需要在`/etc/shells`最后加上：`/sbin/nologin`

> 最后需要重启ftp服务器

### 测试
```
# ftp
ftp> open 192.168.1.100
user:ftp0
pwd:xxxxx
success!
```

### 附javaftp上传代码
```
package com.kangyonggan.api.biz;

import org.apache.commons.net.ftp.FTPClient;
import org.apache.commons.net.ftp.FTPReply;

import java.io.File;
import java.io.FileInputStream;

/**
 * @author kangyonggan
 * @since 2016/12/25
 */
public class TestFtp {

    private FTPClient ftp;

    /**
     * @param path     上传到ftp服务器哪个路径下
     * @param addr     地址
     * @param port     端口号
     * @param username 用户名
     * @param password 密码
     * @return
     * @throws Exception
     */
    private boolean connect(String path, String addr, int port, String username, String password) throws Exception {
        boolean result = false;
        ftp = new FTPClient();
        int reply;
        ftp.connect(addr, port);
        ftp.login(username, password);
        ftp.setFileType(FTPClient.BINARY_FILE_TYPE);
        reply = ftp.getReplyCode();
        if (!FTPReply.isPositiveCompletion(reply)) {
            ftp.disconnect();
            return result;
        }
        ftp.changeWorkingDirectory(path);
        result = true;
        return result;
    }

    /**
     * @param file 上传的文件或文件夹
     * @throws Exception
     */
    private void upload(File file) throws Exception {
        if (file.isDirectory()) {
            ftp.makeDirectory(file.getName());
            ftp.changeWorkingDirectory(file.getName());
            String[] files = file.list();
            for (int i = 0; i < files.length; i++) {
                File file1 = new File(file.getPath() + "\\" + files[i]);
                if (file1.isDirectory()) {
                    upload(file1);
                    ftp.changeToParentDirectory();
                } else {
                    File file2 = new File(file.getPath() + "\\" + files[i]);
                    FileInputStream input = new FileInputStream(file2);
                    ftp.storeFile(file2.getName(), input);
                    input.close();
                }
            }
        } else {
            File file2 = new File(file.getPath());
            FileInputStream input = new FileInputStream(file2);
            ftp.storeFile(file2.getName(), input);
            input.close();
        }
    }

    public static void main(String[] args) throws Exception {
        TestFtp t = new TestFtp();
        t.connect("/home/ftp0/", "192.168.1.100", 21, "ftp0", "123456");
        File file = new File("/Users/kyg/Downloads/blog.sql");
        t.upload(file);
    }

}
```




