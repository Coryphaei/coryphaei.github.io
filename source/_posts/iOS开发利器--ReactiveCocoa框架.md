---
title: 新Objective-C纪元--ReactiveCocoa框架
date: 2015-12-15 14:56:46
tags: [iOS, ReactiveCocoa]
categories: 叶帆
---
# 前言
很久之前我就准备写有关于ReactiveCocoa的文章，前面林林总总写过几篇，但是都是简单的讲述，并没有深刻的去总结这个技术。根本的原因在于这个技术确实很难入门，但是ReactiveCocoa的出现确实可以给iOS带来很多新的思考和实现，ReactiveCocoa更加被Mattt Thompson大神称为开启一个新Objective-C纪元。。

# 函数响应式编程
ReactiveCocoa的基本思想就是`函数响应式编程（Function Reactive Programming，以下简称FRP）`。FRP是一种响应变化的编程范式。我们通常会拿一个经典的例子来解释概念。

## 理念
```c
a = 2
b = 2
c = a + b// c = 4

b = 4
// 现在c的值是多少？
```
上面的问题，正常人一眼就能看出答案，因为我们`响应`了`b = 2`这个值的变化，所以说`c`的值会随着`b`的值的改变而改变。FRP就是使用这样的基本原理，所以称之为`响应式编程`。

## 如何实现
FRP提供了`信号（Signal）`机制来实现这样的效果，通过信号来记录值的变化。通过信号的组合，从而不再去监听值的变化，甚至是事件的变化。在上述例子中加入了signal的图解：
![FRP_signal](http://7xp57v.com1.z0.glb.clouddn.com/coryphaei/FRP_signal.png)

## ReactiveCocoa作者对于FRP的解释

[Josh Abernathy这样解释它：](http://blog.maybeapps.com/post/42894317939/input-and-output)

> 程序接收输入产生输出。输出就是对输入做了一些事的结果。输入，转换，输出，完成。
> 输入是应用动作的全部来源。点击、键盘事件、定时器事件、GPS时间、网络请求响应都算是输入。这些事件被传递到应用中，应用将他们以某种方式混合，产生了结果：就是输出。
> 输出通常会改变应用的UI。开关状态变化、列表有了新的元素都是UI变化。也有可能让磁盘上某个文件产生变化，或者产生一个API请求，这都是应用的输出。
> 但不像传统的输入输出设计，应用的输入输出可以产生很多次。应用打开后，不只是一个简单的 输入→工作→输出 就构成了一个生命周期。应用经常有大量的输入并基于这些输入产生输出。

## 话外
关于ReactiveCocoa的灵感来源，我们可以看到官方README中提到了`ReactiveCocoa是基于.NET的Reactive Extension（Rx）`。加上最近朋友在看RxJava，我正好了解到了[reactivex](http://reactivex.io)，目前已经支持了Java，swift等（原谅我只注意到了这两个），所以说Rx所带来的编程思想可能会成为下个时代的开端。

# ReactiveCocoa
