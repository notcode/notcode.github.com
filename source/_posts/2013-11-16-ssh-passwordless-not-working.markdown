---
layout: post
title: "ssh故障诊断方法"
date: 2013-11-16 12:04
comments: true
categories: 
- system
- devops
---

#### ssh故障诊断方法

有时在配置或使用ssh/scp时发生异常,可以用以下方法排查原因:

*   开启`-vvv`选项

    可以清楚的看到ssh/scp在建立连接时发生了什么，如登录过程中使用的哪个密钥文件、server端支持那些认证方式，都可以在此看到。

    
*   查看server端日志：/var/log/secure

    每一次ssh链接的client ip,prot,user,是否成功, 以及相关错误信息都会记录到此。
    最近遇到过.ssh下权限配置错误导致ssh登录失败，从日志里可以很方便的看到是因为哪个文件权限配置错误造成的。


