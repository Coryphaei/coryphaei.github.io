---
title: Objective-C新纪元--ReactiveCocoa框架
date: 2015-12-15 14:56:46
tags: [iOS, ReactiveCocoa, Objective-C]
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
关于ReactiveCocoa的灵感来源，我们可以看到官方README中提到了`ReactiveCocoa深受Microsoft's Reactive Extension的思想，并基于Reactive Extension（Rx）`。但是官方列举了很多ReactiveCocoa有别于Rx的地方，有兴趣的可以去了解下。

# ReactiveCocoa
[ReactiveCocoa](http://reactivecocoa.io/) is a framework developed by [GitHub](https://github.com/ReactiveCocoa/ReactiveCocoa) to support functional reactive programming on iOS and OS X.

## 起因
作为一个移动开发者，应用中经常有大量的输入，大部分的代码都用来响应这些输出并且基于这些输入来产生输出。我们需要响应的事件非常多：按钮点击事件（target-action）、网络消息回调事件（Block or delegate）、属性变化事件（KVO）、通知事件（NSNotification）等，而这边响应事件在代码中的表现形式却并不统一。为了定义一个标准统一的事件处理接口，并且通过定义的接口来进行组合使用，ReactiveCocoa出现了。

## 基本思想
我们GitHub主页上看到，官方这样给出了概念：
> ReactiveCocoa (RAC)是采用FRP的一个Cocoa framework。RAC提供了API用来组合、转换一直变化的数据流。

ReactiveCocoa采用FRP思想，`信号`则是这个思想的精髓所在，灵魂所在。在ReactiveCocoa简称是RAC，所有的类都以`RAC`开头，所以说ReactiveCocoa中的信号就用`RACSignal`类来表示，用来展示`事件流`的变化，并且可以通过链接、过滤、组合等方式来进行处理。

> 引用我在很多博客中看到的一段话，但是我对其做了改动，加入了桶的概念：
可以把信号(signal)想象成水龙头，只不过里面不是水，而是玻璃球(stream of value)，直径跟水管的内径一样，这样就能保证玻璃球是依次排列，不会出现并排的情况(数据都是线性处理的，不会出现并发情况)。只要你打开水龙头的开关，就会有玻璃球出来。但是，并不是所有的玻璃球都能被使用，除非有了桶(subscriber)来接收掉下来的玻璃球，这样才能运往需要的地方。这样有新的玻璃球进来，有桶在监听，就会自动传送给接收者。可以在水龙头上加一个过滤嘴(filter)，不符合的不让通过，也可以加一个改动装置，把球改变成符合自己的需求(map)。也可以把多个水龙头合并成一个新的水龙头(combineLatest:reduce:)，这样只要其中的一个水龙头有玻璃球出来，这个新合并的水龙头就会得到这个球。

## 思考
通过上述对其的了解，总结ReactiveCocoa带来的影响。
- 定义标准的事件处理接口
- 解决了状态过多依赖的问题

PS：关于巧哥说的给Controller瘦身的问题，我认为这个是MVVM框架所带来的影响，ReactiveCocoa只是很好的配合了MVVM。因此我并没有把这一点归纳在内。

# 开始
进入正轨，开始介绍ReactiveCocoa的机制和常用方法。

## 安装
推荐大家用[CocoaPods](http://code4app.com/article/cocoapods-install-usage)进行安装，这么好的工具肯定要掌握的。
![CocoaPods](http://7xp57v.com1.z0.glb.clouddn.com/coryphaei/cocoapods.png)
目前4.0的alpha版本正在开发，建议大家先使用发布的版本。如果你用swift来写可以用3.0，我是用的Objective，所以用的2.5版本，Podfile:
```
platform :ios, '8.0'
pod 'ReactiveCocoa', '~> 2.5'

```

## RACStreams
`RACStreams`官方定义`An abstract class representing any stream of values`，我翻译下RACStreams是展现任何数据流的一个抽象类。RACStreams通俗点讲就是上面那段话中`水管里面线性流动的、具有顺序的玻璃球`。RACStreams因为是一个抽象类，我们使用中很少直接接触到，我们一般是使用继承自RACStreams的`RACSignal`和`RACSequence`。对于RACSignal和RACSequence与RACStreams联系，我觉得可以直接用[NShipster](http://nshipster.cn/reactivecocoa/)中一句话：
> signal是push驱动的stream，sequence是pull驱动的stream。

## RACSignal and RACSubscriber
`RACSignal`是ReactiveCocoa的核心所在，有了它就能开始使用ReactiveCocoa。RACSignal通俗点讲就是上面那段话中所提到的`水龙头`，表示未来要到到达的值。比较类似于一个概念，叫做`future and promise`，大家可以自行去了解下。
`RACSubscriber`是订阅者，通俗点说就是上面那段话中用来装玻璃球的`桶`。我们可以用一个更好的比喻来理解一下。把RACSignal比作插头，把RACSubscriber比作插座，插头负责去用电，插座负责去取点，插头插座配套才能使用。

### 冷信号(Cold)和热信号(Hot)
在上文中提到的插头插座比喻中，如果说只有插头，没有插座，即只有RACSignal，而没有RACSubscriber，则把RACSignal称之为冷信号，而冷信号默认是不进行任何操作的。只要加上RACSubscriber，就可以进行操作，这个时候RACSignal就被称作是热信号。如果说只有插座，没有插头，那么只要去找到插头就能解决问题。

### RACReplaySubject
我们继续上文中的插头插座比喻，如果现在同时有多个插座在等待一个插头用电，那么我就要把这个插头多次拔下来插到所有的插座上。大家都不愿意重复这个操作，ReactiveCocoa提供了`RACReplaySubject`方法，保证`RACSignal`只触发一次。把需要send的value存起来，直接发送缓存数据。

### 详解
RACSignal一共会发送三种事件给RACSubscriber，RACSubscriber通过-subscribeNext:error:completed:对不同事件作出相应反应
- next 继续进行发送
- error 出现错误  
- completed 完成
一个RACSignal会因为error和completed的出现而终止，即生命周期中只会有一个errot或者completed，但是却可以多次发送next事件。而我们接下来要讨论的就是如何来处理这些多次next事件。

## RACSequence
