---
title: Objective-C封装技巧（一）：Category和继承的博弈
date: 2016-01-05 15:24:19
tags: [iOS, Objective-C, 封装, 继承, Category]
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
本篇博客对第一个问题进行回答分析，如果阐述过程中有错误或者疑问，大家可以在文章下面留言。

# 继承--子类化
我们首先从继承--对面对象的三大基本特征讲起，继承的概念得益于能够方便的对父类的方法进行实现和overwrite。在使用场景上，我想很多开发者应该在项目之初会经常写了很多Base的基类，封装一些基本的常用方法，在子类中可以直接实用，防止大量的重复代码。下图，使用的是[DeveloperLx](https://github.com/DeveloperLx)大神的项目框架，特此表示感谢，我也从他的框架中学到了很多。
![baseclass](http://7xkvt5.com1.z0.glb.clouddn.com/package%2Fbaseclass.png)

应该说所有人的最初封装都是从继承开始，这个是最容易想到的方法。那么这个继承使用的场景其实是在我们需要对方法进行重写的时候（如生命周期等等，当然其实有更好的方法替代），或者对于父类Delegate通用方法来进行封装的时候来使用。但是继承的缺点在于耦合度比较高，比如我们写了BaseViewController来进行继承，那么通用模块移植到别的项目中的时候就会出现依赖于BaseViewController基类的问题。出现了非常高的耦合度，所以我们经常在面试题中，面试官也会问你这样的问题：你项目中使用继承吗？优缺点？是否有改进的方法？

# Category
说到Category，大家肯定不陌生，可以看到Apple官方也大量的使用了Category。Category使用的设计模式其实就是装饰模式，是对于改设计模式的具体实现。Category区别于继承的最大不同点在于，它是在不改变原有类的前提下，动态的去扩展该类的类方法和实例方法。

## 使用场景
对于Category的使用场景，我们根据[Apple](https://developer.apple.com/library/ios/documentation/General/Conceptual/DevPedia-CocoaCore/Category.html#//apple_ref/doc/uid/TP40008195-CH5-SW1)官方的描述，可以大致分为以下三类：
1. 官方给出的使用最广泛的场景，对一个已经存在的类（不管实现已知还是未知）进行增加方法的扩展，而不去子类化。扩展出来的类均可以被子类和原始类来使用。
2. 把代码量的类中的方法划分成多个Category文件。
3. 声明私有方法。

在日常使用的过程中，我们通常使用的是一种方法，因为其封装的便利性，使得其应用非常的广泛。我们使用Category的基本原则：
> “The answer is don't do that. Only add methods,don't try to replace or otherwise.It's hard to understand for people reading your code"

如果说你在使用的时候发现你试图在Category中去overwrite父类的方法，虽然是可以这么做的，但是不推荐。因为重写之后，在使用方法的时候会优先选择Category中的方法实现，导致原来的方法不能使用。如果出现了这种情况，那么一定是你使用场景出现大的错误，要尝试继承或者其他的封装方法。

## 注意
- 不需要实现所有的方法
在我们平时的使用中，在Category中声明的方法并不需要都实现，关键在于你会不会调用方法。

- 命名
开发者在项目的过程中，常常会出现积累了自己Category的情况。当使用三方库的时候，有可能出现Category重名的问题，所以建议大家在创建Category的时候，加上自己特有的prefix。可以在最初使用的时候统一命名为+xxTools，等到Category方法越来越多的时候在进行根据功能划分的Category拆分，当然同时也要加上prefix，避免重名。

- 属性
大家都会说Category中不能自定义属性。这种说法其实是不正确的，在Category中可以声明property，也同样会实现setter和getter方法，但是对于属性的实现是不行的。想要进行弥补，就只能使用Associated Objects来实现。我们将在该系列的下一篇[博客](http://blog.coryphaei.com/2016/01/06/Objective-C%E5%B0%81%E8%A3%85%E6%8A%80%E5%B7%A7%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9A%E4%B8%8E%E2%80%9C%E9%AD%94%E9%AC%BC%E7%9A%84%E4%BA%A4%E6%98%93%E2%80%9D%20--%20Associated%20Objects/)中来详细展开。

## 参考
大家对于Category的封装，可以参考[Yaoyuan](https://github.com/ibireme)大神的[YYCategory](https://github.com/ibireme/YYCategories)，直接拿来就能用。对他为开源做出的贡献表示感谢。

# 总结
通过上述的展开，我们可以大概总结一下。对于需要重写的父类的方法，或者需要对实例属性进行操作的时候，我们需要选择继承，继承通常应在UIKit的对象中。对于现有类方法的扩展，或者对于Foundation中对象的封装，通常首先考虑使用Category。
最后，本篇文章回答了第一次问题，其他的两个问题，将在下面两篇博客中来展开。
