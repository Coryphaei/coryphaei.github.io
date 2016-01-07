---
title: Objective-C封装技巧（二）：与“魔鬼的交易” -- Associated Objects
date: 2016-01-06 09:42:32
tags: [iOS, runtime, category, Objective-C]
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
本篇博客对第二个问题进行回答分析，如果阐述过程中有错误或者疑问，大家可以在文章下面留言。

# Associated Objects
我们知道上次再写Category的时候提到了Category的缺点，不能自定义实例变量。我们在上篇博客的中给出的答案是可以考虑继承，这里引入一个新的技术方案 -- Associated Objects。

## 前提
```
#import <objc/runtime.h>
```
首先需要导入runtime这个类，runtime是个双刃剑。大家使用的时候一定要小心。Associated Objects就是其中一个利器。
有关于runtime，可以参考下Apple官方的[文档](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/ObjCRuntimeRef/#//apple_ref/c/func/objc_setAssociatedObject)。

## 开始
Associated Objects弥补了Objective-C不能在存在的类中扩展自定义属性的缺点，非常的便捷。Associated Objects（对相关联），用一句通俗的话来概括下就是将键值对在运行是关联到对象函数。
一共三个方法：
```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
id objc_getAssociatedObject(id object, const void *key);
void objc_removeAssociatedObjects(id object);
```

## 创建关联
```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
```
一共需要四个参数：源对象，关键字，关联的对象和一个关联策略。我们一一解释。

- **object** 源对象：需要进行关联的对象。                                                                        
- **key** 关键字：关键字是一个void类型的指针。每一个关联的关键字必须是唯一的。有三种方式来进行关键的定义，下面会给出。      
- **value** 关联的对象：在Catogory中一般就是自定义的变量。                                                         
- **policy** 关联策略：相关的对象是通过赋值，保留引用还是复制的方式、通过原子还是非原子的方式进行关联。一共有五种方式，下面给出。

### key-关键字定义
1. 声明 **static char kAssociatedObjectKey;** ，使用 **&kAssociatedObjectKey** 作为key值;
2. 声明 **static void *kAssociatedObjectKey = &kAssociatedObjectKey;**，使用 **kAssociatedObjectKey** 作为key值；
3. 使用 **selector** ，使用getter方法的名称 **@selector(associatedObject)** 作为 key 值。

通常使用 **static char** 类型来定义，更加推荐的是指针类型。另外，尽量保证该属性是常量且唯一，试用范围在本类当中。当然有的人喜欢 **selector** 的方式，因为解决了计算机中最难的取名字问题。

### policy-关联对象的行为
| Behavior                           | @property                      | Description                             |
| ---------------------------------- |:------------------------------:| ---------------------------------------:|
| OBJC_ASSOCIATION_ASSIGN            | @property (assign)             | 指定一个关联对象的弱引用。                  |
| OBJC_ASSOCIATION_RETAIN_NONATOMIC  | @property (nonatomic, strong)  | 指定一个关联对象的强引用，不能被原子化使用。   |
| OBJC_ASSOCIATION_COPY_NONATOMIC    | @property (nonatomic, copy)    | 指定一个关联对象的copy引用，不能被原子化使用。 |
| OBJC_ASSOCIATION_RETAIN            | @property (atomic, strong)     | 指定一个关联对象的强引用，能被原子化使用。     |
| OBJC_ASSOCIATION_COPY              | @property (atomic, copy)       | 指定一个关联对象的copy引用，能被原子化使用。   |

## 获取关联
```
id objc_getAssociatedObject(id object, const void *key);
```
这个非常简单，不再赘述。

## 断开关联
理论上说，我们会使用 **void objc_removeAssociatedObjects(id object);** 但是我们不应手动去调用这个函数。
> 此函数的主要目的是在“初试状态”时方便地返回一个对象。你不应该用这个函数来删除对象的属性，因为可能会导致其他客户对其添加的属性也被移除了。规范的方法是：调用 objc_setAssociatedObject 方法并传入一个 nil 值来清除一个关联。

通常这样去断开关联
```
objc_setAssociatedObject(array, &overviewKey, nil, OBJC_ASSOCIATION_ASSIGN);  
```

## 生命周期
> 根据 WWDC 2011, Session 322 (第36分22秒) 中发布的内存销毁时间表，被关联的对象在生命周期内要比对象本身释放的晚很多。它们会在被 NSObject -dealloc 调用的 object_dispose() 方法中释放。

> 关联对象的释放时机与移除时机并不总是一致，比如实验中用关联策略 OBJC_ASSOCIATION_ASSIGN 进行关联的对象，很早就已经被释放了，但是并没有被移除，而再使用这个关联对象时就会造成 Crash 。

## 代码示例
这里附上[Yaoyuan](https://github.com/ibireme)大神对于Associated Objects的具体实现：

**UIBarButtonItem+YYAdd.h**
```objc
#import <UIKit/UIKit.h>

/**
 Provides extensions for `UIBarButtonItem`.
 */
@interface UIBarButtonItem (YYAdd)

/**
 The block that invoked when the item is selected. The objects captured by block
 will retained by the ButtonItem.

 @discussion This param is conflict with `target` and `action` property.
 Set this will set `target` and `action` property to some internal objects.
 */
@property (nonatomic, copy) void (^actionBlock)(id);

@end
```

**UIBarButtonItem+YYAdd.m**
```objc
#import "UIBarButtonItem+YYAdd.h"
#import "YYCategoriesMacro.h"
#import <objc/runtime.h>

YYSYNTH_DUMMY_CLASS(UIBarButtonItem_YYAdd)


static const int block_key;

@interface _YYUIBarButtonItemBlockTarget : NSObject

@property (nonatomic, copy) void (^block)(id sender);

- (id)initWithBlock:(void (^)(id sender))block;
- (void)invoke:(id)sender;

@end

@implementation _YYUIBarButtonItemBlockTarget

- (id)initWithBlock:(void (^)(id sender))block{
    self = [super init];
    if (self) {
        _block = [block copy];
    }
    return self;
}

- (void)invoke:(id)sender {
    if (self.block) self.block(sender);
}

@end


@implementation UIBarButtonItem (YYAdd)

- (void)setActionBlock:(void (^)(id sender))block {
    _YYUIBarButtonItemBlockTarget *target = [[_YYUIBarButtonItemBlockTarget alloc] initWithBlock:block];
    objc_setAssociatedObject(self, &block_key, target, OBJC_ASSOCIATION_RETAIN_NONATOMIC);

    [self setTarget:target];
    [self setAction:@selector(invoke:)];
}

- (void (^)(id)) actionBlock {
    _YYUIBarButtonItemBlockTarget *target = objc_getAssociatedObject(self, &block_key);
    return target.block;
}

@end
```

# 总结
对于Associated Objects，提供一种为Category添加自定义属性的方法。那么，在我们有自定义属性的时候，我们去使用继承还是使用Associated Objects呢？[Yaoyuan](https://github.com/ibireme)大神在issue中这样回复我：
![比较](http://7xkvt5.com1.z0.glb.clouddn.com/package%2FAssociatedObjects.png)

所以大家还是根据自己的使用场景去确定，最后一个问题将在下篇博客中进行讲解。
