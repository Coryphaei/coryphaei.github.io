---
title: Objective-C新纪元--ReactiveCocoa框架
date: 2015-12-15 14:56:46
tags: [iOS, ReactiveCocoa, Objective-C]
categories: 叶帆
---
# 前言
很久之前我就准备写有关于ReactiveCocoa的文章，前面林林总总写过几篇，但是都是简单的讲述，并没有深刻的去总结这个技术。根本的原因在于这个技术确实很难入门，但是ReactiveCocoa的出现确实可以给iOS带来很多新的思考和实现，ReactiveCocoa更加被Mattt Thompson大神称为开启一个新Objective-C纪元。另外提醒大家，我看到的优秀的讲ReactiveCocoa的文章篇幅都很长，其实大家都在简洁的语言来讲，我的这边文章应该写完也是长篇幅，希望大家可以耐心的看完。

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

```objc
[self.usernameTextField.rac_textSignal subscribeNext:^(id x) {
    NSLog(@"%@", x);
}];
```

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
`RACSequence`官方的解释是一组immutable且有序的values，很多人说把这个看做是`NSArray`。但是注意用词是`看做`，因为这些values的值是`懒加载`(只有需要的时候才加载)，这样sequence只有一部分被用到，会一定程度得提升性能。那么NSArray可以通过rac_sequence方法转换成RACSequence来调用RAC中的方法了。像Cocoa的集合类型一样，RACSequence不接受`nil`。

## map -- 修改
`map` calls its block with each user that's fetched and returns a new. 解释一下就是将事件中获得的数据映射为你想要的对象，可以看做对玻璃球的重新包装。
```objc
[[[self.usernameTextField.rac_textSignal map:^id(NSString *text) {
    return @(text.length);
 }]
 subscribeNext:^(id x) {
     NSLog(@"%@", x);
 }];
```

## filter -- 过滤
`Filters` out values in the receiver that don't pass the given test. 非常简单对事件中的内容进行过滤，可以看做不合要求的玻璃球进行拦击，不允许通过水管。
```objc
[[[self.usernameTextField.rac_textSignal map:^id(NSString *text) {
    return @(text.length);
 }]
 filter:^BOOL(NSNumber *length) {
     return [length intValue] > 3;
 }]
 subscribeNext:^(id x) {
     NSLog(@"%@", x);
 }];
```

## combineLatest -- 组合
`Combines` the latest values from the receiver and the given signal into RACTuples, once both have sent at least one next. 将一组事件组合为一个输出最新事件的signal。可以看做是对水管进行改造，使得任何时刻都输出最新的玻璃球。
```objc
RACSignal *signUpActiveSignal = [RACSignal combineLatest:@[validUsernameSignal, validPasswordSignal]
                                                  reduce:^id(NSNumber *usernameValid, NSNumber *passwordValid){
                                                      return @([usernameValid boolValue] && [passwordValid boolValue]);
                                                  }];
```

## flatten -- 合并
`flatten`把事件进行合并，对于其中的内容都进行显示，来一个显示一个，可以交叉显示。可以看做把多个水管进行了合并，哪个水管中的玻璃球到了就放出玻璃球。

## flattenMap -- 解决signal of signals
Maps `block` across the values in the receiver and flattens the result.
这个问题首先要先解释一下。就是说事件完成block后有可能会返回signal的实例，这个时候外部信号中就会包含一个内部信号，这个时候使用map去讲信号转换为另一种信号，造成了嵌套的麻烦。所以说通过flattenMap将事件从内部信号发送到外部信号，并且映射到另外一个信号上去，这样这个过程就变得扁平化。Signal被按序的链接起来执行异步操作，而且不用嵌套block。
```objc
- (RACSignal *)signInSignal
{
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [self.signInService signInWithUsername:self.usernameTextField.text
                                      password:self.passwordTextField.text
                                      complete:^(BOOL success) {
                                          [subscriber sendNext:@(success)];
                                          [subscriber sendCompleted];
                                      }];
        return nil;
    }];
}

[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside] flattenMap:^RACStream *(id value) {
    return [self signInSignal];
}] subscribeNext:^(id x) {
    NSLog(@"Sign in result: %@", x);
}];
```

## 循环引用
ReactiveCocoa使用时大量的使用了block，而由于Ojective-C语言的内存管理机制使用的引用计数，会造成循环引用的问题。为了避免循环引用的问题，通常的解决办法是声明其中的一个变量为弱引用weak，将其赋值给self，在block中来使用这个弱引用的self，为了简单，通常使用了一个语法糖：`@weakify(self)`和`@strongify(self)`。

## 常用宏定义
- RAC()可以将信号的某个属性与其他的信号进行联动。
```objc
RAC(self.submitButton.enabled) = [RACSignal combineLatest:@[self.usernameField.rac_textSignal, self.passwordField.rac_textSignal] reduce:^id(NSString *userName, NSString *password) {
    return @(userName.length >= 6 && password.length >= 6);
}];
```
- RACObserve()监听信号的属性的改变，使用block的KVO
```objc
[RACObserve(self.textField, text) subscribeNext:^(NSString *newName) {
    NSLog(@"%@", newName);
}];
```


# MVVM
![MVVM](http://7xp57v.com1.z0.glb.clouddn.com/coryphaei/mvvm.png)

## 为什么要提到MVVM
MVVM其实是MVC的变形框架，主要来解决目前iOS应用中日益增长的重量级Controller的问题。在你使用ReactiveCocoa的时候会发现将事件定义统一接口后确实方便了代码的编写，但是都在Controller中来进行使得Conttroller异常的臃肿。这个也就是为什么很多人写到ReactiveCocoa的时候一定会提到MVVM的原因，建议大家配合使用，将ReactiveCocoa处理事件的代码写在ViewModel中，这样也方便做测试，昨天听了LeanCloud智维大神的自动化和测试之后，也准备来探究一下，应该到时候会出一篇博客。

## 关于MVVM
关于MVVM，这里不做详细的讲解，不是本章的重点。但是可以给出几篇参考，有兴趣的同学可以去了解一下。
- [MVVM 介绍](http://objccn.io/issue-13-1/)
- [被误解的MVC和被神化的MVVM
](http://www.infoq.com/cn/articles/rethinking-mvc-mvvm)

# 最后
我尽管认真的学习了一周ReactiveCocoa，但是仍然还处在入门阶段，也许等我实战之后会有更多的体会和坑来告诉大家，但是这个是重框架，入门还是比较难的，我尽我所能的理解写下这个博客，希望能帮助大家入个门，同时我也给出几篇参考文章，希望对大家有帮助。
- [Reactive​Cocoa](http://nshipster.cn/reactivecocoa/)
- [说说ReactiveCocoa 2](http://limboy.me/ios/2013/12/27/reactivecocoa-2.html)
- [ReactiveCocoa学习笔记](http://yulingtianxia.com/blog/2014/07/29/reactivecocoa/)
- [ReactiveCocoa Tutorial – the Definitive Introduction: Part 1/2](http://southpeak.github.io/blog/2014/08/02/reactivecocoazhi-nan-%5B%3F%5D-:xin-hao/)
- [使用ReactiveCocoa实现iOS平台响应式编程](http://www.itiger.me/?p=38)
- [唐巧 ReactiveCocoa - iOS开发的新框架](http://blog.devtang.com/blog/2014/02/11/reactivecocoa-introduction/)

# update
- 2015.12.22 上周六的时候，[DeveloperLx](https://github.com/DeveloperLx)讲了有关于ReactiveCocoa的很多干货，我写了一篇[博客]()，大部分都是对他将的内容的整理和一点感悟。
