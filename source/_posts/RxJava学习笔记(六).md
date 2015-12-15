---
title: RxJava学习笔记6
date: 2015-12-15 13:43:20
tags: [Android, RxJava]
categories: 施博文
---
# RxJava笔记6
## 补充之前的一点
当调用Obeservable.subscibe(subscriber)时，会返回一个Subscription对象，这个对象代表了被观察者和订阅者的关系
Subscription subscription = Obeservable.just("Hello")
  .subscibe(s -> system.out.println(s));

可以在Activity的onPause()中使用 subscription.unsubscribe();停止被观察者和订阅者的关系以防止内存泄露的问题。
