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


http://blog.leichunfeng.com/blog/2015/06/14/objective-c-method-swizzling-best-practice/
http://blog.csdn.net/yiyaaixuexi/article/details/9374411
http://nshipster.com/method-swizzling/
http://southpeak.github.io/blog/2014/11/06/objective-c-runtime-yun-xing-shi-zhi-si-:method-swizzling/
http://tech.glowing.com/cn/method-swizzling-aop/
