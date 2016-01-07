---
title: Objective-C封装技巧（三）：比继承更加灵活的overwrite方法 -- Method Swizzling
date: 2016-01-06 10:38:24
tags: [iOS, Objective-C, Swizzling, 封装]
categories: 叶帆
---
# 写在前面
有经验的工程师在经历过一些项目之后，会慢慢的去考虑之前项目中遇到的坑，思考的过程中就会诞生设计模式和架构的雏形。我这次毕业设计的项目架构正在酝酿中，因为自己本身经验并没有很丰富，所以浅谈自己封装的一些想法，权当抛砖引玉。

平时项目中我们经常会对重复使用的代码进行封装，那么对于封装的场景是否有过思考？是否想过各种封装方法的使用场景和原则。我们实现提出在封装常用的方法：继承和Category。另外在继承的基础上我们发现了比继承更加灵活的Swizzling和在Category的使用过程出现的Associated Objects。关于上述的所有方法，我将会写一个系列来阐述。
- [Objective-C封装技巧（一）：Category和继承的博弈](http://blog.coryphaei.com/2016/01/05/Objective-C%E5%B0%81%E8%A3%85%E6%8A%80%E5%B7%A7%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9ACategory%E5%92%8C%E7%BB%A7%E6%89%BF%E7%9A%84%E5%8D%9A%E5%BC%88/)
- [Objective-C封装技巧（二）：与“魔鬼的交易” -- Associated Objects](http://blog.coryphaei.com/2016/01/06/Objective-C%E5%B0%81%E8%A3%85%E6%8A%80%E5%B7%A7%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9A%E4%B8%8E%E2%80%9C%E9%AD%94%E9%AC%BC%E7%9A%84%E4%BA%A4%E6%98%93%E2%80%9D%20--%20Associated%20Objects/)
- [Objective-C封装技巧（三）：比继承更加灵活的overwrite方法 -- Method Swizzling](http://blog.coryphaei.com/2016/01/06/Objective-C%E5%B0%81%E8%A3%85%E6%8A%80%E5%B7%A7%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9A%E6%AF%94%E7%BB%A7%E6%89%BF%E6%9B%B4%E5%8A%A0%E7%81%B5%E6%B4%BB%E7%9A%84overwrite%E6%96%B9%E6%B3%95%20--%20Method%20Swizzling/)

# 提出问题
希望大家在看系列文章之前，首先思考下面提出的几个问题：
1. 对于方法封装，我们平时经常使用的就是继承子类化Base基类和使用Category，那么在使用这两个方法是具体场景是什么？它们之前的区别和使用优先原则是什么？
2. Category不允许自定义实例属性的缺点，可以用什么来弥补？（提示：Associated Objects）Associated Objects的具体使用场景又是什么呢？
3. 对于需要重复写的通用方法，又存在生命周期中的方法，除了工厂方法继承多个工厂类，还有没有更好的封装机制？（提示：Method Swizzling）关于Method Swizzling的使用场景和机制是什么？
本篇博客对第三个问题进行回答分析，如果阐述过程中有错误或者疑问，大家可以在文章下面留言。

# 前提
在我们对继承进行使用的时候，通常有个使用场景，类似于UMeng的对用户行为进行追踪和分析的操作。通常有这么三种做法：
- 简单粗暴，直接修改每个页面的 **view controller** 代码，简单粗暴；
- 子类化一个Base基类，在基类的 **view controller** 代码里面进行操作；
- Category来进行方法的扩展，然后在**view controller** 代码里面进行调用；
- Method Swizzling
![UMeng](http://7xkvt5.com1.z0.glb.clouddn.com/package%2FUMeng.png)

1，2，3两种的方法，通常会造成大量的重复代码，显得代码很不优雅。1的方法还会存在忘记添加的情况，2的方法存在需要子类化多个子类，如 **UIViewController** 、 **UITableViewController** 、 **UINavigationController**。3的方法只能对代码量并没有缩减，用在这个场景下也不合适。

# Method Swizzling
这个时候我们想用优雅的方式来解决这个问题，既不影响当前方法的实现，又能够在方法注入一些新的操作，动态的来增加方法。我们引入了runtime中的技术方案 -- Method Swizzling。利用的是@selector机制，修改响应事件或者方法所对应的函数指针。

## 使用场景
大致可以概括成下列三个场景：
- 在向视图控制器的生命周期中注入操作、事件的响应、视图的绘制；
- Foundation中的网络堆栈中对于调用时机的记录；
- 类似于Logging，Analytics，Authentication和Caching。这些事务琐碎，跟主要业务逻辑无关，在很多地方都有，又很难抽象出来单独的模块。

## 代码
```objc
#import <objc/runtime.h>
#import "MobClick.h"

@interface UIViewController (MRCUMAnalytics)

@end

@implementation UIViewController (YFUMengAnalytics)

- (void)swizzleMethod(Class class, SEL originalSelector, SEL swizzledSelector) {
    // the method might not exist in the class, but in its superclass
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

    // class_addMethod will fail if original method already exists
    BOOL didAddMethod = class_addMethod(class, originalSelector, method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod));

    // the method doesn’t exist and we just added one
    if (didAddMethod) {
        class_replaceMethod(class, swizzledSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
    }
    else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];
        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);
        swizzleMethod(class, @selector(viewDidAppear:), @selector(umeng_viewWillAppear:));
    });
}

- (void)umeng_viewWillAppear:(BOOL)animated {
    [self umeng_viewWillAppear:animated];
    [MobClick beginLogPageView:NSStringFromClass([self class])];
}

@end
```

## 解释
- 不需要显性调用
我们发现我们使用了Method Swizzling之后，对现有的方法进行了新方法的注入，我们在调用现有方法的时候，就会自动触发我们注入的方法。这种做法非常类似于Hook。

- 步骤
  1. 创建需要交换的方法 **- (void)umeng_viewWillAppear:(BOOL)animated** ，在方法中添加需要实现的方法。这里有个问题解释下，为什么umeng_viewWillAppear又调用了自身？首先我们通过Method Swizzling之后，这两个方法的实现是被交换的，也就是说在定义的时候这样写，但是在实际调用的时候这个方法调用的viewWillAppear，而在viewWillAppear中调用的是umeng_viewWillAppear。

  2. 进行方法的交换 **- (void)swizzleMethod(Class class, SEL originalSelector, SEL swizzledSelector)** ，这一步是具体的实现过程。将两个方法进行交换。我们发现其中有一个判断条件，是否是增加方法。通常使用Method Swizzling是来进行方法的增加，而不是单纯的替换，所以说会进行判断。

  3. 触发时间，是通过 **+ (void)load** 方法来触发。关于这个方法的调用时间是在类被加载的时候调用的，此外load方法还有一个非常重要的特性，那就是子类、父类和分类中的load方法的实现是被区别对待的。换句话说在 Objective-C runtime 自动调用load方法时，分类中的load方法并不会对主类中的load方法造成覆盖。

# 总结
至此，有关于这个系列的所有文章都已经结束，希望大家能够从文章有所收获，在自己的项目里面用到这些，多思考多重构。
