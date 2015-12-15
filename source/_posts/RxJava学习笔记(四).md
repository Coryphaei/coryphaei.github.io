---
title: RxJava学习笔记4
date: 2015-12-15 13:41:00
tags: [Android, RxJava]
categories: 施博文
---
# RxJava笔记4

## 线程控制 Schedulers
### 基本概念
多次切换线程：observeOn()指定的是它之后操作的线程，只要在每个想要切换线程的位置调用observeOn()即可
```Java
Observable.just(1,2,3,4)
               .subscribeOn(Schedulers.io())
               .observeOn(Schedulers.newThread())
               .map(map1)//新线程，有observeOn指定
               .observeOn(Schedulers.io())
               .map(map2)  //io线程，由observeOn指定
               .observeOn(AndroidSchedulers.mainThread())
               .subscribeOn(subscriber);//主线程，由observeOn指定
```
## 小结
第四章是第三章遗留的问题，到此，RxJava的基础已经差不多了，看到现在还是迷糊的，因为没有一个好的使用场景，准备再去Github上找找好的例子学习一下
